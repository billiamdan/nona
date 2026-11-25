# ADR-0023: Safety policy for actions initiated by a model

**Status:** Proposed · **Date:** 2026-11-25

## Context

A model that can call tools can operate real devices, and text that reaches it can steer it.

What the research found (November 2026):

- Demonstrated attacks exist: in 2025 a poisoned calendar invite made a commercial assistant open windows, switch lights and start a boiler.
- The published defences are architectural, not model-based: let the model only choose among pre-defined actions, keep untrusted text out of the context that can act, require confirmation for sensitive actions, expose only what is needed, and keep safety-critical automations independent of the model.
- Small models also refuse too often, so "never acts" must not be mistaken for "safe and useful".
- No research was found on injection through device names or sensor text specifically; that part applies the general patterns.

## Decision

1. **Action selector.** A model can only select a registered tool with arguments from enumerated values: entity IDs and bounded numeric ranges. The output is constrained to that schema and validated again in code. An injected instruction cannot create a new capability; the worst case is a wrong but permitted action.
2. **Risk class per entity**, set by the owner in the hub:

| Class | Examples | Model-initiated action |
|---|---|---|
| `free` | Lights, sockets with benign loads | Executes |
| `confirm` | Sockets feeding heaters or irons, the curtain drive, anything at night while nobody is up | The hub asks back on the same channel and executes only after a yes |
| `never` | Anything the owner chooses | Not visible to the model at all |

   A command resolved by the deterministic matcher is the user's own explicit command and is not subject to confirmation. A rule the owner approved runs as approved.
3. **Untrusted text is data.** Names reported by devices, sensor text and transcribed background speech are length-limited and passed as quoted values or IDs, never as instructions. The names and aliases a model sees are the ones the owner typed into the hub.
4. **Rules drafted by a model** are validated against the rule schema, checked for existing entities and sane limits, and activated only after the owner approves them in the app.
5. **Audit log.** Append-only: the request, the agent, the model and endpoint, the tool call, the validation result and the action executed.
6. **Limits.** A rate limit on model-initiated actions, and one global switch: "AI control off".
7. **Measurement.** The regression set counts unsafe executions and missed legitimate commands as two separate numbers ([ADR-0018](0018-llm-runtime-and-tiers.md)).

## Consequences

- A model's mistakes are bounded to permitted, logged and mostly reversible actions.
- The owner has to classify each entity once. New entities default to `confirm` until classified.
- Confirmation adds a turn to some voice exchanges.

## Revisit when

- Locks, heating or anything else with security or safety consequences is added. Each such family gets its own record first.
- Cameras arrive: images and their descriptions are untrusted input too ([ADR-0008](0008-cameras-later.md)).

Evidence: [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), section 9.
