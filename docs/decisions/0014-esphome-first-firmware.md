# ADR-0014: ESPHome first for controller firmware

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief includes firmware for the controllers. Writing firmware from scratch means rebuilding Wi-Fi provisioning, over-the-air updates and a driver for every sensor.

What the research found (November 2026):

- ESPHome without Home Assistant is practical and documented when nodes talk MQTT: entities, self-description, over-the-air updates, provisioning and the on-device web page all work with no hub.
- Some features exist only on ESPHome's native API: the voice assistant, the Bluetooth proxy, infrared and radio proxies. A third-party controller can use that API through the `aioesphomeapi` Python library, at the cost of tracking a fast-moving library whose only reference consumer is Home Assistant.
- Running MQTT for ordinary entities and the native API only on the few nodes that need it looks possible, but was not found described as a tested setup.
- All the sensor types the brief needs have native ESPHome components.
- No tested ESPHome configuration was found for either of the two owned M5Stack devices.

## Decision

- Our nodes run ESPHome. Their YAML lives in the repository under `firmware/esphome/`: one shared base package (Wi-Fi, MQTT with self-description, over-the-air updates, time from the hub, logging) and one file per node.
- Nodes speak MQTT for ordinary entities. The native API is switched on only for nodes that need it, such as voice satellites, and the hub consumes it through `aioesphomeapi` ([ADR-0020](0020-voice-pipeline-and-speech-stack.md)).
- A node without the native API leaves out the `api:` block; a node with it sets `reboot_timeout: 0s`. Otherwise the node restarts every 15 minutes while no client is connected.
- Custom code is written only where ESPHome has no component, and then first as an ESPHome external component. A standalone firmware is the last resort and needs its own record.
- The ESPHome version is pinned and updated deliberately.

## Consequences

- Firmware is declarative, reviewable and updated from the hub.
- The two M5Stack devices need pin-level configurations written and tested by us. This is a known risk of Phase 0, and their role there is to prove the path, not to be permanent fixtures.
- The mixed MQTT-plus-native-API setup is unproven and gets a spike before any voice satellite is bought.
- Battery-powered Wi-Fi nodes are a poor fit; battery sensors are bought as Zigbee devices instead ([ADR-0015](0015-radio-strategy.md)).

## Revisit when

A node needs behaviour ESPHome cannot express even with an external component.

Evidence: [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), section 2; [radio, controller and sensor notes](../../research_notes/Local%20smart%20home%20stack%20options/radio_mcu_sensors.md), sections 4 and 5.
