# Radio protocols, ESP32-family controllers, sensors and Zigbee coordinators for a local DIY smart home (state: 25 November 2026)

Conventions used in these notes
- Prices are USD unless another currency is shown. "Retrieved" means the page was read on 2026-11-25 during this research. Several retail pages were showing Black Friday discounts (20-30 Nov 2026); where a "was" price was visible it is given as the regular price.
- **[PRE-2025]** marks a fact whose only source is older than 2025 or an undated listing, i.e. not confirmed with a 2025-2026 source.
- Many prices come from retailer or aggregator pages, not vendor list prices. shop.m5stack.com and sonoff.tech returned HTTP 429 to direct fetches, so their prices come from search-result snippets or third parties.

---

## 1. Radio options for a local DIY smart home: Wi-Fi + MQTT, ESP-NOW, BLE (BTHome v2 / GATT), Bluetooth Mesh, Zigbee 3.0/4.0, Thread/Matter, LoRa, 433 MHz sub-GHz, Z-Wave

### Takeaway
No single radio fits every role. The evidence supports a three-layer mix: Zigbee 3.0 through Zigbee2MQTT for bought battery sensors, sockets, switches and curtain motors (5,868 supported devices, multi-year coin-cell life); Wi-Fi (ESPHome native API or MQTT) for mains-powered DIY nodes, voice satellites and gateways; and BLE advertising (BTHome v2) or ESP-NOW for the cheapest battery sensors. Thread/Matter, LoRa/Meshtastic, 433 MHz and Z-Wave are all usable locally but are niche add-ons here, and Bluetooth Mesh has no evident open-hub path.

### Cited Findings

