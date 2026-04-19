# Toggling Thinking & Sampling Parameters

*Synthesized from a research conversation on 2026-04-19.*

## Two Flags, Two Layers

Qwen3.6 exposes two distinct controls that are often confused:

| Flag | Lives In | Controls |
|---|---|---|
| `enable_thinking` | Inference engine / API / Jinja generation prompt | Whether the model *generates* a `<think>` block for the current turn. |
| `preserve_thinking` | Jinja template / `chat_template_kwargs` | Whether *previous* `<think>` blocks are retained in history. |

`enable_thinking` is a **generation-time** behavior — it controls what the model does next. `preserve_thinking` is a **formatting** rule — it controls how history is packaged before being sent.

This is why Alibaba's reference Jinja template contains `preserve_thinking` logic but not `enable_thinking` as a structural toggle — the latter is passed via API parameters, while the former is a template decision about whether to render `reasoning_content` for past assistant turns.

## The Hard-Coded Override Trap

LM Studio presets sometimes ship with this at the top of the Jinja template:

```jinja
{%- set preserve_thinking = false %}
{%- set enable_thinking = false %}
```

This **overrides** any UI toggles. Even if the user ticks "Enable Thinking" in LM Studio's interface, the template has already set the variables to `false`. Result: the template emits an empty `<think></think>` block and the model jumps straight to the answer, with no scratchpad and no state to preserve.

To actually use reasoning + persistence in LM Studio, those two lines must be set to `true` (or removed so the runtime can pass them in).

## The Official Sampling Parameters

Per the [Qwen3.6 model card](https://huggingface.co/Qwen/Qwen3.6-35B-A3B), recommended parameters vary by mode and task:

| Mode | Task | `temperature` | `top_p` | `top_k` | `min_p` | `presence_penalty` | `repetition_penalty` |
|---|---|---|---|---|---|---|---|
| **Thinking** | General | 1.0 | 0.95 | 20 | 0.0 | **1.5** | 1.0 |
| **Thinking** | Precise coding (WebDev) | **0.6** | 0.95 | 20 | 0.0 | 0.0 | 1.0 |
| **Instruct** (non-thinking) | General | 0.7 | 0.8 | 20 | 0.0 | 1.5 | 1.0 |
| **Instruct** (non-thinking) | Reasoning tasks | 1.0 | 0.95 | 20 | 0.0 | 1.5 | 1.0 |

## Why These Specific Values

### `presence_penalty = 1.5` (Thinking / General)
Unusually high. It's designed to **penalize repetition of the thinking process itself** — preventing the model from getting stuck re-walking the same reasoning loop over multi-turn agent sessions.

**Important interaction with `preserve_thinking`:**
- *With* thinking preserved, the penalty keeps the model from re-chewing over its previous `<think>` block.
- *Without* thinking preserved, the same penalty can actively *encourage* hallucination. The model sees only the prior final answer, wants to generate something "different," and invents new content rather than repeating visible tokens.

### `temperature = 0.6` (Thinking / Precise Coding)
Lowered for code tasks because high-entropy reasoning tokens can cascade into "hallucinated syntax" in the output — the model gets "creative" with API calls. Dropping temperature keeps the logical chain rigid and reduces divergent token paths.

### `min_p = 0.0`
Allows the model to consider low-probability tokens if they are logically consistent with the thinking chain. Reasoning often chains through tokens that have low initial probabilities but build into a valid logical sequence — a higher `min_p` would prune them and break the chain.

### `repetition_penalty = 1.0` (neutral)
Kept off for reasoning. Penalizing repetition interferes with the model reusing variable names, mathematical constants, or identifiers it established earlier in the trace.

### `top_p = 0.8` (Instruct / General)
Tighter nucleus. Standard, deterministic, minimal "waffle" for direct answer mode.

## The "Different Hats" — When to Use Each Mode

### Thinking + Preserve (agent planning)
- Long-horizon multi-turn tasks
- Coding agents that need to retain reasoning across tool calls
- Tasks with cumulative state (the 20-digit number test; multi-step debugging)

### Thinking (one-shot)
- Complex single-turn reasoning with no follow-up
- Saves input tokens on Turn 1 since there are no prior traces to preserve

### Instruct (no thinking)
- Execution steps where a plan already exists
- Simple factual or format-constrained responses
- Tool execution where verbose reasoning would slow syntax generation
- Low-latency agent actions once the "plan" has been locked in

### Dynamic switching
Qwen3.6 and Gemma 4 both ship with **autonomous gating** — the model decides per-prompt whether to enter thinking mode. See [Dynamic Thinking & The Model Router](04-dynamic-thinking-router.md).

## Parameter Interaction Cheatsheet

| You want... | Use |
|---|---|
| Model to remember its own reasoning across turns | `enable_thinking=true` + `preserve_thinking=true` |
| Fast, deterministic tool execution | `enable_thinking=false`, `temperature=0.7`, `top_p=0.8` |
| Rigorous step-by-step coding logic | `enable_thinking=true`, `temperature=0.6`, `presence_penalty=0.0` |
| Exploratory reasoning without loops | `enable_thinking=true`, `presence_penalty=1.5`, `repetition_penalty=1.0` |
| Low-latency chat with no scratchpad | `enable_thinking=false`, `preserve_thinking=false` (fast path) |

---

## Related Pages

- [Interleaved Thinking — Concepts](01-interleaved-thinking.md)
- [Preserve Thinking in Qwen 3.6](02-preserve-thinking.md)
- [Dynamic Thinking & The Model Router](04-dynamic-thinking-router.md)
- [Qwen 3.6 vs. Gemma 4: Two Philosophies](05-qwen-vs-gemma4.md)
