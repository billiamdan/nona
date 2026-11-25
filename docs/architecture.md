# Architecture

How the system is put together. Every choice on this page is backed by a decision record in [decisions/](decisions/README.md); this page shows how they fit.

## The stack at a glance

```
 PEOPLE     web app on phones, tablets, wall panels          voice: push-to-talk, satellites
                        │ HTTPS + WebSocket                              │ audio, point to point
 ───────────────────────┼────────────────────────────────────────────────┼──────────────────────────
 HUB                    ▼                                                ▼
 Python,          ┌───────────┐       ┌────────────────┐        ┌────────────────┐
 on the laptop,   │    API    │       │ intent matcher │◄───────│ voice pipeline │◄──► speech servers
 always on        └─────┬─────┘       └───┬────────┬───┘        └────────────────┘     (STT, TTS)
                        │         resolved│        │unresolved
                        │                 │        ▼
                        │                 │   ┌──────────┐   tool call or rule draft
                        │                 │   │  agents  │◄─────────────────────────► model endpoints
                        │                 │   └────┬─────┘                            (Mac, else laptop)
                        ▼                 ▼        ▼
                  ┌───────────────────────────────────────────────┐
                  │ command router · safety policy · audit log    │
                  ├───────────────────────────────────────────────┤
                  │ registry · state · scheduler · rule engine    │
                  └───────────────────────┬───────────────────────┘
                                          │ adapters
 ─────────────────────────────────────────┼─────────────────────────────────────────────────────────
 BUS                               Mosquitto (MQTT)
 ───────────────┬─────────────────────────┬──────────────────────────────┬──────────────────────────
 EDGE           │                         │                              │
        ESPHome nodes (Wi-Fi)     Zigbee2MQTT + coordinator      perception modules
        sensors, LED strips,      bulbs, plugs, buttons,         virtual sensors from sound,
        433 MHz, infrared         sensors, curtain drive         presence, later vision
```

Three ideas carry the design:

1. **One bus, self-describing devices.** Everything that senses or acts is a client of one MQTT broker and announces what it is. The hub builds its registry from those announcements, and the app, the rules, voice and the model's tools are all generated from the registry.
2. **A deterministic core.** Manual control, schedules and rules need no model. A model only proposes: a tool call that the hub validates, or a rule draft that the owner approves.
3. **Tiers that degrade.** The laptop alone runs the whole home. The Mac adds model quality while it is there. Nothing depends on the internet.

## Layers and choices

| Layer | Part | Choice | Built or reused | Record |
|---|---|---|---|---|
| People | Control app | Installable web app served by the hub | Built | [0010](decisions/0010-web-app-first.md) |
| | Wall panels | The same web app on spare phones and the iPad; later a small Android shell | Built | [0010](decisions/0010-web-app-first.md) |
| | Voice endpoints | Phone push-to-talk, then the M5 devices, Android satellites, dedicated satellites | Built and reused | [0007](decisions/0007-voice-input-in-stages.md) |
| Intelligence | Intent matcher | Deterministic patterns in English and Russian over registry names | Built | [0016](decisions/0016-deterministic-core.md) |
| | Agents | `operator`, `rule-author`, later `narrator` and `perception`; a hand-written loop | Built | [0019](decisions/0019-agent-design.md) |
| | Model servers | `llama-server` on the laptop; a larger model on the Mac; one OpenAI-compatible interface | Reused | [0018](decisions/0018-llm-runtime-and-tiers.md) |
| | Speech | Separate servers for speech-to-text and text-to-speech; wake word on the endpoint | Reused | [0020](decisions/0020-voice-pipeline-and-speech-stack.md) |
| | Perception | Modules that publish virtual sensors: sound level with presence first, an audio classifier and vision later | Built on reused models | [0017](decisions/0017-declarative-rules-virtual-sensors.md), [0008](decisions/0008-cameras-later.md) |
| Hub core | Registry, state, commands | Python, SQLite | Built | [0009](decisions/0009-python-hub.md), [0013](decisions/0013-self-describing-devices.md) |
| | Scheduler and rule engine | Declarative rules with a published schema | Built | [0017](decisions/0017-declarative-rules-virtual-sensors.md) |
| | Safety and audit | Risk classes, confirmation, append-only log, one "AI off" switch | Built | [0023](decisions/0023-ai-action-safety.md) |
| Bus | Broker | Mosquitto | Reused | [0012](decisions/0012-mqtt-control-bus.md) |
| | Zigbee bridge | Zigbee2MQTT with a USB coordinator | Reused | [0015](decisions/0015-radio-strategy.md) |
| Edge | Our nodes | ESP32 boards running ESPHome over MQTT | Reused firmware, our configuration | [0014](decisions/0014-esphome-first-firmware.md) |
| | Bought devices | Zigbee bulbs, plugs, buttons, sensors and curtain drive | Bought | [0004](decisions/0004-mains-safety.md), [0005](decisions/0005-reversible-installation.md) |
| | Legacy radio | 433 MHz and infrared through the M5 devices | Our configuration | [0015](decisions/0015-radio-strategy.md) |
| Foundation | Host | The Linux laptop around the clock; the Mac as an optional model server | Owned | [0003](decisions/0003-laptop-host-tiered-compute.md) |
| | Network | Our own router with an IoT zone that has no internet; a private certificate authority for HTTPS | Bought and configured | [0021](decisions/0021-network-and-security.md) |
| | Repository and deployment | One repository; containers for third-party services, native services for ours | | [0022](decisions/0022-repo-layout-and-deployment.md) |

