# Open questions

What is not decided or not known yet. Each row has a default, so work is not blocked; answering a question replaces the default and, where it changes the design, gets a decision record.

## Facts the owner can supply

| # | Question | Why it matters | Default until answered | Needed by |
|---|---|---|---|---|
| 1 | What exactly is the Linux laptop: maker and model, CPU, integrated graphics? | Sizes the small model; a recent integrated GPU can read prompts several times faster than the CPU; the maker decides how the battery charge cap is set | A mid-range CPU, no usable GPU; the Phase 0 benchmark measures it | Phase 0 |
| 2 | Which M5 stick is it: M5StickC Plus2 or StickS3? The model is printed on the back. | Pins, whether it has a real speaker, whether it can receive infrared | M5StickC Plus2 | Phase 0 |
| 3 | Which EU country? | Plug type (F or E) and shops | Type F (Schuko) | The first order |
| 4 | Can the apartment's router be configured, or is it the building's or the ISP's? | Whether and when our own router is needed | Not configurable; our own router arrives with Tier 2 | Phase 4 |
| 5 | What are the LED strips: voltage, plain or addressable, any controller already attached? | Which controller to buy | Not bought until known | Tier 2 |
| 6 | Are there appliances with a 433 MHz or infrared remote: remote-controlled sockets, fans, air conditioner, TV? | Gives the CC1101 and infrared path a real job early | None assumed | Phase 0, spike S2 |
| 7 | The curtain track: U-shaped or I-shaped profile, curtain weight, one panel or two opening from the middle? | Which retrofit drive fits, and whether one or two are needed | One panel on a U-rail | Tier 3 |
| 8 | Which other window coverings, if any? | Further drives | None | Later |
| 9 | Who else lives in the home and uses the system? | Microphones in shared rooms need everyone's agreement; whether per-person rights are needed | One household, equal rights for every paired device | Phase 4 |

To answer question 1, run this on the laptop and paste the output:

```bash
lscpu | grep -E 'Model name|^CPU\(s\)'; free -h | head -2; lspci | grep -Ei 'vga|3d|display'; cat /sys/class/dmi/id/sys_vendor /sys/class/dmi/id/product_name
```

## Choices for the owner

| # | Choice | Options | Default until decided | Needed by |
|---|---|---|---|---|
| 10 | Review of the proposed records, [ADR-0012](decisions/0012-mqtt-control-bus.md) to [ADR-0023](decisions/0023-ai-action-safety.md) | Accept, change or reject each | Work proceeds as proposed | Before Phase 0 |
| 11 | Web app toolkit | Svelte (proposed), React, or another | Svelte with Vite and TypeScript | Phase 0 |
| 12 | Language of the app's own text | English only, or English and Russian | English, with device names in both languages | Phase 1 |
| 13 | Disk encryption on the hub laptop | Passphrase at each boot; automatic unlock bound to the laptop's security chip; no encryption | Passphrase at boot, relying on the battery to make reboots rare | Phase 0 |
| 14 | Wake words | One shared name, or one per language to select the language; which names | A stock wake word first | Phase 7 |
| 15 | Notifications on iPhones | In-app only; or encrypted push through Apple's push service, which is an exception to [ADR-0001](decisions/0001-local-only.md) | In-app only | Phase 8 |
| 16 | Control from outside the home | None; a free third-party mesh VPN; a rented relay server | None | Phase 8 |
| 17 | A dedicated AI computer | Stay with laptop plus Mac; buy an always-on box | Stay; decide after Phase 5 | After Phase 5 |

## To be settled by measurement

| # | Question | Settled in |
|---|---|---|
| 18 | Which model runs on the laptop, and is it fast enough for live voice with a cached prompt? | Phase 0, spike S1 |
| 19 | How good are the candidate models at Russian and mixed-language commands? | Phase 0, spike S1 |
| 20 | Do the two M5 devices work under ESPHome, including the CC1101? | Phase 0, spike S2 |
| 21 | Do iPhones and Android phones accept our certificate authority and grant the microphone to the web app? | Phase 0, spike S3 |
| 22 | What is the real voice latency on the laptop, stage by stage? | Phase 4 |
| 23 | Can our hub serve ESPHome voice endpoints, and does the mixed MQTT-plus-native-API node setup work? | Phase 7, spike S4 |
| 24 | Does sound level plus presence detect waking reliably enough, or is the audio classifier needed? | Phase 3 |
