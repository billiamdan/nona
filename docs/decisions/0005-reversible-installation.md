# ADR-0005: Reversible installation only (rented apartment, EU 230 V)

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

The home is a rented apartment in the EU (230 V, 50 Hz). The wiring behind switches and sockets must not be touched, and everything has to be restorable when moving out.

## Decision

Every device must be removable without a trace.

| Area | Allowed | Not allowed |
|---|---|---|
| Ceiling lights | Smart bulbs in the existing sockets (E27, E14, GU10) | Relay or dimmer modules behind the switch |
| Floor and table lamps | Smart plugs or smart bulbs | |
| LED strips | A low-voltage controller with a certified power supply | |
| Wall control | Battery buttons and remotes stuck beside or over the existing switch; a guard that keeps the switch on | Replacing the wall switch |
| Sockets | Plug-in smart plugs in the country's plug type | Replacing wall sockets |
| Curtains | A retrofit drive that rides on the existing track | Replacing the rail, drilling |
| Sensors | Adhesive or freestanding, battery or USB powered | |
| Network | Our own small router for an isolated IoT network when the building's router cannot be configured ([ADR-0021](0021-network-and-security.md)) | Changes to the building's network equipment |

## Consequences

- A smart bulb loses power when someone flips the wall switch. Switch guards and stick-on buttons reduce this, and the hub must treat an unreachable bulb as "switched off at the wall", not as a fault.
- A fixture with built-in LEDs cannot take a smart bulb; only a switch-pressing robot would work there. None is planned, because the owner's lights have replaceable bulbs.
- Everything moves with the owner to the next home.
- The plug type must match the country. Type F (Schuko) is assumed until the country is known; France, Belgium, Poland, Czechia and Slovakia use Type E sockets.

## Revisit when

The owner moves to an owned home, or the landlord agrees in writing to specific in-wall devices.
