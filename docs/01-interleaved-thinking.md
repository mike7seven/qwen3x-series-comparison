# Interleaved Thinking

*Synthesized from a research conversation on 2026-04-19.*

## What "Interleaved Thinking" Means in 2026

The phrase is used two different ways in the LLM space, and conflating them causes most of the confusion around Qwen3.6 documentation:

1. **Interleaved Thinking (Multimodal / Token-level)** — the model's ability to process mixed modalities (text, image, audio tokens) within a single sequence rather than as separate blocks. This keeps the "hidden states" grounded in all inputs simultaneously and prevents the model from forgetting an image while it's halfway through writing the text description.

2. **Reasoning Persistence (`preserve_thinking`)** — keeping the Chain-of-Thought `<think>` tokens in the conversation history across multiple turns so the model can reference its own prior reasoning. This is a *context management* strategy, not a multimodal one.

Qwen3.6 uses both, but they are distinct features. When Alibaba's docs say the default behavior is an "interleaved thinking pattern," they are using the term loosely to describe turn-by-turn alternation between hidden thought and visible output — **not** the multimodal definition.

## Why Interleaved Thinking Exists in Reasoning Models

Three primary reasons LLMs use interleaved-style processing:

### 1. Modality Synchronization
Multimodal models interleave text, image tokens, and audio data within one sequence so attention stays grounded across all inputs at every step.

### 2. Dynamic Chain-of-Thought
Reasoning-heavy models alternate between **drafting a solution** and **self-correcting** within a single response. Instead of generating one long linear answer, the model interleaves its internal monologue with output tokens. This lets it catch logic errors in real time — if a reasoning token predicts a contradiction, the output token can pivot immediately.

### 3. Contextual RAG & Long Context
For large context windows, interleaved attention lets the model "think" back and forth between the user's constraints and the provided data, rather than reading the document once and answering. This reduces the "lost in the middle" problem.

## Blocked vs. Interleaved — Quick Reference

| Feature | Blocked Thinking (Traditional LLM) | Interleaved Thinking (Qwen3.x Style) |
|---|---|---|
| **Logic** | Linear: follows the most probable next word. | Iterative: loops between reasoning and output. |
| **Multimodal** | Processes image first, then text. | Processes image and text tokens together. |
| **Error Correction** | Only "realizes" mistakes after finishing. | Evaluates and adjusts during generation. |

---

## Related Pages

- [Preserve Thinking in Qwen 3.6](02-preserve-thinking.md)
- [Toggling Thinking & Sampling Parameters](03-thinking-toggle-and-parameters.md)
- [Dynamic Thinking & The Model Router](04-dynamic-thinking-router.md)
- [Qwen 3.6 vs. Gemma 4: Two Philosophies](05-qwen-vs-gemma4.md)
