# ADR-0016: Deterministic core: the model proposes, the engine executes

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief puts local language models at the centre. The 24/7 host has no GPU ([ADR-0003](0003-laptop-host-tiered-compute.md)).

What the research found (November 2026):

- Nearly every account of a working local voice assistant converges on the same shape: deterministic intent matching first, a small tool-calling model as the fallback, few exposed entities, a cached prompt, and no reasoning mode in the voice loop.
- Model-only control on CPU-class hardware is what failed.
- The deterministic layer is not only faster. It is the safety net for an unreliable model, and condition-based triggers belong in rules.

## Decision

1. Manual control, schedules and rules run in the hub core with no model involved. With every AI process stopped, the home still works.
2. A voice or text request first goes to a deterministic intent matcher: patterns per language over the names and aliases in the device registry. Only a request it cannot resolve goes to a model.
3. A model never controls a device directly and never evaluates a rule at run time. It can do two things:
   - select a registered tool with arguments drawn from enumerated values, which the hub validates and then executes;
   - draft a rule as data, which the owner approves before it becomes active.
4. A condition that involves perception, such as "people are waking up", is evaluated by the rule engine against a virtual sensor ([ADR-0017](0017-declarative-rules-virtual-sensors.md)), not by asking a model.

## Consequences

- Common commands answer in well under a second on the laptop, with or without the Mac.
- Every action can be explained: which pattern or rule fired, or which tool call a model proposed.
- The model's job is narrow, so a small model is enough for most of it.
- The cost is up-front work on intent patterns in two languages, and a rule engine we write ourselves.

## Revisit when

Local models on the always-on host become fast and reliable enough that the deterministic matcher is mostly bypassed. Even then, schedules and rules stay model-free.

Evidence: [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), sections 7 and 8; [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), sections 3 and 9.
