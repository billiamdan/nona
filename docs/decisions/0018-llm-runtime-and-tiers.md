# ADR-0018: One model interface, an ordered list of endpoints, starting models by tier

**Status:** Proposed · **Date:** 2026-11-25 · Model choices are provisional until the Phase 0 benchmark has run

## Context

Models run on two machines of very different strength ([ADR-0003](0003-laptop-host-tiered-compute.md)), and the best model changes every few months. The hub must not care which model answers.

What the research found (November 2026):

- Current generations: Gemma 4 (April 2026, Apache 2.0; E2B, E4B, 12B, 26B A4B, 31B) and Qwen3.5 small models (March 2026; 0.8B, 2B, 4B, 9B), with Qwen's 27B class advanced to Qwen3.8-27B (August 2026). Older generations are superseded at the same memory footprint.
- Among roughly 4B models, Qwen3.5-4B is at or near the top for executing tool calls, while Gemma 4 E4B is better at deciding when not to call a tool. For a home, wrongly firing an action costs more than declining.
- With about eight tools, 4–9B models pick the right tool almost every time. They remain weak at multi-step chains and exact arguments, and failures are often caused by the runtime's templates, not the model.
- `llama-server` from llama.cpp is the most complete single runtime for an always-on backend: an OpenAI-compatible endpoint, tool templates, schema and grammar constraints, a prompt cache, parallel slots, several models behind one server; MIT licence. Ollama is the easiest to operate.
- What matters most for response time is processing the prompt, so a stable prompt prefix that the cache can reuse is a first-order design concern.

## Decision

- The hub talks to models through one interface only: an OpenAI-compatible chat endpoint with tool calling.
- Endpoints are an ordered list with health checks: the Mac first while it is reachable, then the laptop.
- Runtime on the laptop: `llama-server` as a system service. Runtime on the Mac: decided by the benchmark between Ollama and `llama-server`.

Starting candidates, all at 4-bit:

| Tier | Machine | Candidates | Notes |
|---|---|---|---|
| B | Laptop, 16 GB, CPU only | Qwen3.5-4B; Gemma 4 E4B; Nemotron 3.5 Bolt E4B; GigaChat3.1-10B-A1.8B (Russian-first, MIT, about 6.5 GB) | Thinking off. Roughly 5–17 tokens per second expected for the 4B class. A 1–2B first-stage model is an option if the benchmark supports it. Try the integrated GPU as well as the CPU. |
| C | MacBook, 48 GB | Qwen3.8-27B or Gemma 4 31B for quality; Gemma 4 26B A4B or Qwen3.6-35B-A3B for speed; T-pro-it-2.1 (32B, Apache 2.0) for Russian | Image input is a selection criterion here ([ADR-0008](0008-cameras-later.md)) |

Settings that apply to every tier:

- an explicit context length, and the model kept loaded;
- a stable prompt prefix: a fixed tool list and a fixed order of entities and aliases;
- thinking off for device commands, on only for drafting rules;
- the tool name and the entity arguments constrained to enumerated values, everything else left free;
- the server bound to localhost on the laptop; on the Mac, bound to the LAN with an API key.

**Selection is by our own regression set**, not by public leaderboards: 30–50 commands in English, Russian and mixed language, including cases where the right answer is to do nothing. It counts unsafe executions and missed commands separately and is rerun after every model or runtime update. Known-good runtime builds are pinned.

## Consequences

- A model or runtime can be swapped by editing configuration.
- Russian is the least certain part. In the paired figures available, the same model scores about 5–12 points lower on Russian function calling than on English. The best-documented Russian tool callers are T-Tech's Qwen3 fine-tunes (T-lite-it-2.1 at 8B, T-pro-it-2.1 at 32B); the 8B one is too slow for live voice on the laptop. No Russian tool-calling data was found for Qwen3.5 or Gemma 4, and nothing on Cyrillic or mixed-language entity names. The regression set decides.
- New model families typically need weeks of runtime fixes before tool calling is dependable, so the newest release is not adopted on day one.

## Revisit when

- A new small-model generation is released (the research flags a small Qwen3.8 as the likely next change).
- The benchmark contradicts the expected speeds.

Evidence: [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), sections 1 to 5, 10 and 11.
