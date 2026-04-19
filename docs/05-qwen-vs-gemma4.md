# Qwen 3.6 vs. Gemma 4: Two Philosophies of Reasoning State

*Synthesized from a research conversation on 2026-04-19.*

## The Split in 2026

Both Qwen3.6 and Gemma 4 ship with dynamic thinking (autonomous gating), but they disagree sharply on **what to do with reasoning traces between turns**.

| Philosophy | Lead Model | Primary Goal | History Strategy |
|---|---|---|---|
| **Stateful Persistence** | **Qwen 3.6** | Multi-turn consistency & KV cache efficiency | Keep `<think>` blocks in history when `preserve_thinking` is true. |
| **The Clean Slate** | **Gemma 4** (released April 2, 2026) | Avoiding "Ghost Thoughts" & loop prevention | Strip `<think>` blocks from history; only provide the final response to the next turn. |

## Google's Position: The Clean Slate

From Gemma 4 guidance on multi-turn conversations:

> **No Thinking Content in History:** In multi-turn conversations, the historical model output should only include the final response. Thoughts from previous model turns must not be added before the next user turn begins.

Google's reasoning: preserving previous `<think>` blocks in history triggers a **feedback loop** they call **"Ghost Thoughts."** If a model sees its own complex reasoning in its history, it can recurse — thinking about its own thinking instead of addressing the new prompt. Gemma 4's templates include specific "empty thinking tokens" to suppress these ghost channels.

The tradeoff: each turn is a fresh, focused logical operation — but any state that lived only in the scratchpad is gone.

## Alibaba's Position: Stateful Persistence

Qwen3.6 takes the opposite view: reasoning traces are load-bearing, and throwing them away costs you more than you save.

- Without persistence, Turn 2 often has to re-derive Turn 1's logic ("re-dwelling"), producing redundant reasoning tokens.
- Without persistence, the KV cache prefix is invalidated on every turn — even simple non-thinking follow-ups require a full re-prefill.
- Without persistence, the model **hallucinates** state it "should" have (see the [two 20-digit numbers test](02-preserve-thinking.md#the-validation-test)).

## The Validation Divide

The two-number test reveals the philosophical split cleanly:

| Philosophy | Lead Model | Two-Number Test | Why |
|---|---|---|---|
| **Stateful Reasoner** | Qwen 3.6 (with preservation on) | **Passes** — returns the actual second number | Model can see its previous internal logic in the cache. |
| **Clean Slate** | Gemma 4 | **Fails** — invents a new number to stay helpful | Reasoning that didn't reach the final output is purposefully discarded. |

For agentic workflows, the Gemma approach gives the model **goldfish memory** for any logic that didn't get printed.

## Failure Modes, Side-by-Side

| | The Clean Slate (Gemma 4) | The Stateful Reasoner (Qwen 3.6) |
|---|---|---|
| **Logic** | Output-focused: strips thoughts to avoid recursive loops. | Context-focused: keeps thoughts for internal consistency. |
| **Memory model** | Mimics human *semantic* memory — facts only. | Mimics a *scratchpad* that never gets erased. |
| **Failure mode** | Loses data that wasn't explicitly printed. | Token bloat — history becomes large and expensive. |

## The "Human Terms" Argument

An intuitive argument for the Gemma approach: humans generally discard the scaffolding of a thought once we commit the conclusion to long-term memory. You don't remember every synaptic firing it took to calculate your taxes; you just remember the number.

The problem: **LLMs have no biological long-term memory.** They only have the current context window. If a number existed only in the scaffolding and the scaffolding is discarded, the number is gone — and the model, trained to be cooperative, will invent a plausible replacement rather than admit the loss.

### The Middle Ground: Explicit Handoff

A workaround that works on both philosophies: force the model to **summarize its state** into the final visible output, so nothing load-bearing lives only in the scratchpad.

- **State block:** *"At the end of every turn, provide a 'System State' block containing any variables or numbers you generated but did not use."*
- **Compressed reasoning:** before the next turn, distill the previous `<think>` block into a 50-token summary using a secondary model call.
- **Metadata channel:** on APIs that support it, pass hidden state via `metadata` fields to keep thoughts out of the visible chat but present in the request.

## Picking a Side (or Both)

- **Pick Qwen 3.6 (stateful) when:** building long-horizon agents, multi-turn coding workflows, anything where the reasoning is load-bearing across turns, anything where KV cache reuse and TTFT matter.
- **Pick Gemma 4 (clean slate) when:** you want deterministic per-turn behavior, loops are a historical problem in your workflow, you'd rather engineer explicit state than rely on persistence, or you're running on constrained contexts where every token matters.
- **Run both and force explicit handoffs:** write prompts so state lives in visible output. This decouples behavior from the underlying philosophy and makes the workflow portable across models.

## A Final Reframing

Alibaba is handing the agent a **permanent notebook**.
Google is trying to give the agent **perfect intuition**.

Neither is universally right — it depends on whether your workflow's state is best kept in the scratchpad or in the output.

---

## Related Pages

- [Interleaved Thinking — Concepts](01-interleaved-thinking.md)
- [Preserve Thinking in Qwen 3.6](02-preserve-thinking.md)
- [Toggling Thinking & Sampling Parameters](03-thinking-toggle-and-parameters.md)
- [Dynamic Thinking & The Model Router](04-dynamic-thinking-router.md)
