# Decision records

One short record per decision. Why this practice exists: [ADR-0000](0000-record-decisions.md).

- **Accepted** records were decided by the owner, in the project brief or in the design Q&A of 25 November 2026.
- **Proposed** records are recommendations that follow from those decisions and from the research. Work proceeds on them, and they await the owner's review.

## Foundations

| # | Decision | Status |
|---|---|---|
| [0000](0000-record-decisions.md) | Record every decision as a short Markdown record | Accepted |
| [0001](0001-local-only.md) | Local only: no cloud, no subscription, no vendor account | Accepted |
| [0002](0002-own-hub-reuse-edges.md) | Build our own hub, reuse open building blocks at the device edge | Accepted |
| [0003](0003-laptop-host-tiered-compute.md) | The owned Linux laptop is the 24/7 host; compute is tiered | Accepted (host), Proposed (tiering) |
| [0004](0004-mains-safety.md) | Mains safety: certified parts under open local control, DIY only at low voltage | Accepted |
| [0005](0005-reversible-installation.md) | Reversible installation only (rented apartment, EU 230 V) | Accepted |
| [0006](0006-english-and-russian-voice.md) | Voice works in English and Russian | Accepted |
| [0007](0007-voice-input-in-stages.md) | Voice input arrives in four stages | Accepted (sources), Proposed (order) |
| [0008](0008-cameras-later.md) | Cameras later: vision as an optional perception module | Accepted |
| [0009](0009-python-hub.md) | The hub is written in Python | Accepted |
| [0010](0010-web-app-first.md) | The phone app is a web app first | Accepted |
| [0011](0011-phased-purchasing.md) | Buy in phases; the first round stays within €100 | Accepted |

## Architecture

| # | Decision | Status |
|---|---|---|
| [0012](0012-mqtt-control-bus.md) | MQTT is the control-plane bus | Proposed |
| [0013](0013-self-describing-devices.md) | Devices describe themselves; everything else is generated from the registry | Proposed |
| [0014](0014-esphome-first-firmware.md) | ESPHome first for controller firmware | Proposed |
| [0015](0015-radio-strategy.md) | Radio strategy: Zigbee for bought devices, Wi-Fi for our own nodes | Proposed |
| [0016](0016-deterministic-core.md) | Deterministic core: the model proposes, the engine executes | Proposed |
| [0017](0017-declarative-rules-virtual-sensors.md) | Complex conditions are declarative rules; perception arrives as virtual sensors | Proposed |
| [0018](0018-llm-runtime-and-tiers.md) | One model interface, an ordered list of endpoints, starting models by tier | Proposed |
| [0019](0019-agent-design.md) | A few narrow agents behind a deterministic router, not a swarm | Proposed |
| [0020](0020-voice-pipeline-and-speech-stack.md) | Voice pipeline and starting speech stack | Proposed |
| [0021](0021-network-and-security.md) | Network and security baseline | Proposed |
| [0022](0022-repo-layout-and-deployment.md) | One repository; third-party services in containers, our own services native | Proposed |
| [0023](0023-ai-action-safety.md) | Safety policy for actions initiated by a model | Proposed |

## Adding a record

1. Copy the template below to `NNNN-short-title.md` with the next free number.
2. Add a row to the table above.
3. If it replaces an older record, set that record's status to "Superseded by ADR-NNNN". Do not rewrite the older record.

```markdown
# ADR-NNNN: Title that states the decision

**Status:** Proposed · **Date:** YYYY-MM-DD · **Decided by:** who, and where

## Context

What forces are at play, and what was found out.

## Decision

What we do, stated so that someone could act on it.

## Alternatives considered

| Option | Why not |
|---|---|

## Consequences

What becomes easier, what becomes harder, what we now owe.

## Revisit when

The condition that should reopen this decision.

Evidence: links to research notes, measurements or sources.
```
