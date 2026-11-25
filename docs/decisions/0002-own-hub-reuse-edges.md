# ADR-0002: Build our own hub, reuse open building blocks at the device edge

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

There are three ways to get a local smart home: adopt Home Assistant as the core, build a hub and reuse open components for the device plumbing, or build everything.

What the research found (November 2026):

- Home Assistant is the most complete and most active option and now has first-party hooks for local models. Its documented costs are a monthly stream of breaking changes, a large surface, and model control that is only fast and dependable with few exposed entities and deterministic matching in front of the model.
- The device edge (firmware, Zigbee device support) is where the bulk of community effort sits. Nobody who built a custom hub and stayed happy wrote their own firmware or Zigbee stack; they kept ESPHome and Zigbee2MQTT and wrote only the logic.
- The successful local-model setups converge on a narrow, cache-stable tool set with deterministic routing first. Owning the hub means owning that interface.

## Options considered

| Option | For | Against |
|---|---|---|
| Home Assistant as the core | Voice, phone app, Matter and the largest integration catalogue work today | Upgrade churn; a large surface we do not control; the model interface is Home Assistant's, not ours; standard push notifications and Matter commissioning lean on phone vendors |
| **Own hub, reuse the edge (chosen)** | Full control of the model interface; a small codebase we understand; devices describe themselves over MQTT | We must build the discovery consumer, the schedule and rule engine, the phone UI, the voice server side and the tool layer; every device family that does not describe itself costs adapter work |
| Everything from scratch | Maximum control | Rebuilds an enormous amount of device support; no account of anyone succeeding was found |

## Decision

We build the hub core, the agents and the app. We reuse, unmodified:

| Reused block | Role |
|---|---|
| Mosquitto | MQTT broker ([ADR-0012](0012-mqtt-control-bus.md)) |
| ESPHome | Firmware for our ESP32 nodes ([ADR-0014](0014-esphome-first-firmware.md)) |
| Zigbee2MQTT | Bridge for Zigbee devices ([ADR-0015](0015-radio-strategy.md)) |
| A local model server and local speech servers | Separate processes behind standard interfaces ([ADR-0018](0018-llm-runtime-and-tiers.md), [ADR-0020](0020-voice-pipeline-and-speech-stack.md)) |

What we write ourselves:

1. the consumer for device self-descriptions and the device registry,
2. the schedule and rule engine,
3. the web app,
4. the server side of voice,
5. the tool layer between the model and the bus.

## Consequences

- The decision is reversible at the edge. Devices announce themselves over MQTT in a format Home Assistant also reads, so Home Assistant can later be attached beside our hub, or instead of it, without touching a device.
- Scope discipline is required: a narrow set of device families (ESPHome and Zigbee2MQTT first; small adapters for other firmware only when such a device is actually bought).
- Other builders report the most pain in the phone UI and the voice server side. Both therefore get early, thin slices in [roadmap.md](../roadmap.md).
- Fallback, recorded so it is not forgotten: if the UI or voice work proves too costly, Home Assistant can run as a device-and-app layer beside our service, which would keep voice, model routing and complex conditions.

## Revisit when

- After the rules phase, if the effort of building the hub is far beyond plan.
- When a wanted device family has no self-description over MQTT and no small adapter is feasible.

Evidence: [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), sections 1, 9 and 10.
