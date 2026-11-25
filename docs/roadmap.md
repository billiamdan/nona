# Roadmap

The step-by-step implementation plan. The architecture it builds is in [architecture.md](architecture.md); what to buy for each phase is in [bom.md](bom.md).

## Rules of the road

1. Each phase is one thin slice through every layer it needs, and it ends with a demo that either works or does not.
2. Nothing is built or bought for a later phase.
3. A risky assumption is tested with a small spike before anything is built on it.
4. A phase is done when its exit demo passes with the internet cable unplugged, new decisions are recorded in [decisions/](decisions/README.md), and [open-questions.md](open-questions.md) is up to date.

Phase 0 scaffolds every layer thinly. Each later phase adds one feature from the brief.

## Overview

| Phase | Result | Brief's features | Buys | Size |
|---|---|---|---|---|
| 0 | A walking skeleton on owned hardware | 4, 6, 7 | Nothing | Medium |
| 1 | Real lights and a socket, from the app | 4, 5, 7 | Tier 1, within €100 | Medium |
| 2 | Schedules | 2 | Nothing | Small |
| 3 | Complex conditions | 3 | Nothing | Medium |
| 4 | Voice for known commands | 1 | Tier 2 | Large |
| 5 | A model handles free-form requests | 1, 6 | Nothing | Medium |
| 6 | A model drafts automations | 3, 6 | Nothing | Small |
| 7 | The curtain, and always-listening voice | 1, 5 | Tier 3 | Large |
| 8 | Hardening and later modules | | As needed | Ongoing |

Feature numbers refer to [project-brief.md](project-brief.md).

## Phase 0: Walking skeleton

**Goal.** One message path through every layer, on hardware already owned.

**Build**

- The repository scaffold ([ADR-0022](decisions/0022-repo-layout-and-deployment.md)) and the first contracts: the capability model for `switch`, `sensor` and `binary_sensor`.
- `deploy/`: the broker with credentials and the HTTPS reverse proxy with our private certificate authority, as containers.
- `tools/simulator`: fake devices that announce themselves, report state and accept commands.
- Hub, first version: bus client, the `ha_discovery` adapter, registry and state, command router, HTTP and WebSocket API.
- App, first version: entity cards generated from the registry, with live updates and a toggle.
- Firmware: ESPHome on the M5 stick (its button as a binary sensor, its LED or buzzer as a switch) and on the Cardputer.
- The laptop prepared as a server: lid and sleep settings, battery charge cap, services that start at boot ([ADR-0003](decisions/0003-laptop-host-tiered-compute.md)).

**Spikes**

| Spike | Question | Decides |
|---|---|---|
| S1 Model benchmark | How fast are the candidate models on the laptop, on its CPU and on its integrated GPU, and on the Mac, with the real tool prompt, cached and uncached? How do they score on 30–50 commands in English, Russian and mixed language, including cases where the right answer is to do nothing? | The models for both tiers, and whether the laptop's model is usable for live voice ([ADR-0018](decisions/0018-llm-runtime-and-tiers.md)) |
| S2 M5 radios | Under ESPHome, can the stick's CC1101 receive and replay a 433 MHz remote, and can either device send infrared? | Whether legacy radio gets a real job ([ADR-0015](decisions/0015-radio-strategy.md)) |
| S3 Certificates | Do one iPhone and one Android phone trust our certificate authority, open the app and grant the microphone? | The HTTPS route ([ADR-0021](decisions/0021-network-and-security.md)) |

**Exit demo.** From a phone, toggle the M5 stick's LED in the web app and watch its button press arrive live. A second, simulated device of a different type appears in the app with no code change.

## Phase 1: First real devices

**Goal.** Control real lights and a socket.

**Buys.** Tier 1.

**Build**

- Zigbee2MQTT with the coordinator, the `zigbee2mqtt` adapter, the `light` capability and its card.
- Rooms, names and aliases in both languages, and risk classes, in the registry.
- Availability: a bulb switched off at the wall is shown as such.
- A stick-on button that toggles a bulb through the hub, and that is also bound directly to the bulb as a fallback.
- A first sensor node of our own: an ESP32 board with illuminance and presence sensors.
- A metering smart plug on a lamp.

**Exit demo.** Dim a ceiling bulb and switch a lamp on a plug from the phone. The wall button toggles the bulb. Illuminance and presence update live. All of it works with the internet cable unplugged.

## Phase 2: Schedules

**Goal.** Actions run at set times.

**Build**

