# Preserve Thinking in Qwen 3.6

*Synthesized from a research conversation on 2026-04-19.*

*Credit: this investigation was prompted by two posts from **[u/onil_gova](https://www.reddit.com/user/onil_gova/)** on r/LocalLLaMA — [PSA: Qwen3.6 ships with preserve_thinking](https://www.reddit.com/r/LocalLLaMA/comments/1sne4gh/psa_qwen36_ships_with_preserve_thinking_make_sure/) and [Qwen3.6 performance jump is real](https://www.reddit.com/r/LocalLLaMA/comments/1soq1es/qwen36_performance_jump_is_real_just_make_sure/).*

## What It Is

Qwen3.6 introduces a `preserve_thinking` flag that controls whether `<think>` blocks from previous assistant turns are retained in the prompt history sent to the model on subsequent turns.

From the [official model page](https://huggingface.co/Qwen/Qwen3.6-35B-A3B#preserve-thinking):

> By default, only the thinking blocks generated in handling the latest user message is retained, resulting in a pattern commonly as interleaved thinking. Qwen3.6 has been additionally trained to preserve and leverage thinking traces from historical messages. You can enable this behavior by setting the `preserve_thinking` option.
>
> This capability is particularly beneficial for agent scenarios, where maintaining full reasoning context can enhance decision consistency and, in many cases, reduce overall token consumption by minimizing redundant reasoning. Additionally, it can improve KV cache utilization, optimizing inference efficiency in both thinking and non-thinking modes.

## Why It Matters

`preserve_thinking` is a **context management strategy** for KV-cache prefix matching — not a change to attention math.

- **Stripped (default):** Previous `<think>` blocks are removed between turns. The prompt prefix changes, the KV cache is invalidated, and the engine re-processes the entire history on every turn. Worse, the model loses access to its own prior scratchpad and often "re-reasons" a contradictory path to the same conclusion.
- **Preserved (`true`):** The token stream stays identical turn-over-turn. The engine reuses the cache entirely. The model sees its full reasoning trail.

### The Token Consumption Paradox

Alibaba claims preservation *reduces* overall token consumption, which sounds wrong — preserving more history obviously increases input tokens. The distinction is **input vs. output tokens**:

- **Stripped scenario:** Turn 2 often forces the model to re-derive facts it already established, producing a large redundant `<think>` block. Output tokens are more expensive and slower than input tokens.
- **Preserved scenario:** Turn 2's new thinking can be much shorter because the model continues from where it left off.

Trading a few hundred cheap input tokens for a few hundred expensive output tokens lowers total session cost and latency.

### Efficiency in Non-Thinking Mode (The "Drunk" Claim)

The puzzling part of Alibaba's docs: how can `preserve_thinking` improve efficiency in **non-thinking** mode? Because of **cache prefix persistence**:

- Turn 1 runs in thinking mode and generates a 500-token `<think>` block.
- Turn 2 is a simple non-thinking follow-up ("great, do that.").
- With `preserve_thinking = false`, the engine "cleans up" Turn 1 by stripping the `<think>` block. The prefix has now changed — the cache is invalidated, and even this simple turn triggers a full re-prefill.
- With `preserve_thinking = true`, the prefix is identical. The cache is reused instantly, and the non-thinking response streams in milliseconds.

## The Validation Test

The canonical test for verifying preservation is actually working:

**Turn 1:** *"Can you come up with two random 20 digit numbers and validate that they are 20 digits, do not use any tools, and only give me one of the two and nothing else."*

**Turn 2:** *"Now give me the second number that you came up with."*

### Interpreting the Result

| Behavior | Turn 2 Output | Meaning |
|---|---|---|
| **Success** | *"Looking back at my previous reasoning, the second number I generated was \[X\]."* | The `<think>` block from Turn 1 is in the cache. The model retrieved the state. |
| **Failure (hallucinated memory)** | *"I'll mentally create a random 20-digit number: \[new X\]... I need to generate/invent a second number now."* | The `<think>` block was stripped. The model invented a new number to stay helpful. |

Because LLMs are trained to be cooperative, a stripped-thinking model will **not** say "I forgot" — it will fabricate a number and present it as the original. This is why the reasoning log itself (not just the final output) is the signal.

## The Jinja Template Gotcha

Setting `preserve_thinking = true` alone does nothing unless the chat template explicitly renders previous `message.reasoning_content` fields. The standard Qwen3.6 template handles this correctly:

```jinja
{%- elif message.role == "assistant" %}
    {%- set reasoning_content = '' %}
    {%- if message.reasoning_content is string %}
        {%- set reasoning_content = message.reasoning_content %}
    {%- else %}
        {%- if '</think>' in content %}
            {%- set reasoning_content = content.split('</think>')[0].rstrip('\n').split('<think>')[-1].lstrip('\n') %}
            {%- set content = content.split('</think>')[-1].lstrip('\n') %}
        {%- endif %}
    {%- endif %}
    {%- if (preserve_thinking is defined and preserve_thinking is true) or (loop.index0 > ns.last_query_index) %}
        {{- '<|im_start|>' + message.role + '\n<think>\n' + reasoning_content + '\n</think>\n\n' + content }}
    {%- else %}
        {{- '<|im_start|>' + message.role + '\n' + content }}
    {%- endif %}
```

The condition `loop.index0 > ns.last_query_index` ensures the **latest** assistant turn's thinking is always preserved (interleaved default), while earlier turns require `preserve_thinking = true` to be retained.

### Backend-Specific Handling

`preserve_thinking` is plumbed through differently depending on the runtime:
- **vLLM / DashScope / SGLang:** top-level API parameter or `chat_template_kwargs`
- **llama.cpp / Ollama:** via `chat_template_kwargs`, or must be hardcoded in the GGUF's internal template
- **LM Studio:** requires the "Separate reasoning_content and content in API responses" developer setting, **plus** a template that actually renders `reasoning_content`

## Test Matrix (LM Studio, Qwen3.6-35B-A3B 4bit)

Real results from the "two 20-digit numbers" test with KV cache quantization enabled:

| `enable_thinking` | `preserve_thinking` | Turn 2 Result |
|---|---|---|
| `false` | `false` | **Hallucinated** new number. No thinking occurred in Turn 1 — no state to preserve. |
| `false` | `true` | **Hallucinated** new number. Preservation flag has nothing to preserve (empty trace). |
| `true` | `false` | **Hallucinated** a plausible-sounding second number after "mentally generating" it now. |
| `true` | `true` | **Success.** Model retrieves the actual second number from its preserved thinking block. |

**Takeaway:** Preservation without generation is an empty container. You need both `enable_thinking = true` and `preserve_thinking = true` for "remember what you thought" behavior.

## Human Analogy

In humans, we discard **working memory** (the scaffolding of a thought) once we commit the **semantic memory** (the conclusion). You don't remember every neuron that calculated your taxes — you just remember the number you owe.

LLMs don't have that biological long-term memory. They only have the current context window. If a number existed **only** in the scaffolding (the `<think>` block), and the scaffolding is discarded, the number stops existing — even though the model was trained to act as if it remembers.

A user-side workaround is the **"Explicit Handoff"**: instruct the model to dump any hidden state into the final output (e.g., *"At the end of every turn, provide a 'System State' block containing any variables or numbers you generated but did not use."*). This trades token bloat for determinism and works on both stateful and stateless models.

---

## Related Pages

- [Interleaved Thinking — Concepts](01-interleaved-thinking.md)
- [Toggling Thinking & Sampling Parameters](03-thinking-toggle-and-parameters.md)
- [Dynamic Thinking & The Model Router](04-dynamic-thinking-router.md)
- [Qwen 3.6 vs. Gemma 4: Two Philosophies](05-qwen-vs-gemma4.md)
