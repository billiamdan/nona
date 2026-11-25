# ADR-0020: Voice pipeline and starting speech stack

**Status:** Proposed · **Date:** 2026-11-25 · Component picks are provisional until measured on the laptop

## Context

Voice must work in English and Russian ([ADR-0006](0006-english-and-russian-voice.md)) on a 16 GB laptop without a GPU ([ADR-0003](0003-laptop-host-tiered-compute.md)), from four kinds of endpoint ([ADR-0007](0007-voice-input-in-stages.md)).

What the research found (November 2026):

- With the wake word detected on the endpoint, a compact speech-to-text model per utterance and streaming speech output, compute is no longer the bottleneck on a CPU-only machine. The pause that marks the end of speech dominates. A rough estimate for a three-second command is 1–2 seconds from the end of speech to the reply. No end-to-end measurement on a laptop CPU was found.
- Whisper large-v3-turbo is the one-model fallback but is roughly ten times slower on CPU and weaker on Russian commands.
- Piper is the practical CPU baseline for speech output and has Russian and English voices. Silero v5 is the documented leader for Russian stress, under a licence that allows personal non-commercial use only.
- Detecting the wake word on the device is the default for ESPHome satellites, and the same detector runs in the Android and Linux satellite apps. Training a custom wake word is officially described as very difficult; community trainers are the practical route.
- A custom hub can be the server side for ESPHome satellites: the `aioesphomeapi` library exposes every voice call, and Home Assistant's own satellite code is the open reference. No other server implementation was found.

## Decision

**Pipeline.** Button or wake word on the endpoint → audio streamed point to point to the hub → end-of-speech detection → speech-to-text for the request's language → intent matcher → an agent only if unresolved ([ADR-0016](0016-deterministic-core.md)) → reply text → text-to-speech → audio back to the endpoint.

**Starting components.** Each sits behind a small interface and can be swapped.

| Stage | Pick | Alternative |
|---|---|---|
| Wake word | microWakeWord on the endpoint; a stock wake word first, a custom one later | openWakeWord on the hub |
| End of speech | Silero VAD on the hub | |
| Speech-to-text, Russian | GigaAM v3 through ONNX | |
| Speech-to-text, English | A Parakeet TDT v3 family model through ONNX | Whisper large-v3-turbo as a single-model fallback |
| Text-to-speech | Piper with one Russian and one English voice | Silero v5 for Russian if stress is poor |
| Satellite protocol | ESPHome native API, with the hub as client through `aioesphomeapi` | |
| Push-to-talk from the app | Audio frames over the app's WebSocket | |

The two speech-to-text models can be served by one existing open-source speech server that already routes Russian to GigaAM and English to Parakeet, or loaded directly in a small Python service. Which of the two is decided in the voice phase.

**Rules.**

- Audio never passes through the MQTT broker and is not stored. A debug switch, off by default, can keep the last few utterances locally for tuning.
- Speech models stay loaded. Replies are streamed sentence by sentence. A simple command gets a short cue instead of a spoken sentence.
- Speech stays on the laptop even when the Mac is at home; this stack has headroom there.
- English device names in Russian speech are matched through Cyrillic aliases and transliterated before synthesis.
- Spoken confirmations of device commands come from templates in code, in the language of the request, not from a model.
- When an agent is needed, the endpoint gets an immediate "thinking" cue.
- No speaker identification in the first version. If added later, it is a personalisation hint and never authentication.

## Consequences

- Deterministic voice commands should feel responsive on the laptop alone.
- Two unproven points get spikes before any purchase: the hub as server for ESPHome satellites, and wake-word detection on the owned hardware.
- If serving ESPHome satellites from our hub proves impractical, the fallback the research found is the open-source xiaozhi-esp32 firmware, which talks to a self-hosted server over WebSocket and supports M5Stack boards.
- Custom wake words are deferred; which names to use is an open question ([open-questions.md](../open-questions.md)).
- The latency estimate must be replaced by a measurement in the voice phase.

## Revisit when

- Measured latency on the laptop is far from the estimate.
- A single bilingual speech-to-text or text-to-speech model proves itself in a listening test.

Evidence: [speech and audio notes](../../research_notes/Local%20smart%20home%20stack%20options/speech_and_audio.md); [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), sections 2 and 8.
