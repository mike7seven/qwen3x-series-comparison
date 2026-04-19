# Dynamic Thinking & The Model Router

*Synthesized from a research conversation on 2026-04-19.*

## The Shift: Manual Toggle → Autonomous Gating

In the prior generation of reasoning models, "thinking" was a manual choice — you either invoked a reasoning model (o1, DeepSeek-R, early Qwen thinking variants) or a standard instruct model.

With **Qwen3.6** and **Gemma 4**, the model acts as its own **router**. An internal **gating mechanism** (sometimes called a "routing head") evaluates the prompt before the first token is generated and decides how much reasoning budget — if any — to spend.

This is called **dynamic thinking** (or **dynamic reasoning** / **autonomous gating**):

- **Fast path:** low-complexity prompts ("what time is it in Colorado?") bypass the reasoning block to save latency and power.
- **Deep path:** math, logic, multi-step challenges autonomously trigger a `<think>` block.

## Why Hard-Coded Templates Defeat It

The Jinja trap from the preserve-thinking discussion:

```jinja
{%- set enable_thinking = false %}
```

This is effectively a lobotomy — it overrides the model's autonomous ability to choose the deep path, even when the prompt genuinely requires it. The model wants to reason; the template forbids it.

For dynamic thinking to work as intended, `enable_thinking` should either be left undefined or passed through from the runtime so the model's own gating head can decide.

## Adaptive Reasoning Latency ("Dwelling")

Beyond the binary fast/deep choice, both Qwen3.6 and Gemma 4 tune **how much** reasoning to spend when they do think. This was observed across multiple model sizes and prompt types:

### The "Hello" Test

A plain greeting with no other context triggers very different behavior across model sizes:

| Model Class | "Hello" (No Tools) | Behavior |
|---|---|---|
| **Large (35B+, MoE)** | **High dwell.** Model debates persona, ethics, style, "how should I respond?" | Long, circular, persona-focused reasoning log. |
| **Small (7B–9B)** | **Zero or shallow dwell.** Direct response. | "Fast Path" bypass — no `<think>` block at all. |

Larger models have more idle "experts" (in MoE) and more parameter surface area, which makes them prone to **reasoning leakage** — even a trivial input activates some reasoning expert because the model is always seeking higher-confidence paths. Smaller models are tuned for **inference efficiency** and have a higher gating threshold: no clear contradiction, no thinking.

### The Tool Anchor Effect

Adding tool definitions to the same "Hello" prompt radically changes both large and small model behavior:

| Input Scenario | Reasoning "Dwell" Focus | Outcome |
|---|---|---|
| **Simple prompt, no tools** | Internal: persona, ethics, style, self-regulation | Long, circular, "lost in thought" log |
| **Simple prompt, tools present** | External: tool matching, constraint checking | Short, decisive, utility-focused log |

**Real example from Qwen3.6 with tools enabled + a "Hello" prompt:**

> "The user has said 'Hello'. This is a greeting. I should respond politely and ask how I can help them. I can also check if there are any existing projects available to mention if relevant, but for a simple greeting, a friendly response is best. I'll list the available projects to be helpful."

**Same model, same prompt, no tools enabled:**

> *(paraphrased from the observed output)* — Analyzes the greeting → determines appropriate response → checks identity/constitution constraints → drafts a response → refines it → second-guesses variations → refines again → finalizes. The log is several times longer and gets stuck in self-regulation loops.

The tool definitions in the system prompt act as a **gating anchor** — they route the model's reasoning budget into the "action space" (matching tools to intent) rather than the "persona space" (debating how to greet).

### Prediction: Small Models + Tools

A reverse pivot appears likely:
- **Large model:** thinking *shortens* when tools are added (tools limit over-analysis).
- **Small model:** thinking may *lengthen* slightly when tools are added. Plain "Hello" gets skipped entirely, but tool-call syntax (JSON/XML formatting) is a "high difficulty" task for an SLM. You may see a brief surgical `<think>` just for syntax validation.

## Gating by Utility, Not Just Size

The 2026 reasoning models aren't judged by thought length — they're judged by the model's ability to **know when to shut up and work**. Alibaba's claim about efficiency in *non-thinking* modes makes sense here: when the router correctly identifies a tool-use intent, suppressing reasoning isn't a feature loss — it's the router doing its job.

## Summary Matrix

| Model Class | "Hello" (No Tools) | "Hello" (Tools Enabled) | Reason for Shift |
|---|---|---|---|
| **SLM (Small)** | Zero dwell — bypasses `<think>` for speed | Micro dwell — brief syntax check | Complexity increased |
| **LLM (Large)** | High dwell — persona/ethics spiral | Low dwell — decisive action check | Complexity anchored |

## Human Analogy

- **Large model, no tools:** the overly-analytical professor with nothing to do that day — spends an hour over-thinking a casual "hey."
- **Large model, with tools:** the same professor handed a specific task — suddenly focused and efficient.
- **Small model, no tools:** the first responder — no time for philosophy, just move.
- **Small model, with tools:** the clerk — only starts thinking when the paperwork actually gets complicated.

---

## Related Pages

- [Interleaved Thinking — Concepts](01-interleaved-thinking.md)
- [Preserve Thinking in Qwen 3.6](02-preserve-thinking.md)
- [Toggling Thinking & Sampling Parameters](03-thinking-toggle-and-parameters.md)
- [Qwen 3.6 vs. Gemma 4: Two Philosophies](05-qwen-vs-gemma4.md)
