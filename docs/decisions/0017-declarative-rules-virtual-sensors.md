# ADR-0017: Complex conditions are declarative rules; perception arrives as virtual sensors

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief's example of a complex condition is: morning, a certain illuminance level, and the sound of people waking up. Two of those are plain sensor facts. The third has to be inferred.

What the research found (November 2026):

- For recognising household sounds, small dedicated audio models match or beat audio-capable language models, and the language models state wrong answers confidently. A language model per audio clip is also too costly to run continuously.
- The usual default classifier, YAMNet, was the weakest in the 2026 comparisons and must not be assumed adequate without testing on the home's own sounds.
- ESPHome's sound-level sensor gives relative loudness only; it cannot tell what kind of sound it hears.
- Automations triggered by perception should rely on classifier scores and corroborating sensors, not on a model's own claim of certainty.

## Decision

**A rule is data.** It has three parts and is stored in the hub's database:

| Part | Contents |
|---|---|
| Triggers | A state change, a time, a sun event, a device event |
| Conditions | A tree of and / or / not over: comparisons, time windows, "this state has held for N minutes", sun position |
| Actions | Device commands, scenes, delays, notifications |

- Rules are JSON with a published schema. The same schema validates a rule made in the app and a rule drafted by a model ([ADR-0016](0016-deterministic-core.md)).
- Every rule can run in dry-run mode, which logs what would have fired. Every evaluation leaves a trace that answers "why did this fire?".

**Perception modules publish virtual sensors.** A module turns raw signals into an ordinary entity on the bus, described like any device ([ADR-0013](0013-self-describing-devices.md)). Rules cannot tell a virtual sensor from a physical one.

The brief's example then becomes one rule:

> between 06:30 and 10:00, and bedroom illuminance below a threshold, and `bedroom_waking_activity` true for two minutes → open the curtain to 30% and set the lamp to a warm 20%.

**Waking activity is built in two stages:**

1. First, on the node: sound level only, combined in the hub with radar presence. No audio leaves the node.
2. Later, on the host: a dedicated audio classifier on short clips, tested on this home's own recordings before it is trusted.

A vision module, when cameras arrive, plugs in the same way ([ADR-0008](0008-cameras-later.md)).

## Consequences

- Rules are deterministic, testable and explainable.
- Noisy sensors need hysteresis and hold times; the schema has them from the start.
- What a rule can express is limited to the schema, and the schema is extended deliberately.
- Raw audio is never stored by default ([ADR-0001](0001-local-only.md)).

## Revisit when

A wanted condition cannot be expressed without making the schema a programming language. Then consider a scripted-rule escape hatch, with its own record.

Evidence: [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), section 7; [radio, controller and sensor notes](../../research_notes/Local%20smart%20home%20stack%20options/radio_mcu_sensors.md), section 5.