## What runs where

| Machine | Runs | When |
|---|---|---|
| Linux laptop | In containers: broker, Zigbee2MQTT, HTTPS reverse proxy. As system services: the hub, a small model server, the speech servers. | Always |
| MacBook Pro | A larger model behind the same interface | While it is at home |
| ESP32 nodes | ESPHome | Always |
| Zigbee devices | Their own firmware, joined to our coordinator | Always |
| Phones and tablets | The web app in a browser; later an Android shell on wall panels | On demand |

## How a device gets in

This is the modularity the brief asks for.

1. The device announces itself on the bus. An ESPHome node publishes a description of each of its entities; a Zigbee device appears in Zigbee2MQTT's device document.
2. The matching adapter turns the announcement into entities, each with one capability type: `switch`, `light`, `cover`, `sensor`, `binary_sensor`, `button`, `number` or `select`.
3. The registry stores them. The owner assigns a room, names and aliases in both languages, and a risk class.
4. From then on, with no new code, the entity has a card in the app, can be used in rules as a trigger, a condition or an action, can be addressed by voice, and can be selected by a model.

A new device of a known family needs no code. A new family needs one adapter: a small plugin that describes entities, receives state and sends commands.

## How things flow

**A tap in the app**

1. The app posts a command for an entity, for example "turn on, brightness 40" for the bedroom lamp.
2. The command router checks the caller's token and checks the command against the entity's capability, then hands it to the entity's adapter.
3. The adapter publishes the device's native MQTT message.
4. The device reports its new state on the bus. The adapter updates the state store, and the hub pushes the change to every open app.

The app shows the new state only once the device confirms it, and "no response" after a timeout.

**A schedule or a rule**

1. A trigger arrives: the clock, a sun event or a state change.
2. The rule engine evaluates the rule's conditions against the state store and writes the evaluation to the trace.
3. If they hold, the actions go through the same command router as a tap.

No model is involved.

**A spoken command the matcher resolves**

1. A button or a wake word starts it. The endpoint streams audio to the hub.
2. End of speech is detected. The audio goes to the speech-to-text model for the request's language.
3. The intent matcher resolves the text against its patterns and the registry's names.
4. The command goes through the command router, as above.
5. A short cue or a spoken reply returns to the endpoint.

Target on the laptop alone: one to two seconds after the speaker stops.

**A spoken request that needs a model**

Steps 1 and 2 are the same. Then:

3. The matcher finds no match. The endpoint gets an immediate "thinking" cue.
4. The router picks the `operator` agent. Its prompt has a fixed prefix (instructions, tool list, exposed entities in a stable order) followed by the request, so the model server can reuse its cache.
5. The first healthy model endpoint returns a tool call whose tool name and entity are constrained to enumerated values.
6. The hub validates the call, applies the safety policy (execute, ask back, or refuse), logs it, and runs it through the command router.
7. The reply is spoken.

**"Make me an automation"**

1. A request in text or voice goes to the `rule-author` agent.
2. The agent outputs a rule draft. The hub validates it against the rule schema and the registry.
3. The app shows the draft in plain words with the entities involved. The owner approves, edits or discards it.
4. An approved rule can run in dry-run mode first, then goes live.

## Contracts

**Bus topics**

| Source | Topic | Meaning |
|---|---|---|
| ESPHome node | `homeassistant/<type>/<node>/<name>/config` (retained) | Self-description of one entity |
| | `<node>/<type>/<name>/state` and `.../command` | State and command |
| | `<node>/status` | Online or offline, set by last will |
| Zigbee2MQTT | `zigbee2mqtt/bridge/devices` (retained) | Description of every Zigbee device |
| | `zigbee2mqtt/<device>` and `zigbee2mqtt/<device>/set` | State and command, as JSON |
| | `zigbee2mqtt/<device>/availability` | Online or offline |
| Hub | `nona/status` | Online or offline, set by last will |
| Perception modules | A `homeassistant/.../config` description, with state under `nona/perception/<module>/<name>/state` | Virtual sensors, described like devices |

