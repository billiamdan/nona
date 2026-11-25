# Components list

What to buy, in tiers that follow the [roadmap](roadmap.md). Nothing is bought before the phase that uses it ([ADR-0011](decisions/0011-phased-purchasing.md)).

Prices are from the research of 25 November 2026, in euros including VAT unless marked otherwise. **Checked** means read from a shop or maker page that day. **Estimate** means not verified. Black-Friday prices were live that week, so re-check everything before ordering.

## Rules that shape the list

| Rule | Record |
|---|---|
| Everything on mains is a bought, certified product | [ADR-0004](decisions/0004-mains-safety.md) |
| Everything is removable without a trace: no rewiring, no drilling | [ADR-0005](decisions/0005-reversible-installation.md) |
| No device may need a cloud or an account in normal operation | [ADR-0001](decisions/0001-local-only.md) |
| Bought devices are Zigbee where the product exists; battery devices are bought, not built | [ADR-0015](decisions/0015-radio-strategy.md) |
| One unit first; more only after it has proved itself | [ADR-0011](decisions/0011-phased-purchasing.md) |

## Tier 0: already owned (Phase 0, €0)

| Item | Role |
|---|---|
| Linux laptop, 16 GB | The 24/7 host |
| MacBook Pro, M5 Pro, 48 GB | Development; optional larger-model server |
| M5 stick with CC1101 | First ESPHome node; sound-level microphone; 433 MHz and infrared gateway |
| M5 Cardputer mesh kit | Second node; handheld panel; push-to-talk prototype |
| Android phones, iPhones, iPad | The app; later wall panels and voice satellites |
| Soldering tools, wires, USB chargers and cables | Sensor nodes |

## Tier 1: first real devices (Phases 1 to 3, within €100)

| # | Item | Qty | Price | Status | Why this one | Alternative |
|---|---|---|---|---|---|---|
| 1 | USB Zigbee coordinator, SMLIGHT SLZB-07 | 1 | €16.95 | Checked (sale; list €18.95) | Cheapest coordinator found; the shop lists it as working with Zigbee2MQTT | SONOFF ZBDongle-E, about $21–27 |
| 2 | USB extension lead, 1–2 m | 1 | about €3 | Estimate | Keeps the radio away from the laptop's USB 3 ports | Any lead already owned |
| 3 | Zigbee E27 bulbs, Innr Comfort, pack of two | 1 pack | about €30 | Checked page; pack size inferred | Zigbee, dimmable, can be bound to a button; IKEA no longer sells Zigbee bulbs | Shelly Duo E27 Gen3, two for €15.12 on offer (€18.90 list): Wi-Fi with local MQTT, needs a small Shelly adapter in the hub |
| 4 | Zigbee metering plug with its own Zigbee2MQTT definition: NOUS A1Z, Sonoff S60ZBTPF or Aqara SP-EUC01 | 1 | about €10–12 | Estimate | No new hub code; also strengthens the Zigbee mesh | Shelly Plug S MTR Gen3, €9.56 on offer (€23.90 list), checked: Wi-Fi with local MQTT, needs the same Shelly adapter |
| 5 | Stick-on Zigbee button, Sonoff SNZB-01P | 1 | about €9–12 | Estimate | Cheapest; its presses go to the hub | Philips Hue dimmer switch, about €20–25: can be bound directly to the bulbs, so it works with the hub down |
| 6 | ESP32-C6 board for the first sensor node | 1 | €9.95 | Checked: ESP32-C6 DevKitC at an EU maker shop. A Seeed XIAO ESP32-C6 is $5.20 at the maker's store | The all-round chip in the research; a dependable board for the node the rules rely on | ESP32-C3 SuperMini, €3.95 checked; clones have recurring Wi-Fi problems |
| 7 | Illuminance sensor, BH1750 | 1 | about €3 | Estimate | Native ESPHome support | VEML7700 |
| 8 | Presence radar, HLK-LD2410C | 1 | about €6 | Estimate | Detects a still person, unlike a motion sensor; native ESPHome support | LD2412 |
| | Shipping, two or three shops | | about €7–13 | One shop checked at €6.95 | | |
| | **Total** | | **about €97–103** | | Goods about €90; drop to the cheaper board or the offer-priced Shelly plug to stay under €100 | |

Notes:

- Sound level for the "waking" condition comes from the M5 stick's built-in microphone, so no microphone is bought in this tier. A separate I2S microphone (INMP441, about $5) can be added later.
- If the total runs over, the first cut is the cheaper Wi-Fi bulbs in line 3, at the cost of one more adapter in the hub.
- If there is room, the first upgrade is the bindable button in line 5.
- Until a switch guard is fitted, the existing wall switch still works as a plain on/off for a smart bulb, which is a useful fallback while the hub is young.

## Tier 2: network, more light, LED strips (around Phase 4; buy per need)

