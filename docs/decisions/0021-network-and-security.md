# ADR-0021: Network and security baseline

**Status:** Proposed · **Date:** 2026-11-25

## Context

The home is a rented apartment whose building or ISP router may not be configurable ([ADR-0005](0005-reversible-installation.md)). IoT devices must have no internet access ([ADR-0001](0001-local-only.md)). The web app needs HTTPS for the microphone and for installation to the home screen ([ADR-0010](0010-web-app-first.md)).

What the research found (November 2026):

- A user-owned router in routing mode behind the building's router gives a private network that the building side cannot reach. The double NAT is harmless for outgoing traffic but rules out incoming connections, so a self-hosted VPN endpoint cannot be reached from outside.
- A private certificate authority is the only HTTPS route with no cloud and no recurring cost. Each phone needs a one-time step to trust it.
- Android resolves `.local` names inconsistently; a normal DNS name served by our router is more robust.
- Full TLS for one broker connection is affordable on an ESP32; ESPHome encrypts its updates from version 2026.9.

## Decision

**1. Network.** A small OpenWrt-capable router of our own, in routing mode, behind the building's router, which is treated as untrusted. Two zones:

| Zone | Members | Rules |
|---|---|---|
| Trusted | Phones, the Mac, the hub laptop (by cable) | Normal internet access |
| IoT | ESP32 nodes, plugs, satellites, wall panels; its own 2.4 GHz network | No route to the internet. Devices may reach only the hub's broker and API ports, and the router for addresses, names and time. Only the trusted zone may open connections into it. |

Devices get fixed addresses and reach the hub by a DNS name served by our router, not by mDNS. Time comes from the router or the hub.

**2. Broker.** Anonymous access off, one credential per device or bridge, topic permissions per credential. TLS on MQTT is a later hardening step.

**3. App and API.** HTTPS and secure WebSocket only. Certificates come from our own private certificate authority, run by the reverse proxy in front of the hub (Caddy with its built-in certificate authority); its root is installed once on each phone and tablet. A device is paired with a one-time code shown on the LAN, which yields a token that can be revoked. Authentication is enforced even on the LAN.

**4. Firmware.** Updates are pushed from the hub only, authenticated and encrypted.

**5. Secrets.** Never in the repository. The certificate authority's key, broker credentials and Wi-Fi keys live on the hub laptop only.

**6. Remote access.** None in the first version.

Phasing: the first phase runs on the existing Wi-Fi with broker credentials only, because the first purchase round is small ([ADR-0011](0011-phased-purchasing.md)). The router arrives with a later tier, before the number of Wi-Fi devices grows.

## Consequences

- Devices in the IoT zone cannot phone home, and the building's network cannot see them.
- A one-time trust step per phone, and one more box to own.
- Remote access will need an outbound tunnel: either a free third-party mesh VPN, which adds a company's control plane, or a rented relay server, which is a monthly fee. Both touch [ADR-0001](0001-local-only.md), so the choice needs its own record and the owner's decision.
- Whether the hub laptop's disk is encrypted, and how it then reboots unattended, is an open question ([open-questions.md](../open-questions.md)).

## Revisit when

- The owner wants control from outside the home.
- The building's router turns out to be configurable, which would simplify the topology.

Evidence: [app and security notes](../../research_notes/Local%20smart%20home%20stack%20options/app_remote_security.md), sections 4, 5, 6 and 9; [radio, controller and sensor notes](../../research_notes/Local%20smart%20home%20stack%20options/radio_mcu_sensors.md), section 2.
