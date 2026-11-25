# ADR-0003: The owned Linux laptop is the 24/7 host; compute is tiered

**Status:** Accepted (host choice by the owner) · Proposed (the tiering) · **Date:** 2026-11-25

## Context

The owner chose the Linux laptop already owned as the always-on machine: 16 GB RAM, no dedicated GPU, exact CPU still unknown. The MacBook Pro (M5 Pro, 48 GB) is far faster but leaves the house.

What the research found (November 2026):

- On laptop-class CPUs a 4B-class model at 4-bit generates roughly 5–17 tokens per second and reads a prompt at about 50–200 tokens per second. A prompt of 1,500–2,000 tokens that is not cached therefore costs roughly 8–12 seconds before the first token. With the fixed part of the prompt cached, one measured CPU test reached the tool call in about 1 second and finished in 2–4 seconds.
- People who run a local voice assistant without that discipline report replies of 10–20 seconds or worse on CPU or integrated graphics, against 1–3 seconds on GPU-class hardware. Commands handled without a model answer in well under a second.
- A recent integrated GPU can read prompts several times faster than the CPU beside it (7–9 times on one 2024 Intel laptop), so the benchmark must try both.
- No trustworthy direct measurements on an M5 Pro were found. Estimates scaled from other Apple chips are roughly 13–25 tokens per second for dense 27–31B models and 45–90 for mixture-of-experts models of similar size.
- Dedicated boxes became expensive in 2026 because of memory prices: the base Mac mini is $899 (16 GB), a Mac mini with 48 GB about $2,299, a 16 GB N150 mini PC about $349, a Jetson Orin Nano Super $399. Since the MacBook is already owned, a dedicated box is justified only by availability, not by capability.

## Decision

1. The laptop is the single always-on machine. It runs everything the home needs in order to function: broker, hub, Zigbee bridge, schedules, rules, web app, speech models and a small language model.
2. Compute is tiered. Each tier falls back to the one below with no action from the user.

| Tier | Where | What runs there | Availability |
|---|---|---|---|
| A | Laptop | Deterministic intent matching, schedules, rules. No model. | Always |
| B | Laptop | A 4B-class model for requests the deterministic layer cannot resolve, and for jobs nobody is waiting on (drafting a rule, summaries) | Always; a few seconds with a cached prompt, far slower without |
| C | MacBook, while it is on the home network | A 27B-class model served over the LAN. The hub prefers it when a health check passes. | Sometimes |

3. No dedicated AI box is bought until the Phase 0 benchmark and real use show that it is needed.

Settings for a laptop that acts as a server:

- ignore the lid switch and disable every sleep state;
- cap the battery charge at 40–60% with a firmware charge threshold, which also leaves the battery as a short-outage UPS;
- run every component as a system service that restarts on failure and starts at boot without a login;
- connect it by Ethernet to our own router ([ADR-0021](0021-network-and-security.md)).

## Consequences

- The home never depends on the Mac. The quality of free-form voice requests varies with whether the Mac is at home.
- The deterministic path must cover the common commands in both languages, and voice must acknowledge at once when a slower tier is thinking.
- Both model tiers must look identical to the hub: one interface, an ordered list of endpoints ([ADR-0018](0018-llm-runtime-and-tiers.md)).
- Because the CPU is unknown, no model choice is final until the benchmark has run on the laptop itself.

## Revisit when

- The benchmark shows Tier B is unusable even for jobs nobody is waiting on.
- The owner wants model-quality voice control around the clock. Then a dedicated box replaces Tier C; options and prices are in [bom.md](../bom.md).

Evidence: [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), sections 5, 6 and 10; [platform landscape notes](../../research_notes/Local%20smart%20home%20stack%20options/platform_landscape.md), section 8; [app and security notes](../../research_notes/Local%20smart%20home%20stack%20options/app_remote_security.md), section 10.
