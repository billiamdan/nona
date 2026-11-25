# Nona

A lean smart home that runs entirely on the local network: a self-built hub on an MQTT bus, ESP32 controllers with open firmware, Zigbee devices, and open-weight AI models on a home computer that is always on. No subscription, no vendor cloud, no account.

**Status (25 November 2026):** designed, not yet built. The next step is Phase 0 of the [roadmap](docs/roadmap.md), which needs no purchases.

## Principles

1. **Local only.** Everything works with the internet cable unplugged.
2. **One bus, self-describing devices.** A new device announces itself and gets its app card, its place in rules, its voice names and its model tool arguments with no new code.
3. **A deterministic core.** Manual control, schedules and rules never depend on a model. A model only proposes, and the hub validates.
4. **Reuse the edge, build the brain.** Broker, firmware and the Zigbee bridge are proven open-source parts; the hub, the agents and the app are ours.
5. **Lean.** One thin end-to-end slice at a time, each ending in a demo; nothing is bought ahead of the phase that uses it.
6. **Every decision is written down.**

## The stack in brief

| Layer | Choice |
|---|---|
| App | Installable web app served by the hub |
| Voice | Push-to-talk first, then satellites; English and Russian; speech models on the laptop |
| Intelligence | Deterministic intent matcher, then a few narrow agents on local models |
| Hub | Python, one process, SQLite; registry, rules, scheduler, safety policy |
| Bus | Mosquitto (MQTT) |
| Edge | ESPHome on ESP32 over Wi-Fi; Zigbee devices through Zigbee2MQTT; 433 MHz and infrared through the M5 devices |
| Host | A Linux laptop around the clock; a MacBook Pro as an optional larger-model server |

## Documents

| Document | What it answers |
|---|---|
| [docs/project-brief.md](docs/project-brief.md) | What is being built, with what, and what the owner decided up front |
| [docs/architecture.md](docs/architecture.md) | How the parts fit: layers, flows, contracts, failure behaviour |
| [docs/roadmap.md](docs/roadmap.md) | The step-by-step plan, phase by phase, with exit demos |
| [docs/bom.md](docs/bom.md) | What to buy, in tiers, with prices and alternatives |
| [docs/decisions/](docs/decisions/README.md) | Every decision, with context, alternatives and consequences |
| [docs/open-questions.md](docs/open-questions.md) | What is not decided or not known yet |
| [reports/Local smart home stack options.md](reports/Local%20smart%20home%20stack%20options.md) | The research report behind the decisions: existing approaches, pros and cons, sources |
| [research_notes/](research_notes/) | The detailed research notes, with sources |
