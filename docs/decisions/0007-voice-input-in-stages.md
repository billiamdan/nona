# ADR-0007: Voice input arrives in four stages

**Status:** Accepted (the owner wants all four sources) · Proposed (the order) · **Date:** 2026-11-25

## Context

The owner wants voice from the phone app, from the owned M5 devices as prototypes, from spare Android phones as room satellites, and from dedicated satellites.

What the research found (November 2026):

- Push-to-talk from a web app works on iPhone and Android as long as the page is served over HTTPS.
- Always-listening is Android-only in practice. iOS does not let a third-party app start recording from the background.
- ESPHome's native API is now the main satellite protocol. Open-source satellite apps for Android and for Linux present themselves as ESPHome devices, so one hub-side implementation covers ESP32, Android and Linux endpoints.
- The owned M5 stick has a microphone but only a buzzer, so it can capture a command but not speak a reply. The Cardputer has a microphone and a speaker.
- The reference dedicated satellite costs about €59; a four-microphone board with beamforming is the stronger far-field option on paper.

## Decision

| Stage | Endpoint | Starts listening on | What it needs | Buys |
|---|---|---|---|---|
| 1 | The web app on any phone | A held button | HTTPS; audio over the app's WebSocket | Nothing |
| 2 | The Cardputer, then the M5 stick | A key or button | An ESPHome voice configuration on hardware we own; the hub as satellite server | Nothing |
| 3 | Spare Android phones as room satellites and wall panels | A wake word detected on the phone | An open-source satellite app that speaks the ESPHome protocol, installed over ADB | Nothing |
| 4 | Dedicated satellites | A wake word detected on the device | A purchase, after stage 3 has shown where far-field pickup matters | Later tier |

- Stage 1 uses the app's own audio channel. Stages 2 to 4 share one hub-side implementation of the satellite protocol. All four feed the same pipeline ([ADR-0020](0020-voice-pipeline-and-speech-stack.md)).
- Stage 2 exists to prove the satellite protocol on owned hardware before anything is bought.

## Consequences

- Nothing has to be bought for voice until stage 4.
- iPhones and the iPad are push-to-talk only, by platform design.
- A spare phone used as a satellite must stay in the foreground, powered and pinned, and its battery needs care ([bom.md](../bom.md)).

## Revisit when

Stage 2 shows that serving ESPHome satellites from our hub is impractical. Then stages 3 and 4 need another endpoint path, with a new record.

Evidence: [speech and audio notes](../../research_notes/Local%20smart%20home%20stack%20options/speech_and_audio.md), sections on protocols and satellite hardware; [app and security notes](../../research_notes/Local%20smart%20home%20stack%20options/app_remote_security.md), sections 7 and 8.
