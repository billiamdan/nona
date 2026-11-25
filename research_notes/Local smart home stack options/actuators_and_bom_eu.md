# Off-the-shelf mains devices, curtain/blind motors and a phased starter BOM for a local smart home in the EU (230 V), as of November 2026

Method note (read first): all pages were fetched on 2026-11-25. The web-search budget of the session was exhausted before the first query, so the first pass of this file is built from direct fetches of primary pages (manufacturer knowledge bases and stores, zigbee2mqtt.io, esphome.io, devices.esphome.io, templates.blakadder.com, GitHub READMEs, IKEA Germany, European Commission pages). Several Shopify-based stores (shelly.com, sonoff.tech, eu.aqara.com, eu.switch-bot.com, innr.com, m5stack) rate-limited the fetcher, so price coverage is partial. A second pass (after an interruption) again had no web search available; it added EUR shop prices from one EU maker shop that could be fetched (hobbyelectronica.nl, Netherlands; consumer webshop prices, VAT-inclusive pricing is assumed but not stated on the pages) and rebuilt the bill of materials around the EUR 100 first-round limit. Anything marked "ESTIMATE" is from the researcher's background knowledge (pre-mid-2026) and was NOT confirmed with a 2025-2026 source in this session. Scope decisions from the coordinator: rented apartment, wiring may not be modified; lights are ceiling fixtures with replaceable bulbs, floor/table lamps on sockets and LED strips (no built-in-LED fixtures); curtains hang on an existing track (rail); first purchase round must cost at most EUR 100.

## 1. Smart plugs with energy metering that work with no cloud and no account (EU plug types)

### Takeaway
Three families work fully locally with an MQTT hub in November 2026: Shelly Gen3 plugs on stock firmware (local MQTT/RPC, no flashing needed), plugs sold pre-flashed with ESPHome or Tasmota (Athom, NOUS A1T/A8T, Refoss P11), and Zigbee plugs through Zigbee2MQTT (Sonoff S60ZBTPF, NOUS A1Z, Aqara SP-EUC01, generic Tuya TS011F with a polling caveat). Flashing stock firmware off a device after purchase has become the hard path (Shelly Plug S Gen3: serial only after destructive opening; Tuya: OTA exploit patched since 2022), and IKEA's current plug (GRILLPLATS) is Matter-over-Thread, which does not fit a Zigbee2MQTT/ESPHome stack without an extra Thread border router and Matter controller.

### Cited Findings

