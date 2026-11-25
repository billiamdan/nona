# ADR-0012: MQTT is the control-plane bus

**Status:** Proposed · **Date:** 2026-11-25

## Context

Devices, bridges, the hub and later modules need one contract between them. ESPHome, Zigbee2MQTT, Shelly, Tasmota, OpenBeken and WLED all speak MQTT. The research found Mosquitto to be the default broker for a single household; EMQX is no longer open source.

## Decision

- All device state and all device commands travel over MQTT through one Mosquitto broker on the host.
- Each firmware and bridge keeps its native topics (for example `zigbee2mqtt/...`, an ESPHome node's own prefix, and the `homeassistant/...` discovery tree). The hub adapts to them; devices are not reconfigured to suit the hub.
- The hub owns the `nona/` namespace for what it originates: its own status, and the virtual devices published by perception modules ([ADR-0017](0017-declarative-rules-virtual-sensors.md)).
- MQTT carries control only. Audio for voice travels point to point and never through the broker ([ADR-0020](0020-voice-pipeline-and-speech-stack.md)).
- Conventions: retained messages for device descriptions and last state; an availability topic with a last-will message for every device; anonymous access off, one credential per device or bridge, topic permissions per credential ([ADR-0021](0021-network-and-security.md)).

## Alternatives considered

| Option | Why not |
|---|---|
| Talk to each device over its own API (ESPHome native API, HTTP) | The hub would have to connect to every device and know every protocol; nothing can be observed or tested from outside |
| The Homie convention on MQTT | Technically close to what we need, but almost no mainstream firmware emits it |
| Another bus (NATS, Redis streams) | Devices do not speak it; a bridge per device family would be needed |

## Consequences

- Any component can be developed, observed and tested independently with standard tools.
- A module can be written in any language.
- The broker is a single point of failure. It runs as a supervised service, and devices keep working locally when it is down: plugs keep their physical button, bulbs keep their power-on behaviour.
- In the first version the hub does not mirror its normalised state back onto the bus. That can be added if modules outside the hub process need it.

## Revisit when

A needed device family cannot be reached through MQTT without unreasonable effort.

Evidence: [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), sections 2, 3 and 5.