- The rule store and engine with time and sun triggers and device actions.
- Time zone and daylight-saving handling, and a defined behaviour for a run that was missed during a restart.
- A schedule screen and one-off timers.
- Dry-run mode and the trace.

**Exit demo.** A lamp turns on 20 minutes before sunset and off at 23:30. "Off in 20 minutes" works. Both survive a hub restart.

## Phase 3: Complex conditions

**Goal.** The brief's example works: morning, low light, and people waking up.

**Build**

- State triggers and the condition tree, with hold times and hysteresis.
- A rule editor in the app.
- The first perception module: `waking_activity` from sound level and radar presence ([ADR-0017](decisions/0017-declarative-rules-virtual-sensors.md)).
- The trace view: why a rule did or did not fire.

**Exit demo.** The brief's example runs in dry-run mode in the bedroom for a week. Its trace is reviewed, the thresholds are adjusted, and it goes live with a lamp.

## Phase 4: Voice for known commands

**Goal.** A spoken command in either language controls a device, with no model involved.

**Buys.** Tier 2.

**Build**

- Every phone and tablet trusts the certificate authority.
- Push-to-talk in the app, with audio over the WebSocket.
- Speech-to-text and text-to-speech servers on the laptop ([ADR-0020](decisions/0020-voice-pipeline-and-speech-stack.md)).
- The intent matcher in English and Russian, over the registry's names and aliases.
- A cue and a spoken reply, and a measurement of latency stage by stage.
- Our own router with the IoT zone ([ADR-0021](decisions/0021-network-and-security.md)).

**Exit demo.** With the Mac switched off, holding the button and saying "turn on the bedroom lamp" or «включи лампу в спальне» works within about two seconds of the end of speech.

## Phase 5: A model handles free-form requests

**Goal.** Requests that no pattern matches are handled by a model, safely.

**Build**

- The model server on the laptop as a service, and the ordered endpoint list with the Mac first.
- The `operator` agent with its tool loop ([ADR-0019](decisions/0019-agent-design.md)).
- The safety policy: risk classes, confirmation, the audit log and the "AI off" switch ([ADR-0023](decisions/0023-ai-action-safety.md)).
- The regression set as a command in `tools/`, run after every model or runtime change.

**Exit demo.** A request such as "it is too bright in here" or «сделай поуютнее» leads to a sensible, logged action. With the Mac at home it takes a few seconds. With the Mac away it still completes. Deterministic commands are unaffected either way.

## Phase 6: A model drafts automations

**Goal.** An automation described in words becomes a rule the owner approves.

**Build**

- The `rule-author` agent.
- Validation of the draft against the rule schema and the registry.
- An approval screen that renders the rule in plain words, with dry run before live.

**Exit demo.** "On weekdays at seven, if it is still dark, turn on the hall lamp at 30%" becomes a correct draft that the owner approves in the app.

## Phase 7: The curtain, and always-listening voice

**Goal.** The morning rule moves the curtain, and voice works without touching a phone.

**Buys.** Tier 3.

**Build**

- The curtain drive on the existing track, the `cover` capability and its card.
- Spike S4: the hub as server for an ESPHome voice endpoint, first on the Cardputer ([ADR-0007](decisions/0007-voice-input-in-stages.md)).
- A spare Android phone as a satellite and wall panel, with a stock wake word.
- One dedicated satellite, if far-field pickup turns out to need it.
- A small Android shell for wall panels that reports battery level, so the hub can cycle the charger's plug.

**Exit demo.** The morning rule opens the curtain. A request spoken from across the room is answered through the satellite.

## Phase 8: Hardening and later modules

Picked by need, each with its own decision record where it changes the design:

- Backup, and a restore drill on a second machine.
- The update routine: pinned versions, the regression set after each update.
- Notifications ([ADR-0010](decisions/0010-web-app-first.md)).
- Control from outside the home ([ADR-0021](decisions/0021-network-and-security.md)).
- An audio classifier for waking activity, tested on the home's own sounds.
- Custom wake words.
- Cameras ([ADR-0008](decisions/0008-cameras-later.md)).
- A local MCP server over the hub's tools.
- Energy views from the metering plugs.
- The decision on a dedicated AI computer ([ADR-0003](decisions/0003-laptop-host-tiered-compute.md)).

## Changing the order

Phases 0 to 3 build on each other. After that, the curtain part of Phase 7 can move ahead of voice if the owner prefers; it depends only on Phase 1 and on its purchase.