Shelly (stock firmware, local MQTT/RPC):
- Shelly's knowledge base lists these Gen3 plugs: Plug M Gen3, Plug S MTR Gen3, Plug PM Gen3, Outdoor Plug S Gen3. The only Gen4 plug listed is "Plug US Gen4" (US type), plus the "Power Strip 4 Gen4" — i.e. no EU single plug in Gen4 as of the fetch date — [Shelly KB device index](https://kb.shelly.cloud/knowledge-base/devices)
- Shelly Plug S MTR Gen3 (model S3PL-00112EU; the KB page gives the device name as "Shelly Plug S Gen3"): max 12 A / 2500 W "resistive load only", power consumption "< 1 W", CEE 7/7 plug, Schuko (Type F) output socket, listed as compatible with CEE 7/3 (Type F) and CEE 7/5 (Type E) wall sockets, MCU "ESP-Shelly-C38F" with 8 MB flash, Wi-Fi 802.11 b/g/n, Bluetooth 4.2, Matter supported, MQTT yes, scripting "only supported when Matter is not enabled", voltmeter/ammeter/power and energy meters yes (accuracy not stated), protections: overheating, overvoltage, overcurrent, overpower; 44x44x70 mm; EU declaration of conformity dated 2025-12-05 — [Shelly KB: Plug S MTR Gen3](https://kb.shelly.cloud/knowledge-base/shelly-plug-s-mtr-gen3)
- Price: "Shelly Plug S MTR Gen3 White" 9.56 EUR, compare-at (list) price 23.90 EUR, store JSON fetched 2026-11-25 (Black-Friday week); product text says "With local scripting support, automations continue operating reliably even without cloud connectivity" and lists MQTT, REST API and webhooks — [shelly.com product JSON](https://www.shelly.com/products/shelly-plug-s-gen3.json)
- Shelly Plug PM Gen3 (S3PL-30116EU): 16 A / 3680 W (resistive loads only), "under 1 W" own consumption, Schuko output, same 44x44x70 mm housing, ESP-Shelly-C38F, Wi-Fi b/g/n + Bluetooth 4.2, voltage/current/power/energy metering, MQTT and scripting yes; Matter not mentioned on its page — [Shelly KB: Plug PM Gen3](https://kb.shelly.cloud/knowledge-base/shelly-plug-pm-gen3)
- Shelly Power Strip 4 Gen4: four Schuko sockets, each individually switchable and metered ("Power metering on every plug"), 16 A total / 12 A per socket / 3680 W, MCU ESP-Shelly-C68F, Wi-Fi 802.11 b/g/n/ax, Bluetooth 5.0, Zigbee (with repeater support), Matter, "< 1 W", MQTT yes — [Shelly KB: Power Strip 4 Gen4](https://kb.shelly.cloud/knowledge-base/shelly-power-strip-gen4)
- Shelly Gen2+ MQTT: outbound broker connection configured locally via RPC (`MQTT.SetConfig`); RPC requests on `<shelly-id>/rpc`, notifications on `<topic_prefix>/events/rpc`, status on `<topic_prefix>/status/<component:id>`, control on `<topic_prefix>/command/<component:id>`, online/LWT on `<id>/online`, QoS 1, optional TLS — [Shelly API docs: MQTT](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Mqtt)
- Flashing Shelly Plug S Gen3 to ESPHome: chip is ESP32-C3 with a BL0942 metering IC; "At this moment there is NO way to Flash it OTA. You need to open the Plug and use UART!"; opening is destructive (tamper seal drilled out, hot air ~300 °C, soldering difficulty rated 4/5) — [ESPHome devices: Shelly Plug S Gen3](https://devices.esphome.io/devices/Shelly-Plug-S-Gen3)
- Flashing Shelly to Tasmota over the air: the converter project lists "PlugSG3 (no Tasmota support)", says "All Matter devices are impossible to convert", does not support Shelly firmware 1.5.x or later, warns "There is no way back to Shelly firmware", and was archived on 6 Aug 2025 with development stopped "until known, if new available devices will be flashable" — [tasmota/mgos32-to-tasmota32](https://github.com/tasmota/mgos32-to-tasmota32)

Pre-flashed open-firmware Wi-Fi plugs:
- Athom "EU Plug Made For ESPHome": ESP8285 (2 MB flash), ESPHome pre-flashed, 16 A, HLW8032 metering IC, US$9.94 sale / US$18.40 regular (manufacturer store, fetched 2026-11-25); standby power and certifications not stated on the page — [athom.tech ESPHome EU plug](https://www.athom.tech/blank-1/esphome-eu-plug)
- Athom "ESP32-C5 Dual-Band Wi-Fi EU Plug Made For ESPHome": ESP32-C5 (4 MB), 2.4 + 5 GHz Wi-Fi 6, ESPHome pre-installed, 16 A, BL0942 metering IC, Bluetooth proxy, US$15.60 sale / US$20.00 regular; a Tasmota variant and an "EU Dual Smart Plug" also exist — [athom.tech ESP32-C5 EU plug](https://www.athom.tech/blank-1/esp32-c5-dual-band-wi-fi-eu-plug-made-for-esphome), [athom.tech home](https://www.athom.tech/)
- Athom Plug V3 (PG01V3-EU16A): ESP32-C3, CSE7766 metering IC, sold with Tasmota pre-installed; the same hardware is listed as a "Made for ESPHome" device — [Blakadder: Athom Plug V3](https://templates.blakadder.com/athom_PG01V3-EU16A.html), [ESPHome Made-for-ESPHome list](https://devices.esphome.io/made-for-esphome/)
- Athom ships EU orders from China via 4PX (Germany/Italy 5-10 working days, other EU 8-12); VAT/customs handling is not stated on the shipping page — [athom.tech shipping](https://www.athom.tech/shipping)
- NOUS A1T: ESP8266, "Tasmota pre-installed", BL0937 metering, 16 A; calibration recommended for accurate readings — [Blakadder: NOUS A1T](https://templates.blakadder.com/nous_A1T.html)
- NOUS A8T: ESP32, Tasmota pre-installed, 10 A, BL0937 metering — [Blakadder: NOUS A8T](https://templates.blakadder.com/nous_A8T.html)
- Refoss P11: ESP8266, Tasmota pre-installed, CSE7766 metering; note about wrong factory temperature calibration causing unexpected switch-off (fix: `ADCParam1 2, 4000,80000,4250,1`) — [Blakadder: Refoss P11](https://templates.blakadder.com/refoss_P11.html)
- Other EU devices sold with Tasmota pre-installed: NOUS A4T/A5T/A6T (outdoor, power strip), Delock 11826/11827, CloudFree P1EU, CurrySmarter 16A plugs, Eightree ET28 — [Blakadder pre-flashed list](https://templates.blakadder.com/preflashed.html)
- "Made for ESPHome" now requires an ESP32-family chip ("ESP8266 is not eligible"), a public configuration, no cloud lookalike, user OTA of own builds and "Serial flashing left enabled, as the escape hatch" — [ESPHome: Made for ESPHome](https://esphome.io/guides/made_for_esphome.html)

Zigbee plugs (via Zigbee2MQTT):
- Sonoff S60ZBTPF (Type F): exposes power, current, voltage, energy (incl. daily/monthly totals), overload protection (0.1-4000 W, 0.1-17 A, 165-277 V), inching, power-on behaviour; OTA supported — [Zigbee2MQTT: S60ZBTPF](https://www.zigbee2mqtt.io/devices/S60ZBTPF.html)
- NOUS A1Z: "Smart plug (with power monitoring)", exposes power/current/voltage/energy, child lock, indicator mode, power-outage memory; OTA supported — [Zigbee2MQTT: A1Z](https://www.zigbee2mqtt.io/devices/A1Z.html)
- Aqara SP-EUC01 ("Smart plug EU"): power, energy, voltage, current; overload-protection setting range 100-2300 W; firmware v42 issue where control switches routed through the plug can toggle it — [Zigbee2MQTT: SP-EUC01](https://www.zigbee2mqtt.io/devices/SP-EUC01.html)
- Generic Tuya TS011F: from firmware 1.0.5 "TuYa has disabled the automatic reporting of power, voltage and current values meaning they need to be polled instead" (`measurement_poll_interval`); plugs produced since Q4 2021 usually ship that firmware; some variants report unreliably or switch off randomly — [Zigbee2MQTT: TS011F_plug_1](https://www.zigbee2mqtt.io/devices/TS011F_plug_1.html)
- Sonoff's store (Nov 2026) also lists a new "iPlug Gen2 S61s Type F Matter Over Wi-Fi Smart Energy Monitoring Plug" (S61STPF) and an outdoor Type F version (S61STPF-PM-O, 17.90, currency not stated in the JSON) — [sonoff.tech products JSON](https://sonoff.tech/products.json?limit=250)
- Aqara EU store lists "Power Plug H2 EU" and "Outdoor Plug H2 EU" at 32.99 each (both unavailable at fetch time) — [eu.aqara.com products JSON](https://eu.aqara.com/products.json?limit=250)
- Innr lists an "Outdoor Smart Plug 16A with Power Metering" at EUR 24.99-44.99 and "Innr Smart Plug" EUR 19.99-59.99 (pack-size ranges) — [innr.com](https://www.innr.com/en/)

IKEA (Matter over Thread, not Zigbee):
- IKEA Germany's smart-control category lists GRILLPLATS smart plug at EUR 8.99 (EUR 12.00 with remote) and the outdoor TOFSMYGGA plug at EUR 12.99, all "kompatibel mit Matter"; the earlier Zigbee plugs TRETAKT and INSPELNING do not appear in the category — [IKEA DE: Smart Home Steuerung](https://www.ikea.com/de/de/cat/smart-home-steuerung-36814/)
- GRILLPLATS: "Dieses Produkt nutzt Matter over Thread, was bedeutet, dass du einen Thread Border Router benötigst"; it can show power consumption; maximum load and standby power are not given on the product page — [IKEA DE: GRILLPLATS](https://www.ikea.com/de/de/p/grillplats-steckdose-smart-60604238/)

Tuya/Beken Wi-Fi plugs (flash-yourself path):
- tuya-cloudcutter exploits only unpatched Beken BK7231T/N and Realtek RTL8720CF/RTL8710BN devices; "Tuya has patched their SDK as of February 2022" and "Any device with a firmware compiled against a patched SDK will not be exploitable"; patched devices can still be flashed over serial — [tuya-cloudcutter](https://github.com/tuya-cloudcutter/tuya-cloudcutter)

### Inferences
- For this project the lowest-effort metering plug is whichever of these is cheapest on the day: Shelly Plug S Gen3 on stock firmware (enable MQTT, never create a Shelly account), an Athom ESPHome plug (already speaks ESPHome; add the `mqtt:` component and re-flash OTA), or a Zigbee plug (also acts as a mesh router for battery sensors).
- "Buy pre-flashed or buy an open local API" has replaced "buy cheap and flash it" in 2026: the Shelly OTA-to-Tasmota route is frozen, the Plug S Gen3 needs destructive opening, and Tuya OTA exploits are patched.
- Type E countries (France, Belgium, Poland, Czechia): Shelly lists its CEE 7/7 plug as compatible with Type E sockets, but the outlet side of these plugs is Schuko. Appliances with the hybrid CEE 7/7 plug fit; dedicated Type E-socket variants (Athom "FR", NOUS "A1T-F", Sonoff "TPE") were not verified in this session.
- Shelly rates its plugs for resistive loads only (2500 W / 3680 W); motor or compressor loads should be kept well below the nameplate.
- A Zigbee plug choice should avoid the generic TS011F lottery (polling-only firmware) in favour of models with their own Zigbee2MQTT definitions (Sonoff S60ZBTPF, Aqara SP-EUC01, NOUS A1Z).

### Gaps
- Metering accuracy is not stated by Shelly, Athom, NOUS or Sonoff on the fetched pages; standby power is only published by Shelly ("< 1 W").
- Prices for NOUS A1T/A8T/A1Z, Sonoff S60TPF/S60ZBTPF, Refoss P11, Aqara SP-EUC01 and generic TS011F plugs could not be fetched (nous.technology returned HTTP 403; Shopify stores rate-limited). ESTIMATES (unverified): NOUS A1T ~EUR 10-13, NOUS A1Z ~EUR 10-12, Sonoff S60ZBTPF ~EUR 12-15, TS011F plugs ~EUR 8-12, Aqara SP-EUC01 ~EUR 20-25.
- Chip, local API and flashability of the Sonoff S60 Wi-Fi (S60TPF) and the new S61 "iPlug Gen2" Matter plugs were not verified (no Tasmota/ESPHome template page found by direct URL).
- CE/RED certification documents were only confirmed for Shelly (EU DoC on the KB page); Athom and NOUS pages fetched do not state certifications.
- Whether IKEA INSPELNING/TRETAKT (Zigbee) are still sold anywhere in the EU was not verified; they are absent from the IKEA Germany category page.

## 2. Lighting without rewiring: smart bulbs (E27/E14/GU10), the wall-switch problem, and LED strip controllers

### Takeaway
For a Zigbee2MQTT stack the bulb market shifted in 2026: IKEA Germany now lists only KAJPLATS bulbs, which are Matter-over-Thread (from EUR 4.99) rather than Zigbee, so Zigbee bulbs must come from Innr, Philips Hue (works without the Hue Bridge through Zigbee2MQTT), Aqara or Tuya-type brands, or one uses Wi-Fi bulbs with a local API (Shelly Duo/Multicolor Gen3) or pre-flashed open firmware (Athom). The wall-switch problem is handled by leaving the switch on (cover or guard), setting the bulb's power-on behaviour, and sticking a battery button next to the switch that is bound directly to the bulb or group so it keeps working when the hub is down. LED strips are best run from a WLED controller on an ESP32 (pre-flashed Athom units cost US$8-24) fed by a closed, certified plug-in power adapter, which keeps all DIY work on the low-voltage side.

### Cited Findings

IKEA (Matter over Thread):
- IKEA Germany's smart LED bulb category shows 13 products, all KAJPLATS and all labelled "kompatibel mit Matter": E27 470 lm white spectrum EUR 4.99; E27 806 lm clear EUR 7.99; E27 1055 lm white spectrum EUR 8.99; E27 1055 lm colour + white EUR 9.99; E27 1521 lm white spectrum EUR 9.99; GU10 575 lm white spectrum EUR 6.99; GU10 470 lm colour EUR 7.99; E14 806 lm white spectrum EUR 6.99; E14 806 lm colour EUR 7.99; E14 470 lm clear EUR 6.99; starter sets with remote EUR 9.99 (white) / EUR 12.99 (colour) — [IKEA DE: Smart Home LED-Leuchtmittel](https://www.ikea.com/de/de/cat/smart-home-led-leuchtmittel-36813/)
- IKEA's new range uses Matter over Thread and needs a Thread border router (e.g. the DIRIGERA hub, EUR 79.99) for app control; stated on the GRILLPLATS and BILRESA product pages — [IKEA DE: GRILLPLATS](https://www.ikea.com/de/de/p/grillplats-steckdose-smart-60604238/), [IKEA DE: BILRESA](https://www.ikea.com/de/de/p/bilresa-fernbedienung-weiss-smart-dualschalter-10604165/)
- IKEA Germany's smart-lighting category (49 products, first 24 visible) shows no TRÅDFRI Zigbee bulbs and no RODRET/STYRBAR/SOMRIG remotes among the visible items; it lists the ORMANÄS smart LED strip (4 m) at EUR 29.99 — [IKEA DE: Smart Home Beleuchtung](https://www.ikea.com/de/de/cat/smart-home-beleuchtung-36812/)
- A self-hosted Matter controller exists: the Open Home Foundation "Matter (.js) Server" is a WebSocket Matter controller (port 5580, `/ws`), "drop-in replacement for the Python Matter Server", supports Matter 1.6.0, can run as a standalone Docker container, Apache-2.0; the older python-matter-server is archived ("version 8.1.2 is the final version") — [matter-js/matterjs-server](https://github.com/matter-js/matterjs-server), [python-matter-server](https://github.com/home-assistant-libs/python-matter-server)
- Espressif's ESP Thread Border Router is built on ESP-IDF and the open-source OpenThread stack, combines Espressif Wi-Fi and 802.15.4 SoCs, is Thread 1.4 certified and supports Matter — [ESP Thread BR docs](https://docs.espressif.com/projects/esp-thread-br/en/latest/)

Zigbee bulbs through Zigbee2MQTT (no vendor bridge):
- Philips Hue bulbs pair directly: "New lights are automatically in pairing mode"; factory reset without a bridge is possible by power-cycling 5 times (off 2 s, on 8 s), by serial number from the Zigbee2MQTT frontend (Touchlink → Philips Hue reset), with a Hue dimmer switch held next to the lamp, or via the Hue Bluetooth app; OTA updates supported; exposes brightness, colour temperature, colour, effects and power-on behaviour — [Zigbee2MQTT: Philips 9290022166](https://www.zigbee2mqtt.io/devices/9290022166.html)
- Hue power-on behaviour can be set (`on`, `off`, `recover`, `default`); warning: with `off`, "You will NOT be able to turn the bulb on by sequentially switching power on and off" — [Zigbee2MQTT: Philips 9290022166](https://www.zigbee2mqtt.io/devices/9290022166.html)
- Philips Hue Germany prices (fetched 2026-11-25): Essential A60 E27 806 lm White and Color 3-pack EUR 59.99 / 4-pack EUR 69.99; White Ambiance A60 E27 1100 lm 2-pack EUR 64.99; White GU10 2-pack EUR 39.99; White Ambiance GU10 2-pack EUR 49.99 (3-pack on offer EUR 48.99); White Ambiance E14 2-pack EUR 59.99 — [philips-hue.com DE bulbs](https://www.philips-hue.com/de-de/products/smart-light-bulbs)
- Innr RB 279 T ("Smart bulb tunable white E27") is supported with brightness, colour temperature, power-on behaviour (off/on/toggle/previous) and OTA — [Zigbee2MQTT: Innr RB 279 T](https://www.zigbee2mqtt.io/devices/RB_279_T.html)
- Innr prices on the manufacturer site: Smart Bulb Colour E27 1200 lm EUR 24.99-39.99, Smart Bulb Comfort 1100 lm E27 EUR 29.99-44.99 ("€15,00 per item"), Smart Spot Colour GU10 EUR 19.99-49.99 (pack-size ranges); the site says the products "Work with Philips Hue*, Homey & SmartThings" and shows a Zigbee logo — [innr.com](https://www.innr.com/en/)
- Aqara LED Bulb T2 E14: 20.99 on the Aqara EU store — [eu.aqara.com products JSON](https://eu.aqara.com/products.json?limit=250)

Wi-Fi bulbs with local API or open firmware:
- Shelly Duo Bulb E27 Gen3: up to 806 lm, 8.5 W, 2700-6500 K, dimmable, "Shelly ESP C3" MCU, Wi-Fi b/g/n + Bluetooth 4.2, MQTT yes, scripting yes, idle draw < 1 W with Wi-Fi on, 15000 h — [Shelly KB: Duo Bulb E27 Gen3](https://kb.shelly.cloud/knowledge-base/shelly-duo-bulb-e27-gen3)
- Shelly store prices (fetched 2026-11-25, Black-Friday week; the bulk product feed does not state a currency, the single-product feed of the same store reports EUR): Duo Bulb E27 Gen3 2-pack 15.12 (list 18.90), 4-pack 27.92 (list 34.90); Multicolor Bulb E27 Gen3 2-pack 15.12 (list 18.90), 4-pack 27.92 (list 34.90) — [shelly.com products JSON](https://www.shelly.com/products.json?limit=250), [shelly.com product JSON](https://www.shelly.com/products/shelly-plug-s-gen3.json)
- Athom sells bulbs pre-flashed with WLED (and ESPHome/Tasmota lines): WLED 12W colour bulb US$14.63 (regular 19.50, ESP32-C3), WLED E14 colour bulb US$12.98 (ESP8285), WLED GU10 RGBCW bulb (ESP32-C3), BR30 RGBCW US$13.98 — [athom.tech WLED](https://www.athom.tech/wled)

The wall-switch problem (bulb loses power when the wall switch is off):
- Zigbee binding lets a remote control a bulb or group directly, "allowing devices to directly control each other without the intervention of Zigbee2MQTT"; it "Works even when home automation software, Zigbee2MQTT, or the coordinator is down"; bind with `zigbee2mqtt/bridge/request/device/bind` and payload `{"from": "my_remote", "to": "my_bulb"}`; battery remotes must be woken (press a button) right before binding — [Zigbee2MQTT: Binding](https://www.zigbee2mqtt.io/guide/usage/binding.html)
- Bulb power-on behaviour after power loss is configurable on Hue (`hue_power_on_behavior`) and Innr (`power_on_behavior`: off/on/toggle/previous) — [Zigbee2MQTT: Philips 9290022166](https://www.zigbee2mqtt.io/devices/9290022166.html), [Zigbee2MQTT: Innr RB 279 T](https://www.zigbee2mqtt.io/devices/RB_279_T.html)

LED strip controllers with open firmware (WLED):
- WLED's controller list says "ESP8266 should not be used for any new installations" and treats ESP32-C3/S2/S3 as "experimentally supported"; listed pre-installed ESP32 controllers include QuinLED Dig-Uno (level shifters, 5-24 V, onboard fuse), Dig-Quad, Dig-Octa, Dig2go (USB-C powered, fuses, IR, digital microphone, "Recommended for small projects up to 15W"), Athom controllers, ABC! WLED boards, cod.m WLED Controller, SMLIGHT A1-SLWF-03 (ESP32, mic, 5-24 V) — [WLED: compatible controllers](https://kno.wled.ge/basics/compatible-controllers/)
- Athom pre-flashed WLED controllers (manufacturer store, fetched 2026-11-25): WS2812B LED Strip Controller US$7.84 (ESP32-C3), Slim Addressable US$11.85, Sound Reactive Slim US$13.30, High Power US$14.88, RGBCCT Analog + Digital US$14.88, ESP32 Music Reactive US$19.53, Ethernet ESP32 Addressable/PWM US$23.80 — [athom.tech WLED](https://www.athom.tech/wled)
- WLED notes that Athom Technology Co. (the Chinese maker) is "not affiliated in any way with the European company Athom B.V." (trademark dispute) — [WLED: compatible controllers](https://kno.wled.ge/basics/compatible-controllers/)
- WLED's getting-started guide: "For a new project, an ESP32 is the better pick" (ESP8266 support is ending); addressable strips need power, data and ground; analog (single-colour/RGB) strips need one GPIO and one MOSFET per channel (IRLZ44N and STP55NF06L are named); "Make sure to connect ESP and LED-strip grounds together!"; "If the connecting wire cannot be kept short, use a level shifter"; the page links to an LED power, wiring and fuse calculator and to wiring guides for 12 V strips and multiple supplies — [WLED: getting started](https://kno.wled.ge/basics/getting-started/)
- EU maker-shop prices for strip parts (hobbyelectronica.nl, fetched 2026-11-25): WS2812B RGB LED strip EUR 5.95 (was 7.95) and "WS2812B RGB 60" EUR 6.95 (was 8.95) (length per item not stated in the listing); plug-in power adapters 12 V 1 A EUR 8.95, 12 V 2 A EUR 11.95, 5 V 2 A (micro-USB) EUR 9.95; no certification or brand is stated in these listings and no Mean Well units appear on the first page of the category — [hobbyelectronica: Licht](https://www.hobbyelectronica.nl/product-categorie/licht-2/), [hobbyelectronica: Voeding](https://www.hobbyelectronica.nl/product-categorie/voeding-2/)

### Inferences
- With IKEA now on Matter-over-Thread, the cheapest route to many bulbs (KAJPLATS from EUR 4.99) requires adding a Thread border router plus a Matter controller (for example matterjs-server bridged to MQTT by the user's hub); that is a second radio stack next to Zigbee and should be a deliberate decision, not an accident of shopping at IKEA.
- For a Zigbee2MQTT-first stack the practical bulb choices are Innr (about EUR 15 per bulb in multipacks), Hue Essential/White multipacks (about EUR 15-20 per bulb) or Tuya-type Zigbee bulbs; Wi-Fi alternatives with fully local MQTT are Shelly Duo/Multicolor Gen3 (about EUR 7-9.50 per bulb in packs) and Athom pre-flashed bulbs.
- Rental-friendly switch handling: leave the wall switch on and physically block it (a clip-on guard or a cover plate that sits over the rocker; the Shelly BLU Wall Switch 4 and several remotes are designed to sit in or over standard frames), set power-on behaviour to "previous"/"recover" so a power cut does not light every lamp, and bind a stick-on battery remote to the bulb group for hub-independent control.
- Floor and table lamps on sockets can use either a smart bulb or a metering smart plug; a plug keeps the lamp's own switch semantics and gives energy data, a bulb gives dimming and colour temperature.
- LED strips fit the project's safety rule well: the only mains part is a plug-in power adapter, and everything after it (5/12/24 V strip, WLED controller) is low-voltage DIY. The adapter is therefore the part to buy certified: a closed desktop or wall-plug unit from a recognised brand with CE plus a test-house mark, rather than an open-frame supply with exposed mains screw terminals. (Background knowledge, not verified in this session: Mean Well's GST/GSM desktop-adapter series, e.g. GST25E05 or GST60A05/12/24, is the usual recommendation; open-frame LRS units are common in WLED builds but need a mains-rated enclosure.)
- Sizing rule of thumb (background knowledge, not verified in this session): budget up to about 60 mA per RGB pixel at full white on 5 V WS2812B, i.e. a 1 m / 60-LED strip fits a 5 V 3-4 A adapter, and use WLED's current limiter to cap draw to the adapter rating; longer runs are easier at 12 V (WS2815) or 24 V with lower current.
- Pre-flashed Athom WLED controllers (US$8-15) or a bare ESP32 board (EUR 4-11) with a level shifter both run stock WLED; IKEA's ORMANÄS strip (EUR 29.99) is the no-DIY alternative but sits in the Matter/Thread camp.

### Gaps
- No EU price was fetched for Tuya-type Zigbee bulbs, Ledvance Smart+ or Müller-Licht tint; ESTIMATE (unverified): EUR 8-15 per E27 Zigbee bulb.
- No source was fetched for clip-on switch guards/covers for EU 55 mm rocker switches (products exist from several accessory makers; names and prices unverified).
- Whether remaining IKEA TRÅDFRI Zigbee bulbs/remotes are still sold in other EU countries is unverified.
- Whether a BILRESA remote can be bound directly to KAJPLATS bulbs without any hub is not stated on the IKEA page (a reviewer found it "schwer zu verbinden" without hub).
- Certified LED power supplies: no specification or price could be verified. The Mean Well GST60A datasheet URL (https://www.meanwell.com/Upload/PDF/GST60A/GST60A-SPEC.PDF) returns a PDF that could not be parsed, and the EU distributor page returned HTTP 403. ESTIMATE (unverified): a branded 5 V 4-6 A or 12 V 5 A desktop adapter costs about EUR 12-25 at EU distributors (Reichelt, TME, BerryBase).
- WLED's power-sizing figures (mA per LED, default current limiter value) were not on the fetched page; the rule of thumb above is unverified.
- EU-stocked WLED controllers (GLEDOPTO, QuinLED via resellers, cod.m, SMLIGHT) and their EUR prices were not fetched; only Athom's USD store prices are verified.
- Athom ESPHome/Tasmota bulb prices: category pages were truncated; only the WLED bulb prices were retrieved.

## 3. Stick-on battery buttons, remotes and rotary dimmers (Zigbee or BLE)

### Takeaway
Zigbee remotes that support direct binding (Hue dimmer switch, IKEA RODRET, Aqara H1 wireless rocker and knob in "command" mode, Tuya knob) are the best match for smart bulbs because they keep working when the hub is down; the Sonoff SNZB-01P cannot bind and only sends events to the hub. BLE buttons (Shelly BLU, Athom BTHome) need an ESP32 or Shelly device in range as a gateway, and IKEA's current BILRESA remotes are Matter-over-Thread.

### Cited Findings
- Philips Hue dimmer switch gen 2 (929002398602): actions on/off/up/down with press/hold/release variants; "it can also be used to directly control other lights and switches in the network" (binding; wake the device by pressing a button after sending the bind command); OTA supported — [Zigbee2MQTT: Hue dimmer switch gen 2](https://www.zigbee2mqtt.io/devices/929002398602.html)
- IKEA RODRET (E2201): 1x AAA, actions on/off/brightness_move_up/down/stop, "This remote can directly control individual devices and/or groups (v1.0.57)", OTA supported — [Zigbee2MQTT: IKEA E2201](https://www.zigbee2mqtt.io/devices/E2201.html)
- Aqara wireless remote switch H1 double rocker (WXKG15LM): `operation_mode` "command" enables bindings (only one endpoint for binding even with two rockers), "event" mode gives 12 action values (single/double/hold, left/right/both) — [Zigbee2MQTT: WXKG15LM](https://www.zigbee2mqtt.io/devices/WXKG15LM.html)
- Aqara smart rotary knob H1 wireless (ZNXNKG02LM): rotation events with angle/speed, single/double/hold; "Command mode is useful for binding. Event mode is useful for processing"; mode can only be changed within about a minute after pairing — [Zigbee2MQTT: ZNXNKG02LM](https://www.zigbee2mqtt.io/devices/ZNXNKG02LM.html)
- Tuya smart knob (ERS-10TZBVK-AA): event mode (rotate_left/right, single, double, hold) or command mode (brightness_step, toggle, hue_move); caveat: may toggle unrelated devices because devices are placed "into group 0 by default" — [Zigbee2MQTT: ERS-10TZBVK-AA](https://www.zigbee2mqtt.io/devices/ERS-10TZBVK-AA.html)
- Sonoff SNZB-01P: single/double/long, CR2477, OTA; "This button can not directly control individual devices or groups (v2.2.0)" — [Zigbee2MQTT: SNZB-01P](https://www.zigbee2mqtt.io/devices/SNZB-01P.html)
- Shelly BLU Wall Switch 4 ZB (SBBT-104CEU): dual protocol Bluetooth 5 LE + Zigbee 802.15.4 (4 presses = Bluetooth pairing, 5 presses = Zigbee pairing), four buttons with single/double/triple/long press ("16 different actions"), CR2032 "Up to 2 years", "Mount it anywhere with adhesive or screws", "Fits in popular wall switch frame systems", 46x46x13 mm — [Shelly KB: BLU Wall Switch 4 ZB](https://kb.shelly.cloud/knowledge-base/shelly-blu-wall-switch-4-zb)
- Shelly BLU Button1: Bluetooth, 1/2/3 short presses + long press, beacon mode, CR2032 up to 2 years, 36x36x6 mm, AES (CCM) encryption — [Shelly KB: BLU Button1](https://kb.shelly.cloud/knowledge-base/shellyblu-button1)
- Shelly's KB also lists Zigbee-capable "ZB" versions of the BLU Button Tough 1, BLU RC Button 4 and a BLU Remote Control ZB — [Shelly KB device index](https://kb.shelly.cloud/knowledge-base/devices)
- BTHome "is an open standard for broadcasting sensor data and button presses over Bluetooth LE", allows encryption; Shelly BLU Button1 reports single/double/triple/long clicks in this format — [bthome.io](https://bthome.io/)
- An ESP32 running OpenMQTTGateway/Theengs decodes Shelly BLU Button1, BLU Wall Switch 4, BLU RC Button 4 (button press type per button, battery) and publishes them to MQTT — [Theengs Decoder device list](https://decoder.theengs.io/devices/devices.html)
- Athom BTHome Wireless Smart Button: US$7.98 sale (regular US$11.40) — [athom.tech BTHome](https://www.athom.tech/bthome)
- IKEA BILRESA remote (dual button EUR 4.99; scroll wheel EUR 5.99): "Dieses Produkt nutzt Matter over Thread", AAA batteries, controls up to 10 products, 70x40x19 mm, customer rating 3.1/5 from 866 ratings — [IKEA DE: BILRESA](https://www.ikea.com/de/de/p/bilresa-fernbedienung-weiss-smart-dualschalter-10604165/), [IKEA DE: Smart Home Steuerung](https://www.ikea.com/de/de/cat/smart-home-steuerung-36814/)

### Inferences
- For "one stick-on Zigbee button" in Tier 1, pick a bindable remote (Hue dimmer, RODRET if still obtainable, Aqara H1 rocker) if it should dim the bulbs directly; pick the cheapest event-only button (Sonoff SNZB-01P or a Tuya scene switch) if every action is handled by the hub anyway.
- The Shelly BLU Wall Switch 4 ZB is the most rental-friendly "looks like a wall switch" option: it sits in a standard frame, sticks with adhesive and can join Zigbee2MQTT directly (Zigbee2MQTT support was not verified in this session).

### Gaps
- EUR prices for the Hue dimmer switch, Aqara H1 remote/knob, Tuya knob, Sonoff SNZB-01P and Shelly BLU buttons were not fetched. ESTIMATES (unverified): Hue dimmer ~EUR 20-25, Aqara mini switch ~EUR 13-18, Aqara H1 double rocker ~EUR 20-25, Aqara knob H1 ~EUR 20-25, Tuya knob ~EUR 10-15, Sonoff SNZB-01P ~EUR 9-12, Shelly BLU Button1 ~EUR 10-12, Shelly BLU Wall Switch 4 ~EUR 17-20.
- Zigbee2MQTT support for the Shelly BLU "ZB" devices was not checked.
- IKEA RODRET/SOMRIG/STYRBAR availability in November 2026 is unverified (not on the IKEA Germany category pages fetched).

## 4. Curtains and blinds with local control (retrofit for an existing track first)

### Takeaway
For curtains on an existing track, two retrofit robots ride the rail without drilling: the Aqara Curtain Driver E1 Track Version (Zigbee 3.0, native Zigbee2MQTT support with position, battery and light level; U- and I-rails only) and the SwitchBot Curtain 3 (BLE; controllable and readable locally through an ESP32 gateway such as OpenMQTTGateway or Home Assistant's local Bluetooth integration with no hub, EUR 89.99). Track-replacement motors (Tuya/Zemismart/Dooya Zigbee, Sonoff ZBCurtain) and wired tubular motors are a second step, and IKEA Germany currently sells no electric blinds at all.

### Cited Findings

Retrofit robots for an existing track or rod:
- Aqara Curtain Driver E1: Rod and Track versions; the Track Version "only supports U- and I-Rails" (U-rail bottom slot width ≤ 8 mm, gliders without wheels not supported; I-rail bottom width ≥ 10 mm); moves "a maximum of 12 kilograms"; 6000 mAh battery, "Up to 1-Year Battery Life between Each Charge"; USB-C charging that can also power it constantly; Zigbee 3.0; built-in light sensor; "No tools are needed" — [Aqara EU: Curtain Driver E1](https://www.aqara.com/eu/product/curtain-driver-e1/)
- Aqara Curtain Driver E1 in Zigbee2MQTT (ZNCLBL01LM): cover state and position 0-100, battery, charging state, power source, hooks lock/unlock, limits calibration (start/end/reset), illuminance, motor state, "hand open" (pulling the curtain starts the motor), OTA supported — [Zigbee2MQTT: ZNCLBL01LM](https://www.zigbee2mqtt.io/devices/ZNCLBL01LM.html)
- SwitchBot Curtain 3: EUR 89.99 on the SwitchBot EU store (fetched 2026-11-25); U Rail and Rod variants and a solar combo are listed in the store navigation — [eu.switch-bot.com Curtain 3](https://eu.switch-bot.com/products/switchbot-curtain-3)
- SwitchBot local control: Home Assistant's integration states "A SwitchBot Hub is not required for this integration" and "The integration connects locally to the devices without going via the SwitchBot Cloud"; Curtain 3 and Roller Shade are "certified for Bluetooth"; curtain features: open/close/pause, set position, get position, light level, battery, calibration state, plus movement speed on Curtain 3; curtains are not listed among the encrypted device types — [Home Assistant: SwitchBot Bluetooth](https://www.home-assistant.io/integrations/switchbot/)
- OpenMQTTGateway on an ESP32 controls "SwitchBot Curtain (2/3)" (model_id `W070160X`) over MQTT: publish to `home/OpenMQTTGateway/commands/MQTTtoBT` a payload such as `{"model_id":"W070160X","position":55,"id":"AA:BB:CC:DD:EE:FF"}` (or "open"/"close"/"stop"); Blind Tilt (`W270160X`) and Bot (`X1`) are also controllable — [OpenMQTTGateway: BLE](https://docs.openmqttgateway.com/use/ble.html)
- Theengs Decoder (used by OpenMQTTGateway) decodes SwitchBot Curtain (2/3) broadcasts: moving, position, light level, battery, calibration state — [Theengs Decoder device list](https://decoder.theengs.io/devices/devices.html)
- SwitchBot publishes an official "SwitchBot BLE open API"; the Curtain 3 document describes broadcast fields (calibrated flag, battery, movement status, position %, light level 1-10) and command packets whose header specifies "Encryption Mode 0 – no encryption", including a low-speed/"silent mode" setting — [SwitchBotAPI-BLE](https://github.com/OpenWonderLabs/SwitchBotAPI-BLE), [Curtain 3 BLE doc](https://github.com/OpenWonderLabs/SwitchBotAPI-BLE/blob/latest/devicetypes/curtain3.md)
- A dedicated ESP32 firmware bridges SwitchBot bots and curtains to MQTT ("no switchbot hub used/required"; topic `switchbot/esp32/curtain/<name>/set` with OPEN/CLOSE/PAUSE or 0-100; classic ESP32 boards only, ESP32-C3 listed as possibly not working); its README does not mention Curtain 3 — [devWaves/SwitchBot-MQTT-BLE-ESP32](https://github.com/devWaves/SwitchBot-MQTT-BLE-ESP32)

Roller blinds and venetian blinds (undecided coverings):
- Aqara Roller Shade Driver E1 (ZNJLBL01LM) in Zigbee2MQTT: cover position, battery, charging status, motor speed (low/medium/high), limits set with the device buttons, OTA — [Zigbee2MQTT: ZNJLBL01LM](https://www.zigbee2mqtt.io/devices/ZNJLBL01LM.html)
- AM43-type chain drives over BLE: ESPHome has a native `am43` cover platform that "connects to the device over the ESP32's BLE peripheral and both controls and fetches the position and state of the motor"; "sold under various brands, such as Zemismart and A-OK"; it "does NOT support Tuya-based versions"; limits must first be set with the vendor app or buttons; "The BLE stack on the ESP32 has a limit of three connected devices" — [ESPHome: AM43 cover](https://esphome.io/components/cover/am43.html)
- AM43 Zigbee version (Moes AM43-0.45/40-ES-EB, "Roller blind/shades drive motor"): position, motor direction/speed, upper/bottom limit setting via Zigbee2MQTT — [Zigbee2MQTT: AM43-0.45/40-ES-EB](https://www.zigbee2mqtt.io/devices/AM43-0.45_40-ES-EB.html)
- SwitchBot Blind Tilt (venetian blinds) is controllable locally via OpenMQTTGateway (tilt 0-100, open/close_up/close_down) — [OpenMQTTGateway: BLE](https://docs.openmqttgateway.com/use/ble.html)
- IKEA Germany's "Elektrische Rollos & Jalousien" category is empty on 2026-11-25: "Es tut uns leid, im Moment führen wir leider keine Produkte in diesem Bereich" — [IKEA DE: Elektrische Rollos](https://www.ikea.com/de/de/cat/elektrische-rollos-44531/)
- IKEA's earlier Zigbee blinds remain supported by Zigbee2MQTT (FYRTUR E1757: position, battery, OTA; needs the bundled repeater paired first) — [Zigbee2MQTT: IKEA E1757](https://www.zigbee2mqtt.io/devices/E1757.html)

Track-replacement and tubular motors (second priority; involve replacing the rail or the blind tube):
- Zigbee2MQTT's generic Tuya definition "Curtain motor/roller blind motor/window pusher/tubular motor" covers Zemismart ZM79E-DT, M515EGB, AM43, M2805EGBZTN, BCM500DS-TYZ; Tuya DT82LEMA-1.2N and M515EGZT; A-OK AM25; Yushun YS-MT750 and others; exposes state, position, motor speed and reverse direction — [Zigbee2MQTT: TS0601_cover_1](https://www.zigbee2mqtt.io/devices/TS0601_cover_1.html)
- Sonoff ZBCurtain ("Zigbee smart curtain motor"): position 0-100 and battery — [Zigbee2MQTT: ZBCurtain](https://www.zigbee2mqtt.io/devices/ZBCurtain.html)
- Mains tubular motors with a wall switch can be driven by a Shelly 2PM Gen4 in cover mode (calibration, positioning, slat tilt, obstacle detection) — this is in-wall wiring and therefore out of scope for a rental — [Shelly KB: 2PM Gen4](https://kb.shelly.cloud/knowledge-base/shelly-2pm-gen4)

DIY and RF options:
- ESPHome cover platforms: AM43, Current Based, Endstop, Feedback, Template, Time Based, Tuya and others; position is a value from 0.0 (closed) to 1.0 (open) — [ESPHome: Cover](https://esphome.io/components/cover/)
- TMC2209 for ESPHome (external component by slimcdk): `stepper` platform with an `on_stall` automation usable for sensorless homing, StealthChop/SpreadCycle selection, microsteps up to 256, current settings; UART wiring via 1 kΩ resistor to PDN_UART; no ready-made blinds example in the README; 76 stars — [slimcdk/esphome-custom-components](https://github.com/slimcdk/esphome-custom-components), [TMC2209 README](https://github.com/slimcdk/esphome-custom-components/tree/master/esphome/components/tmc2209)
- Classic DIY roller-blind project: ESP8266 + 28BYJ-48 stepper, MQTT and WebSocket control, 0-100 % positioning, printable parts on Thingiverse (thing:2392856); archived 29 Jan 2019, 116 stars — [nidayand/motor-on-roller-blind-ws](https://github.com/nidayand/motor-on-roller-blind-ws)
- ESPSomfy-RTS: ESP32 + CC1101 "controller for Somfy RTS blinds and shades that supports up to 32 individual shades and 16 groups", web UI, MQTT, tracks position even when the original remote is used; not for IO Home Control motors — [rstrouse/ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS)
- ESPHome has a native `cc1101` component (300-928 MHz, default 433.92 MHz, ASK/OOK default) that works with `remote_transmitter`/`remote_receiver`; `remote_transmitter` includes a Dooya action ("sends a Dooya RF remote code": 24-bit id, channel, button, check) and, per the fetched protocol list, A-OK, RC-Switch and KeeLoq entries — [ESPHome: CC1101](https://esphome.io/components/cc1101.html), [ESPHome: Remote Transmitter](https://esphome.io/components/remote_transmitter.html)
- Sonoff RF Bridge 433 with Tasmota: ESP8285 + EFM8BB1 RF chip, sends/receives 433.9 MHz codes over MQTT, learns up to 16 fixed-code remotes (PT2260/PT2262/PT2264/EV1527); users report "limited success flashing Tasmota to R2 V2.2", whose alternative RF firmware (OB38S003) is incomplete — [Tasmota: Sonoff RF Bridge 433](https://tasmota.github.io/docs/devices/Sonoff-RF-Bridge-433/)

### Inferences
- Because the curtains hang on a track, the decision is between Aqara E1 Track Version (Zigbee: fits the Zigbee2MQTT stack, reports position/battery/light, mesh-routed) and SwitchBot Curtain 3 U-rail (BLE: needs an ESP32 within a few metres running OpenMQTTGateway or equivalent, but is documented, unencrypted and hub-free). The first check is mechanical: rail profile (U-rail slot ≤ 8 mm with wheeled gliders, or I-rail ≥ 10 mm for Aqara) and curtain weight (Aqara: 12 kg).
- Both robots push the leading glider along the rail; a two-panel curtain that opens from the middle needs two units, which doubles the price. One unit per panel is the normal assumption.
- RF (433 MHz) motors give no position feedback; position is time-estimated by the controller. Zigbee and BLE drives report real position.
- DIY stepper drives are a fun low-voltage project (12/24 V, TMC2209, ESP32) but need 3D-printed mechanics; there is no single dominant maintained design, so treat it as a later experiment rather than the first curtain solution.

### Gaps
- EUR price of the Aqara Curtain Driver E1 (track version) was not fetched (Aqara EU store rate-limited; a guessed product URL returned 404, and the product did not appear in the first ~50 entries of the store's product feed, so current EU availability is also unconfirmed). ESTIMATE (unverified): EUR 70-100 per unit.
- SwitchBot Curtain 3 mechanical specs (max load, noise, battery life) were not retrieved from an official page; whether first-time calibration can be done without the SwitchBot app/account is not confirmed.
- No Zigbee2MQTT page was fetched for other Zigbee "curtain robots" that ride an existing track (Tuya/Zemismart/Moes); existence and prices unverified.
- Prices for AM43 chain drives, Zemismart track motors and tubular motors were not fetched (zemismart.com returned HTTP 403). ESTIMATES (unverified): AM43 chain drive EUR 35-55, Zigbee track motor with rail EUR 90-160, Zigbee tubular motor EUR 60-100.
- Noise levels were not found in any fetched source.
- The presence of a Somfy RTS action in ESPHome `remote_transmitter` was reported by the page summary but not double-checked.

## 5. Switch-pressing robots (low priority: the home has no built-in-LED fixtures)

### Takeaway
If one is ever needed, the SwitchBot Bot is controllable locally over BLE from an ESP32 gateway with no hub, and the Tuya "Fingerbot Plus" exists in a Zigbee version supported by Zigbee2MQTT; the Bluetooth Fingerbot needs a key from the Tuya cloud, so it is not account-free.

### Cited Findings
- SwitchBot Bot: local on/off, press action and battery level without hub or cloud; optional device password; a `switch mode` attribute distinguishes pull/retract mode from momentary press mode — [Home Assistant: SwitchBot Bluetooth](https://www.home-assistant.io/integrations/switchbot/)
- OpenMQTTGateway controls the SwitchBot Bot (model_id `X1`: press, on, off, up, down) over MQTT from an ESP32 — [OpenMQTTGateway: BLE](https://docs.openmqttgateway.com/use/ble.html)
- Tuya "Zigbee fingerbot plus" (TS0001_fingerbot): state ON/OFF, mode click/switch/program, lower/upper movement limits, sustain time 0-10 s, reverse, touch, battery — [Zigbee2MQTT: TS0001_fingerbot](https://www.zigbee2mqtt.io/devices/TS0001_fingerbot.html)
- Bluetooth Fingerbots (Fingerbot, Adaprox Fingerbot, Fingerbot Plus, CubeTouch): "The integration works locally, but connection to Tuya BLE device requires device ID and encryption key from Tuya IOT cloud" — [PlusPlus-ua/ha_tuya_ble](https://github.com/PlusPlus-ua/ha_tuya_ble)

### Inferences
- Order of preference if needed later: Zigbee Fingerbot Plus (fits Zigbee2MQTT) or SwitchBot Bot (BLE via ESP32 gateway); avoid BLE-only Tuya Fingerbots.

### Gaps
- Prices not fetched. ESTIMATES (unverified): SwitchBot Bot ~EUR 25-30, Zigbee Fingerbot Plus ~EUR 25-35.

## 6. Adhesive or freestanding sensors

### Takeaway
Battery Zigbee sensors from Sonoff and Aqara are the default for a Zigbee2MQTT stack; Shelly's BLU sensors now come in dual Bluetooth/Zigbee "ZB" versions, BLE sensors can be read through an ESP32 gateway, and IKEA's new EUR 7-8 sensors are Matter-over-Thread only.

### Cited Findings
- Sonoff SNZB-02P temperature/humidity sensor: CR2477, OTA, calibration offsets — [Zigbee2MQTT: SNZB-02P](https://www.zigbee2mqtt.io/devices/SNZB-02P.html)
- Sonoff store listing (prices as in the store JSON, currency not stated): AirGuard TH Lite SNZB-02B 9.90; AirGuard TH Pressure SNZB-02M 16.90; SenseGuard Motion Gen2 SNZB-03PR2 16.50; Presence Core 24 GHz Zigbee SNZB-06P24 23.90; SenseGuard DW Gen3 Zigbee & Thread door/window SNZT-04P 22.90; E-Ink TH sensor SNZB-02UL 22.00 — [sonoff.tech products JSON](https://sonoff.tech/products.json?limit=250)
- Aqara temperature and humidity sensor T1 (WSDCGQ12LM): temperature, humidity, pressure, battery; Aqara/Xiaomi devices may drop off when routed through routers that "cannot deal with Xiaomi devices" — [Zigbee2MQTT: WSDCGQ12LM](https://www.zigbee2mqtt.io/devices/WSDCGQ12LM.html)
- Aqara EU store: Presence Multi-Sensor FP300 4-pack 169.99 (compare-at 199.99) — [eu.aqara.com products JSON](https://eu.aqara.com/products.json?limit=250)
- Shelly BLU Door/Window ZB: 14.32 EUR (list 17.90), BLU Door/Window Mini ZB 15.90, BLU Door/Window Outdoor ZB 18.90 (store JSON, fetched 2026-11-25) — [shelly.com products JSON](https://www.shelly.com/products.json?limit=250)
- Theengs Decoder decodes Shelly BLU Door/Window, H&T and Motion, SwitchBot Meter/Contact/Motion sensors, and Xiaomi LYWSD03MMC thermometers with ATC/PVVX/BTHome firmware ("137 of 137 devices") — [Theengs Decoder device list](https://decoder.theengs.io/devices/devices.html)
- IKEA Germany: MYGGBETT door/window sensor EUR 7.99, TIMMERFLOTTE temperature/humidity EUR 7.99, MYGGSPRAY motion EUR 7.99, KLIPPBOK water leak EUR 6.99, ALPSTUGA air quality EUR 29.99, all Matter — [IKEA DE: Smart Home Steuerung](https://www.ikea.com/de/de/cat/smart-home-steuerung-36814/)
- Athom BTHome door contact sensor with temperature/humidity US$7.98; BTHome presence sensor (PIR + mmWave) US$13.68 — [athom.tech BTHome](https://www.athom.tech/bthome)
- "Made for ESPHome" mains/USB-powered presence sensors exist (Everything Presence Lite/One/Pro, Apollo MSR-1/MSR-2/MTR-1) — [ESPHome Made-for-ESPHome list](https://devices.esphome.io/made-for-esphome/)
- Generic battery Zigbee sensors at an EU maker shop (hobbyelectronica.nl, fetched 2026-11-25; brand/model not stated in the listing): door/window sensor EUR 10.95 (in stock), temperature/humidity sensor EUR 10.95 and PIR motion sensor EUR 10.95 (both on waitlist) — [hobbyelectronica: Zigbee en Matter](https://www.hobbyelectronica.nl/product-categorie/platformen/zigbee-matter/)
- Wired sensor modules for DIY ESP32 nodes at the same shop: DHT22/AM2302 temperature + humidity EUR 7.95, DS18B20 temperature EUR 3.95 (waterproof probe EUR 2.95), DHT11 EUR 2.25, HC-SR501 PIR EUR 2.45, AM312 mini PIR EUR 2.25, MC-38 wired door/window contact EUR 2.95 (page 1 of 5 of the sensor category; BME280, SHT4x, BH1750, LD2410 were not on the first page) — [hobbyelectronica: Sensors](https://www.hobbyelectronica.nl/product-categorie/sensors/)

### Inferences
- DIY ESP32 sensors (the user owns M5Stack devices and soldering tools) cover USB-powered spots; battery Zigbee sensors cover doors, windows and places without power.
- Mixing Aqara end devices with non-Aqara routers can cause drop-offs; Sonoff sensors are the lower-risk default on a mixed mesh.

### Gaps
- EUR prices for Aqara single sensors and Sonoff sensors in EU shops were not fetched; currency of the Sonoff store JSON is not confirmed.

## 7. In-wall relay/dimmer modules and hard-wired wall switches (short reference only: not usable in a rental without touching the wiring)

### Takeaway
If the wiring could ever be touched, Shelly Gen4 modules are the reference: same local MQTT/RPC API as Gen3 plus Wi-Fi 6 (802.11ax), Bluetooth 5 and a Zigbee/Matter mode on a new "ESP-Shelly-C68F" chip, in housings about 16-17 mm deep; Sonoff's Zigbee minis (ZBMINIR2 and the newer MINI-ZB series) are the Zigbee2MQTT equivalents with detach-relay mode.

### Cited Findings
- Shelly 1 Mini Gen4: 8 A, 29x34x16 mm, MCU "ESP-Shelly-C68F" (8 MB), Wi-Fi 802.11 b/g/n/ax, Bluetooth 5.0, Zigbee 802.15.4, Matter (Matter is the default mode; five button presses switch to Zigbee), "< 1.2 W", dry contact, MQTT/HTTP/webhooks/scripting; described as "improved version of Shelly 1 Mini Gen3 with a more advanced processor and Zigbee connectivity" — [Shelly KB: 1 Mini Gen4](https://kb.shelly.cloud/knowledge-base/shelly-1-mini-gen4)
- Shelly 1PM Mini Gen4: 8 A with power metering, same 29x34x16 mm housing, "External protection: B or C tripping, max 10 A rated" — [Shelly KB: 1PM Mini Gen4](https://kb.shelly.cloud/knowledge-base/shelly-1pm-mini-gen4)
- Shelly 2PM Gen4: 10 A per channel, 16 A total, 37x42x16 mm, "< 1.4 W", cover mode with calibration, slat tilt and obstacle detection — [Shelly KB: 2PM Gen4](https://kb.shelly.cloud/knowledge-base/shelly-2pm-gen4)
- Shelly Dimmer Gen3 / Gen4: trailing-edge, max 200 W (dimmable LED up to 150 W), neutral optional, minimum 20 W load without neutral and without bypass, metering (±5 %) only with neutral, external protection 16 A B/C; Gen3 38x42.5x16.5 mm with ESP32-C3 + STM32G051; Gen4 38.5x43.5x17 mm with ESP-Shelly-C68F, Wi-Fi ax, Zigbee repeater, Matter — [Shelly KB: Dimmer Gen3](https://kb.shelly.cloud/knowledge-base/shelly-dimmer-gen3), [Shelly KB: Dimmer Gen4](https://kb.shelly.cloud/knowledge-base/shelly-dimmer-gen4)
- Shelly 1L Gen4 (no-neutral switch): neutral not needed, max 200 W, Shelly Bypass included and wired in parallel with the load, external protection 10 A, 41.2x42.2x16 mm, Zigbee and Matter — [Shelly KB: 1L Gen4](https://kb.shelly.cloud/knowledge-base/shelly-1l-gen4)
- Sonoff ZBMINIR2 (Zigbee): detach relay mode, external trigger modes (edge, pulse, following), turbo radio mode, inching, OTA; ZBMINIL2 is the no-neutral version and is an "End Device" that "cannot function as a Zigbee router"; the dual-channel MINI-ZB2GS has per-outlet detach relay mode — [Zigbee2MQTT: ZBMINIR2](https://www.zigbee2mqtt.io/devices/ZBMINIR2.html), [Zigbee2MQTT: ZBMINIL2](https://www.zigbee2mqtt.io/devices/ZBMINIL2.html), [Zigbee2MQTT: MINI-ZB2GS](https://www.zigbee2mqtt.io/devices/MINI-ZB2GS.html)
- Sonoff's November 2026 store lists a renewed range: MINI ONE Zigbee (MINI-ZB1GS) 17.90, MINI ONE PM Zigbee (MINI-ZB1GSP) 18.50, MINI Dry Zigbee (MINI-ZBD) 13.50, MINI ONE Matter over Wi-Fi (MINI-1GS) 17.90, SwitchMan ZBM5 80-type EU Zigbee wall switch 24.90, M5 Matter 80-type wall switch 18.90 (currency not stated) — [sonoff.tech products JSON](https://sonoff.tech/products.json?limit=250)
- Aqara single switch module T1 with neutral (SSM-U01): power/energy metering, switch type toggle/momentary — [Zigbee2MQTT: SSM-U01](https://www.zigbee2mqtt.io/devices/SSM-U01.html)
- Aqara EU store: Light Switch H2 EU 4-pack 147.99, Dual Relay Module T2 4-pack 125.99 — [eu.aqara.com products JSON](https://eu.aqara.com/products.json?limit=250)
- Pre-flashed Tasmota in-wall modules and switches for the EU: NOUS B1T/B2T (1/2-channel modules), B3T (2-channel/curtain with metering), L13T, L1T/L2T touch switches; Athom SW011EU/SW012EU key switches and no-neutral SW111EU/SW112EU, SD01-EU dimmer; Athom Mini Relay (RS01C3) and 2CH relay board are also "Made for ESPHome" — [Blakadder pre-flashed list](https://templates.blakadder.com/preflashed.html), [ESPHome Made-for-ESPHome list](https://devices.esphome.io/made-for-esphome/)
- Shelly wall products listed in the KB: Wall Display, Wall Display X2, X1i, X2i, XL; "Shelly Wall Switch (1, 2 and 4)" accessories; Shelly Bypass — [Shelly KB device index](https://kb.shelly.cloud/knowledge-base/devices)

### Inferences
- What Gen4 adds over Gen3, per Shelly's own pages: a new processor with Wi-Fi 6, Bluetooth 5 and an 802.15.4 radio, so each device can run either as a Wi-Fi/MQTT device or as a Zigbee (router) or Matter device. For an MQTT hub the Wi-Fi mode with stock MQTT stays the simplest.
- Shelly Mini modules require a ≤ 10 A breaker, which many 16 A socket circuits do not have; this is a real constraint for in-wall use.
- DIY relay/SSR boards (ESP32 + relay module) are not CE-assessed as mains appliances and lack approved enclosures and fusing; the certified modules above cost EUR 10-20, so there is no economic case for home-made mains switching.

### Gaps
- Depth/fit claims for 60 mm round boxes were not checked beyond the housing dimensions; EU prices for Shelly Gen4 modules were not fetched (Shelly store rate-limited). ESTIMATE (unverified): Shelly 1 Mini Gen4 ~EUR 12-15, 1PM Mini Gen4 ~EUR 15-18, 2PM Gen4 ~EUR 25-30, Dimmer Gen4 ~EUR 30-35.
- Sonoff MINIR4/MINIR4M, Aqara H1/H2 EU wall switches, Zemismart/Moes/BSEED wall switches and the Sonoff TX Ultimate were not researched in detail after the scope change.

## 8. Safety and regulation for an EU buyer

### Takeaway
RED cybersecurity requirements (Delegated Regulation (EU) 2022/30) have applied to internet-connected radio equipment since August 2025 and will be replaced by the Cyber Resilience Act, whose main obligations apply from 11 December 2027 (vulnerability reporting from 11 September 2026). Neither law forbids owners from flashing their own devices, and the Commission formally abandoned the separate "software upload" rule in January 2026, but manufacturers have been locking firmware (Shelly Matter devices cannot be converted) and vendors decline warranty for third-party firmware.

### Cited Findings
- The Commission adopted Delegated Regulation (EU) 2022/30 in October 2021 "activating Articles 3(3)(d), (e) and (f) for certain categories of radio equipment"; it "entered into application in August 2025" — [European Commission: RED](https://single-market-economy.ec.europa.eu/sectors/electrical-and-electronic-engineering-industries-eei/radio-equipment-directive-red_en)
- Scope: Article 3(3)(d) (network protection) "shall apply to any radio equipment that can communicate itself over the internet"; 3(3)(e) (personal data and privacy) applies to internet-connected equipment, childcare equipment, toys and wearables capable of processing personal data; 3(3)(f) (fraud) applies to internet-connected equipment enabling money transfers; the original text set application "from 1 August 2024" (the Commission page above gives August 2025 as the actual start) — [Delegated Regulation (EU) 2022/30](https://eur-lex.europa.eu/eli/reg_del/2022/30/oj/eng)
- The planned rule on software uploads to radio equipment (Article 3(3)(i)) was abandoned on 20 January 2026: "there is currently insufficient evidence to justify regulation related to software-reconfigurable radio systems" — [European Commission: RED](https://single-market-economy.ec.europa.eu/sectors/electrical-and-electronic-engineering-industries-eei/radio-equipment-directive-red_en)
- Delegated Regulation (EU) 2026/339 (adopted 16 February 2026, published 29 April 2026) repeals 2022/30 "to avoid overlapping legislative requirements"; "The repeal will be effective as of 11 December 2027" — [European Commission: RED](https://single-market-economy.ec.europa.eu/sectors/electrical-and-electronic-engineering-industries-eei/radio-equipment-directive-red_en)
- Cyber Resilience Act: "entered into force on 10 December 2024"; "reporting obligations to apply as of 11 September 2026"; "The main obligations introduced by the Act will apply from 11 December 2027"; covers "software or hardware products with digital elements"; compliance shown by CE marking; has "a special approach to free and open-source software" — [European Commission: Cyber Resilience Act](https://digital-strategy.ec.europa.eu/en/policies/cyber-resilience-act)
- EU legal guarantee: "You always have the right to a minimum 2-year guarantee from the moment you received the goods"; "If a defect becomes apparent within 1 year of delivery, you don't have to prove it existed at the time of delivery"; a commercial guarantee "can never exclude or reduce your rights under the minimum 2-year guarantee"; suppliers of goods with digital content "should also ensure that you get the necessary updates" — [Your Europe: guarantees](https://europa.eu/youreurope/citizens/consumers/shopping/guarantees-returns/index_en.htm)
- Vendor positions on third-party firmware: Cudy: "We do not assume any warranty for products that have installed any software" not released by Cudy; Tasmota's Shelly converter: "There is no way back to Shelly firmware"; by contrast "Made for ESPHome" devices must let "the user ... push their own builds over the air" — [Cudy OpenWrt page](https://www.cudy.com/blogs/faq/openwrt-software-download), [tasmota/mgos32-to-tasmota32](https://github.com/tasmota/mgos32-to-tasmota32), [ESPHome: Made for ESPHome](https://esphome.io/guides/made_for_esphome.html)
- Electrical limits printed by the manufacturer: Shelly Plug S Gen3 2500 W and Plug PM Gen3 3680 W are for resistive loads only; Shelly 1PM Mini Gen4 requires external protection of max 10 A (B or C); Dimmer Gen4 and Gen3 require a 16 A B/C breaker; ambient temperature limit 40 °C — [Shelly KB: Plug S MTR Gen3](https://kb.shelly.cloud/knowledge-base/shelly-plug-s-mtr-gen3), [Shelly KB: Plug PM Gen3](https://kb.shelly.cloud/knowledge-base/shelly-plug-pm-gen3), [Shelly KB: 1PM Mini Gen4](https://kb.shelly.cloud/knowledge-base/shelly-1pm-mini-gen4), [Shelly KB: Dimmer Gen4](https://kb.shelly.cloud/knowledge-base/shelly-dimmer-gen4)
- Germany as an example of "electrician required": under § 13 NAV, work on the electrical installation behind the house connection may, apart from the grid operator, only be carried out by an installation company registered with a grid operator ("nur durch ein in ein Installateurverzeichnis eines Netzbetreibers eingetragenes Installationsunternehmen"); CE, VDE or GS marks create a presumption that materials comply — [§ 13 NAV](https://www.gesetze-im-internet.de/nav/__13.html)

### Inferences
- Plug-in devices, bulbs, battery sensors and clip-on curtain robots involve no fixed-installation work, so the electrician question does not arise for the rental-friendly path; anything behind a wall plate does, and in Germany formally requires a registered installer (other countries differ and were not checked).
- Flashing third-party firmware changes the software the manufacturer's CE/RED declaration was issued for; the practical consequences for a private owner are loss of the manufacturer's commercial warranty and support, not illegality. The statutory 2-year guarantee still covers defects that existed at delivery, but a seller can dispute a claim if the modification plausibly caused the fault. Buying devices that ship with open firmware (Athom, NOUS) or an open local API (Shelly) avoids the issue.
- The CRA will push vendors toward signed firmware and defined update periods from December 2027; expect more locked bootloaders, which strengthens the case for "open by design" products.

### Gaps
- The postponement act (Delegated Regulation (EU) 2023/2444, moving the date to 1 August 2025) and the EN 18031 harmonised-standards decision (EU) 2025/138 could not be fetched from EUR-Lex (empty pages); the August 2025 date is confirmed by the Commission page only.
- No vendor statement from Shelly on warranty after custom firmware was found.
- Electrician/DIY rules for other EU countries (France NF C 15-100, Poland SEP, Netherlands, Czechia) were not researched.
- CRA details on open-source/non-commercial firmware and "substantial modification" were not fetched from the legal text.

## 9. EU availability and typical November 2026 prices

### Takeaway
Manufacturer EU stores (Shelly, IKEA, SwitchBot, Aqara, Sonoff, Philips Hue, Innr) and distributor networks (Home Assistant hardware via BerryBase, mediarath, Reichelt, Domadoo, Botland and others) cover everything except Athom and generic Tuya gear, which ship from China; Black-Friday week pricing was live on 25 November 2026 (Shelly Plug S Gen3 at EUR 9.56 instead of EUR 23.90). EU maker shops sell the DIY side cheaply: a USB Zigbee coordinator from EUR 16.95, ESP32-C3 boards at EUR 3.95 and sensor modules at EUR 2-8.

### Cited Findings
- Shelly EU store prices in EUR on 2026-11-25: Plug S MTR Gen3 9.56 (list 23.90); Duo Bulb E27 Gen3 2-pack 15.12 (18.90); Multicolor Bulb E27 Gen3 4-pack 27.92 (34.90); BLU Door/Window ZB 14.32 (17.90); i4 Gen4 16.90 — [shelly.com product JSON](https://www.shelly.com/products/shelly-plug-s-gen3.json), [shelly.com products JSON](https://www.shelly.com/products.json?limit=250)
- IKEA Germany prices on 2026-11-25: see sections 1-3 and 6 (GRILLPLATS EUR 8.99, KAJPLATS bulbs EUR 4.99-9.99, BILRESA EUR 4.99-5.99, sensors EUR 6.99-7.99, DIRIGERA EUR 79.99) — [IKEA DE: Smart Home Steuerung](https://www.ikea.com/de/de/cat/smart-home-steuerung-36814/), [IKEA DE: LED-Leuchtmittel](https://www.ikea.com/de/de/cat/smart-home-led-leuchtmittel-36813/)
- SwitchBot EU store: Curtain 3 EUR 89.99 — [eu.switch-bot.com](https://eu.switch-bot.com/products/switchbot-curtain-3)
- Home Assistant Connect ZBT-2 (Zigbee/Thread coordinator, Silicon Labs MG24): recommended price "EUR €45 (including taxes)"; used "by many people with Zigbee2MQTT"; EU retailers include BerryBase, Mediarath, Reichelt, Seeed Studio, SmartDomo (DE), Domadoo, Kubii (FR), Hashop, Robbshop (NL), Botland (PL), Alza (CZ) — [Home Assistant Connect ZBT-2](https://www.home-assistant.io/connect/zbt-2/)
- Sonoff coordinators in the Sonoff store (currency not stated in the JSON): Dongle Plus CC2674P10 Zigbee/Thread USB dongle (Dongle-PP10) 28.70 (regular 35.90); Dongle Max CC2674P10 Zigbee/Thread PoE dongle (Dongle-MP10) 39.10 (regular 48.90) — [sonoff.tech products JSON](https://sonoff.tech/products.json?limit=250)
- SMLIGHT SLZB-06M (EFR32MG21, Ethernet/PoE/USB/Wi-Fi, "Zigbee2MQTT support: Yes"); EU resellers listed include mediarath (DE), Domadoo (FR), hobbyelectronica (NL), pcblab (PL), homebrainz (CZ/SK); no price on the manufacturer page — [SMLIGHT SLZB-06M](https://smlight.tech/product/slzb-06m)
- Home Assistant Voice Preview Edition: "€59 (recommended MSRP, including applicable taxes)", ESP32-S3 + XMOS XU316, dual microphones, ESPHome preloaded with "Fully open-source firmware for both the ESP32 and XMOS chip"; EU distributors include BerryBase, mediarath, Reichelt, Domadoo, Botland, Alza — [Home Assistant Voice PE](https://www.home-assistant.io/voice-pe/)
- Seeed Studio: XIAO ESP32-C3 USD 4.90; XIAO ESP32-C6 (Wi-Fi 6, BLE, Zigbee/Thread) USD 5.20; reSpeaker Lite 2-Mic Array Voice Kit with XIAO ESP32S3 (XMOS XU316, ESPHome firmware integration) USD 29.90; prices exclude shipping and duties, "Ship Tax-Free from Local Warehouses" — [Seeed XIAO ESP32-C3](https://www.seeedstudio.com/Seeed-XIAO-ESP32C3-p-5431.html), [Seeed XIAO ESP32-C6](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html), [Seeed reSpeaker Lite kit](https://www.seeedstudio.com/ReSpeaker-Lite-Voice-Assistant-Kit-p-5929.html)
- Athom ships EU orders from China (4PX; 5-12 working days) and prices in USD — [athom.tech shipping](https://www.athom.tech/shipping)
- EU maker shop hobbyelectronica.nl (Netherlands), fetched 2026-11-25 — Zigbee coordinators: SMLIGHT SLZB-07 USB coordinator EUR 16.95 (was 18.95; 42 in stock; Silicon Labs EFR32MG21 with CP2102 USB-serial, "Works with Zigbee2MQTT and Home Assistant ZHA", Thread/Matter-ready via multipan firmware), SLZB-07 MG24 EUR 28.95, SLZB-07p7 EUR 26.95, SLZB-06 (Ethernet/PoE/USB) EUR 37.95 (was 39.95), SLZB-06M EUR 39.95, SLZB-06p7 EUR 49.95, SLZB-MR1 EUR 54.95; shipping EUR 6.95 for orders under EUR 100, free above — [hobbyelectronica: SLZB-07](https://www.hobbyelectronica.nl/product/slzb-07-zigbee-coordinator-usb/), [hobbyelectronica: SLZB-06](https://www.hobbyelectronica.nl/product/slzb-06-zigbee-ethernet-poe-usb-adapter/), [hobbyelectronica: Zigbee en Matter](https://www.hobbyelectronica.nl/product-categorie/platformen/zigbee-matter/)
- Same shop — ESP32 boards: ESP32-C3 SuperMini EUR 3.95, ESP32-C6 SuperMini EUR 7.95, ESP32-H2 SuperMini EUR 7.95, ESP32-S3 SuperMini EUR 8.95, D1 mini ESP32 (USB-C) EUR 6.95, NodeMCU ESP32 dual-core EUR 10.95 (was 12.95), NodeMCU ESP32 CH340C USB-C EUR 7.95, ESP32-C6 DevKitC 16 MB EUR 9.95, WT32-ETH01 Ethernet board EUR 18.95 — [hobbyelectronica: ESP32](https://www.hobbyelectronica.nl/product-categorie/platformen/esp32/)
- Buying from outside the EU: VAT is payable on purchases from non-EU countries "irrespective of the value of the goods"; if the seller is registered in the Import One Stop Shop (IOSS) VAT is paid at checkout, otherwise the postal operator or courier collects it on delivery and may charge a clearance fee; customs duty is not due where the value "does not exceed €150" — [European Commission: buying goods online from a non-EU country](https://taxation-customs.ec.europa.eu/buying-goods-online-coming-non-european-union-country_en)
- The VAT exemption for imported goods below EUR 22 was abolished from 1 July 2021; IOSS covers distance sales in consignments with an intrinsic value not exceeding EUR 150 — [European Commission: low value consignments](https://taxation-customs.ec.europa.eu/customs-4/customs-procedures-import-and-export/customs-procedures/customs-formalities-low-value-consignments_en)
- OpenWrt-capable routers: OpenWrt's MediaTek Filogic build definitions include Cudy WR3000 (v1), WR3000S/E/H/P, TR3000 (v1 and 256 MB), AP3000, M3000 and GL.iNet GL-MT3000, GL-MT6000, GL-MT2500, GL-X3000; Cudy itself publishes OpenWrt builds/intermediate firmware for its routers — [OpenWrt filogic.mk](https://raw.githubusercontent.com/openwrt/openwrt/main/target/linux/mediatek/image/filogic.mk), [Cudy OpenWrt page](https://www.cudy.com/blogs/faq/openwrt-software-download)

### Inferences
- Black-Friday week (late November) is a good time for the first purchase round: manufacturer stores showed 20-60 % discounts on 25 November 2026.
- For AliExpress and Athom orders, expect VAT to be collected at checkout through IOSS on orders up to EUR 150; "EU warehouse" listings avoid customs handling and usually arrive within a week.

### Gaps
- A flat customs duty of about EUR 3 per item on low-value parcels (the EUR 150 duty exemption being removed) was reported as agreed by the Council in December 2025 for application from 1 July 2026; this could NOT be confirmed in this session (consilium.europa.eu returned HTTP 403 and the Commission pages fetched do not mention it). Treat as unverified until checked.
- No Amazon, AliExpress, idealo or geizhals prices could be fetched; EU-reseller prices for NOUS, Sonoff, Aqara, Zemismart and Moes are missing. EU maker-shop prices come from a single shop (hobbyelectronica.nl); other shops named by vendors as EU distributors (BerryBase, mediarath, Reichelt, Botland, Domadoo) could not be fetched (Shopify throttling or HTTP 403).
- The currency of the Sonoff store's product feed is not stated; its figures (e.g. 28.70 for the Dongle Plus PP10) should not be read as EUR without checking.
- Router prices were not fetched. ESTIMATES (unverified): Cudy WR3000 ~EUR 40-50, Cudy TR3000 ~EUR 45-60, GL.iNet GL-MT3000 (Beryl AX) ~EUR 70-90, GL.iNet GL-MT300N-V2 (Mango, 2.4 GHz only) ~EUR 25-30.

## 10. Phased starter bill of materials for a rented 2-3 room apartment (first round ≤ EUR 100)

### Takeaway
A first round under EUR 100 is achievable in late November 2026: a USB Zigbee coordinator (SMLIGHT SLZB-07, EUR 16.95), a two-pack of bulbs (Zigbee Innr about EUR 30, or Wi-Fi/MQTT Shelly Duo Gen3 EUR 15-19), one metering plug (Shelly Plug S Gen3, EUR 9.56 on Black-Friday offer, EUR 23.90 list), one stick-on Zigbee button (about EUR 10-20, unverified) and two ESP32-C3 boards with a climate and a motion sensor (about EUR 18) come to roughly EUR 87 plus EUR 7-13 shipping. The track-riding curtain robot (EUR 90 class per curtain panel), the LED-strip kit, more sensors and the optional OpenWrt router form round two; voice satellites (EUR 59 for the open reference device) form round three. The total is sensitive to promotions: at list prices the same basket exceeds EUR 100 unless the cheaper Wi-Fi bulbs are chosen.

### Cited Findings
Price anchors verified on 2026-11-25 (source tags are reused in the tables below):
- [S1] SMLIGHT SLZB-07 USB Zigbee coordinator EUR 16.95 (was 18.95); SLZB-07 MG24 EUR 28.95; SLZB-06M EUR 39.95; shipping EUR 6.95 under EUR 100 — [hobbyelectronica: SLZB-07](https://www.hobbyelectronica.nl/product/slzb-07-zigbee-coordinator-usb/), [hobbyelectronica: Zigbee en Matter](https://www.hobbyelectronica.nl/product-categorie/platformen/zigbee-matter/)
- [S2] ESP32-C3 SuperMini EUR 3.95; D1 mini ESP32 USB-C EUR 6.95; ESP32-C6 SuperMini EUR 7.95 — [hobbyelectronica: ESP32](https://www.hobbyelectronica.nl/product-categorie/platformen/esp32/)
- [S3] DHT22 EUR 7.95; DS18B20 EUR 3.95; AM312 mini PIR EUR 2.25; HC-SR501 PIR EUR 2.45 — [hobbyelectronica: Sensors](https://www.hobbyelectronica.nl/product-categorie/sensors/)
- [S4] Generic Zigbee door/window, temperature/humidity and PIR sensors EUR 10.95 each — [hobbyelectronica: Zigbee en Matter](https://www.hobbyelectronica.nl/product-categorie/platformen/zigbee-matter/)
- [S5] Shelly Plug S MTR Gen3 EUR 9.56 (list 23.90) — [shelly.com product JSON](https://www.shelly.com/products/shelly-plug-s-gen3.json)
- [S6] Shelly Duo Bulb E27 Gen3 2-pack EUR 15.12 (list 18.90), 4-pack EUR 27.92 (list 34.90); Shelly BLU Door/Window ZB EUR 14.32 (list 17.90) — [shelly.com products JSON](https://www.shelly.com/products.json?limit=250)
- [S7] Innr Smart Bulb Comfort E27 1100 lm "€29,99 to €44,99 (€15,00 per item)"; Innr Smart Bulb Colour E27 EUR 24.99-39.99 — [innr.com](https://www.innr.com/en/)
- [S8] IKEA KAJPLATS E27 bulbs EUR 4.99-9.99, GRILLPLATS plug EUR 8.99, BILRESA remote EUR 4.99 (all Matter over Thread) — [IKEA DE: LED-Leuchtmittel](https://www.ikea.com/de/de/cat/smart-home-led-leuchtmittel-36813/), [IKEA DE: Steuerung](https://www.ikea.com/de/de/cat/smart-home-steuerung-36814/)
- [S9] Philips Hue Essential E27 colour 3-pack EUR 59.99 — [philips-hue.com DE](https://www.philips-hue.com/de-de/products/smart-light-bulbs)
- [S10] Home Assistant Connect ZBT-2 EUR 45 — [home-assistant.io](https://www.home-assistant.io/connect/zbt-2/)
- [S11] Athom ESPHome EU plug US$9.94; Athom ESP32-C5 EU plug US$15.60 — [athom.tech](https://www.athom.tech/blank-1/esphome-eu-plug), [athom.tech](https://www.athom.tech/blank-1/esp32-c5-dual-band-wi-fi-eu-plug-made-for-esphome)
- [S12] Athom WLED controllers US$7.84-23.80 — [athom.tech WLED](https://www.athom.tech/wled)
- [S13] WS2812B strip EUR 5.95-6.95; 12 V 2 A adapter EUR 11.95; 5 V 2 A adapter EUR 9.95 — [hobbyelectronica: Licht](https://www.hobbyelectronica.nl/product-categorie/licht-2/), [hobbyelectronica: Voeding](https://www.hobbyelectronica.nl/product-categorie/voeding-2/)
- [S14] SwitchBot Curtain 3 EUR 89.99 — [eu.switch-bot.com](https://eu.switch-bot.com/products/switchbot-curtain-3)
- [S15] Home Assistant Voice Preview Edition EUR 59 — [home-assistant.io](https://www.home-assistant.io/voice-pe/)
- [S16] Seeed reSpeaker Lite 2-Mic voice kit with XIAO ESP32S3 US$29.90; XIAO ESP32-C3 US$4.90 — [Seeed reSpeaker Lite](https://www.seeedstudio.com/ReSpeaker-Lite-Voice-Assistant-Kit-p-5929.html), [Seeed XIAO ESP32-C3](https://www.seeedstudio.com/Seeed-XIAO-ESP32C3-p-5431.html)
- [S17] OpenWrt support for Cudy WR3000/TR3000 and GL.iNet GL-MT3000/GL-MT6000 — [OpenWrt filogic.mk](https://raw.githubusercontent.com/openwrt/openwrt/main/target/linux/mediatek/image/filogic.mk), [Cudy OpenWrt page](https://www.cudy.com/blogs/faq/openwrt-software-download)

### Inferences

The tables below are the researcher's proposal built on the anchors above. "EST" marks a price that was NOT verified in this session. Prices in EUR including VAT unless stated; USD store prices exclude shipping and import VAT.

Tier 1 — first end-to-end slice, target ≤ EUR 100 (already owned: two M5Stack ESP32 devices, phones, laptops, soldering tools)

| # | Item | Qty | Unit | Total | Purpose | One alternative | Price status |
|---|---|---|---|---|---|---|---|
| 1 | SMLIGHT SLZB-07 USB Zigbee coordinator (EFR32MG21) | 1 | 16.95 | 16.95 | Radio for Zigbee2MQTT on the hub | SLZB-07 MG24 EUR 28.95, or Home Assistant Connect ZBT-2 EUR 45 (Zigbee or Thread) | [S1] sale price, list 18.95; [S10] |
| 2 | Zigbee E27 bulbs, Innr Smart Bulb Comfort 1100 lm, 2-pack | 1 pack | 29.99 | 29.99 | Ceiling fixtures; mains-powered Zigbee routers; can be bound to the button | Shelly Duo Bulb E27 Gen3 2-pack EUR 15.12 offer / 18.90 list (Wi-Fi, stock MQTT, no binding) | [S7] pack size inferred from "€15,00 per item"; [S6] |
| 3 | Shelly Plug S MTR Gen3 metering plug (12 A, stock MQTT) | 1 | 9.56 | 9.56 | Floor/table lamp or appliance; first MQTT actuator with energy data | Athom ESPHome EU plug US$9.94 + shipping from China (ESPHome pre-flashed) | [S5] Black-Friday price, list 23.90; [S11] |
| 4 | Stick-on Zigbee button/dimmer | 1 | ~12 | ~12 | Sits beside the (always-on) wall switch; bound to the bulb group | Philips Hue dimmer switch (bindable, EST EUR 20-25) or Sonoff SNZB-01P (events only, EST EUR 9-12) | EST (range EUR 9-22) |
| 5 | ESP32-C3 SuperMini board | 2 | 3.95 | 7.90 | ESPHome sensor nodes over MQTT | D1 mini ESP32 USB-C EUR 6.95 (classic ESP32, also suitable as BLE gateway) | [S2] |
| 6 | DHT22/AM2302 temperature + humidity module | 1 | 7.95 | 7.95 | First climate node | DS18B20 EUR 3.95 (temperature only); BME280/SHT4x breakout EST EUR 3-6 | [S3] |
| 7 | AM312 mini PIR module | 1 | 2.25 | 2.25 | First motion node | HC-SR501 PIR EUR 2.45 | [S3] |
| – | Shipping allowance (two or three shops) | – | – | ~7-13 | – | – | EUR 6.95 verified for one shop [S1]; others EST |
| | Tier 1 total | | | ≈ EUR 94-100 | (EUR 86.60 goods + shipping) | | |

Notes on Tier 1:
- Variant B (two plugs, Wi-Fi bulbs): replace line 2 with the Shelly Duo Bulb Gen3 2-pack (EUR 15.12) and take two Shelly plugs (EUR 19.12): goods EUR 81.29, about EUR 88-94 with shipping. Without Zigbee bulbs the button is handled by the hub instead of direct binding.
- Outside promotions (Shelly plug at EUR 23.90 list) variant A rises to about EUR 101 before shipping; use the Shelly Duo bulbs or defer the button to stay under EUR 100.
- IKEA KAJPLATS bulbs (from EUR 4.99) and the GRILLPLATS plug (EUR 8.99) are the cheapest hardware [S8] but are Matter-over-Thread: they only make sense if a Thread border router and a Matter controller are added to the stack.

Tier 2 — Zigbee expansion, curtain and LED strip (buy per need; not a single basket)

| # | Item | Qty | Unit | Total | Purpose | One alternative | Price status |
|---|---|---|---|---|---|---|---|
| 1 | Track-riding curtain robot, Aqara Curtain Driver E1 Track Version (Zigbee 3.0) | 1 per curtain panel | ~70-100 | ~70-100 | Motorise the existing U- or I-rail with position/battery/light in Zigbee2MQTT | SwitchBot Curtain 3 (U Rail) EUR 89.99, BLE via an ESP32 gateway | EST for Aqara; [S14] for SwitchBot |
| 2 | Classic ESP32 board as BLE-to-MQTT gateway (only for the SwitchBot route) | 1 | 6.95 | 6.95 | OpenMQTTGateway within a few metres of the curtain | Reuse an owned M5Stack ESP32 | [S2] |
| 3 | Battery Zigbee sensors (2 door/window, 2 temperature/humidity, 1 motion) | 5 | 10.95 | 54.75 | Adhesive sensors for rooms and windows | Shelly BLU Door/Window ZB EUR 14.32 offer / 17.90 list; Sonoff/Aqara sensors (EUR prices EST 9-20) | [S4] generic models, two types on waitlist; [S6] |
| 4 | Second and third metering plug, Zigbee (also mesh routers) | 2 | ~12 | ~24 | More loads; strengthens the Zigbee mesh | Shelly Plug S MTR Gen3 EUR 9.56-23.90 | EST; [S5] |
| 5 | More bulbs: Innr Comfort E27 3-pack | 1 pack | 44.99 | 44.99 | Remaining ceiling fixtures | Philips Hue Essential E27 colour 3-pack EUR 59.99 | [S7] pack size inferred; [S9] |
| 6 | WLED controller, Athom WS2812B LED Strip Controller (ESP32-C3, pre-flashed) | 1 | US$7.84 | ≈ EUR 7-8 + shipping | Open-firmware LED strip control over Wi-Fi/MQTT | Bare D1 mini ESP32 EUR 6.95 + level shifter, flashed with WLED | [S12]; [S2] |
| 7 | Addressable LED strip WS2812B | 1 | 5.95-6.95 | ~7 | Accent lighting | 12 V or 24 V strip for longer runs (EST EUR 10-20 per 5 m) | [S13] length not stated |
| 8 | Certified plug-in power adapter for the strip (5 V 3-6 A or 12 V 2-5 A, closed housing, CE + test mark) | 1 | ~12-25 | ~12-25 | The only mains part of the LED-strip build | Generic 12 V 2 A adapter EUR 11.95 / 5 V 2 A EUR 9.95 (certification not stated in the listing) | EST for branded unit; [S13] |
| 9 | Optional user-owned OpenWrt-capable Wi-Fi router/AP for an isolated IoT network | 1 | ~40-60 | ~40-60 | Separate 2.4 GHz IoT network behind the landlord/ISP router, local DNS/NTP, firewall to block cloud access | GL.iNet GL-MT3000 (OpenWrt-based stock firmware, EST EUR 70-90) instead of Cudy WR3000/TR3000 | EST price; OpenWrt support verified [S17] |
| 10 | Optional LAN coordinator upgrade, SMLIGHT SLZB-06M (Ethernet/PoE) | 1 | 39.95 | 39.95 | Place the Zigbee radio centrally, away from the hub | Stay on the USB coordinator with a USB extension cable | [S1] |

Tier 3 — voice satellites

| # | Item | Qty | Unit | Total | Purpose | One alternative | Price status |
|---|---|---|---|---|---|---|---|
| 1 | Home Assistant Voice Preview Edition (ESP32-S3 + XMOS, ESPHome, open firmware) | 1 (then 1 per room) | 59.00 | 59.00 | Reference open voice satellite hardware | Seeed reSpeaker Lite 2-Mic kit with XIAO ESP32S3 US$29.90 plus speaker and enclosure | [S15]; [S16] |
| 2 | Pilot on already-owned M5Stack ESP32 devices | 0 | 0 | 0 | Try wake word and audio streaming before buying | M5Stack Atom Echo class device (EST EUR 13-18) | owned / EST |

- Order of operations implied by the scope answers: round 1 proves the whole chain (Zigbee2MQTT + one Zigbee bulb group + one bound button + one MQTT plug + two ESPHome nodes); round 2 starts with the curtain robot after measuring the rail profile; the router is only needed if the building's router cannot be configured.
- Running costs: standby draw of the always-on devices is small (Shelly plug "< 1 W", Shelly bulb idle < 1 W); battery devices use CR2032/CR2477/AAA cells.

### Gaps
- The stick-on Zigbee button price and the Aqara curtain driver price are estimates; a verified EUR price exists only for the SwitchBot Curtain 3 (product price; the U-rail variant price was not separately confirmed).
- The Innr pack sizes (2-pack EUR 29.99, 3-pack EUR 44.99) are inferred from the range and the "per item" figure shown on innr.com.
- Shipping costs for shelly.com, innr.com, eu.switch-bot.com and athom.tech were not verified; Athom orders from China may additionally incur import VAT/handling.
- Router prices (Cudy, GL.iNet) and branded power-adapter prices are estimates.
- No Type E (France/Belgium/Poland/Czechia) variants were priced; Shelly's CEE 7/7 plug fits Type E sockets, other brands need the "FR/Type E" variant.
