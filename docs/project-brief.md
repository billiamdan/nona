# Project brief

What is being built, under which constraints, and what the owner decided up front.
Source: the owner's brief and the design Q&A of 25 November 2026. The decisions that follow from it are indexed in [decisions/README.md](decisions/README.md).

## Goal

A lean smart home that runs entirely on the local network. It is assembled from open-source software, ESP32-class controllers and off-the-shelf parts, and it is driven by open-weight AI models on a home computer that is on around the clock. No subscription, no vendor cloud, and full control over which data exists and where it goes.

## Features for the first version

| # | Feature | Meaning |
|---|---|---|
| 1 | Voice | A spoken request starts an action. |
| 2 | Schedules | Actions run at set times. |
| 3 | Complex conditions | An action fires on a combination such as: morning, a certain illuminance level, and the sound of people waking up. |
| 4 | Modularity | Adding a sensor or a device brings its own module to read and control it; nothing else has to change. |
| 5 | First devices | Sockets, light switches, curtain motors; more to be decided. |
| 6 | Whole system | A phone app, a host application that operates local agents and language models, and firmware for the controllers. |
| 7 | Wireless | Wireless wherever possible: Wi-Fi, Bluetooth and other radio links. |

## Way of working

- Lean: scaffold the architecture first as a thin end-to-end slice, then add one feature at a time. See [roadmap.md](roadmap.md).
- Every good decision is written down as a Markdown record. See [ADR-0000](decisions/0000-record-decisions.md).

## Hardware already owned

| Item | Identified as | Planned role |
|---|---|---|
| M5 stick with radio module | An M5Stick (most likely M5StickC Plus2; the exact model is still to be read off the device) with a CC1101 sub-GHz add-on, 433/868 MHz. The owner confirmed the CC1101. | Development node, 433 MHz gateway, infrared blaster, Bluetooth scanner |
| M5 Cardputer, mesh version | Cardputer Mesh Kit: a Cardputer-Adv (ESP32-S3, keyboard, screen, microphone, speaker, infrared transmitter) with a LoRa SX1262 cap and GNSS, Meshtastic preinstalled | Handheld control panel, push-to-talk microphone prototype, infrared blaster; LoRa is a niche extra |
| Several Android phones | | Wall panels and always-listening voice satellites |
| Several iPhones, an iPad | | Control app; the iPad as a wall panel |
| Linux laptop, 16 GB RAM, no dedicated GPU | Exact CPU still unknown | The 24/7 host |
| MacBook Pro, M5 Pro, 48 GB | | Development machine; optional larger model while it is at home |
| Soldering iron, solder, wires | | Low-voltage DIY nodes |

## Answers from the design Q&A (25 November 2026)

| Topic | Answer | Recorded in |
|---|---|---|
| Build or reuse | Build our own hub, agents and app; reuse an MQTT broker, ESPHome firmware and Zigbee2MQTT at the device edge | [ADR-0002](decisions/0002-own-hub-reuse-edges.md) |
| 24/7 host | The owned Linux laptop to start (16 GB RAM, no dedicated GPU); a dedicated box only if benchmarks demand it | [ADR-0003](decisions/0003-laptop-host-tiered-compute.md) |
| Region | EU, 230 V | [ADR-0005](decisions/0005-reversible-installation.md) |
| Mains-voltage devices | Certified parts with open firmware or a local interface; DIY only at low voltage | [ADR-0004](decisions/0004-mains-safety.md) |
| Home | Rented apartment, no rewiring | [ADR-0005](decisions/0005-reversible-installation.md) |
| Voice languages | English and Russian | [ADR-0006](decisions/0006-english-and-russian-voice.md) |
| Voice input | All four, in stages: phone push-to-talk, the M5 devices as prototypes, old Android phones as satellites, dedicated satellites | [ADR-0007](decisions/0007-voice-input-in-stages.md) |
| Cameras | Later: not in the first phases, but the use cases are planned now | [ADR-0008](decisions/0008-cameras-later.md) |
| Hub language | Python | [ADR-0009](decisions/0009-python-hub.md) |
| Phone app | Web app first; a native wrapper later if needed | [ADR-0010](decisions/0010-web-app-first.md) |
| First purchase round | Up to €100 | [ADR-0011](decisions/0011-phased-purchasing.md) |
| Window coverings | Curtains on a track; the rest not decided yet | [bom.md](bom.md) |
| Lights | Ceiling lights with replaceable bulbs, floor and table lamps, LED strips | [bom.md](bom.md) |
| M5 stick radio | Sub-GHz CC1101, 433/868 MHz | [ADR-0015](decisions/0015-radio-strategy.md) |

## Not in scope for now

- Cameras (planned for a later phase, [ADR-0008](decisions/0008-cameras-later.md)).
- Control from outside the home network (later, through a self-hosted VPN, [ADR-0021](decisions/0021-network-and-security.md)).
- Anything that needs rewiring or drilling ([ADR-0005](decisions/0005-reversible-installation.md)).
- Home-made mains-voltage electronics ([ADR-0004](decisions/0004-mains-safety.md)).

What is still undecided is tracked in [open-questions.md](open-questions.md).
