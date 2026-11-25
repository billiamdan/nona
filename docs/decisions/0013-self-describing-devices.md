# ADR-0013: Devices describe themselves; everything else is generated from the registry

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief asks for modularity: adding a sensor or device must bring its own module to read and control it.

What the research found (November 2026):

- Home Assistant's MQTT discovery format is the de-facto way for a device to describe itself over MQTT. Dozens of tools emit it, including ESPHome, OpenBeken and OpenMQTTGateway, and other hubs consume it.
- Zigbee2MQTT publishes a richer description of every Zigbee device in one retained document (`bridge/devices`, with an `exposes` list carrying value ranges and access rights).
- Tasmota, Shelly and WLED use their own announcement schemes and would each need a small adapter.

## Decision

**One internal model.** A device has entities. Each entity has exactly one capability type from a small fixed set, with typed state and typed commands:

| Capability | State | Commands |
|---|---|---|
| `switch` | on/off | on, off, toggle |
| `light` | on/off, brightness, colour temperature, colour | the same, with transitions |
| `cover` | position, moving | open, close, stop, set position |
| `sensor` | number with unit and class (illuminance, temperature, sound level, power) | none |
| `binary_sensor` | true/false with class (presence, contact, waking activity) | none |
| `button` / `event` | none | press; emits events |
| `number` / `select` | a setting | set |

**Adapters bring devices in.** An adapter is a small Python plugin that turns one announcement scheme into this model and translates commands back.

| Adapter | Covers | When |
|---|---|---|
| `ha_discovery` | ESPHome nodes, OpenBeken, OpenMQTTGateway, our own virtual sensors. Targets the stable core of the format and ignores unknown keys. | First version |
| `zigbee2mqtt` | Every Zigbee device, from the `bridge/devices` document | When the Zigbee tier is bought |
| Shelly, Tasmota, WLED | One adapter each | Only when such a device is actually bought |
| Standalone Matter Server | Matter and Thread devices | Only if Thread is adopted ([ADR-0015](0015-radio-strategy.md)); the registry leaves room for it |

**Everything downstream is generated, never hand-written per device:** the app's cards (one component per capability type), the conditions and actions offered in rules, the arguments the model may use (enumerated entity IDs), and the names voice understands.

**Human knowledge lives in the hub, not in firmware:** room, display names and aliases in English and Russian, risk class ([ADR-0023](0023-ai-action-safety.md)), and whether an entity is exposed to voice and to the model.

## Consequences

- A new device of a known family needs no code. A new family needs one adapter.
- The capability set must stay small. An entity of an unknown type is shown as unsupported instead of breaking anything.
- Home Assistant can read the same announcements, which keeps [ADR-0002](0002-own-hub-reuse-edges.md) reversible.
- A device's own name is treated as untrusted text and is never shown to the model as an instruction ([ADR-0023](0023-ai-action-safety.md)).

## Revisit when

A wanted device does not fit any capability type. Extend the set deliberately, with a new record.

Evidence: [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), sections 3 and 5.