#### Wi-Fi + MQTT (or ESPHome native API)
- A measured full cycle of wake, Wi-Fi connect and MQTT publish on an ESP32 took 3.877 s; the author estimated 6.9 months on a 2500 mAh cell — [ThingPulse](https://thingpulse.com/esp32-ultra-long-battery-life-with-espnow/)
- Protocol comparison articles agree that Wi-Fi radios draw far more power than Zigbee, Z-Wave or Thread, which is why battery sensors rarely use Wi-Fi — [Homey protocol comparison](https://homey.app/en-ae/wiki/smart-home-protocols/); [Silicon Labs comparison](https://www.silabs.com/blog/smart-home-technology-comparison)
- ESPHome has an MQTT client component as an alternative to its native API — [ESPHome MQTT component](https://new.esphome.io/components/mqtt)
- ESPHome made native ESP-IDF the default ESP32 toolchain in 2026.7.0 — [ESPHome 2026.7.0](https://esphome.io/changelog/2026.7.0/)
- Router-side client limits and network practices: see Question 2.

#### ESP-NOW
- ESPHome's `espnow` component runs on ESP32 variants with a Wi-Fi radio and on ESP8266; default payload is 250 bytes, ESP-NOW v2 frames carry up to 1470 bytes between v2-capable peers; when Wi-Fi is active ESP-NOW must use the Wi-Fi network's channel; broadcasts are unacknowledged while unicast gives `on_sent`/`on_error`; the component has no link-layer encryption (encryption comes from the Packet Transport layer); ESP32-H2 is not supported and ESP32-P4 needs a hosted co-processor — [ESPHome ESP-NOW](https://esphome.io/components/espnow.html)
- ESP8266 support for ESP-NOW was added in ESPHome 2026.11.0 — [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- ESPHome 2026.7.0 added `max_payload_size` for ESP-NOW v2 frames; 2026.8.0 stopped the component polling the Wi-Fi driver every loop — [ESPHome 2026.7.0](https://esphome.io/changelog/2026.7.0/); [ESPHome 2026.8.0](https://esphome.io/changelog/2026.8.0/)
- ESPHome Packet Transport lets nodes share sensor data directly; supported transports are ESP-NOW, SX126x, SX127x, UART and UDP — [ESPHome Packet Transport](https://esphome.io/components/packet_transport/)
- A full wake, voltage read and ESP-NOW send cycle took about 200 ms versus 5-10 s over Wi-Fi + MQTT; the author estimated more than 5 years at one message per 15 minutes, ignoring sleep current and self-discharge — [ThingPulse ePulse Feather](https://thingpulse.com/epulse-feather-espnow-a-dream-team/)

#### Bluetooth LE advertising (BTHome v2) and GATT
- BTHome is an open standard for broadcasting sensor data and button presses in BLE advertisements; v2 is current (v1 deprecated November 2022); it supports encryption; the site claims "over a year on a single battery"; Home Assistant decodes it out of the box; it is part of the Open Home Foundation and Allterco (Shelly) sponsored the UUID under a free licence — [BTHome](https://bthome.io/)
- Products and firmware using BTHome named by the project: Shelly BLU Button1, Shelly BLU Door/Window, ATC MiThermometer custom firmware, the open-source b-parasite; DIY libraries exist for ESP32, nRF52 (CircuitPython), Espruino and MicroPython — [BTHome](https://bthome.io/)
- ESPHome Bluetooth proxy: passive advertisements (e.g. BTHome) are forwarded without using connection slots; active GATT connections default to 3 slots on ESP32, maximum 9, with 5 or fewer recommended; Ethernet-connected proxies are recommended because Wi-Fi shares the radio; esp-idf is recommended over Arduino; ESPHome does not decode device data itself (Home Assistant integrations do) — [ESPHome Bluetooth Proxy](https://esphome.io/components/bluetooth_proxy.html)
- OpenMQTTGateway (docs header shows v1.9.1) bridges BLE (via Theengs Decoder, "more than 100 BLE sensors out of the box"), RF 433/315/868/915 MHz, infrared and LoRa to MQTT on ESP32/ESP8266; Home Assistant MQTT auto-discovery is on by default; it advertises "Local first, cloud optional"; Theengs Bridge is a pre-flashed BLE gateway with Ethernet — [OpenMQTTGateway docs](https://docs.openmqttgateway.com/)
- The pvvx firmware for Xiaomi thermometers supports Xiaomi, ATC, Custom and BTHome v2 advert formats, claims more than a year on a CR2032, and one project page puts the default settings (2.5 s advertising, 10 s measurement) at roughly 14-20 µA or "over 6 months" — [pvvx ATC_MiThermometer](https://github.com/pvvx/ATC_MiThermometer)
- Shelly rates the BLU H&T at up to 3 years and the BLU Door/Window at up to 5 years on a CR2032 (vendor claims) — [Shelly BLU H&T](https://us.shelly.com/products/shelly-blu-h-t-us-list); [Shelly BLU Door/Window](https://us.shelly.com/products/shelly-blu-door-window-us-list)

#### Bluetooth Mesh
- ESP-BLE-MESH on ESP32 joins devices in two phases (provisioning, then configuration) through a Provisioner; Espressif's examples use the nRF Mesh mobile app; the theoretical node limit is 32,767 but is memory-bound in practice — [ESP-BLE-MESH FAQ](https://docs.espressif.com/projects/esp-idf/en/v5.3/esp32/api-guides/esp-ble-mesh/ble-mesh-faq.html)
- A mesh-protocol comparison says latency rises with network size for all three mesh protocols and most for Bluetooth mesh **[PRE-2025]** — [TechTarget](https://internetofthingsagenda.techtarget.com/feature/Comparing-IoT-mesh-network-protocols-Whats-the-difference)

#### Zigbee 3.0 and Zigbee 4.0
- Zigbee2MQTT states "Currently 5868 devices are supported from 616 different vendors" (retrieved 2026-11-25) — [Zigbee2MQTT supported devices](https://www.zigbee2mqtt.io/supported-devices/)
- Zigbee2MQTT needs an adapter flashed with coordinator firmware; recommended driver families are zStack (TI), EmberZNet (Silicon Labs) and deCONZ; ZiGate is unmaintained and ZBOSS is experimental; only zStack and EmberZNet adapters support coordinator backup — [Zigbee2MQTT adapters](https://www.zigbee2mqtt.io/guide/adapters/)
- Zigbee 4.0 and "Suzi" (the sub-GHz brand) were released by the CSA on 18 November 2025; the set comprises Base Device Behavior v3.1, Zigbee Core R23.2, Device Type Library v1.0 and Zigbee Direct v1.1 — [CSA announcement](https://csa-iot.org/newsroom/the-connectivity-standards-alliance-announces-zigbee-4-0-and-suzi-empowering-the-next-generation-of-secure-interoperable-iot-devices/); [CNX Software](https://www.cnx-software.com/2025/11/19/zigbee-4-0-introduces-suzi-sub-ghz-connectivity-and-security-enhancements/)
- Silicon Labs' Zigbee SDK 9.0 (22 January 2026) is described as Zigbee 4.0 certification-ready with BDB 3.1 and R23 features — [Silicon Labs release notes](https://docs.silabs.com/zigbee/9.0.0/sisdk-zigbee-release-notes/sisdk-zigbee-sdk-release-notes)
- A CSA document says subsequent Zigbee 4.x updates are expected to progressively complete the specification, testing and certification changes needed for full harmonization — [CSA Zigbee 4.0 Step 5 (PDF)](https://csa-iot.org/wp-content/uploads/2026/11/Zigbee-4.0-Step-5.pdf)
- ESPHome has a native Zigbee component for ESP32-C5/C6/H2 (H4 and H21 listed as future) and nRF52: end device, router (`router: true`) or sleepy end device (`sleepy: true`, polls about every 3-5 s); exposed entity types are binary_sensor, sensor, switch and number; Zigbee2MQTT 2.8.0 or later is advised; the ESP32-H2 is reported to work more reliably than the ESP32-C6 (some C6 boards only work next to the coordinator); Wi-Fi station + end device works, Wi-Fi + router is unstable, Wi-Fi AP + Zigbee is unsupported; after a config change the device must be removed, re-added and re-interviewed; commercial Zigbee hubs are unlikely to support such devices — [ESPHome Zigbee](https://esphome.io/components/zigbee.html)
- ESPHome Zigbee timeline in 2026: 2026.7.0 moved to SDK 2.0.2 with endpoint merging (breaking: devices must be re-joined), 2026.9.0 added a Zigbee time component, 2026.10.0 added switch and number, 2026.11.0 added sleepy-end-device support and a report action on ESP32 — [ESPHome 2026.7.0](https://esphome.io/changelog/2026.7.0/); [ESPHome 2026.9.0](https://esphome.io/changelog/2026.9.0/); [ESPHome 2026.10.0](https://esphome.io/changelog/2026.10.0.html); [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- SONOFF rates its SNZB-02P temperature/humidity sensor at about four years on a CR2477 (vendor lab data) — [SONOFF SNZB-02P](https://sonoff.tech/en-us/products/sonoff-zigbee-temperature-and-humidity-sensor-snzb-02p)
- An older comparison table lists operating range per hop as about 35 ft for Zigbee, 100 ft for Z-Wave, 100 ft (theoretical) for Thread and 330 ft for Bluetooth mesh **[PRE-2025]** — [Tom's Guide](https://www.tomsguide.com/us/smart-home-wireless-network-primer,news-21085.html)

#### Thread and Matter
- Matter 1.5 (November 2025) added cameras, closures, soil sensors and energy management; 1.5.1 followed in March 2026; Matter 1.6 was announced 17 June 2026 with NFC commissioning, Joint Fabric and Thermostat Suggestions — [CSA Matter 1.5](https://csa-iot.org/newsroom/matter-1-5-introduces-cameras-closures-and-enhanced-energy-management-capabilities/); [Matter version history](https://matteralpha.com/explainer/matter-version-history-every-update-feature-explained); [CSA Matter 1.6](https://csa-iot.org/newsroom/matter-1-6-enables-more-intuitive-setup-multi-ecosystem-experiences-and-context-driven-control/)
- Matter 1.4 made it easier to add Thread devices and border routers to an existing Thread network (credential sharing, one mesh instead of parallel ones) — [CSA Matter 1.4](https://csa-iot.org/newsroom/matter-1-4-enables-more-capable-smart-homes/)
- tvOS 27 added Thread 1.4 support, while Apple Home still does not fully support Matter 1.5 — [MacRumors](https://www.macrumors.com/2026/06/17/matter-1-6-specification/)
- ESPHome gained initial OpenThread support and official ESP32-C6/H2/P4 variants in 2025.6.0; 2026.11.0 added ESP32 OpenThread Border Router support — [CNX Software on ESPHome 2025.6.0](https://www.cnx-software.com/2025/06/24/esphome-2025-6-0-adds-esp32-p4-and-openthread-support/); [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- Zigbee2MQTT states "Multiprotocol firmware is not supported" and recommends one adapter per protocol — [Zigbee2MQTT EmberZNet adapters](https://www.zigbee2mqtt.io/guide/adapters/emberznet.html)
- A DIY ESP32-H2 Thread deep-sleep sensor (ESPHome, BME280, 18650 measured at 3300 mAh) drained in about 2 months; normal wake-to-first-reading took 6-8 s and failed cycles 87-88 s; the default ESPHome deep-sleep component was reported not to work properly on ESP32-H2 (a fork was needed) — [Home Assistant community thread](https://community.home-assistant.io/t/esp32-h2-thread-with-deep-sleep-for-cheap-esp32-battery-sensors/919132)
- IKEA's 2025-2026 Matter-over-Thread range: an AppleInsider review (15 May 2026) says lingering connection issues with bulbs, controls and sensors are the main problem — [AppleInsider](https://appleinsider.com/articles/26/05/15/ikea-matter-over-thread-review-amazing-smart-home-tech-when-they-work)

#### LoRa (point-to-point, Meshtastic, LoRaWAN)
- A typical ESP32 LoRa add-on (M5Stack Cap LoRa-1262, SX1262) covers 868-923 MHz with +22 dBm transmit power and -147 dBm receive sensitivity in low-data-rate mode — [M5Stack Cardputer Mesh Kit docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit)
- Meshtastic region limits: EU_868 uses 869.4-869.65 MHz with a 10 % duty cycle (rolling hour) and a 27 dBm firmware cap; EU_433 is 433-434 MHz, 10 %, 10 dBm; US is 902-928 MHz, 100 %, 30 dBm; default hop limit is 3 (maximum 7); slower presets imply message delays of several seconds; nodes must share a modem preset — [Meshtastic LoRa configuration](https://meshtastic.org/docs/configuration/radio/lora/)
- Meshtastic MQTT: a node with Wi-Fi/Ethernet acts as gateway with per-channel uplink/downlink; JSON topics are `msh/REGION/2/json/CHANNELNAME/USERID`; JSON is not supported on nRF52; published JSON types include text, telemetry, position, detection-sensor and remote-hardware messages; downlink from MQTT supports only `sendtext` and `sendposition` via a channel named "mqtt"; `mqtt.address` can point at a private broker — [Meshtastic MQTT docs](https://meshtastic.org/docs/software/integrations/mqtt/)
- For Home Assistant, Meshtastic recommends a private broker with JSON output; a separate HACS integration talks to the node directly but a 2026 blog notes its last release was April 2025 and that it carries deprecation warnings for Home Assistant 2027.7/2027.8 — [Meshtastic Home Assistant docs](https://meshtastic.org/docs/software/integrations/mqtt/home-assistant/); [raspberry.tips](https://raspberry.tips/en/smart-home/meshtastic-in-home-assistant-mqtt-hacs)
- ESPHome's SX126x component drives SX1261/SX1262/SX1268/LLCC68 at 150-960 MHz in LoRa or FSK mode (up to 22 dBm), has `on_packet`/`send_packet` automations and sleep modes, and can be a Packet Transport platform between ESPHome nodes — [ESPHome SX126x](https://esphome.io/components/sx126x.html)
- LoRa is described as suitable for indoor and underground coverage because the signal penetrates walls and ceilings — [AWS LPWAN whitepaper](https://docs.aws.amazon.com/whitepapers/latest/implementing-lpwan-solutions-with-aws/lorawan.md)

#### 433 MHz sub-GHz (CC1101, RF bridges)
- ESPHome has a native CC1101 component: 300-928 MHz, ASK/OOK (default), 2-FSK, 4-FSK, GFSK and MSK, -30 to +11 dBm; async mode pairs with `remote_transmitter`/`remote_receiver` for OOK remotes, packet mode handles FSK links up to 64 bytes — [ESPHome CC1101](https://esphome.io/components/cc1101.html)
- A community guide states native CC1101 support arrived in ESPHome 2025.12 — [Home Assistant community guide](https://community.home-assistant.io/t/guide-direct-davis-vantage-receiver-868mhz-via-esphome-cc1101-no-custom-components/1014474)
- OpenMQTTGateway's RF gateway runs on ESP32 with a CC1101 or SX1276/SX1278; the RTL_433 library is receive-only and lists only OOK decoders; RCSwitch, Pilight and RF2 can send and receive; the active receiver and frequency (300-348, 387-464, 779-928 MHz) can be switched over MQTT; the Sonoff RF Bridge is a separate option that "doesn't currently receive correct values from some switch remotes" — [OpenMQTTGateway RF docs](https://docs.openmqttgateway.com/use/rf)
- In the OpenMQTTGateway issue tracker, CC1101 reception with rtl_433 is described as hit-or-miss for some users and worse than an RTL-SDR — [OpenMQTTGateway issue](https://www.github.com/1technophile/OpenMQTTGateway/issues/1887)
- Forum users report that a Sonoff RF Bridge and a Broadlink RM4 Pro could not capture a remote assumed to use rolling codes, and that no usable software for rolling codes exists — [Home Assistant thread](https://community.home-assistant.io/t/433mhz-kl180-remote-programming/1005908); [Home Assistant thread](https://community.home-assistant.io/t/any-rf-experts-out-there-specifically-433mhz/90619)

#### Z-Wave
- Home Assistant Connect ZWA-2 uses an 800-series chipset and runs classic Z-Wave and Long Range at once; launch price $69 (August 2025), £51 in the UK; Long Range is limited to US and EU regions — [CNX Software](https://www.cnx-software.com/2025/08/14/home-assistant-connect-zwa-2-z-wave-usb-adapter/); [Home Assistant ZWA-2](https://www.home-assistant.io/connect/zwa-2/); [The Pi Hut](https://thepihut.com/products/home-assistant-connect-zwa-2)
- US retailer listings show Zooz 800LR devices at $25.95-$42.95 each and a Zooz 800 Long Range USB stick at $26.95-$30.00 (undated listings; which of the two retailers carried which figure was not verified) — [CloudFree Z-Wave category](https://cloudfree.shop/product-category/z-wave); [Ameridroid Z-Wave collection](https://ameridroid.com/collections/z-wave-automation-devices)
- None of the ESP32-family chips lists a Z-Wave radio (radios are Wi-Fi, Bluetooth and 802.15.4 only) — [DroneBot Workshop ESP32 guide 2026](https://dronebotworkshop.com/esp32-2026/)

### Inferences
- **Synthesis table (my reading of the cited findings plus general engineering knowledge; cells without a cited number above are estimates, not sourced facts).**

| Radio | Through-wall range | Battery life (realistic) | Latency | Topology | Hub needed | Openness | Cheap devices | Ease on ESP32 | Path into MQTT |
|---|---|---|---|---|---|---|---|---|---|
| Wi-Fi + MQTT | Whole flat with one AP | Poor: months on 18650 with deep sleep (3.9 s per wake) | Low when mains-powered; seconds from deep sleep | Star to AP | Router only | Fully open | Many (Shelly, Athom, Tasmota/ESPHome-flashable) | Easiest | Direct (ESPHome MQTT, Tasmota) |
| ESP-NOW | Similar to Wi-Fi 2.4 GHz | Good: years possible (200 ms wake) | Very low | Peer-to-peer/broadcast | An always-on ESP32 gateway | Espressif-proprietary but documented | None to buy; DIY only | Easy (native in ESPHome) | Gateway node relays to Wi-Fi/MQTT |
| BLE adverts (BTHome v2) | One or two rooms per receiver | Very good: 1-5 years on coin cell | 1-10 s typical (advert interval) | Broadcast, one-way | Receivers (ESP32 proxies) | Open standard | Very cheap (LYWSD03MMC, Shelly BLU) | Easy as receiver; DIY senders better on nRF52 | Theengs/OpenMQTTGateway, or ESPHome proxy + Home Assistant |
| BLE GATT | Same | Device-dependent | Connection setup 1-3 s | Point-to-point, 3-5 slots per proxy | Proxy | Open | Many vendor devices, per-device support | Moderate | Via Home Assistant integrations |
| Bluetooth Mesh | Mesh | Mains nodes mostly | Grows with size | Mesh | Provisioner | Open spec, weak open tooling | Mostly cloud-brand lamps | Hard | No evident bridge |
| Zigbee 3.0 | 10-20 m per hop, mesh extends | Excellent: 2-4 years coin cell | Low (sub-second; sleepy devices poll every few s) | Mesh | Coordinator | Open spec, open stack via Z2M | Largest catalogue | Moderate (C6/H2 only) | Zigbee2MQTT |
| Thread / Matter | Like Zigbee | Like Zigbee for commercial; poor in DIY tests | Low | IPv6 mesh | Border router + Matter controller | Open spec; IP-native | Growing (IKEA) | Moderate-hard | Via Matter controller (e.g. Home Assistant), not natively MQTT |
| LoRa P2P / Meshtastic | Best: building-to-building, km outdoors | Good for sensors with long intervals | Seconds; duty-cycle limited | Star or flood mesh | Gateway node | Open (Meshtastic GPL); LoRa PHY proprietary | Few home devices | Moderate (ESPHome SX126x, RadioLib) | Meshtastic MQTT JSON; ESPHome gateway |
| 433 MHz (CC1101) | Good through walls | Excellent (one-way transmitters) | Low | One-way broadcast mostly | RF bridge | Ad-hoc protocols | Cheapest remotes, sockets, weather sensors, blind motors | Easy for OOK, hard for rolling codes | OpenMQTTGateway, ESPHome remote_receiver, rtl_433 |
| Z-Wave (800 LR) | Better than 2.4 GHz mesh | Excellent | Low | Mesh / LR star | Controller stick | Spec open since 2020, single silicon source | Costlier ($26-43 per device) | Not possible | Z-Wave JS UI to MQTT |

- **Protocol-to-role recommendation.** (1) Bought battery sensors, door contacts, buttons, sockets, wall switches and curtain motors: Zigbee 3.0 via Zigbee2MQTT. (2) Mains-powered DIY nodes (relays, radar presence, microphones, IR blasters, BLE proxies, voice satellites): Wi-Fi with ESPHome. (3) Cheapest temperature/humidity and door sensors: BLE adverts in BTHome v2 received by Wi-Fi/Ethernet ESP32 proxies. (4) DIY battery nodes: ESP-NOW with deep sleep, or an nRF52840/ESP32-H2 Zigbee sleepy end device once measured. (5) Legacy 433 MHz sockets, remotes and blind motors: one CC1101 gateway. (6) LoRa: only for a garage, garden or mailbox out of Wi-Fi/Zigbee reach. (7) Thread/Matter: optional, only if specific devices (IKEA) are wanted; needs a separate radio from the Zigbee coordinator. (8) Skip Bluetooth Mesh and, unless specific devices require it, Z-Wave.
- Zigbee 4.0 is a published specification with certification-ready vendor SDKs, but nothing found indicates that Zigbee2MQTT users need to act on it in late 2026; Zigbee 3.0 devices and coordinators remain the practical target.
- Because Zigbee2MQTT rejects multiprotocol firmware and single-radio sticks run Zigbee or Thread but not both, a home wanting both needs two radios (two sticks or a dual-radio SLZB-MR unit).
- For "sound of people waking up" plus illuminance plus time-of-day triggers, latency requirements are loose (seconds), so every protocol in the table is fast enough; reliability and battery cost matter more than latency.

### Gaps
- No 2025-2026 measurement study with indoor range in metres and command latency in milliseconds for all protocols was found; the only tabulated range figures are from an older Tom's Guide article.
- No source was found describing a Bluetooth Mesh integration for Home Assistant, Zigbee2MQTT-style bridges or other open hubs; absence of evidence, not proof of absence.
- LoRaWAN for a local home (private network server such as ChirpStack, gateway cost) was not researched to a citable level.
- A general "1 % duty cycle on most EU868 sub-bands (36 s per hour)" figure appeared in search summaries but could not be tied to a specific verified source; Meshtastic's own EU_868 preset is documented at 10 %.
- Whether ESPHome can act as a Matter device was not confirmed; only OpenThread (transport) support was confirmed.
- Whether the ESPHome Bluetooth proxy can be consumed without Home Assistant (pure MQTT stack) is not stated in its documentation; for an MQTT-only stack OpenMQTTGateway/Theengs is the documented route.
- The September 2026 "Suzi Product Certification is Now Open" notice was seen only as a CSA newsroom navigation snippet, not as a full article.
- Some qualitative statements from protocol-comparison articles (Homey, Silicon Labs, TechTarget) reached these notes through search summaries; which article made which statement was not individually verified.
- The ESPHome version in which each of `espnow`, `sx126x` and `cc1101` first appeared was not confirmed from release notes (only a community statement that CC1101 became native in 2025.12).

---

## 2. How many Wi-Fi IoT clients consumer routers handle, and best practices (IoT SSID/VLAN, DHCP reservations, blocking internet)

### Takeaway
There is no universal number: vendor statements range from 16 recommended to 64 hard-limited clients per band, and users report trouble anywhere from 15 ESP32 devices upward on consumer routers. The consistent advice is a dedicated 2.4 GHz IoT network (20 MHz, WPA2-AES, band steering off), DHCP reservations, a narrow firewall allow-rule toward the hub, and no internet for devices that do not need it.

### Cited Findings
- TP-Link support says the Archer AX20 allows a maximum of 64 associated wireless clients per band, and that the real limit depends on router hardware and environment — [TP-Link community](https://community.tp-link.com/en/home/forum/topic/271106)
- In a TP-Link thread about a home-automation design, the poster cites a theoretical 32 clients per band per device and a vendor recommendation of 16 connected clients per band — [TP-Link community](https://community.tp-link.com/en/home/forum/topic/829152)
- An ASUS forum user reports Wi-Fi clients disconnecting once 15 or more ESP32 devices are connected — [ASUS ROG forum](https://rog-forum.asus.com/t5/gaming-routers/wi-fi-clients-disconnect-when-15-esp32-devices-are-connected-gt/td-p/1110167)
- An ASUS AiMesh user with about 165 smart and wireless devices across 6 nodes describes devices dropping out and staying unresponsive for 45 minutes to 2 hours — [SNBForums](https://snbforums.com/threads/asus-aimesh-have-i-exceeded-max-clients-for-6-nodes-lots-of-unresponsive-clients.84425/)
- A general-audience article says a typical home router is good for only about 20 to 50 devices — [BGR](https://www.bgr.com/2141119/how-many-devices-average-home-router-can-connect-to)
- TP-Link's IoT guidance is to disable band steering so IoT devices connect on 2.4 GHz — [TP-Link FAQ](https://service-provider.tp-link.com/faq/806/)
- Router-settings guides recommend a dedicated 2.4 GHz IoT network, 20 MHz channel width on 2.4 GHz and WPA2-AES (not WPA3-only) for compatibility, noting many sensors and plugs cannot complete setup on 5 or 6 GHz — [TechBloat guide](https://www.techbloat.com/best-router-settings-for-reliable-smart-home-devices-a-practical-guide.html); [Zyxel community](https://community.zyxel.com/en/discussion/34242/does-a-separate-2-4ghz-iot-ssid-improve-the-speed-of-my-main-network)
- A guest network is the simplest segmentation on consumer routers but usually applies to Wi-Fi only; a VLAN covers wired and wireless clients and needs a VLAN-capable router and managed switch — [SpeedtestHQ IoT VLAN guide](https://speedtesthq.com/guides/smarthome/iot-vlan-separate-network)
- Recommended firewall pattern: allow the IoT VLAN to reach the hub's IP on specific ports (8123 for the Home Assistant UI, 1883 for MQTT), block it to other VLANs and to the internet, with the allow rule first; mDNS reflection is needed for cross-VLAN discovery and slightly reduces isolation — [Botmonster segmentation guide](https://botmonster.com/smart-home/secure-smart-home-network-segmentation-vlans/)
- A 2026 UniFi zone-based-firewall write-up warns that replies to hub-initiated connections must be allowed or they are caught by the IoT block rule — [vNinja](https://vninja.net/2026/08/13/unifi-zone-based-firewall-changes-the-iot-vlan-rule/)
- Home Assistant forum users note ESPHome and Shelly devices work locally without internet but must be able to reach the hub, whereas cloud-dependent devices such as Tuya are the exception; a common approach is to block internet in both directions for devices that do not need it — [Home Assistant community](https://community.home-assistant.io/t/ha-networking-how-do-you-protect-your-home-network-from-iot-devices/728023); [Home Assistant community](https://community.home-assistant.io/t/how-to-properly-set-up-secure-iot-network-with-home-assistant/505156)
- An OpenWrt user needed both an mDNS reflector and a firewall rule for UDP 5353 to get discovery across an IoT VLAN — [OpenWrt forum](https://forum.openwrt.org/t/advice-needed-iot-vlan-mdns-avahi-ssdp/242639)
- A 2026 home-segmentation write-up gives each VLAN its own /24 subnet and DHCP scope and treats the mDNS reflector as a per-VLAN toggle separate from firewall rules — [Kunal Khosla blog](https://kunalkhosla.github.io/blogs/2026/04/20/home-network-segmentation.html)

### Inferences
- A practical planning figure for a consumer router is roughly 20-30 Wi-Fi IoT clients per 2.4 GHz radio before reliability becomes router-dependent; beyond that, add access points or move sensor-class devices to Zigbee/BLE/ESP-NOW. This is a judgment from the anecdotes above, not a measured limit.
- Keeping battery sensors off Wi-Fi (Question 1) also keeps the Wi-Fi client count low: only mains nodes, gateways and voice satellites need Wi-Fi.
- ESPHome discovery relies on mDNS, so either keep the hub on the IoT VLAN, enable an mDNS reflector, or use static IPs/DHCP reservations and address devices by IP.
- NTP should be allowed only to a local server or the gateway if internet is blocked (no source found; reasonable practice).

### Gaps
- No authoritative, tested client-capacity figures per router chipset were found; all numbers are vendor forum statements or user anecdotes.
- Several best-practice statements come from blogs and community posts surfaced through search summaries; per-sentence attribution among the listed guides was not individually verified.
- No source specifically on NTP handling for internet-blocked IoT VLANs.

---

## 3. ESP32 family in late 2026 (classic, S3, C3, C6, C5, C61, H2, P4, plus newcomers): radios, memory, deep sleep, price, node role, boards and framework support

### Takeaway
ESP32-C6 is the default all-rounder for smart-home nodes (Wi-Fi 6 + BLE + 802.15.4), ESP32-S3 with PSRAM is the choice for voice satellites and displays, ESP32-C3 is the cheapest Wi-Fi/BLE node, and ESP32-H2 is the dedicated Zigbee/Thread end device. ESP32-C5 and C61 reached full ESP-IDF support in v6.0 (March 2026), and board-level deep-sleep current (13-490 µA) matters far more than the chip figure (5-8 µA).

### Cited Findings

#### Chip comparison
| Chip | CPU | RAM | Wi-Fi | Bluetooth | 802.15.4 (Zigbee/Thread) | Positioning per source |
|---|---|---|---|---|---|---|
| ESP32 (classic) | 2x Xtensa LX6, 240 MHz | 520 KB | 2.4 GHz Wi-Fi 4 | Classic + BLE 4.2 | No | Only option with Bluetooth Classic; otherwise not recommended for new designs |
| ESP32-S3 | 2x Xtensa LX7, 240 MHz | 512 KB | 2.4 GHz Wi-Fi 4 | BLE 5.0 | No | General-purpose favourite; USB OTG, camera/LCD, TinyML |
| ESP32-C3 | 1x RISC-V, 160 MHz | 400 KB | 2.4 GHz Wi-Fi 4 | BLE 5.0 | No | Budget Wi-Fi + BLE |
| ESP32-C6 | 1x RISC-V, 160 MHz | 512 KB | 2.4 GHz Wi-Fi 6 | BLE 5.3 | Yes | Recommended all-rounder for IoT and smart home |
| ESP32-C5 | 1x RISC-V, 240 MHz | 400 KB | 2.4 + 5 GHz Wi-Fi 6 | BLE 5.0 | Yes | Only ESP32 with 5 GHz Wi-Fi |
| ESP32-C61 | 1x RISC-V, 160 MHz | 256 KB | 2.4 GHz Wi-Fi 6 | BLE 5.0 | No | Cheap Wi-Fi 6, co-processor or budget use |
| ESP32-H2 | 1x RISC-V, 96 MHz | 320 KB | None | BLE 5.0 (5.3 per DevKit entry) | Yes | Thread/Zigbee endpoint, ultra-low power |
| ESP32-H4 | 2x RISC-V, 96 MHz | 320 KB | None | BLE 5.4 | Yes | Thread/Zigbee + BLE audio; dev kits limited |
| ESP32-P4 | 2x RISC-V, 400 MHz | 768 KB | None (needs companion chip) | None | No | High-performance multimedia; boards often add a C6 |

- All rows of the table above — [DroneBot Workshop ESP32 guide 2026](https://dronebotworkshop.com/esp32-2026/)
- ESP32-S31 (new in 2026): dual-core RISC-V up to 320 MHz, 512 KB SRAM, 2.4 GHz Wi-Fi 6, Bluetooth 5.4 LE plus Classic, 802.15.4 (Thread/Zigbee), 1000 Mbps Ethernet MAC, 60 GPIO — [Espressif ESP32-S31](https://www.espressif.com/en/products/socs/esp32-s31)
- Espressif announced ESP32-S31 mass production and availability on 27 July 2026 — [Espressif news](https://www.espressif.com/en/news/ESP32_S31_Mass_Production)
- ESP32-E22 (Wi-Fi 6E radio co-processor) launched January 2026 with engineering samples and no pricing or general-availability information at launch — [It's FOSS](https://itsfoss.com/news/espressif-launches-esp32-e22.md); [Elektor](https://www.elektormagazine.com/news/new-espressif-microcontrollers-ces-2026)
- ESP32-H4 (dual-core RISC-V, 96 MHz, 802.15.4 + BLE) was announced in March 2026 with samples on request; no mass-production date was found — [Matter Alpha](https://www.matteralpha.com/industry-news/espressif-releases-next-gen-matter-chipsets-and-open-source-sdk)

#### Deep-sleep current
- Espressif's low-power guide gives deep-sleep base current of 5 µA for ESP32-C3 and 7 µA for ESP32-S3 — [ESP Techpedia](https://docs.espressif.com/projects/esp-techpedia/en/latest/esp-friends/advanced-development/performance/lowpower/lowpower-fundamental.html)
- Espressif measured 8.14 µA in deep sleep on a bare ESP32-C6 module — [ESP-IDF current consumption guide](https://docs.espressif.com/projects/esp-idf/en/v6.0.3/esp32c6/api-guides/current-consumption-measurement-modules.html)
- Board-level measurements of Seeed XIAO boards (2026): XIAO ESP32-S3 13.4 µA, XIAO ESP32-C6 16.9 µA, XIAO ESP32-C3 41.6 µA, although the C3 chip has the lowest datasheet figure — [Qoitech](https://www.qoitech.com/?p=19961)
- Seeed lists 15 µA typical deep sleep, 3.1 mA light sleep and 30 mA modem sleep for the XIAO ESP32-C6 — [Seeed XIAO ESP32-C6](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html)
- An ESP32-H2 SuperMini drew 488 µA in deep sleep unmodified, 223 µA without the WS2812 LED, 148 µA without the LDO, 115 µA without the charger IC and about 17 µA after also removing the charger LED (PPK2 measurements quoted in a forum post, January 2026) — [Home Assistant community thread](https://community.home-assistant.io/t/esp32-h2-thread-with-deep-sleep-for-cheap-esp32-battery-sensors/919132)

#### Framework support
- ESP-IDF v6.0 (announced March 2026) moved ESP32-C5 and ESP32-C61 from preview to full support and added preview support for ESP32-H21 and ESP32-H4 — [Espressif ESP-IDF 6.0](https://www.espressif.com/en/news/ESP_IDF_6.0); [CNX Software](https://www.cnx-software.com/2026/03/24/esp-idf-v6-0-framework-adds-support-for-esp32-c5-and-esp32-c61-preview-for-esp32-h21-and-esp32-h4/)
- Arduino-ESP32 core 3.3.x covers ESP32, C3, C6, H2, P4, S2 and S3; ESP32-C2 and ESP32-C61 are supported only when Arduino is used as an ESP-IDF component or static libraries are rebuilt; 3.3.12 was the latest version listed — [arduino-esp32 3.3.0 readme](https://components.espressif.com/components/espressif/arduino-esp32/versions/3.3.0/readme); [arduino-esp32 3.3.6 readme](https://components.espressif.com/components/espressif/arduino-esp32/versions/3.3.6/readme); [arduino-esp32 3.3.12](https://components.espressif.com/components/espressif/arduino-esp32/versions/3.3.12/readme)
- Rust: esp-hal 1.1.x lists ESP32-C5, C6, C61, H2 and P4 (P4 chip revision v3.x or later) among supported chips, with partial driver coverage on many peripherals; the "Rust on ESP" book's table still omits C5 and P4 — [esp-hal on docs.rs](https://docs.rs/crate/esp-hal/latest); [Rust on ESP book](https://docs.espressif.com/projects/rust/book/introduction/hardware-overview.html)
- MicroPython: v1.27.0 (10 December 2025) added ESP32-C5 and ESP32-P4; ESP32-C6 has been supported since v1.24.0; ESP32-H2 arrived in v1.29.0 (24 August 2026) — [CNX Software](https://www.cnx-software.com/2025/12/16/micropython-v1-27-adds-support-for-esp32-c5-esp32-p4-and-stm32u5-microcontrollers/); [MicroPython ESP32_GENERIC_H2](https://micropython.org/download/ESP32_GENERIC_H2); [MicroPython v1.29.0 tag](https://gitlab.com/alelec/micropython/-/tags/v1.29.0)
- ESPHome: C6, H2 and P4 became official variants in 2025.6.0 (described then as "still being refined"); C5 gained internal temperature (2026.1.0) and dual-band `band_mode` (2026.3.0); 2026.4.0 added ESP32-C61 PSRAM support and made 240 MHz the default for ESP32/S2/S3/C5; P4 pre-rev3 samples need `engineering_sample: true`; 2026.11.0 mentions ESP32-S31 I2S audio support — [Hackster on ESPHome 2025.6.0](https://www.hackster.io/news/esphome-2025-6-0-brings-espressif-esp32-p4-compatibility-initial-openthreads-support-741dd11dbbb4); [ESPHome 2026.1.0](https://esphome.io/changelog/2026.1.0.html); [ESPHome 2026.3.0](https://esphome.io/changelog/2026.3.0/); [ESPHome 2026.4.0](https://esphome.io/changelog/2026.4.0/); [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- ESPHome Zigbee runs on ESP32-C5/C6/H2 and nRF52; ESP-NOW does not run on ESP32-H2 — [ESPHome Zigbee](https://esphome.io/components/zigbee.html); [ESPHome ESP-NOW](https://esphome.io/components/espnow.html)

#### Development boards and prices
| Board | Price and date | Notes | Source |
|---|---|---|---|
| Seeed XIAO ESP32-C6 | $5.20 (retrieved 2026-11-25) | 4 MB flash, Wi-Fi 6 + BLE + 802.15.4, LiPo charging, 15 µA deep sleep | [Seeed](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html) |
| Seeed XIAO ESP32-C5 | $7.24 incl. US tariffs at RobotShop (2026, undated) | Dual-band Wi-Fi 6 | [RobotShop](https://www.robotshop.com/products/seeedstudio-seeed-studio-xiao-esp32-c5) |
| M5Stack NanoC6 | $5.95 (a second price of $6.95 also shown; snippet, 2026) | ESP32-C6, Grove port | [M5Stack store](https://shop.m5stack.com/collections/controllers) |
| M5Stack Atom Lite | $7.69 at Mouser (2026, undated) | Classic ESP32 | [Mouser](https://www.mouser.com/c/rf-wireless/rf-wireless-development-tools/multiprotocol-development-tools/?m=M5Stack) |
| M5Stack Atom VoiceS3R (renamed from Atom EchoS3R in April 2026) | $14.50 excl. tax reference price; ¥2,739 incl. tax at Switch Science Japan (2026) | ESP32-S3-PICO-1-N8R8, 8 MB flash + 8 MB PSRAM, ES8311 codec, MEMS mic, NS4150B amp, IR transmitter | [Switch Science](https://www.switch-science.com/products/10808); [M5Stack docs](https://docs.m5stack.com/en/core/Atom_EchoS3R) |
| M5Stack StickS3 | about $21.50 (January 2026) | See Question 4 | [CNX Software](https://www.cnx-software.com/2026/01/23/m5stack-sticks3-esp32-s3-mini-iot-dev-kit-features-1-14-inch-color-display-speaker-and-mic-for-voice-interaction/) |
| Espressif ESP32-C5-DevKitC-1-N8R8 | 21.60 at Mouser Canada (currency probably CAD; 2026) | Official dual-band devkit | [Mouser.ca](https://www.mouser.ca/fr/c/?q=esp32) |
| Waveshare ESP32-H2 dev kit | $4.99-$5.99 (2026 listing) | Zigbee/Thread only | [Waveshare](https://www.waveshare.com/product/esp32-related/boards-kits.htm?dir=asc&mode=list) |
| Waveshare ESP32-C6 dev board | $5.99-$8.49 (2026 listing) | | [Waveshare](https://www.waveshare.com/product/esp32-related/boards-kits.htm?dir=asc&mode=list) |
| Waveshare ESP32-P4-Core | $11.99-$12.99 (2026 listing) | | [Waveshare](https://www.waveshare.com/product/esp32-related/boards-kits.htm?dir=asc&mode=list) |
| ESP32-C6 / ESP32-H2 SuperMini | about $5 each (comparison guide, undated) | H2 has no Wi-Fi | [ESPBoards SuperMini comparison](https://www.espboards.dev/blog/esp32-super-mini-comparison/) |
| WeAct ESP32-C6-Mini | $3.86 on AliExpress **[PRE-2025]** | | [CNX Software](https://cnx-software.com/?p=129692) |
| Waveshare ESP32-S3-Relay-6CH | $28.99 (Waveshare store, undated) | 6 relays rated up to 10 A 250 VAC / 30 VDC, RS485; vendor docs mention Arduino and MicroPython | [Waveshare](https://www.waveshare.com/product/modules/esp32-s3-relay-6ch.htm) |
| LilyGO T-Relay (4-channel, classic ESP32) | $10.01, shown sold out (undated) | Needs LilyGO T-U2T adapter to flash over USB | [LilyGO](https://lilygo.cc/products/t-relay); [LilyGO wiki](https://wiki.lilygo.cc/products/t-relay-series/t-relay/) |
| LilyGO T-Relay-S3 (6-channel) | Price not found | ESP32-S3-WROOM-1U, 16 MB flash, 8 MB PSRAM; wiki lists an ESPHome example | [LilyGO wiki](https://wiki.lilygo.cc/products/t-relay-series/t-relay-s3/) |
| Home Assistant Voice Preview Edition | $59 at launch (December 2024); a 2026 review cites $69 MSRP | ESP32-S3 + XMOS XU316, dual mics, stock ESPHome | [Home Assistant blog](https://home-assistant.io/blog/2024/12/19/voice-preview-edition-the-era-of-open-voice); [Botmonster review](https://botmonster.com/smart-home/home-assistant-voice-preview-edition-review/) |

- ESP32-C3 SuperMini clones have recurring Wi-Fi problems: users report "Auth Expired"/handshake failures on cheap AliExpress boards (suspected missing antenna matching components) and very limited range from the tiny antenna — [Home Assistant community](https://community.home-assistant.io/t/esp32-c3-super-mini-wi-fi-auth-expired-no-tx-current-spikes-wroom-32d-works-perfectly/1021889); [WLED forum](https://wled.discourse.group/t/some-c3-super-minis-cant-connect-to-wi-fi/12153)
- M5Stack publishes ready-made ESPHome voice-assistant packages for CoreS3 (ESPHome 2026.1.0 or later) and Atom VoiceS3R (ESPHome 2026.10.0 or later) — [M5Stack CoreS3 voice assistant](https://docs.m5stack.com/en/homeassistant/applications/voice_assistant/cores3); [M5Stack Atom VoiceS3R voice assistant](https://docs.m5stack.com/en/homeassistant/applications/voice_assistant/atom_voices3r)
- ESPHome 2026.11.0 voice changes: configurable TTS playback start timeout; micro wake word gains opt-in internal-RAM placement, selectable inference core and configurable capture buffer — [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- A community comparison thread covers Home Assistant Voice PE, FutureProofHomes Satellite1 and Seeed reSpeaker XVF3800 as ESPHome voice satellites (2026; exact post dates not read) — [Home Assistant community](https://community.home-assistant.io/t/voice-satellite-comparison-ha-voice-pe-vs-futureproofhomes-satellite1-vs-seeed-respeaker-xvf3800/1026509)

### Inferences
- **Chip-to-role mapping.** Mains relay/sensor node: ESP32-C6 (or C3 where cost dominates and a good antenna is present). Battery sensor: ESP32-C6 or H2 on a board with a low-quiescent regulator (XIAO class), or better an nRF52840; avoid SuperMini boards unless modified. Voice satellite: ESP32-S3 with 8 MB PSRAM plus a codec (Atom VoiceS3R at about $14.50 is the cheapest confirmed ready-made option; Voice PE at $59-69 has far-field DSP). BLE gateway/proxy: classic ESP32 or S3/C6, ideally with Ethernet. Zigbee/Thread end device or router: ESP32-H2 (reported more reliable than C6) or C6 when Wi-Fi is also needed. ESP32-C5: only where 5 GHz Wi-Fi is needed. ESP32-C61: not worth it for DIY yet (ESPHome support limited to a PSRAM entry, Arduino only as IDF component). ESP32-P4: display/camera panels only, needs a companion radio. ESP32-S31: promising single-chip Wi-Fi 6 + BT + 802.15.4 + Ethernet part, but too new for an ESPHome-first build.
- ESPHome is the most complete path across the family; Rust (esp-hal) now names all current chips but with partial peripheral coverage; MicroPython covers C3/C6/S3/C5/P4/H2 but without first-class Zigbee/Thread device stacks (not confirmed).
- For a buy list in late 2026: XIAO ESP32-C6 ($5.20) or M5Stack NanoC6 (about $6) as generic nodes, Atom VoiceS3R for voice experiments, a 6-channel S3 relay board ($29) for a DIN/box-mounted mains node.

### Gaps
- Chip-level deep-sleep figures for classic ESP32, ESP32-C5, C61, H2 and P4 were not found in fetched sources (only C3, S3 and C6 module figures).
- Per-chip unit prices and flash/PSRAM options were not found in a single 2026 source.
- USD prices not found: Espressif ESP32-C6-DevKitC-1, ESP32-S3-DevKitC-1, ESP32-H2-DevKitM-1, ESP32-P4 Function EV board, ESP32-C61 devkit; Seeed XIAO ESP32-S3, XIAO nRF52840, XIAO MG24, XIAO ESP32-C3 single-unit (a Hackster listing showed $4.99 at 10+ units, unverified); M5Stack AtomS3 Lite, AtomS3R and Stamp modules; current ESP32-C3 SuperMini street price.
- The Mouser.ca figure for the ESP32-C5-DevKitC-1 is of uncertain currency.
- ESPHome support for the Waveshare ESP32-S3-Relay-6CH was not confirmed by a source (it is a plain ESP32-S3 with GPIO relays, so support is likely).
- PSRAM requirements for ESPHome voice assistant with on-device wake word were not confirmed by a primary source.

---

## 4. Identifying the two owned M5Stack devices and their smart-home roles

### Takeaway
"Cardputer mesh version" matches a specific 2026 product: the Cardputer Mesh Kit (SKU K152), a Cardputer-Adv with the Cap LoRa-1262 (SX1262 + GNSS) and Meshtastic preinstalled. "M5 stick with radio module" is not one official product; the most likely match is an M5StickC Plus2 (or the 2026 StickS3) with a third-party NRF24 + CC1101 hat sold for Bruce firmware, and the exact stick model must be checked on the device.

### Cited Findings

#### (b) Cardputer "mesh version"
- M5Stack sells a Cardputer Mesh Kit (SKU K152): 1x Cardputer-Adv (green back shell), 1x Cap LoRa-1262, RP-SMA rubber-duck LoRa antenna, screws and hex key; Meshtastic firmware is factory-flashed — [M5Stack docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit)
- It launched around 30 April 2026 at $48.00 for the kit; the Cap LoRa-1262 alone was $14.50 — [CNX Software](https://www.cnx-software.com/2026/04/30/m5stack-cardputer-goes-off-grid-with-new-mesh-kit-featuring-lora-gnss-and-meshtastic-support/)
- Core specs: ESP32-S3FN8 on a Stamp-S3A (dual-core LX7 240 MHz, 512 KB SRAM), 8 MB flash, no PSRAM mentioned; 2.4 GHz Wi-Fi 4 and Bluetooth 5 LE; 1.14" 240x135 ST7789V2 LCD; 56-key keyboard (TCA8418); ES8311 codec, MEMS microphone (65 dB SNR), NS4150B amplifier with 1 W speaker, 3.5 mm jack; BMI270 IMU; one IR emitter (no receiver mentioned); microSD; Grove HY2.0-4P; EXT 2.54 mm 14-pin bus — [M5Stack Cardputer-Adv docs](https://docs.m5stack.com/en/core/Cardputer-Adv); [CNX Software](https://www.cnx-software.com/2026/04/30/m5stack-cardputer-goes-off-grid-with-new-mesh-kit-featuring-lora-gnss-and-meshtastic-support/)
- Cap LoRa-1262: Semtech SX1262, 868-923 MHz, +22 dBm, -147 dBm sensitivity, LoRa/FSK/GFSK/MSK; ATGM336H-6N (AT6668) GNSS with ceramic antenna; its own Grove port (I2C) — [M5Stack docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit)
- EXT bus pin map: RESET G3, INT G4, BUSY G6, SCK G40, MOSI G14, MISO G39, CS G5, I2C SDA G8 / SCL G9, UART RX G13 / TX G15; microSD shares SPI (CS G12) — [M5Stack docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit)
- Battery: documentation changed the capacity from 1750 mAh to 1500 mAh on 2026-07-13; CNX's launch article gives 1750 mAh — [M5Stack docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit); contradicted on capacity by [CNX Software](https://www.cnx-software.com/2026/04/30/m5stack-cardputer-goes-off-grid-with-new-mesh-kit-featuring-lora-gnss-and-meshtastic-support/)
- Measured draw of the Cardputer-Adv at 4.2 V: 120.2 mA operating, 132.3 mA with Wi-Fi, 154.6 mA with BLE — [M5Stack Cardputer-Adv docs](https://docs.m5stack.com/en/core/Cardputer-Adv)
- Firmware: Meshtastic (official, licensed; only Mesh Kit units can be registered as authorized devices), Arduino (M5Cardputer, RadioLib, TinyGPSPlus), UiFlow2, ESP-IDF, PlatformIO; ESPHome and M5Unified are not listed by M5Stack — [M5Stack docs](https://docs.m5stack.com/en/core/Cardputer_Mesh_Kit)
- Meshtastic lists exactly two M5Stack devices: Unit C6L (ESP32-C6 + SX1262, no GPS) and the Cardputer Mesh Kit (ESP32-S3 + SX1262, GNSS cap) — [Meshtastic M5Stack devices](https://meshtastic.org/docs/hardware/devices/m5stack/)
- Earlier/alternative caps: Cap LoRa868 (Stamp LoRa-1262, SX1262, 868-923 MHz) is documented for the Cardputer-Adv; Cap CC1101 & NFC covers 315/433/868/915 MHz sub-GHz plus 13.56 MHz NFC, $18.95 at M5Stack, and is not a LoRa radio — [M5Stack Cap LoRa868](https://docs.m5stack.com/en/cap/Cap_LoRa868); [M5Stack Cap CC1101](https://docs.m5stack.com/en/cap/Cap_CC1101); [M5Stack store](https://shop.m5stack.com/products/cap-cc1101-nfc-for-cardputer); [The Pi Hut](https://thepihut.com/products/cap-cc1101-nfc-for-cardputer-adv)
- The standalone Cardputer-Adv is listed at $29.90 on M5Stack's store — [M5Stack store](https://shop.m5stack.com/collections/controllers/cardputer)
- Differences from older Cardputers: the original and v1.1 have 120 + 1400 mAh batteries, NS4168 + SPM1423 audio, no IMU, no 3.5 mm jack and no EXT bus, so they cannot take the LoRa cap — [M5Stack Cardputer-Adv docs](https://docs.m5stack.com/en/core/Cardputer-Adv)
- Bruce firmware supports "M5Stack Cardputer (and ADV)" with CC1101, NRF24, FM, PN532, microphone and BadUSB; its feature list has no LoRa item — [Bruce firmware](https://github.com/BruceDevices/firmware)

#### (a) "M5 stick with radio module": candidate sticks
| Model | SoC | Flash / PSRAM | Radios | Audio | IMU | IR | Display | Battery | Expansion | Source |
|---|---|---|---|---|---|---|---|---|---|---|
| M5StickC Plus2 (2023-12) | ESP32-PICO-V3-02, 2 cores 240 MHz | 8 MB / 2 MB | 2.4 GHz Wi-Fi per the docs page (the chip is a classic ESP32, which per the Question 3 chip table also has Bluetooth Classic + BLE 4.2) | SPM1423 PDM mic, passive buzzer | MPU6886 | Transmitter only (G19, shared with red LED) | 1.14" 135x240 ST7789V2 | 200 mAh | Grove HY2.0-4P (G32/G33), HAT pins G0, G25/G26, G36; BM8563 RTC; 3 buttons | [M5Stack docs](https://docs.m5stack.com/en/core/M5StickC%20PLUS2) |
| StickS3 (2026-01, about $21.50) | ESP32-S3-PICO-1-N8R8 | 8 MB / 8 MB | 2.4 GHz Wi-Fi 4, Bluetooth 5 LE | ES8311 codec, MEMS mic (65 dB SNR), AW8737 amp, 1 W speaker | BMI270 | Transmitter and receiver | 1.14" 240x135 ST7789P3 | 250 mAh | 16-pin HAT2 header, Grove (5 V in or out), USB-C OTG | [CNX Software](https://www.cnx-software.com/2026/01/23/m5stack-sticks3-esp32-s3-mini-iot-dev-kit-features-1-14-inch-color-display-speaker-and-mic-for-voice-interaction/) |
| M5StickC Plus (older) | Not fetched | Half the flash of the Plus2 and no PSRAM (implied by the Plus2 announcement: "double the flash, 2 MB of PSRAM, bigger battery") | Not fetched | Not fetched | Not fetched | Not fetched | Not fetched | Smaller than 200 mAh (same implication) | Not fetched | [Hackster](https://hackster.io/news/m5stack-updates-the-compact-m5stickc-plus-with-double-the-flash-2mb-of-psram-and-a-bigger-battery-9845aed35cf2) **[PRE-2025]** |

- The StickS3 is described by a Japanese distributor as the successor to the M5StickC Plus2 (release date 23 January 2026), with basic pin layout and HAT bus compatible but some code changes needed; another source lists the Plus2 as EOL — [Switch Science](https://www.switch-science.com/products/10921); [atomic14](https://www.atomic14.com/esp32/boards/m5stickc-plus2/)
- StickS3 usage warnings: the speaker amplifier must be off while using the IR receiver; on battery keep speaker volume below 75 % to avoid reboots; software support is Arduino, UiFlow2, ESP-IDF, PlatformIO, and M5Stack has a Xiaozhi voice-assistant tutorial — [CNX Software](https://www.cnx-software.com/2026/01/23/m5stack-sticks3-esp32-s3-mini-iot-dev-kit-features-1-14-inch-color-display-speaker-and-mic-for-voice-interaction/)
- M5StickC Plus2 software: Arduino, UiFlow1/2, ESP-IDF, PlatformIO with M5Unified; the AXP192 PMIC was removed in this version — [M5Stack docs](https://docs.m5stack.com/en/core/M5StickC%20PLUS2)

#### (a) Radio add-ons sold with or for sticks
- Third-party "RF 2-in-1" hats for the M5StickC Plus / Plus2 combine an NRF24 (2.4 GHz) and a CC1101 (433 MHz) with a selector switch and are marketed for Bruce firmware; a Tindie board for the Plus2 uses a hot-swap socket for either module — [Newegg listing](https://www.newegg.com/pingequa-rf-2-in-1-module-for-m5stack-stickc-plus/p/3D0-0053-00018); [Tindie](https://www.tindie.com/products/gumslone/m5stickc-plus2-nrf24cc1101-dev-board); [eBay listing](https://www.ebay.de/itm/365779084138)
- Bruce's device table lists M5StickC PLUS2 and M5StickC PLUS with CC1101, NRF24, FM, PN532, microphone and BadUSB support; StickS3 was not in the table when read; the RF module can be set to CC1101 or to M5Stack's "RF433 T/R" units; features include sub-GHz scan/copy/replay, IR TV-B-Gone and custom IR, BLE scan, Wi-Fi tools and ESP-NOW file/command transfer — [Bruce firmware](https://github.com/BruceDevices/firmware)
- M5Stack's shop blog says Bruce is installable on the StickC Plus2 through M5Burner — [M5Stack blog](https://shop.m5stack.com/blogs/blog/all-you-need-to-know-about-stickc-plus2)
- Official M5Stack radio products that connect to a stick's Grove port: Unit LoRaE220-433 (E220-400T22S, LLCC68, 410-493 MHz, 22 dBm, UART) and LoRaWAN Unit 868 (ASR6501) — [M5Stack Unit LoRaE220-433](https://docs.m5stack.com/en/unit/Unit_LoRaE220-433)
- M5Stack's own CC1101 product (SKU M146, E07-900M10S, 855-925 MHz, SPI, stackable) is a Core-series module, not a stick hat, and does not cover 433 MHz — [M5Stack M146](https://docs.m5stack.com/en/products/sku/M146); [Mouser](https://www.mouser.com/new/m5stack/m5stack-cc1101-wireless-communication-module/)
- Unit C6L is a self-contained ESP32-C6 + SX1262 (868-923 MHz) Meshtastic unit with OLED and buzzer, $22.90 at Mouser — [Meshtastic M5Stack devices](https://meshtastic.org/docs/hardware/devices/m5stack/); [Mouser RF modules listing](https://www.mouser.com/c/embedded-solutions/wireless-rf-modules/rf-modules/?m=M5Stack)

#### ESPHome and gateway software relevant to both devices
- No ESPHome device page or M5Stack ESPHome package was found for the Cardputer, StickC Plus2 or StickS3; a community YAML exists for the original M5StickC (IR transmitter on GPIO9), and M5Stack's ESPHome packages cover CoreS3 and Atom VoiceS3R — [Home Assistant community](https://community.home-assistant.io/t/working-esphome-yaml-for-m5stickc/538050); [M5Stack Atom VoiceS3R voice assistant](https://docs.m5stack.com/en/homeassistant/applications/voice_assistant/atom_voices3r)
- ESPHome building blocks that match the hardware: SX126x (LoRa/FSK), CC1101 (sub-GHz), Bluetooth proxy, `sound_level`, IR/RF proxy (`ir_rf_proxy` with `remote_transmitter`/`remote_receiver`) — [ESPHome SX126x](https://esphome.io/components/sx126x.html); [ESPHome CC1101](https://esphome.io/components/cc1101.html); [ESPHome Bluetooth Proxy](https://esphome.io/components/bluetooth_proxy.html); [ESPHome IR/RF Proxy](https://esphome.io/components/ir_rf_proxy/)
- Home Assistant added a native infrared entity platform with transmit in 2026.4 and receive in 2026.6; ESPHome is one of the supported infrared adapters — [Home Assistant developer blog](https://developers.home-assistant.io/blog/2026/03/30/infrared-entity-platform); [Home Assistant infrared integration](https://home-assistant.io/integrations/infrared)
- Meshtastic JSON over MQTT works on ESP32-based nodes (it is excluded only on nRF52), so the ESP32-S3 Cardputer can be a Meshtastic-to-MQTT gateway on a private broker — [Meshtastic MQTT docs](https://meshtastic.org/docs/software/integrations/mqtt/)

### Inferences
- **Identification.** "Cardputer mesh version" = Cardputer Mesh Kit K152 with high confidence (the product name contains "Mesh", it is the only Cardputer bundle with LoRa, and it is Meshtastic-licensed). A green back shell and an RP-SMA antenna on a top-mounted cap confirm it. If the cap is instead labelled Cap LoRa868 or the unit was bought before April 2026, it is a Cardputer-Adv with a separately bought LoRa cap, functionally equivalent.
- **Identification.** "M5 stick with radio module" is most probably an M5StickC Plus2 with a third-party NRF24 + CC1101 hat (the common "Bruce kit"); second most likely a StickS3 with the same kind of hat; less likely an official Grove unit (LoRa E220, RF433T/R). Tell-tales: StickS3 has a speaker and USB OTG and identifies as ESP32-S3; Plus2 has a buzzer and three buttons and identifies as ESP32-PICO-V3-02; an NRF24 module has a 2x4-pin header and a CC1101 module a coiled or SMA 433 MHz antenna.
- **Roles for the Cardputer Mesh Kit.** (1) Meshtastic node and Meshtastic-to-MQTT gateway for long-range sensors (keep stock firmware). (2) Alternatively an ESPHome LoRa point-to-point gateway using `sx126x` with the pin map above (CS G5, BUSY G6, DIO1/INT G4, RST G3, SPI on G40/G14/G39); this mapping is derived from the EXT pin table and needs verification. (3) Handheld control panel and admin terminal (keyboard + display over MQTT). (4) Push-to-talk microphone and small speaker for voice (ES8311 + MEMS mic). (5) IR blaster (transmit only). (6) BLE scanner/BTHome receiver. (7) General S3 development node. Limits: 8 MB flash and apparently no PSRAM constrain heavier ESPHome builds (display + voice + BLE together); 120-155 mA draw (more with the LoRa cap, which M5Stack rates at 163.4 mA from 5 V) means on the order of 8-11 hours on the 1500 mAh battery, so permanent gateway duty needs USB power; the SX1262 cap covers 868-923 MHz only, not 433 MHz.
- **Roles for the stick.** Plus2: IR blaster, small status display with three buttons, BLE scanner/proxy (classic ESP32, 2 MB PSRAM), 433 MHz gateway through the CC1101 hat using ESPHome `cc1101` + `remote_receiver`/`remote_transmitter` or OpenMQTTGateway, PDM microphone for push-to-talk or sound level. StickS3: all of that plus a real codec/speaker and 8 MB PSRAM (better voice satellite) and an IR receiver (can learn remotes). The NRF24 half of a combo hat has little smart-home value (no common home devices use it in an open way). The 200-250 mAh battery makes either stick a tethered device for always-on roles.
- Both devices duplicate Wi-Fi/BLE roles; their unique value is the sub-GHz radios (LoRa on the Cardputer, CC1101 on the stick) and the built-in human interface (keyboard, display, microphone).

### Gaps
- The exact stick model and hat cannot be determined from the description; it needs a physical check (label, `esptool chip_id`, or the hat's markings).
- No source confirms that the third-party NRF24/CC1101 hats fit the StickS3's 16-pin HAT2 header (only a distributor note that the HAT bus is "basically compatible").
- No tested ESPHome YAML was found for Cardputer-Adv, Cap LoRa-1262, StickC Plus2 or StickS3; GPIO maps for the stick hats vary by seller.
- OpenMQTTGateway's documentation front page does not list M5Stack boards; whether a prebuilt OMG environment exists for these exact devices was not verified.
- Whether the ESP32-S3FN8 in the Cardputer-Adv can run ESPHome voice assistant with on-device wake word without PSRAM was not confirmed.
- M5StickC Plus (non-2) detailed specs were not fetched from a primary 2025-2026 source.

---

## 5. Sensors and modules for the required triggers (illuminance, presence, sound, climate/air, contact, power metering, IR, relays, audio output)

### Takeaway
Every sensor class the triggers need has a cheap module with native ESPHome support: BH1750 or VEML7700 for lux ($1-3), HLK-LD2410/LD2412 radar for presence ($3-8), an I2S MEMS microphone plus ESPHome's `sound_level` sensor for loudness, SHT4x/SCD4x for climate and CO2, and a 60 GHz MR60BHA2 kit (about $25) for bed-level breathing/heart-rate sensing. The weak spots are true sound classification (not available on-device in ESPHome) and independent accuracy data for sleep/wake radar.

### Cited Findings

#### Illuminance
| Sensor | Price (source, date) | ESPHome | Notes |
|---|---|---|---|
| BH1750 (GY-302) | $1-2 (checked July 2026) | Native | Good for automation, not photometry; one user reports the default high-resolution mode reading about 2x too high |
| VEML7700 | $2-3 (checked July 2026) | Native | About 0-120,000 lux; better low-light linearity and auto-gain; low-light readings can take several seconds |
| TSL2591 | $6.95 Adafruit (retrieved 2026-11-25, in stock) | Native | 188 µlux to 88,000 lux; gain must be raised for dim light; uncalibrated values are "in the ballpark" |
| LTR-303 | $4.50 Adafruit (retrieved 2026-11-25, in stock) | Not confirmed | 0.01 to 64k lux, separate IR channel, interrupt output |

- BH1750 and VEML7700 prices and native support — [atomic14 sensor index](https://www.atomic14.com/esp32/sensors/)
- BH1750 quality notes — [atomic14 BH1750](https://www.atomic14.com/esp32/sensors/bh1750/); [Home Assistant community](https://community.home-assistant.io/t/esphome-node-for-bh1750-problem-at-the-default-h-resolution-mode2/332385)
- VEML7700 range and behaviour — [ESPHome VEML7700](https://esphome.io/components/sensor/veml7700.html)
- TSL2591 — [Adafruit TSL2591](https://www.adafruit.com/product/1980); [ESPHome TSL2591](https://esphome.io/components/sensor/tsl2591.html)
- LTR-303 — [Adafruit LTR-303](https://www.adafruit.com/product/5610)

#### Presence and motion
| Module | Price (checked July 2026 unless noted) | ESPHome | Notes |
|---|---|---|---|
| HLK-LD2410 (B/C) | $3-6 | Native | Community default; 24 GHz, 8 m motion / 5 m static; B/C suited to ceiling mounting |
| HLK-LD2412 | $5-8 | Native | 9 m / 6 m; better with very still people and big rooms |
| HLK-LD2420 | $2-3 | Native | Cheapest; firmware batches vary (two baud rates) |
| HLK-LD2450 | $6-10 | Native | Tracks up to 3 targets with X/Y; weak on motionless people (degrades beyond about 2 m); pair with LD2410/LD2412 |
| HLK-LD2410S | Price not found | Native since ESPHome 2026.11.0 (community components before) | Low-power variant, 3-3.6 V; about 0.1 mA claimed; DIY battery nodes report 2-3 months per charge |
| DFRobot C4001 (SEN0609 / SEN0610) | $12-15 | Native since ESPHome 2026.11.0 | 25 m / 16 m (SEN0609), 12 m / 8 m (SEN0610); called "the sensor to beat" for ESPHome |
| Seeed MR24HPC1 | Price not found | Native (`seeed_mr24hpc1`) | 24 GHz static presence, 5 m / 4 m; a user reported a compile error after a recent ESPHome update |
| Seeed MR60BHA2 kit (with XIAO ESP32-C6) | $24.90 at launch (October 2024) **[PRE-2025]**; £25.60 incl. VAT at The Pi Hut (2026, "coming soon") | Native (`seeed_mr60bha2`), ships with ESPHome pre-flashed | 60 GHz; breathing and heartbeat to 1.5 m, static presence to 6 m; exposes has_target, breath_rate, heart_rate, distance, num_targets |
| PIR AM312 / HC-SR501 | $1-2 | GPIO binary sensor | Motion only; faster than radar, good to pair for instant lights-on |

- Prices and support for LD2410/LD2412/LD2420/LD2450, C4001 and PIR — [atomic14 sensor index](https://www.atomic14.com/esp32/sensors/); [atomic14 presence sensors](https://www.atomic14.com/esp32/sensors/presence-sensors/)
- Ranges and quality comments — [SmartHomeScene mmWave list](https://smarthomescene.com/blog/the-ultimate-list-of-mmwave-radar-sensor-modules/)
- LD2420 firmware/baud-rate caveat — [Home Assistant community](https://community.home-assistant.io/t/ld2410c-vs-2410b-vs-ld2410s-vs-2411-vs-hlk-ld2420-vs-ld2450/652599)
- ESPHome 2026.11.0 added HLK-LD2410S, HLK-LD2460 and DFRobot SEN0609/SEN0610 components and polygon zones for the LD2450 — [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- LD2410S power and battery reports — [ESPBoards LD2410S](https://www.espboards.dev/sensors/ld2410s/); [Home Assistant community](https://community.home-assistant.io/t/battery-powered-presence-sensor-with-ld2410s/756296)
- MR24HPC1 — [ESPHome MR24HPC1](https://esphome.io/components/seeed_mr24hpc1.html); [Home Assistant community](https://community.home-assistant.io/t/mr24hpc1-communication-issue-with-esphome/971765?page=2)
- MR60BHA2 — [ESPHome MR60BHA2](https://esphome.io/components/seeed_mr60bha2.html); [CNX Software](https://www.cnx-software.com/2024/10/21/seeed-studio-esp32-c6-60ghz-mmwave-human-fall-detection-and-breathing-heartbeat-detection-sensor-kits/); [Seeed wiki](https://wiki.seeedstudio.com/ha_with_mr60bha2/); [The Pi Hut](https://thepihut.com/products/60ghz-mmwave-breathing-and-heartbeat-detection-sensor-mr60bha2)
- Ready-made ESPHome presence devices for comparison: Apollo MSR-2 about $33, Everything Presence Lite about $39 and One about $65, Aqara FP2 about $85; cheap Tuya/Moes Zigbee presence pucks ($15-25) are on an "avoid" tier for false triggers — [atomic14 presence sensors](https://www.atomic14.com/esp32/sensors/presence-sensors/)

#### Sound (microphones and on-device level)
- ESPHome's `sound_level` sensor reports peak and RMS level over a measurement window (50 ms to 60 s, default 1000 ms) in relative dB where 0 dB is the loudest the microphone can measure; a passive mode measures only while another component (e.g. voice assistant) is capturing; audio components use significant RAM/CPU and BLE components are known to conflict with voice/audio components — [ESPHome sound_level](https://esphome.io/components/sensor/sound_level.html)
- INMP441 is the most commonly used and best-documented I2S microphone on ESP32 and won a side-by-side test against the SPH0645; the SPH0645 has I2S timing quirks on ESP32 that need sample shifting; the ICS-43434 is described as a good replacement for the INMP441 (which of these guides made which statement was not individually verified) — [atomic14 video](https://atomic14.com/videos/posts/3g7l5bm7fZ8); [WLED microphone hookup guide](https://mm.kno.wled.ge/WLEDSR/Digital-Microphone-Hookup); [Applied EE MEMS microphone guide](https://applied-ee.github.io/embedded/docs/sensor-integration/audio-and-acoustic/mems-microphone-interfacing/)
- SNR figures conflict across sources (INMP441 61-65 dB; ICS-43434 65-70 dB) — [Applied EE MEMS microphone guide](https://applied-ee.github.io/embedded/docs/sensor-integration/audio-and-acoustic/mems-microphone-interfacing/); contradicted in part by [Adafruit ICS-43434](https://www.adafruit.com/product/6049), which lists 65 dBA
- Prices: INMP441 breakouts $15.99 per 3 or $23.99 per 5 (about $5 each, Newegg bundles, undated); ICS-43434 breakout $8.95 at Adafruit (retrieved 2026-11-25), with a note that the ICS-43434 has been discontinued and the SPH0645LM4H is a drop-in replacement; SPH0645 breakout $6.95 at Adafruit, out of stock — [Newegg INMP441](https://www.newegg.com/aitrip-inmp441/p/0WF-00AB-00002); [Adafruit ICS-43434](https://www.adafruit.com/product/6049); [Adafruit SPH0645](https://www.adafruit.com/product/3421)

#### Climate and air
| Sensor | Price (checked July 2026) | ESPHome |
|---|---|---|
| SHT40 / SHT41 / SHT45 | $1-3 | Native |
| SHT31 | $1-3 | Native |
| AHT20 / AHT30 | $1-2 | Native (aht10) |
| BME280 | $2-4 | Native |
| BME680 / BME688 | $4-9 | Native (bme68x_bsec2) |
| SCD40 | $8-13 | Native (scd4x) |
| SCD41 | $9-16 | Native (scd4x) |

- All rows — [atomic14 sensor index](https://www.atomic14.com/esp32/sensors/)

#### Power metering
- PZEM-004T v3 $7-11 (native `pzemac`), INA219 $1-2 and INA226 $2-4 (native, DC) — [atomic14 sensor index](https://www.atomic14.com/esp32/sensors/)
- ESPHome's `hlw8012` platform covers HLW8012, BL0937 and CSE7759 (BL0937 needs its own model setting and usually an inverted SEL pin); `cse7766` is a UART chip used in the Sonoff POW R2; `bl0942` is a UART chip found in some Tuya devices — [ESPHome HLW8012](https://esphome.io/components/sensor/hlw8012.html); [ESPHome CSE7766](https://esphome.io/components/sensor/cse7766.html); [ESPHome BL0942](https://esphome.io/components/sensor/bl0942.html); [esp32.co.uk HLW8012 guide](https://esp32.co.uk/hlw8012-esphome-voltage-current-power-monitoring/)

#### IR transmit and receive
- ESPHome's IR/RF proxy exposes an `infrared` entity bound to a `remote_transmitter` (and a receiver entry for `remote_receiver`); Home Assistant's native infrared platform shipped transmit in 2026.4 and receive in 2026.6 — [ESPHome IR/RF Proxy](https://esphome.io/components/ir_rf_proxy/); [Home Assistant developer blog](https://developers.home-assistant.io/blog/2026/03/30/infrared-entity-platform)
- A November 2026 guide reports the chain working on Home Assistant 2026.11.1 with ESPHome 2026.10.2 and names the Seeed XIAO IR Mate as a common ESPHome IR proxy — [ComputingForGeeks](https://computingforgeeks.com/?p=187940)
- Built-in IR hardware on M5Stack boards: Atom S3 Lite IR LED on GPIO4; Atom VoiceS3R has an IR transmitter; StickS3 has transmitter and receiver — [ESPHome devices: Atom S3 Lite](https://devices.esphome.io/devices/m5stack-atoms3-lite/); [M5Stack Atom EchoS3R docs](https://docs.m5stack.com/en/core/Atom_EchoS3R); [CNX Software](https://www.cnx-software.com/2026/01/23/m5stack-sticks3-esp32-s3-mini-iot-dev-kit-features-1-14-inch-color-display-speaker-and-mic-for-voice-interaction/)

#### Relays
- Waveshare ESP32-S3-Relay-6CH: $28.99, relays rated up to 10 A 250 VAC or 10 A 30 VDC, stated to control 220 VAC appliances directly — [Waveshare](https://www.waveshare.com/product/modules/esp32-s3-relay-6ch.htm)
- LilyGO T-Relay-8 relays are rated up to 250 VAC/10 A **[PRE-2025]** — [CNX Software](https://www.cnx-software.com/2022/06/11/t-relay-8-an-esp32-board-with-8-relays/)

#### Small I2S amplifiers
- Adafruit MAX98357A I2S class-D amplifier breakout: $5.95 (retrieved 2026-11-25, in stock), 3.2 W into 4 ohm or 1.8 W into 8 ohm at 5 V, no MCLK needed, 2.7-5.5 V supply — [Adafruit MAX98357A](https://www.adafruit.com/product/3006)
- Integrated alternative: M5Stack Atom VoiceS3R combines ES8311 codec, MEMS microphone and NS4150B amplifier for $14.50 — [Switch Science](https://www.switch-science.com/products/10808)

### Inferences
- **Wake-up trigger design.** "Time of day + illuminance + sound of people waking up" maps to: a lux sensor (BH1750/VEML7700) per room, a bedroom radar that distinguishes still from moving targets (LD2410/LD2412 expose moving and still energy; the MR60BHA2 adds breathing/heart rate at bed range), and an I2S microphone publishing `sound_level` RMS/peak. Recognising the kind of sound (alarm, voices, footsteps) is beyond ESPHome's on-device features; it would have to run on the always-on home computer from streamed audio, or be approximated by level/duration thresholds.
- `sound_level` values are relative dBFS, not calibrated dB SPL, so thresholds must be tuned per microphone and enclosure.
- For battery nodes only the LD2410S and PIR are plausible presence sensors; all other radars draw tens of mA and belong on mains nodes.
- Given the ICS-43434's discontinuation note and the SPH0645's ESP32 quirks, INMP441 (or a codec board such as the Atom VoiceS3R) is the pragmatic microphone choice.
- For sockets and wall switches, buying certified devices (Zigbee plugs, or ESPHome/Tasmota-flashable Wi-Fi plugs with BL0942/CSE7766 metering) is safer than building mains relay nodes; DIY relay boards fit low-voltage loads and enclosures with proper isolation.

### Gaps
- No ESPHome source was found for LTR-303 (`ltr_als_ps`) support, and no price/quality source for ATM90E32-based energy monitors or for pre-flashed ESPHome smart plugs (e.g. Athom) in 2026.
- No prices with sources for reed switches, IR LEDs/receivers (TSOP38238 class), bare relay modules, solid-state relays or small speakers.
- No independent 2025-2026 accuracy evaluation of MR60BHA2 heart-rate/breathing or sleep-stage detection was found; the ESPHome page's note that breath and heart rate are measured "only during the first 60 seconds" is ambiguous and should be checked against Seeed's documentation.
- No price found for MR24HPC1 or LD2410S; the atomic14 price ranges are community street prices, not vendor quotes.
- Curtain-motor hardware was out of scope for this question set; only the radio choice (Zigbee or 433 MHz) is addressed in Question 1.

---

## 6. Battery-powered sensors: buy versus build

### Takeaway
Buying wins on battery life and cost: commercial Zigbee, BLE and Thread sensors cost roughly $5-20 and run 1-5 years on a coin cell, while documented DIY ESP32-C6/H2 Zigbee or Thread builds drain in about 2 months unless the board is stripped of LEDs, regulator and charger. Build only for sensors that cannot be bought (custom combinations, ESP-NOW nodes, LD2410S presence), preferably on nRF52840 or a low-quiescent ESP32 board.

### Cited Findings

#### Bought options
- SONOFF SNZB-02P (Zigbee temperature/humidity): about 4 years on a CR2477 by vendor lab data; listed at $16.90 excl. tax on SONOFF's store (undated page); works with Zigbee2MQTT — [SONOFF SNZB-02P](https://sonoff.tech/en-us/products/sonoff-zigbee-temperature-and-humidity-sensor-snzb-02p)
- SONOFF SNZB-03P (PIR) is marketed with a 3-year battery life; SNZB-04P (door/window) exposes tamper and low-battery status in Zigbee2MQTT; SNZB-06P (mmWave presence) is USB-powered and acts as a Zigbee router — [MicrocontrollersLab buying guide](https://microcontrollerslab.com/best-sonoff-zigbee-devices-14-dongles-bridges-sensors-relays-and-buttons-compared-buying-guide/); [CNX Software review](https://www.cnx-software.com/2024/04/29/review-sonoff-snzb-06p-zigbee-human-presence-snzb-04pdoor-window-sensor/)
- An Aqara Zigbee temperature sensor (about $13) lasted about 3 years before the battery died, per a user comparing it with a DIY ESP32-H2 build — [Home Assistant community thread](https://community.home-assistant.io/t/esp32-h2-thread-with-deep-sleep-for-cheap-esp32-battery-sensors/919132)
- IKEA Matter-over-Thread sensors (2025-2026): TIMMERFLOTTE temperature/humidity €6 (Ireland), MYGGSPRAY motion €8.99 (Netherlands) — [IKEA Ireland](https://www.ikea.com/ie/en/p/timmerflotte-temperature-humidity-sensor-smart-60618952); [IKEA Netherlands](https://www.ikea.com/nl/en/p/myggspray-wireless-motion-sensor-smart-70604186/)
- Swedish launch prices: TIMMERFLOTTE 79 kr, KLIPPBOK 79 kr, MYGGSPRAY 99 kr — [Home Assistant community](https://community.home-assistant.io/t/ikea-just-announced-21-new-affordable-matter-over-thread-smart-home-devices/948187?page=3)
- IKEA Thread reliability is mixed: AppleInsider (May 2026) cites lingering connection issues; some Home Assistant users report MYGGSPRAY working first time, others that it must be registered on a DIRIGERA hub first or does not appear; one reviewer called TIMMERFLOTTE humidity readings poor — [AppleInsider](https://appleinsider.com/articles/26/05/15/ikea-matter-over-thread-review-amazing-smart-home-tech-when-they-work); [Home Assistant community](https://community.home-assistant.io/t/ikea-myggspray-matter-over-thread-new-motion-sensor/958847)
- Shelly BLU (BTHome): Door/Window $19.99 (US store), up to 5 years on CR2032; H&T $20.99 (US store, out of stock), up to 3 years; Shelly's app needs a gateway (BLU Gateway or any Shelly Plus/Pro/Gen3 device), while BTHome receivers can read them directly; "ZB" Bluetooth + Zigbee variants also exist — [Shelly BLU Door/Window](https://us.shelly.com/products/shelly-blu-door-window-us-list); [Shelly BLU H&T](https://us.shelly.com/products/shelly-blu-h-t-us-list); [Shelly EU H&T](https://www.shelly.com/products/shelly-blu-h-t-1); [Shelly BLU Door/Window ZB](https://us.shelly.com/products/shelly-blu-door-window-zb-black)
- Xiaomi LYWSD03MMC with pvvx firmware: BTHome v2 output; project claims over a year on a CR2032, another project figure is 14-20 µA or over 6 months at default settings; a user running 5 s advertising with two-minute updates projects 18+ months; battery percentage can be misleading (stuck at 100 % while voltage drops) — [pvvx ATC_MiThermometer](https://github.com/pvvx/ATC_MiThermometer); [Home Assistant community](https://community.home-assistant.io/t/xiaomi-temperature-humidity-sensor-home-assistant-integration-pvvx-custom-firmware-may-2023/572569)

#### DIY options
- ESPHome Zigbee supports sleepy end devices on ESP32-C5/C6/H2 (since 2026.11.0) and on nRF52 (up to 8 endpoints, one per entity) — [ESPHome Zigbee](https://esphome.io/components/zigbee.html); [ESPHome 2026.11.0](https://esphome.io/changelog/2026.11.0.html)
- A Beetle ESP32-C6 Zigbee door sensor (Arduino Zigbee) was on course to drain in about two months against an expected year or more — [DFRobot forum](https://www.dfrobot.com/forum/topic/401992)
- A forum user measured 54 µA sleeping on an ESP32-C6 Zigbee device — [ESP32 forum](https://esp32.com/viewtopic.php?p=151967)
- ESP32-H2 Thread deep-sleep build: about $15 in parts, 3300 mAh 18650 drained in about 2 months; an unmodified H2 SuperMini sleeps at 488 µA and reaches about 17 µA only after removing LED, LDO, charger IC and charger LED; projections of 7 months (2500 mAh at 500 µA) to over 2 years (stripped board, 500 mAh LiFePO4) are estimates, not measurements — [Home Assistant community thread](https://community.home-assistant.io/t/esp32-h2-thread-with-deep-sleep-for-cheap-esp32-battery-sensors/919132)
- A June 2026 thread asking for measured ESP32-H2 Zigbee coin-cell battery life received no data, only the suggestion that nRF52840 has far lower consumption — [Home Assistant community](https://community.home-assistant.io/t/esp32-h2-zigbee-battery-life/1015032)
- A Rust Zigbee stack's documentation estimates 4+ years on a CR2032 for a Telink TLSR8258 (the chip class used in commercial sensors), not for an ESP32 — [zigbee-rs power management](https://faronov.github.io/zigbee-rs/advanced/power.html)
- ESP-NOW deep-sleep nodes: about 200 ms per wake/send cycle and an estimated 5+ years at 15-minute intervals on a low-leakage board (25 µA deep sleep) — [ThingPulse ePulse Feather](https://thingpulse.com/epulse-feather-espnow-a-dream-team/)
- DIY LD2410S presence nodes report 2-3 months per charge; one builder cut consumption further by broadcasting over BLE instead of Wi-Fi — [Home Assistant community](https://community.home-assistant.io/t/battery-powered-presence-sensor-with-ld2410s/756296)
- b-parasite is an open-hardware BTHome soil/temperature/humidity/light sensor; BTHome DIY libraries exist for ESP32 and nRF52 — [BTHome](https://bthome.io/)

### Inferences
- Cost per sensor: bought Zigbee/BLE/Thread sensors are about $5-20 and need no enclosure work; a DIY node is $10-15 in parts (MCU board, sensor, cell, holder) plus an enclosure, and matches commercial battery life only with a purpose-chosen low-leakage board. Build-versus-buy therefore favours buying for temperature/humidity, door/window, motion, leak and buttons.
- The dominant DIY pitfalls are board leakage (LEDs, AMS1117-class regulators, charger ICs), long wake/rejoin times (6-8 s for Thread in ESPHome versus 200 ms for ESP-NOW) and immature deep-sleep/SED support on the newest ESP32 parts; the ESPHome Zigbee SED feature on ESP32 is weeks old as of November 2026 and has no published battery measurements.
- If DIY battery sensors are wanted, the order of preference suggested by the evidence is: nRF52840 with ESPHome Zigbee or BTHome; ESP32-C3/C6 with ESP-NOW and deep sleep on a XIAO-class board; ESP32-H2 Zigbee SED last, after measuring with a power profiler.
- The cheapest robust whole-home temperature/humidity layer is LYWSD03MMC units with pvvx/BTHome firmware received by the Wi-Fi nodes that exist anyway; Zigbee SNZB/Aqara parts are the low-effort alternative that also strengthens nothing (battery devices do not route) but needs no flashing.

### Gaps
- No measured battery life for an ESPHome Zigbee sleepy end device on ESP32-C6/H2 or nRF52840 was found.
- Current 2026 prices were not found for LYWSD03MMC, SONOFF SNZB-03P/04P, Aqara sensors (only one anecdotal $13), IKEA MYGGBETT (door/window) in any market, or US prices for the IKEA range; the SNZB-02P store price is undated.
- The LYWSD03MMC battery-life claims (over a year versus about 6 months at defaults) conflict and depend on advertising interval; no independent measurement was found.
- XIAO nRF52840 price and its measured sleep current under ESPHome were not found.

---

## 7. Zigbee coordinators for Zigbee2MQTT in late 2026

### Takeaway
All the named coordinators work with Zigbee2MQTT; the practical choice is between a cheap USB stick (SONOFF ZBDongle-E about $21-25, Dongle Plus MG24 about $30-36, Home Assistant ZBT-2 $49) and an Ethernet/PoE unit (SONOFF Dongle Max about $35-43, SMLIGHT SLZB-06 family about $73-85 on Amazon US). For a host that is a laptop or a Mac, an Ethernet-attached coordinator reached over `tcp://` is the better fit, and Zigbee2MQTT explicitly prefers Ethernet over Wi-Fi for network adapters.

### Cited Findings

#### Coordinator table
Prices in the "Price" column are from an aggregator read on 2026-11-25 during Black Friday sales unless another source is given; "was" = pre-sale price.

| Coordinator | Radio chip | Connection | Price | Zigbee2MQTT | Thread capability |
|---|---|---|---|---|---|
| SONOFF ZBDongle-E | EFR32MG21 | USB | $20.90 (was $24.90); another site lists $26.99 (June 2026) | Listed (ember; needs NCP 7.4.4+, software flow control) | Zigbee or Thread (OTBR) firmware |
| SONOFF ZBDongle-P | CC2652P | USB | $29.90 (was $39.99) | Listed (zstack) | Not stated |
| SONOFF Dongle Plus MG24 (Dongle-PMG24) | EFR32MG24 | USB | $29.90 (was $35.90) | Listed (ember) | Zigbee or Thread; MultiPAN experimental |
| SONOFF Dongle Lite MG21 | EFR32MG21 | USB | $18.68 (was $24.90) | Not on the Z2M page under this name | Not stated |
| SONOFF Dongle Plus CC2674P10 (Dongle-PP10) | CC2674P10 | USB | $28.72 (was $35.90) | Listed (zstack) | Not stated |
| SONOFF Dongle Max (Dongle-M) | EFR32MG24 + ESP32-D0WD-R2 | Ethernet/PoE, Wi-Fi, USB | $34.90 (was $42.90); $42.90 at launch (Sept 2025); €38.49-€47.90 in EU shops | Listed (ember); built-in web console | Zigbee or Thread; MultiPAN in beta |
| SONOFF Dongle Max CC2674P10 (Dongle-MP10) | CC2674P10 | Ethernet, Wi-Fi, USB, PoE | $48.90 | Listed (zstack) | Not stated |
| Home Assistant Connect ZBT-2 | EFR32MG24 | USB | $49 at launch (Nov 2025); $48.95 at CloudFree; £38 at The Pi Hut (sold out) | Listed (ember; 460800 baud, rtscts) | Zigbee or Thread, one at a time |
| Home Assistant Connect ZBT-1 | EFR32MG21 | USB | Not found | Listed (ember) | Not stated in sources read |
| SMLIGHT SLZB-06 | CC2652P | Ethernet, PoE, USB, Wi-Fi | $72.99 | Listed (zstack) | Experimental Matter-over-Thread |
| SMLIGHT SLZB-06M | EFR32MG21 | Ethernet, PoE, USB, Wi-Fi | $72.99 | Listed (ember; "not suitable for Wi-Fi or high-latency links") | Yes (multiprotocol-capable chip) |
| SMLIGHT SLZB-06p7 | CC2652P7 | Ethernet, PoE, USB, Wi-Fi | $79.99 | Listed (zstack) | Border-router capable per aggregator |
| SMLIGHT SLZB-06P10 | CC2674P10 | Ethernet, PoE, USB, Wi-Fi | $84.99 | Not on the Z2M zStack page when read; vendor lists it as best for Zigbee2MQTT | Not stated |
| SMLIGHT SLZB-06Mg24 (and 06Mg24U with ESP32-S3 host) | EFR32MG24 | Ethernet (W5500), PoE 802.3af, USB-C, Wi-Fi fallback | $83.99 | Listed (ember) | Thread 1.3 / Matter-over-Thread border router via OTBR |
| SMLIGHT SLZB-06Mg26 | EFR32MG26 | Ethernet, PoE, USB, Wi-Fi | Not found (06Mg26U pre-order shipping April 2026) | Listed (ember) | Not stated |
| SMLIGHT SLZB-07 / 07p7 / 07Mg24 | EFR32MG21 / CC2652P7 / EFR32MG24 | USB | $34.99 / $36.99 / not found | Listed | 07Mg24: Zigbee or Thread |
| SMLIGHT SLZB-MR1 | EFR32MG21 + CC2652P7 (+ ESP32) | Ethernet, PoE, USB, Wi-Fi | $101.99 | Listed | Zigbee and Thread simultaneously on separate radios |
| SMLIGHT SLZB-MR2 | EFR32MG21 + CC2652P | Same | $102.99 (one aggregator page) | Listed | Dual radio |
| SMLIGHT SLZB-MR3 | EFR32MG24 + CC2674P10 | Same | £197.72 on the aggregator; €158.13 elsewhere | Listed | Dual radio |
| SMLIGHT SLZB-MR4 | EFR32MG26 + CC2674P10 | Same | $130.00 | Listed | Dual radio |
| SMLIGHT SLZB-MR5U | 2x EFR32MG24 | USB, LAN, PoE, Wi-Fi | €59.95 at one EU retailer (ambiguous listing) | Reviewed as supported | Zigbee and Thread simultaneously, one radio each |
| dresden elektronik ConBee III | EFR32MG21 | USB | €39.95 at launch **[PRE-2025]** | deCONZ driver family (recommended family); no coordinator backup | Thread firmware announced as tech preview at launch |

- Aggregator prices (ZBDongle-E/P, Dongle Plus MG24, Dongle Lite, Dongle Max, PP10, MP10, SLZB-06 family, SLZB-07, MR1, MR3, MR4) — [SmartHomeCompared dongles](https://smarthomecompared.com/dongles)
- ZBDongle-E $26.99 (June 2026) — [SmartHomeCompared ZBDongle-E vs ZBDongle-P](https://smarthomecompared.com/dongles/sonoff-zbdongle-e-vs-sonoff-zbdongle-p)
- EmberZNet adapter list, chips, serial settings and notes (ZBT-2, Dongle-PMG24, ZBDongle-E, ZBT-1, Dongle-M, SLZB-06M/06Mg24/06Mg26, SLZB-07, MR series, Ultima, SMHUB) — [Zigbee2MQTT EmberZNet adapters](https://www.zigbee2mqtt.io/guide/adapters/emberznet.html)
- zStack adapter list (ZBDongle-P, Dongle-PP10, Dongle-MP10, SLZB-06, SLZB-06p7, SLZB-07p7, MR1-MR3, Ultima) — [Zigbee2MQTT zStack adapters](https://www.zigbee2mqtt.io/guide/adapters/zstack.html)
- Thread/ZHA/Z2M/OTBR capability, TX power and verdicts ("Best USB-only" ZBT-2, "Best high-power USB" Dongle Plus MG24, "Best budget USB" ZBDongle-E, "Best overall PoE" SLZB-06MG24, "Best high-power PoE" Dongle Max, "Best dual-radio" SLZB-MR5U), article updated 5 February 2026 — [SmartHomeScene top picks](https://smarthomescene.com/top-picks/best-zigbee-coordinators-for-home-assistant/)
- SONOFF Dongle Max launch price and hardware — [CNX Software](https://www.cnx-software.com/2025/09/17/sonoff-dongle-max-dongle-m-poe-based-zigbee-thread-gateway-esp32-mg24/)
- Dongle Max tested with ZHA and Zigbee2MQTT (January 2026); a UK review calls performance outstanding for the price but notes imperfect setup and beta MultiPAN — [CNX Software review](https://www.cnx-software.com/2026/01/27/sonoff-dongle-max-dongle-m-review-a-zigbee-thread-poe-usb-and-wifi-adapter-tested-with-home-assistant/); [Mighty Gadget](https://mightygadget.co.uk/?p=709815)
- ZBT-2: MG24 chip, launch price $49, cannot run Zigbee and Thread simultaneously, used by many with Zigbee2MQTT — [CNX Software](https://www.cnx-software.com/2025/11/20/home-assistant-connect-zbt-2-zigbee-thread-usb-adapter/); [Home Assistant ZBT-2](https://www.home-assistant.io/connect/zbt-2); [CloudFree](https://cloudfree.shop/products/home-assistant-connect-zbt-2); [The Pi Hut](https://thepihut.com/products/home-assistant-connect-zbt-2)
- SMLIGHT model matrix (chips, max end devices 200-400 for the 06 series and 400-750 for MR1-MR3, vendor "best for" Zigbee2MQTT on TI variants and ZHA on Silicon Labs variants, SLZB-OS features: web UI, OTA, WireGuard, Ethernet-to-Wi-Fi bridge) — [SMLIGHT models overview](https://smlight.tech/manual/slzb-06/guide/slzb-models-overview/)
- SLZB-06Mg24U details (ESP32-S3 host, W5500 Ethernet, PoE 802.3af, Thread 1.3 border router, one-click ESPHome-based Bluetooth proxy firmware, WireGuard) — [SMLIGHT SLZB-06Mg24U](https://smlight.tech/en-US/products/slzb-06mg24u)
- SMLIGHT MR prices from other aggregator pages (MR2 $102.99, MR3 €158.13) and the MR5U €59.95 listing are inconsistent in currency and date — [SmartHomeCompared MR4](https://smarthomecompared.com/dongles/smlight-slzb-mr4); [LOH Electronics comparison](https://www.lohelectronics.se/en-gb/mega_blog/blog/view/smlight-slzb-comparison)
- ConBee III: EFR32MG21, deCONZ firmware with its own serial protocol, €39.95 at launch, Thread border-router firmware announced as a preview; a community thread reported that migrating between ConBee adapters inside Zigbee2MQTT was not possible **[PRE-2025]** — [SmartHomeScene news](https://smarthomescene.com/news/conbee-iii-officially-released-with-zigbee-upcoming-thread-support/); [Home Assistant community](https://community.home-assistant.io/t/conbee-iii-conbee-3-from-dresden-elektronik-based-on-silicon-labs-efr32mg21-20dbm-radio-soc-mcu/649835)

#### Guidance from Zigbee2MQTT and reviewers
- "Network adapters connected via WiFi might have reduced stability" because the serial protocol lacks tolerance for packet loss and latency; a wired Ethernet remote adapter is recommended instead — [Zigbee2MQTT adapters](https://www.zigbee2mqtt.io/guide/adapters/)
- Only zStack and EmberZNet adapters support coordinator backup; switching adapters may require re-pairing all devices — [Zigbee2MQTT adapters](https://www.zigbee2mqtt.io/guide/adapters/)
- EmberZNet firmware 7.4.x through 9.1.x is supported; `adapter: ezsp` is deprecated in favour of `adapter: ember`; multiprotocol firmware is not supported; network adapters are configured as `port: tcp://host:6638` — [Zigbee2MQTT EmberZNet adapters](https://www.zigbee2mqtt.io/guide/adapters/emberznet.html)
- CC2531/CC2530/CC2538 adapters are not recommended and may not handle networks of 20+ devices — [Zigbee2MQTT zStack adapters](https://www.zigbee2mqtt.io/guide/adapters/zstack.html)
- USB sticks are the easiest option but tie the radio to the host and should use a USB extension cable to reduce interference; PoE/LAN coordinators can be placed centrally regardless of where the server sits — [SmartHomeScene top picks](https://smarthomescene.com/top-picks/best-zigbee-coordinators-for-home-assistant/)
- With a network coordinator, Docker device mappings are not needed; Zigbee2MQTT just points at the TCP address — [Zigbee2MQTT Docker docs](https://zigbee2mqtt.io/guide/installation/02_docker.html)
- With Docker Desktop, containers do not run directly on the host kernel (they run in a VM), which is the root of USB serial passthrough trouble; workarounds bridge a serial device over TCP with ser2net/socat — [Level1Techs forum](https://forum.level1techs.com/t/fedora-44-cant-hook-up-to-a-device/253655); [zigbee2mqtt-socat image](https://hub.docker.com/r/mc303/zigbee2mqtt-socat)
- SMLIGHT also sells SMHUB devices that run Zigbee2MQTT (with Mosquitto, Node-RED and Matterbridge) on the unit itself — [Zigbee2MQTT zStack adapters](https://www.zigbee2mqtt.io/guide/adapters/zstack.html)

### Inferences
- **Is a network-attached coordinator preferable for a laptop or Mac host? Yes.** A laptop moves, sleeps and is rarely in the best radio position; on macOS, containerised Zigbee2MQTT cannot reliably receive a USB serial device because Docker Desktop runs containers in a VM (the cited thread documents this for Docker Desktop on Linux; the same architecture applies on macOS, which is my extension, not a sourced statement). An Ethernet/PoE coordinator decouples the Zigbee network from the host, lets Zigbee2MQTT move between machines by changing one `tcp://` line, and allows central placement. Use Ethernet, not the unit's Wi-Fi.
- **Shortlist.** (1) Best value network coordinator: SONOFF Dongle Max (EFR32MG24, Ethernet/PoE, about $35-43). (2) More mature network line with richer firmware: SMLIGHT SLZB-06Mg24/06Mg24U (EmberZNet) or SLZB-06P10 (TI, vendor-recommended for Zigbee2MQTT), about $80-85 at Amazon US pricing. (3) If Thread is wanted alongside Zigbee: a dual-radio SLZB-MR unit (MR1 about $100; MR5U with two MG24 radios) or a second cheap stick as Thread RCP. (4) If the host becomes a fixed Linux box: ZBDongle-E ($21-25) or Dongle Plus MG24 ($30-36) on a USB extension cable; ZBT-2 ($49) is the polished option. (5) ConBee III is a weaker fit for Zigbee2MQTT (no coordinator backup, older price data).
- SMLIGHT aggregator prices appear to be Amazon US retail and are well above what the same units cost from the vendor or EU retailers; treat them as upper bounds.
- An ESP32-C6/H2 can act as a Zigbee coordinator only through the experimental ZBOSS driver, which Zigbee2MQTT does not recommend; it is not a substitute for the adapters above.

### Gaps
- Official list prices from sonoff.tech and smlight.tech could not be read (HTTP 429 / no price on page); all prices are from aggregators, launch articles or resellers, and several are Black Friday sale prices.
- ZBT-1 price and current availability were not found; ConBee III 2025-2026 price and Zigbee2MQTT status were not confirmed with a recent source.
- SLZB-MR5U specifications and price rest on one review and one ambiguous retailer listing; SLZB-MR2/MR3/MR4 prices are inconsistent across pages and currencies.
- The Zigbee2MQTT pages do not label individual adapters "recommended"; "listed" above means the adapter appears on the driver page without a "not recommended" flag.
- No macOS-specific primary source on Docker Desktop USB serial passthrough was found; the conclusion rests on the Docker Desktop VM architecture.