**Hub API**

| Endpoint | Purpose |
|---|---|
| `GET /api/entities` | The registry with current state |
| `POST /api/entities/{id}/command` | Send a command |
| `GET`, `POST`, `PUT`, `DELETE /api/rules` | Schedules and rules; a schedule is a rule with a time trigger |
| `GET /api/rules/{id}/trace` | Why a rule did or did not fire |
| `POST /api/ask` | A text request: matcher first, then an agent |
| `WS /api/ws` | Live state and events |
| `WS /api/voice` | Push-to-talk audio in, reply audio out |
| `POST /api/pair` | Exchange a one-time code for a device token |
| `GET /api/audit` | The audit log |

**Rule format**, shown with the brief's example:

```json
{
  "id": "gentle-morning",
  "name": "Gentle morning",
  "mode": "active",
  "triggers": [
    { "type": "state", "entity": "binary_sensor.bedroom_waking_activity", "to": true, "for": "2m" }
  ],
  "conditions": {
    "all": [
      { "type": "time_between", "after": "06:30", "before": "10:00" },
      { "type": "compare", "entity": "sensor.bedroom_illuminance", "op": "<", "value": 40 },
      { "type": "state", "entity": "cover.bedroom_curtain", "is": "closed" }
    ]
  },
  "actions": [
    { "type": "command", "entity": "cover.bedroom_curtain", "command": "set_position", "args": { "position": 30 } },
    { "type": "command", "entity": "light.bedroom_lamp", "command": "turn_on", "args": { "brightness": 20, "color_temp_kelvin": 2700 } }
  ]
}
```

`mode` is `active`, `dry_run` or `off`.

**Tools a model may call** (the `operator` agent)

| Tool | Arguments | Notes |
|---|---|---|
| `control` | `entity_id` from an enumerated list; `command` from the entity's capability; bounded `args` | Subject to the safety policy |
| `get_state` | `entity_ids` from an enumerated list | Read-only |
| `run_scene` | `scene_id` from an enumerated list | |
| `reply` | Text | When no action is the right answer |

## Inside the hub

One Python process, organised as modules with narrow interfaces.

| Module | Responsibility | Extension point |
|---|---|---|
| `bus` | The MQTT connection and subscriptions | |
| `adapters` | Announcements into entities; commands into native messages | One `Adapter` plugin per device family |
| `registry` | Devices, entities, rooms, names, risk classes | |
| `state` | Current state, availability, history | |
| `commands` | Validation, safety policy, confirmation, dispatch | |
| `rules` | Scheduler, triggers, condition tree, actions, trace, dry run | New condition and action types |
| `intents` | The deterministic matcher | Pattern files per language |
| `agents` | The router, the tool loop, the agents | One module per agent |
| `models` | The ordered list of endpoints and their health checks | |
| `voice` | Satellite server, push-to-talk channel, end-of-speech detection | Speech-to-text and text-to-speech providers; endpoint transports |
| `perception` | Virtual-sensor modules | One module per derived signal |
| `api` | HTTP, WebSocket, pairing, tokens | |
| `audit` | The append-only log | |

## When something fails

| What fails | What still works | What is lost |
|---|---|---|
| The internet | Everything | Nothing |
| The Mac is away | Everything; free-form requests use the small model | Quality and speed of free-form requests |
| The model server | Manual control, schedules, rules, deterministic voice commands | Free-form requests, rule drafting |
| The speech servers | The app, schedules, rules | Voice |
| The hub or the broker | Physical buttons on plugs; bulbs through their wall switch; a Zigbee button bound directly to its bulb | The app, schedules, rules, voice |
| Wi-Fi | Zigbee devices and the rules that use them, if the laptop is on cable | Wi-Fi nodes and the app on phones |
| A bulb switched off at the wall | Everything else | That bulb, shown as "off at the wall" |

## Data and privacy

| Data | Where | Kept |
|---|---|---|
| Device state history | The hub's database | 30 days by default |
| Rule traces | The hub's database | 30 days by default |
| Audit log of model-initiated actions, with the request text | The hub's database, append-only | One year by default |
| Raw audio | Nowhere, unless the debug switch is on | |
| Images | Not applicable until cameras arrive | |

Nothing leaves the local network. IoT devices have no route to the internet.

## Deliberately not in the first version

- Matter and Thread
- Cameras
- Control from outside the home
- A native iPhone app
- Speaker identification
- Per-person permissions: one household, every paired device has the same rights
