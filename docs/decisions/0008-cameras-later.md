# ADR-0008: Cameras later: vision as an optional perception module

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A: "cameras later, plan the use cases now")

## Context

The brief asks for multimodal local models. Cameras add the most privacy risk and the most compute load of any sensor, and the first features (voice, schedules, complex conditions) do not need them. The 24/7 host is a 16 GB laptop without a GPU ([ADR-0003](0003-laptop-host-tiered-compute.md)), which rules out continuous video analysis.

## Decision

- No cameras in the first phases. Presence comes from radar and motion sensors, sound from microphones.
- The architecture reserves the slot now: a vision module is one more publisher of virtual sensors and one more on-demand tool, so adding it changes no core code ([ADR-0017](0017-declarative-rules-virtual-sensors.md)).
- Rules that apply once cameras arrive:
  - a snapshot on demand or on a trigger, never continuous recording;
  - frames are analysed locally and discarded unless the owner explicitly saves one;
  - no camera in a bedroom or a bathroom;
  - a camera's state is visible (indicator light or physical cover);
  - cameras sit on the IoT network with no internet access.

Use cases planned, in order of value for effort:

| # | Use case | How it would work |
|---|---|---|
| 1 | Checking devices that give no feedback | A snapshot confirms that the curtain really opened or that a lamp on a 433 MHz socket is on |
| 2 | Questions on demand | "Is the window open?", "Did I leave the iron on?": snapshot, local vision model, spoken answer |
| 3 | Meter reading | A camera on the electricity or water meter turns the digits into a consumption sensor |
| 4 | Entrance view | A person at the door raises an event and a notification |
| 5 | Activity-aware lighting | For example, someone reading on the sofa; lowest priority |

Hardware candidates: the spare Android phones first, ESP32-S3 camera boards later.

## Consequences

- Until then, "multimodal" means audio: speech and household sound events.
- Image input is a selection criterion for the larger model on the Mac, not for the small model on the laptop ([ADR-0018](0018-llm-runtime-and-tiers.md)).

## Revisit when

The voice and language-model phases are finished and stable (see [roadmap.md](../roadmap.md)).