| Item | Price | Status | Notes |
|---|---|---|---|
| OpenWrt-capable router, Cudy WR3000 or TR3000 | about €40–60 | Estimate; OpenWrt support checked | Only if the apartment's router cannot be configured ([ADR-0021](decisions/0021-network-and-security.md)). Alternative: GL.iNet GL-MT3000, about €70–90 |
| More bulbs, Innr Comfort E27, pack of three | €44.99 | Checked page; pack size inferred | For the remaining ceiling fixtures |
| Battery Zigbee sensors: door or window, climate, motion | €10.95 each | Checked, generic models | Sonoff models are the lower-risk choice on a mixed mesh; their euro prices are estimates of €9–20 |
| LED strip controller | Pre-flashed WLED controllers from Athom: $7.84–14.88 plus shipping, checked | | Chosen once the strip type is known ([open-questions.md](open-questions.md)). A Zigbee LED controller needs no new hub code; a WLED controller needs a small adapter |
| Certified plug-in power supply for the strip | about €12–25 | Estimate | The only mains part of an LED-strip build; it must carry a test mark |
| Switch guard for a wall switch | a few euros | Estimate | Keeps a smart bulb powered |

## Tier 3: the curtain, and always-listening voice (Phase 7)

| Item | Price | Status | Notes |
|---|---|---|---|
| Curtain drive that rides the existing track, Aqara Curtain Driver E1, track version | about €70–100 | Estimate; EU availability not confirmed | Preferred: Zigbee, reports position, battery and light, needs no new hub code. U- or I-shaped rails only, curtains up to 12 kg. One drive per curtain panel |
| Alternative curtain drive, SwitchBot Curtain 3 | €89.99 | Checked | Bluetooth: needs an ESP32 nearby as a gateway and a new adapter. Whether first calibration works without the vendor's app is not confirmed |
| Voice satellite: a spare Android phone | €0 | Owned | The first always-listening endpoint ([ADR-0007](decisions/0007-voice-input-in-stages.md)) |
| Dedicated voice satellite, Home Assistant Voice Preview Edition | €59 | Checked | Open firmware, sold by EU retailers; buy one only after the phone has shown where far-field pickup matters |
| Cheaper satellite kit, Seeed reSpeaker Lite with XIAO ESP32-S3 | $29.90 | Checked | Needs a speaker and an enclosure |
| Zigbee plug for each wall-panel phone's charger | about €10–12 | Estimate | Lets the hub keep the phone's battery in a safe range |

## Tier 4: a dedicated AI computer (optional; decided after Phase 5)

Not planned. The owned laptop and Mac cover the design ([ADR-0003](decisions/0003-laptop-host-tiered-compute.md)). For reference, late-2026 prices after the year's memory price rises:

| Option | Price | Fits |
|---|---|---|
| Mac mini, 16 GB | $899 | A 9–12B model around the clock at about 4 W idle |
| Mac mini with M5 Pro, 48 GB | about $2,299 | The same model class as the MacBook, always on |
| N150 mini PC, 16 GB | about $349 | A 4B-class model for jobs nobody is waiting on |
| Jetson Orin Nano Super | $399 | A low-power 4B-class node |

## Do not buy

| Item | Why |
|---|---|
| IKEA's current smart range: bulbs, plug, remotes, sensors | It is now Matter over Thread, which needs a second radio stack that the first version leaves out ([ADR-0015](decisions/0015-radio-strategy.md)) |
| Cloud-bound Wi-Fi devices in the hope of reflashing them | Reflashing has become the hard path: the common over-the-air exploit has been patched since 2022, and current plugs must be opened destructively |
| Generic Zigbee plugs with no model-specific Zigbee2MQTT definition | Some firmware versions report power only when polled |
| Bluetooth-only button-pressing robots from cloud brands | They need a key from the vendor's cloud |
| In-wall modules and replacement wall switches | Rented apartment ([ADR-0005](decisions/0005-reversible-installation.md)) |
| Mains power supplies without a test mark | [ADR-0004](decisions/0004-mains-safety.md) |
| A dedicated AI computer before the Phase 0 benchmark | [ADR-0003](decisions/0003-laptop-host-tiered-compute.md) |

## Before each order

1. Confirm the country's plug type: Type F is assumed; France, Belgium, Poland, Czechia and Slovakia use Type E.
2. Look up the exact model of every Zigbee device in Zigbee2MQTT's supported-devices list.
3. Confirm pack sizes for bulbs.
4. For the curtain drive: measure the rail profile and the curtain's weight, and count the panels.
5. For LED strips: note the voltage and whether the strip is addressable.
6. Re-check prices, and record any change here with its date.

Evidence: [EU devices notes](../research_notes/Local%20smart%20home%20stack%20options/actuators_and_bom_eu.md); [radio, controller and sensor notes](../research_notes/Local%20smart%20home%20stack%20options/radio_mcu_sensors.md), sections 3, 5 and 7; [speech and audio notes](../research_notes/Local%20smart%20home%20stack%20options/speech_and_audio.md), satellite hardware; [local LLM notes](../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), section 6.
