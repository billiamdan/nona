# ADR-0015: Radio strategy: Zigbee for bought devices, Wi-Fi for our own nodes

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief wants wireless links wherever possible. No single radio suits every device role, and each additional radio is another gateway to run.

What the research found (November 2026):

- Zigbee through Zigbee2MQTT covers 5,868 devices from 616 vendors. Bought battery sensors cost roughly $5–20 and last one to five years.
- Documented DIY battery nodes on ESP32-C6 or H2 drained in about two months unless the board was stripped of LEDs, regulator and charger. Buying battery sensors wins over building them.
- A Wi-Fi node takes seconds to wake and report, so Wi-Fi suits powered nodes. Users report trouble on consumer routers from about 15 ESP32 clients upward.
- Thread and Matter work locally, but commissioning Thread devices without a phone ecosystem is the hard part, the cheapest Matter-over-Thread range had pairing and dropout problems through 2026, and Zigbee2MQTT cannot share one radio with Thread.
- ESPHome has a native CC1101 component (300–928 MHz) that pairs with its remote transmitter and receiver for simple 433 MHz remotes. Rolling-code remotes are hard.

## Decision

| Device role | Radio | Path to the bus |
|---|---|---|
| Bought bulbs, plugs, battery sensors, buttons, the curtain drive | Zigbee 3.0 | A USB coordinator on the laptop, Zigbee2MQTT |
| Our own powered nodes, LED controllers, voice satellites, and any Wi-Fi plug that ships with open firmware | Wi-Fi, 2.4 GHz | ESPHome over MQTT |
| Cheap remote-controlled 433 MHz devices where no feedback is needed | Sub-GHz, through the CC1101 on the M5 stick | ESPHome over MQTT |
| Infrared appliances | Infrared, through the M5 devices | ESPHome over MQTT |

Rules:

1. Battery devices are bought, not built.
2. If a device exists as a Zigbee product, prefer it to a Wi-Fi one, to keep the Wi-Fi client count low.
3. Our Wi-Fi nodes live on a dedicated 2.4 GHz IoT network ([ADR-0021](0021-network-and-security.md)).
4. No Matter or Thread in the first version.
5. The coordinator is a USB dongle on the laptop, on a short extension lead away from USB 3 ports. A network-attached coordinator is the upgrade if the host changes or the radio needs a central position.

Kept in reserve, each needing its own record before use:

| Radio | Possible use |
|---|---|
| Bluetooth LE advertisements (BTHome) | Very cheap thermometers, received by an ESP32 node |
| ESP-NOW | Our own battery nodes, if one is ever needed |
| LoRa on the Cardputer | Anything outside the apartment's radio range |
| Thread and Matter | A wanted device that exists only as Matter-over-Thread |

## Consequences

- 433 MHz control is one-way: the hub assumes the state it last sent, and anyone nearby with a remote can change it. It is used only for low-risk loads and never for security.
- Two gateways run from the start of the Zigbee tier: the broker's Wi-Fi clients and the Zigbee coordinator.
- The LoRa radio on the Cardputer has no role inside one apartment; the device is used for its keyboard, screen, microphone and infrared transmitter.

## Revisit when

- A wanted device is available only on a radio in reserve.
- The home grows beyond one apartment.

Evidence: [radio, controller and sensor notes](../../research_notes/Local%20smart%20home%20stack%20options/radio_mcu_sensors.md), sections 1, 2, 4, 6 and 7; [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), section 6.
