# ADR-0004: Mains safety: certified parts under open local control, DIY only at low voltage

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

Sockets and lights switch 230 V. Home-made relay boards are cheap and flexible, but they carry a real risk of fire and electric shock and have no certification. Many certified products are built on ESP32-class chips and can either run open firmware or expose a documented local interface.

## Decision

- Everything connected to mains is a bought, certified product: smart plugs, bulbs, LED power supplies, curtain drives with their own adapters.
- On those products, local control comes from one of three things:
  - Zigbee, which is preferred wherever the product exists as a Zigbee device ([ADR-0015](0015-radio-strategy.md));
  - open firmware installed by the maker (ESPHome or Tasmota pre-flashed);
  - a documented local interface in the stock firmware.
- Flashing open firmware ourselves is the last resort. The research found that in 2026 it has become the hard path: the common over-the-air exploit has been patched since 2022, and current plugs must be opened destructively. Buy open; do not count on flashing.
- The owner's choice in the Q&A was "certified parts, open firmware". Zigbee devices meet its intent: they are controlled locally through an open protocol and never need a vendor's cloud.
- Our own electronics stay at safe extra-low voltage, typically 5 V from a USB supply: sensors, ESP32 nodes, LED strip controllers fed by a certified power supply, curtain-drive experiments.
- A mains device is never opened or connected to a serial programmer while it is plugged in. Serial flashing is done with the device unplugged and powered from the programmer.
- The load rating on the product label is respected. Heaters and other high-power loads go only on plugs rated for them.

## Alternatives considered

| Option | Why not |
|---|---|
| Certified parts, stock firmware only | Keeps the warranty but gives less control; the owner preferred open firmware |
| Fully DIY, including mains relay boards | Cheapest and most flexible, but the fire and shock risk and the lack of certification are not acceptable |

## Consequences

- A switched load costs a little more than a bare relay module would. The research found certified modules at €10–20, so there is no economic case for home-made mains switching.
- Flashing a device ourselves costs the maker's warranty and support; it is not illegal. Products that ship open avoid the issue.
- Sockets that feed heating loads are treated as risky by the safety policy for AI-initiated actions ([ADR-0023](0023-ai-action-safety.md)).

## Revisit when

The owner moves to an owned home. In-wall modules then become possible, but they remain certified products, installed by a qualified person where the law requires it.
