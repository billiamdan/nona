# ADR-0001: Local only: no cloud, no subscription, no vendor account

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (project brief)

## Context

The purpose of the project is a smart home that depends on no subscription and gives full control over data. Commercial ecosystems tie devices to vendor clouds: a discontinued service or a changed plan can disable hardware, and audio and sensor data leave the home.

## Decision

Every function of the system must keep working with the internet cable unplugged.

1. No component may need a vendor account, a paid plan or a cloud API in order to operate. One-time internet use to download software, models and firmware is fine.
2. A device is acceptable only if it can be controlled locally through an open protocol or open firmware. A device that needs a vendor app once for pairing and never again is a last resort and must be flagged as such in [bom.md](../bom.md).
3. Speech, language and perception models run on our own machines. Raw audio, and later images, stay on the local network and are not stored by default.
4. IoT devices get no internet access at the network level ([ADR-0021](0021-network-and-security.md)).

## Consequences

- Ruled out by default: cloud-bound Wi-Fi devices unless they are reflashed or speak Zigbee, commercial voice assistants, cloud language-model APIs, hosted push and remote-access services.
- Phone notifications and access from outside the home need extra work, because background push on iPhones and NAT traversal normally rely on third parties ([ADR-0010](0010-web-app-first.md), [ADR-0021](0021-network-and-security.md)).
- Model quality is bounded by our own hardware ([ADR-0003](0003-laptop-host-tiered-compute.md), [ADR-0018](0018-llm-runtime-and-tiers.md)).
- More setup effort is accepted in exchange for independence.

## Revisit when

Never as a whole. A single exception, for example a third-party push service that carries only a wake-up signal, needs its own record stating exactly what leaves the home.
