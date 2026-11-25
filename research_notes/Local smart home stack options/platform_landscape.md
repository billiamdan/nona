# Platform landscape: open-source stacks for a fully local, subscription-free smart home with voice and local-LLM control (state as of November 2026)

Research date: 2026-11-25. Version numbers, release dates, star counts and last-push dates labelled "GitHub API" were read from api.github.com on 2026-11-25. Everything else comes from the linked pages as fetched on that date. Statements are paraphrased, not quoted. Items that could only be supported by an older source are marked "(pre-2026 source)". Items marked "(search summary)" come from a search-result summary of the linked page, not from a full read of it. Reddit could not be fetched by the research tooling and the web-search budget ran out late in the session; the affected holes are listed under Gaps.

## 1. Home Assistant as of late 2026: installation types, footprint, Assist and Wyoming, local-LLM plumbing, 2025-2026 changes, documented weaknesses

### Takeaway
Home Assistant (core 2026.11.3, 23 Nov 2026) is the most complete and most actively developed option, and on top of a deterministic local voice pipeline it now has first-party hooks for local models: Ollama, a llama.cpp / OpenAI-compatible integration (since 2026.8), MCP server and client, and AI Task. Its documented costs are a monthly stream of breaking changes, only two supported install types, phone-and-vendor-ecosystem dependence for Matter commissioning and for standard push notifications, and LLM control that is only fast and dependable with a small exposed-entity set, a 12-24 GB GPU and deterministic intent matching in front of the model.

### Cited Findings

**What it is, maturity, activity**
- Latest core release is 2026.11.3, published 2026-11-23; the 2026.11 feature release shipped 2026-11-04 with patch releases on 9, 16 and 23 November — [GitHub releases (API)](https://github.com/home-assistant/core/releases); [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/)
- Repository: Python, Apache-2.0, 92,492 stars, 39,350 forks, 3,837 open issues, last push 2026-11-25 (GitHub API) — [home-assistant/core](https://github.com/home-assistant/core)
- Home Assistant OS latest is 18.5, published 2026-11-06 (GitHub API) — [operating-system releases](https://github.com/home-assistant/operating-system/releases)
- 2026.11 added 38 new integrations in one month and folded the community store HACS into the product as a built-in Marketplace; the standalone HACS installer stops at 2026.11 and custom repositories need a GitHub device sign-in — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/)
- A search summary of the same post reports 5,827 merged pull requests from 557 contributors for 2026.11; those two numbers were not in the part of the page that could be read directly (search summary) — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/)
- The 2025 event recap was headlined with two million homes (pre-2026 source); the 2026 recap (10 April 2026) gives no installation figure and names Home Assistant, ESPHome and Music Assistant as the foundation's projects — [State of the Open Home 2025 recap](https://www.home-assistant.io/blog/2025/04/16/state-of-the-open-home-recap/); [State of the Open Home 2026 recap](https://www.openhomefoundation.org/blog/building-whats-next-state-of-the-open-home-2026/)
- Share of active installations per integration, from the documentation pages in November 2026: MQTT 48.2%, ESPHome 27.5%, Shelly 25.5%, MCP client 8.1%, Tasmota 7.4%, WLED 6% — [MQTT](https://www.home-assistant.io/integrations/mqtt/); [ESPHome](https://www.home-assistant.io/integrations/esphome/); [Shelly](https://www.home-assistant.io/integrations/shelly/); [MCP client](https://www.home-assistant.io/integrations/mcp/); [Tasmota](https://www.home-assistant.io/integrations/tasmota/); [WLED](https://www.home-assistant.io/integrations/wled/)

**Installation types and resource footprint**
- Two installation types are offered: Home Assistant OS (recommended; on Green, Raspberry Pi 4/5, Odroid, x86-64, or as a VM on macOS, Windows, Linux) and Home Assistant Container (Docker). Container has no apps (the former add-ons) and needs manual updates; the page says Thread and Z-Wave need apps and therefore have no out-of-the-box support on Container — [Installation](https://www.home-assistant.io/installation/)
- The Core (Python virtual environment) and Supervised methods and all 32-bit architectures were deprecated on 2025-05-22 with support ending in release 2025.12; they were 2.5% and 3.3% of installations; unsupported means issue reports are no longer accepted (pre-2026 source) — [Deprecation post](https://www.home-assistant.io/blog/2025/05/22/deprecating-core-and-supervised-installation-methods-and-32-bit-systems/)
- Stated minimum for a Raspberry Pi 4/5 running HA OS is 2 GB RAM; no CPU or storage minimum is given — [Installation](https://www.home-assistant.io/installation/)
- Reference appliance Home Assistant Green: Rockchip RK3566 quad-core Cortex-A55 at 1.8 GHz, 4 GB RAM, 32 GB eMMC, about 1.7 W idle and about 3 W under load, MSRP $199 / 179 EUR — [Home Assistant Green](https://www.home-assistant.io/green/)
- The matter.js-based Matter Server (default since June 2026) needs roughly double the RAM of the Python server it replaced, and one specialist site recommends at least 4 GB on a Raspberry Pi that runs Home Assistant — [matter-smarthome.de, 2026-06-23](https://matter-smarthome.de/en/products/home-assistants-new-matter-server-is-a-game-changer/)
- One Hacker News user reported running containerised Home Assistant alongside Zigbee2MQTT, InfluxDB, Telegraf, Grafana and two media servers on a 1 GB Raspberry Pi 3B without issues (2025-12-01, pre-2026 source) — [HN comment](https://news.ycombinator.com/item?id=46110015)

**Assist voice pipeline and the Wyoming protocol**
- The Assist pipeline consists of wake word detection, speech-to-text, a conversation agent and text-to-speech — [OST student project abstract, 2026](https://www.ost.ch/files/daten/abstracts/pdf/3509b232-4aa4-4840-8314-15f71cde761f.pdf)
- The Wyoming integration connects external voice services (speech-to-text, text-to-speech, wake word) to Assist; services are auto-discovered and can also run on another LAN host, added by host name and port — [Wyoming integration](https://www.home-assistant.io/integrations/wyoming/)
- Wyoming itself is a peer-to-peer TCP protocol (one JSON line per event plus optional PCM audio) with event families for audio, speech recognition, synthesis, wake word, voice activity, intent recognition and handling, satellites and timers; listed programs include Faster Whisper, Vosk, whisper.cpp, Piper, openWakeWord, microWakeWord, porcupine and snowboy; it is meant for a trusted network only — [OHF-Voice/wyoming README](https://github.com/OHF-Voice/wyoming)
- Wyoming library latest release v1.11.0, 2026-11-02; MIT; 420 stars (GitHub API) — [OHF-Voice/wyoming releases](https://github.com/OHF-Voice/wyoming/releases)
- Two local speech-to-text options are documented: Speech-to-Phrase (closed set of phrases, under one second on a Green or Raspberry Pi 4, covers only a subset of commands) and Whisper (open-ended, about 8 s on a Raspberry Pi 4, under a second on an Intel NUC). Piper produces about 1.6 s of speech per second on a Raspberry Pi with medium-quality voices — [Fully local assistant docs](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/)
- Speech-to-Phrase is generated from the user's own devices, areas and sentence triggers, takes about 150 ms per command on a Raspberry Pi 5, cannot handle open-ended or wildcard commands and is not meant for LLM conversations; six languages at launch (pre-2026 source, Feb 2025) — [Voice chapter 9](https://www.home-assistant.io/blog/2025/02/13/voice-chapter-9-speech-to-phrase/)
- By default the built-in (non-AI) Assist agent handles a command first and only requests it cannot match go to the configured AI agent; sentence triggers also run before the agent — [AI in Home Assistant, 2025-09-11](https://www.home-assistant.io/blog/2025/09/11/ai-in-home-assistant); [Ollama integration](https://www.home-assistant.io/integrations/ollama/)
- ESP32 voice hardware uses the ESPHome voice_assistant component: the device streams microphone audio to Home Assistant, where Assist processes it; wake word can run on-device through micro_wake_word; it requires Home Assistant 2023.5 or later, and the second microphone channel requires 2026.6.0 or later — [ESPHome voice assistant](https://esphome.io/components/voice_assistant/)
- Voice Preview Edition firmware latest release 26.9.0, 2026-09-17 (GitHub API) — [home-assistant-voice-pe releases](https://github.com/esphome/home-assistant-voice-pe/releases)
- Linux satellites: linux-voice-assistant is described as experimental, speaks the ESPHome protocol (via aioesphomeapi), does on-device wake word with openWakeWord or microWakeWord, and lists a 1 GHz CPU and 512 MB RAM as minimum; v1.1.18, 2026-10-29. The older wyoming-satellite repository is archived (last push 2026-01-24) (GitHub API) — [linux-voice-assistant](https://github.com/OHF-Voice/linux-voice-assistant); [wyoming-satellite](https://github.com/rhasspy/wyoming-satellite)
- The Android companion app gained experimental on-device wake word detection in 2026.3, with a noticeable battery cost — [2026.3 release post](https://www.home-assistant.io/blog/2026/03/04/release-20263/)

**How local LLMs plug in**
- Ollama integration (introduced 2024.4): provides conversation agents and AI Task entities; defaults are model `qwen3:4b-instruct`, context window 8192 tokens (minimum 2048, and it has to hold the exposed entities when device control is on), 20 history messages, keep-alive -1, thinking off. Tool sets are called AI tools: Device control (exposed entities only) and Home Assistant (everything, administrators only, not usable from voice satellites). Only models with tool support can use AI tools — [Ollama integration](https://www.home-assistant.io/integrations/ollama/)
- llama.cpp integration (new in 2026.8): uses a local llama.cpp server or any OpenAI-compatible chat-completions endpoint (llama-cpp-python, vLLM, LM Studio are named) as a conversation agent, with base URL, optional API key, model, instructions and AI tools; Bronze quality scale, 1,218 active installations; the page lists only a conversation agent entity — [llama.cpp integration](https://www.home-assistant.io/integrations/llama_cpp/); [2026.8 release post](https://www.home-assistant.io/blog/2026/08/05/release-20268/)
- LiteLLM integration (new in 2026.8): a LiteLLM proxy as conversation agent, one agent per configured model — [2026.8 release post](https://www.home-assistant.io/blog/2026/08/05/release-20268/)
- LLM API internals: an LLM API is a set of tools plus a prompt; built-in APIs are `assist` (shown as Device control, no administrative actions) and `homeassistant` (registries, logs, automations, config entries; administrators only). Integrations contribute tools through an `llm.py` platform; an intent only becomes a tool when wrapped as an intent tool; custom integrations can register whole APIs. Tools carry MCP-style annotations (read-only, destructive, idempotent, open-world). Several developer deprecations have deadlines in 2027.3, 2027.10 and 2027.11 — [LLM API developer docs](https://contributors.home-assistant.io/core/internals/llm-api/)
- MCP server (introduced 2025.2): Home Assistant exposes its AI tool sets at `/api/mcp` (and one URL per tool set, for example `/api/mcp/assist`), served over Streamable HTTP, with OAuth 2.0 + PKCE, an optional read-only mode and an administrator-only default. Tools, prompts and one resource (a context snapshot of exposed entities) are supported; sampling and notifications are not. Cloud AI apps need Home Assistant reachable from the internet; local clients such as coding agents connect by URL — [MCP server integration](https://www.home-assistant.io/integrations/mcp_server/); [LLM API developer docs](https://contributors.home-assistant.io/core/internals/llm-api/)
- MCP client (introduced 2025.2): gives a conversation agent the tools of an external MCP server, trying Streamable HTTP first and falling back to SSE; stdio servers need a proxy; only tools are supported (no prompts, resources, sampling, notifications); the tool list refreshes every 30 minutes — [MCP client integration](https://www.home-assistant.io/integrations/mcp/)
- AI Task (documentation says introduced 2025.7; the 2025.8 release post presents it as new): actions to generate data (text or structured output, with camera or file attachments), generate images and evaluate; provided by OpenAI, Gemini and Ollama among others. An AI task has no conversation, uses no AI tools and cannot control devices — [AI Task](https://www.home-assistant.io/integrations/ai_task/); [2025.8 release post](https://www.home-assistant.io/blog/2025/08/06/release-20258/)

**What the 2025-2026 releases changed for local voice and LLM control**
- February 2025: Speech-to-Phrase; MCP integrations; recent Assist commands shared with the LLM; LLM replies streamed into chat from 2025.3 (Ollama and OpenAI first); a microWakeWord add-on (pre-2026 source) — [Voice chapter 9](https://www.home-assistant.io/blog/2025/02/13/voice-chapter-9-speech-to-phrase/)
- June 2025: continued conversation when the LLM ends with a question; a start-conversation action; on-device wake words that can be enabled and disabled on the fly; Voice PE firmware CPU load in the heaviest case down from 72% to 35%; a media search-and-play intent; Piper languages 34 to 39; Speech-to-Phrase planned to grow from 6 to 21 languages (pre-2026 source) — [Voice chapter 10](https://www.home-assistant.io/blog/2025/06/25/voice-chapter-10/)
- July-August 2025: AI Task; AI suggestions for automation names and labels (the post suggests a local LLM because the full automation is sent to the model); streaming text-to-speech for Piper, then for the cloud voices; OpenRouter; sub-entries so one Ollama server can back many agents (pre-2026 source) — [2025.8 release post](https://www.home-assistant.io/blog/2025/08/06/release-20258/)
- September 2025: published benchmark showing streaming TTS cutting time-to-first-audio from 5.31 s to 0.56 s with local Piper and from 6.62 s to 0.51 s with the cloud voices (Ollama gemma3:4b on an RTX 3090, Piper on an i5); statement that local models using 8 GB or more of VRAM are close to cloud models (pre-2026 source) — [AI in Home Assistant, 2025-09-11](https://www.home-assistant.io/blog/2025/09/11/ai-in-home-assistant)
- October 2025: up to two wake words and pipelines per satellite (for example one fully local, one with an LLM); an ask-question action for automations; a short beep instead of a spoken confirmation for same-area actions (not for LLM pipelines); new intents; improved fuzzy sentence matching for English only (pre-2026 source) — [Voice chapter 11](https://www.home-assistant.io/blog/2025/10/22/voice-chapter-11)
- 2026.4: the Assist dialog shows the agent's thinking steps and tool calls (desktop web only) — [2026.4 release post](https://www.home-assistant.io/blog/2026/04/01/release-20264/)
- 2026.6: patch 2026.6.3 sorts aliases in LLM prompts to keep prefix caching stable; OpenThread Border Router 1.4 left beta — [2026.6 release post](https://www.home-assistant.io/blog/2026/06/03/release-20266/)
- 2026.8: llama.cpp and LiteLLM integrations; the word "advanced" removed in about 43 places as part of an approachability push — [2026.8 release post](https://www.home-assistant.io/blog/2026/08/05/release-20268/)
- 2026.9: LLM tool names are now prefixed with the providing integration's domain (breaking for prompts that name tools); a new cloud speech-to-text engine (Soniox) is tested in Labs for subscribers — [2026.9 release post](https://www.home-assistant.io/blog/2026/09/02/release-20269/)
- 2026.10: the AI settings page gained an MCP card with URLs to copy into AI apps, administrator-only by default; apps that run their own MCP server are discovered automatically; KNX ships its own AI tools; ESPHome devices can open the connection to Home Assistant themselves — [2026.10 release post](https://www.home-assistant.io/blog/2026/10/07/release-202610/)
- 2026.11: agents that use the Home Assistant LLM API can read the system log; the MCP server can be restricted to read-only tools; an AI conversation can only be continued by the user who started it — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/)
- The blog's Assist category shows no numbered Voice chapter after chapter 11 (22 October 2025) — [Assist category](https://www.home-assistant.io/blog/categories/assist/)

**Documented weaknesses**
- Upgrade breakage: the 2026.11 post lists at least 44 backward-incompatible entries (the readable part ends inside the 45th), including an entity-naming change that requires custom-card updates, MQTT entities without a unique ID no longer creating devices, a newer Z-Wave server requirement and Tuya multi-channel devices being renamed; 2026.9 lists 8; 2026.4 removed the MQTT `object_id` option after six months of deprecation — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/); [2026.9 release post](https://www.home-assistant.io/blog/2026/09/02/release-20269/); [2026.4 release post](https://www.home-assistant.io/blog/2026/04/01/release-20264/)
- Complexity and upgrade fear, as reported by Hacker News commenters in May 2025: a steep learning curve built by and for IT people, configuration drifting from YAML to GUI with breaking changes along the way, a Supervisor that is hard to run outside supported setups, and one user still on a 2020 version for fear of updates; other commenters report years of trouble-free updates (pre-2026 source) — [HN thread, 2025-05-17](https://news.ycombinator.com/item?id=44011381)
- Entity exposure: the best-practice page says to expose only the bare minimum, because every extra name slows sentence matching and, with an LLM, raises the per-request cost through a larger context; it gives no number. The Ollama page adds that smaller models make more mistakes and that some small models chat worse with tools enabled (workaround: a second agent without tools) — [Assist best practices](https://www.home-assistant.io/voice_control/best_practices/); [Ollama integration](https://www.home-assistant.io/integrations/ollama/)
- Conflict: a November 2026 guide states that the integration documentation recommends fewer than 25 exposed entities — [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/); the Ollama page as fetched on 2026-11-25 contains no numeric limit — [Ollama integration](https://www.home-assistant.io/integrations/ollama/)
- Matter commissioning depends on a phone ecosystem: adding a device uses the phone's Bluetooth through the companion app; the Android app needs Google Play Services (the F-Droid build cannot add Matter devices), iOS needs 16 or later, the Mac app cannot do it; the Matter app on HA OS is the only supported option and a standalone Docker Matter server is unsupported — [Matter integration](https://www.home-assistant.io/integrations/matter/)
- Thread: the preferred-network function is not completely implemented, each vendor forms its own Thread network, local announcements do not carry credentials, and Matter updates do not work for Thread devices on networks with Apple border routers — [Thread integration](https://www.home-assistant.io/integrations/thread/); [Matter integration](https://www.home-assistant.io/integrations/matter/)
- Latency and reliability of LLM control: see section 8 for the community numbers.

**Features that depend on the Nabu Casa cloud or on other vendors' clouds**
- The subscription (Home Assistant Cloud, to be renamed Home Assistant Link in 2026.12) provides remote access, Alexa and Google Home linking, online speech-to-text, online backup storage and human support; the vendor states the platform works without it and that most subscription profit funds the Open Home Foundation — [Rename announcement, 2026-10-02](https://home-assistant.io/blog/2026/10/02/big-tech-ruined-the-cloud-so-were-renaming-ours/)
- The 2026.9 post describes the subscription as including faster and more accurate speech-to-text and text-to-speech — [2026.9 release post](https://www.home-assistant.io/blog/2026/09/02/release-20269/)
- One secondary report gives the price as US$6.50 per month (search summary) — [unwire.hk, 2026-10-03](https://unwire.hk/2026/10/03/home-assistant-cloud-renamed-link/life-tech/)
- Standard companion-app push notifications go through Google's Firebase Cloud Messaging, are limited to 500 per device per day and are not encrypted on Firebase — [Companion app notification details](https://companion.home-assistant.io/docs/notifications/notification-details/)
- A local-push alternative over the WebSocket API exists: on iOS only while on the internal URL with configured SSIDs, on Android with a persistent notification and a possible battery cost — [Companion app local push](https://companion.home-assistant.io/docs/notifications/notification-local/)

**Parts reusable without adopting the platform**
- Wyoming services (Faster Whisper, Piper, openWakeWord, Speech-to-Phrase) are ordinary network services behind a documented protocol — [OHF-Voice/wyoming README](https://github.com/OHF-Voice/wyoming)
- Piper (GPL-3.0, v1.8.0, 2026-09-04), Speech-to-Phrase (Apache-2.0, v1.4.3, 2026-04-03), Faster Whisper server (MIT), the intent sentence corpus (CC-BY-4.0), the hassil matcher (Apache-2.0) and the microWakeWord trainer (Apache-2.0) are separate repositories (GitHub API) — [piper1-gpl](https://github.com/OHF-Voice/piper1-gpl); [speech-to-phrase](https://github.com/OHF-Voice/speech-to-phrase); [wyoming-faster-whisper](https://github.com/OHF-Voice/wyoming-faster-whisper); [intents](https://github.com/OHF-Voice/intents); [hassil](https://github.com/OHF-Voice/hassil); [micro-wake-word](https://github.com/OHF-Voice/micro-wake-word)
- The Matter server is explicitly described as suitable for use in other projects and ships standalone Docker and npm instructions — [matterjs-server README](https://github.com/matter-js/matterjs-server)
- The MQTT discovery schema is consumed by other controllers too (openHAB, Domoticz) — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)

### Inferences
- The local-LLM surface in Home Assistant matured through 2026 mostly on the plumbing side (OpenAI-compatible endpoint support, MCP, tool naming and permissions, prompt-cache stability). The speech side had no headline release in 2026 comparable to the 2025 Voice chapters; the visible 2026 voice work is on the paid cloud engine and on satellites.
- For the stated hard constraints, Home Assistant itself passes (it runs offline and without an account), but three convenience paths quietly violate them unless avoided: standard push notifications (Firebase), phone-based Matter commissioning (Google Play Services or Apple), and remote access via the subscription.
- The matter.js server refuses devices with development or test certificates out of the box, and Tasmota's Matter implementation uses development vendor IDs, so DIY Matter devices are likely to need a non-default setting (links: Home Assistant Matter post in section 6 and Tasmota Matter docs in section 3).
- The admin-only "Home Assistant" tool set plus system-log access makes Home Assistant a strong target for a coding-agent style assistant over MCP, which is a different use from low-latency voice control.

### Gaps
- No 2026 figure for active installations was found; the last official number is the 2025 "two million" headline.
- The 2026.11 pull-request and contributor counts come from a search summary only.
- No official measurement of idle RAM or CPU for HA OS or Container was found; footprint evidence is the 2 GB minimum, the Green specification and one user anecdote.
- The subscription price was only found in a secondary source.
- Whether the llama.cpp integration provides an AI Task entity could not be confirmed either way from its page.
- Release posts for 2025.12 and several 2025 monthly releases were not read; the 2025 timeline relies on the Voice chapters and two blog posts.

## 2. ESPHome as of late 2026 used without Home Assistant

### Takeaway
Running ESPHome without Home Assistant is practical and documented when devices talk MQTT: entities, Home Assistant-style discovery, OTA, provisioning and the on-device web UI all work with no hub. The features that make ESPHome nodes more than sensors (voice assistant, Bluetooth proxy, infrared/RF, serial and Z-Wave proxies) exist only on the native API, whose documented clients are the ESPHome tool, Home Assistant and ioBroker; a third-party controller can use that API through the MIT-licensed aioesphomeapi library, which exposes all of those calls, at the cost of tracking a fast-moving library driven by Home Assistant's needs.

### Cited Findings

**What it is, maturity, activity**
- ESPHome is an Open Home Foundation project that generates firmware from YAML for ESP32, ESP8266, RP2040/RP2350, BK72xx, RTL87xx, nRF52 and LN882 chips plus a desktop (host) target, with hundreds of components and local operation without cloud accounts or external servers — [esphome.io](https://esphome.io/)
- Latest release 2026.11.0, published 2026-11-18, with 1,463 pull requests from over 240 contributors and over thirty new components — [GitHub releases (API)](https://github.com/esphome/esphome/releases); [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)
- Repository: C++, 12,166 stars, 5,841 forks, last push 2026-11-25 (GitHub API) — [esphome/esphome](https://github.com/esphome/esphome)
- The Device Builder reached 1.0.0 in 2026.6.0 and replaced the legacy dashboard; it adds a visual component and automation builder, a YAML editor, a component catalogue, a pin viewer and a firmware job queue, and is the default in the Home Assistant ESPHome add-on; the post does not describe a standalone install — [ESPHome 2026.6 post](https://esphome.io/blog/2026/06/17/esphome-2026-6/)
- Breaking changes are routine: 2026.11 requires a relation field on sub-devices, makes `http_request` actions non-blocking, raises the minimum ESP-IDF to 5.1 and drops MQTT messages above the size limit unless `max_payload_size` is raised; 2026.6 removed the `custom_components` auto-loader — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/); [ESPHome 2026.6 post](https://esphome.io/blog/2026/06/17/esphome-2026-6/)

**MQTT component (the Home Assistant-free path)**
- Supported on ESP32, ESP8266, BK72xx, LN882H and RTL87xx. Default topics are `<prefix>/<component_type>/<name>/state` and `.../command`, with a retained `online` on `<prefix>/status` and `offline` as last will — [MQTT component](https://esphome.io/components/mqtt/)
- Home Assistant-style discovery is on by default (prefix `homeassistant`, retained, with options for unique-ID and object-ID generation and abbreviations); separately the device answers `esphome/discover` so the ESPHome tooling can find it for logs and OTA — [MQTT component](https://esphome.io/components/mqtt/)
- Without the native API the `api:` block must be removed or its `reboot_timeout` set to `0s`, otherwise the node reboots every 15 minutes because no API client connected — [MQTT component](https://esphome.io/components/mqtt/); [Native API component](https://esphome.io/components/api/)
- TLS for MQTT is ESP32-only; MQTT over WebSocket needs ESP32 with ESP-IDF; triggers for plain and JSON messages and publish actions are available; a log topic can carry device logs — [MQTT component](https://esphome.io/components/mqtt/)
- 2026.11 made MQTT connect as soon as the network is up after boot — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)

**Native API and aioesphomeapi**
- The native API is a custom TCP protocol on protocol buffers with optional Noise encryption (32-byte base64 key); the documentation states that only the ESPHome tool, Home Assistant and ioBroker use it and points to aioesphomeapi as the Python implementation — [Native API component](https://esphome.io/components/api/)
- Stated advantages over MQTT: about a tenth of the message size for a binary sensor state, one-click setup in Home Assistant, no broker as single point of failure because each node is its own server, tighter protocol control and latency of a few milliseconds; the page also says MQTT will not be removed — [Native API component](https://esphome.io/components/api/)
- Limits: 4-8 simultaneous connections by default depending on platform (compile-time, up to 20), roughly 500-1000 bytes of RAM per connection, 100 ms default batch delay; since 2026 a device can also open the connection itself (`outgoing_connection`, encryption mandatory) — [Native API component](https://esphome.io/components/api/)
- aioesphomeapi: MIT, v46.30.1 published 2026-11-16, 202 stars, last push 2026-11-24 (GitHub API); the README shows connecting, listing entities and subscribing to states, ships log and discovery command-line tools, and notes that password authentication was removed on the device side in ESPHome 2026.1.0 — [aioesphomeapi](https://github.com/esphome/aioesphomeapi); [aioesphomeapi releases](https://github.com/esphome/aioesphomeapi/releases)
- The client class exposes entity commands (cover, fan, light, switch, climate, number, select, lock, valve, media player and others), user-defined service execution, log subscription, Bluetooth LE advertisement subscription and full GATT client calls, voice-assistant subscription with audio, event, announcement and timer calls, Z-Wave proxy, infrared/RF and serial proxy calls, and Noise key provisioning — [aioesphomeapi client.py](https://github.com/esphome/aioesphomeapi/blob/main/aioesphomeapi/client.py)

**Features that only work over the native API**
- Voice assistant: the component is defined as streaming audio to Home Assistant for processing by Assist and states a Home Assistant version requirement — [ESPHome voice assistant](https://esphome.io/components/voice_assistant/)
- Bluetooth proxy: defined as extending Home Assistant's Bluetooth reach; active connections are supported on ESP32 and RP2040/RP2350 (ESP32 up to 9 slots, 5 recommended), BK72xx and LN882x are advertisement-only, each slot costs about 1 KB RAM; MQTT is not mentioned as a transport — [Bluetooth proxy](https://esphome.io/components/bluetooth_proxy/)
- Home Assistant-side features delivered through the API: infrared and RF proxy, serial proxy, event firing, tag scanning, importing Home Assistant entity state into the device — [ESPHome integration](https://www.home-assistant.io/integrations/esphome/)
- A community Node-RED node for the native API was announced as alpha without Bluetooth proxy and encryption support (search summary) — [Node-RED forum](https://discourse.nodered.org/t/announce-node-red-contrib-esphome/74932)

**Zigbee, Thread and nRF52**
- Zigbee component: supported on nRF52 and on ESP32 chips with an 802.15.4 radio (C5, C6, H2); roles are router, end device and sleepy end device, never coordinator, so a coordinator stack such as Zigbee2MQTT or ZHA is required; exposed entity types are binary sensor, sensor, switch and number (no light); nRF52 allows 8 endpoints, ESP32 up to 239 — [ESPHome Zigbee](https://esphome.io/components/zigbee/)
- Documented Zigbee caveats: ESP32-H2 is reported as more reliable than ESP32-C6; Wi-Fi station plus Zigbee end device works but Wi-Fi plus Zigbee router is unstable and Wi-Fi access point plus Zigbee is unsupported; configuration changes require removing and re-adding the device; Zigbee2MQTT 2.8.0 or later is needed for names with spaces — [ESPHome Zigbee](https://esphome.io/components/zigbee/)
- 2026.11 added sleepy end device support on ESP32 and a report action — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)
- OpenThread component: runs on ESP32-C5/C6/H2 (and newer 802.15.4 parts, ESP-IDF 5.5.5 or newer) and on nRF52840 with Zephyr; device types are full (router-eligible), minimal and sleepy; OTA over Thread works except while a sleepy poll period is active; a border router is required, and Thread only carries a higher-level protocol, for ESPHome its own API — [ESPHome OpenThread](https://esphome.io/components/openthread/)
- 2026.11 added an ESP32 OpenThread border router (ESP32 with native radio or with an external radio co-processor), explicitly marked experimental — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)
- A trade article relays ESPHome's own soak results: the native-radio border router ran two weeks without restart, the external co-processor variant occasionally hung in a three-week test — [esp32.co.uk, 2026-11-19](https://esp32.co.uk/?p=20261103072)
- nRF52: nRF52840 boards (Feather, ItsyBitsy, XIAO BLE, nRF52840 dongle); the page states that support for all aspects of ESPHome on nRF52 is still in development; OTA is recommended via MCUboot or BLE; the experimental SDK 3.2.0 has no Zigbee — [ESPHome nRF52](https://esphome.io/components/nrf52/)

**OTA, provisioning and local interfaces**
- 2026.11 lets native OTA fall back from TCP to UDP on lossy links — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)
- Improv over BLE (component renamed `improv_ble` in 2026.10.0, old name valid until 2027.4.0) provisions Wi-Fi credentials over Bluetooth LE using the open Improv standard, optionally gated by a physical authoriser; the BLE stack is RAM-hungry and can crash alongside voice or audio components; Improv over serial is the wired alternative — [Improv via BLE](https://esphome.io/components/esp32_improv/)
- Captive portal: after one minute of failed Wi-Fi attempts the node opens a fallback hotspot where credentials can be changed and a firmware file uploaded — [Captive portal](https://esphome.io/components/captive_portal/)
- The web_server component serves a browser UI plus a REST API and an event-source stream; `local: true` inlines all assets so no internet is needed (about 13 KB extra for version 2, 78 KB for version 3); version 1 is to be removed in 2027.1.0 — [Web server component](https://esphome.io/components/web_server/)

**Evidence from people who run ESPHome without Home Assistant**
- A two-year retrospective (June 2024) describes ESPHome nodes on plain MQTT with Mosquitto, Zigbee2MQTT and Node-RED as remarkably stable, using birth and last-will messages for availability (pre-2026 source) — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)
- Hacker News commenters in 2025 describe whole houses on ESPHome where every device speaks IP and has its own web interface on an isolated Wi-Fi (pre-2026 source) — [HN thread, 2025-05-17](https://news.ycombinator.com/item?id=44011381)
- A small 2026 automation platform (ELARIS) describes itself as built on MQTT, ESPHome and a modular logic engine (GitHub API description) — [elaris-control/core](https://github.com/elaris-control/core)

### Inferences
- `mqtt:` and `api:` are separate components and the MQTT page only asks to remove `api:` when no API client exists, so a hybrid is plausible: MQTT for ordinary entities, and the native API (through aioesphomeapi) only on the few nodes that act as voice satellites or Bluetooth proxies. This was not found described as a tested setup.
- ESPHome's Zigbee and Thread support in late 2026 covers sensor- and switch-class devices but not lights over Zigbee and not a production border router; Wi-Fi with MQTT or the native API remains the mature transport for ESP32-class nodes.
- For a custom hub the lowest-risk reuse is ESPHome firmware plus its MQTT discovery output; the highest-value but highest-maintenance reuse is the native API for voice, because the protocol's only reference server-side consumer is Home Assistant and aioesphomeapi's major version stands at 46.

### Gaps
- The release in which the Zigbee component first shipped was not confirmed.
- No statement on aioesphomeapi API stability or deprecation policy was found; its release cadence was not analysed beyond the latest version.
- Whether the Device Builder can be installed standalone (Docker or pip) was not stated in the 2026.6 post.
- The Home Assistant forum threads on permanent non-Home-Assistant use were only seen as a search summary.
- No documentation was found on what a non-Home-Assistant server must implement to serve the voice-assistant audio pipeline, beyond the client calls visible in the aioesphomeapi source.

## 3. Other device firmware and local device APIs: Tasmota, OpenBeken, WLED, Shelly stock firmware

### Takeaway
All four run with no cloud and no account. Over MQTT, Tasmota announces itself on its own discovery topic tree (legacy Home Assistant discovery was removed), OpenBeken emits Home Assistant discovery on a manual or flag-driven trigger for lights, relays and simple sensors, and WLED and Shelly publish fixed topic schemes without Home Assistant discovery. Shelly Gen2+ has the richest stock local API (JSON-RPC over HTTP, WebSocket, MQTT and UDP, plus on-device scripts).

### Cited Findings

**Tasmota**
- Open-source firmware for ESP8266/ESP8285 and the ESP32 family, controlled over MQTT, web UI, HTTP or serial, with timers, rules and the Berry scripting language; the project states total local control and that it can run standalone with no server; 150+ supported sensors and peripherals — [Tasmota docs](https://tasmota.github.io/docs/)
- Latest release v16.0.0 "Theo", 2026-10-16; adds initial ESP32-S31 plus ESP32-C61 and P4R3 support, optional MQTT 5.0, more Matter device types (spec updated to 1.6.1); drops direct migration from versions before v8.1.0; ESP32-H2 is not in the build list — [Tasmota v16.0.0](https://github.com/arendst/Tasmota/releases/tag/v16.0.0)
- Repository: C, GPL-3.0, 25,023 stars, 16 open issues, last push 2026-11-24 (GitHub API) — [arendst/Tasmota](https://github.com/arendst/Tasmota)
- MQTT layout: commands on `cmnd/<topic>/...`, results on `stat/<topic>/...`, telemetry on `tele/<topic>/STATE` and `SENSOR`, availability on `tele/<topic>/LWT` (Online/Offline) — [Tasmota Home Assistant page](https://tasmota.github.io/docs/Home-Assistant/)
- Self-announcement: with `SetOption19 0` a device is discovered by Home Assistant's dedicated Tasmota integration through the `tasmota/discovery/#` topic tree; the legacy Home Assistant MQTT discovery (`SetOption19 1`) is stated as removed from all builds — [Tasmota Home Assistant page](https://tasmota.github.io/docs/Home-Assistant/)
- The Home Assistant side requires Tasmota 9.2 or later and native discovery, supports lights, relays, sensors, shutters (modes 1-4), fans on two specific modules and buttons as triggers, and does not work with the lite build — [Tasmota integration](https://www.home-assistant.io/integrations/tasmota/)
- Matter: on ESP32 variants since v13.0.0, over Wi-Fi or Ethernet only (no Thread), as end device or as a bridge for other Tasmota or OpenBeken devices over HTTP or MQTT; it cannot be certified and uses development vendor IDs — [Tasmota Matter](https://tasmota.github.io/docs/Matter/)

**OpenBeken (OpenBK7231T_App)**
- A Tasmota/ESPHome alternative for Tuya-style modules on BK7231T/N, BK7238, XR809/XR806, BL602, W800/W600, LN882H, Realtek chips and ESP32 variants, with a Tasmota-like setup, Tasmota-style JSON over HTTP and MQTT, TuyaMCU support, scripting and optional Berry, and web OTA — [OpenBK7231T_App](https://github.com/openshwprojects/OpenBK7231T_App)
- Latest release 1.18.321, 2026-11-05; 2,368 stars, 685 forks, last push 2026-11-24, no licence detected by GitHub (GitHub API) — [OpenBK7231T_App releases](https://github.com/openshwprojects/OpenBK7231T_App/releases)
- Home Assistant pairing: either a generated YAML snippet or a discovery button that publishes one MQTT discovery message per entity under the Home Assistant discovery prefix; discovery is implemented for lights, relays and simple sensors — [OpenBeken Home Assistant docs](https://github.com/openshwprojects/OpenBK7231T_App/blob/main/docs/homeAssistant.md)
- Native topics are `<device>/<channel>/get`, `<device>/connected` and periodic diagnostics; flag 27 re-runs discovery on IP or configuration change and flag 30 enables Tasmota-style `tele`/`stat` publishes for controllers such as ioBroker — [OpenBeken MQTT topics](https://github.com/openshwprojects/OpenBK7231T_App/blob/main/docs/mqttTopics.md); [OpenBeken flags](https://github.com/openshwprojects/OpenBK7231T_App/blob/main/docs/flags.md)
- The docs index lists use as an ESPHome-style Bluetooth proxy including active connections — [OpenBeken docs index](https://github.com/openshwprojects/OpenBK7231T_App/blob/main/docs/README.md)

**WLED**
- Latest release v16.0.1, 2026-07-07 (v16.0.0 on 2026-05-03); C++, EUPL-1.2, 19,120 stars, last push 2026-11-24 (GitHub API) — [wled/WLED releases](https://github.com/wled/WLED/releases)
- MQTT: brightness, colour and API commands on a device topic and an optional group topic; state on `/g`, `/c` and `/v`; no TLS, so only a local broker is advised; `.local` broker names resolve from v16.0.0 — [WLED MQTT docs](https://kno.wled.ge/interfaces/mqtt/)
- Home Assistant talks to WLED through its JSON API, WebSocket first with polling as fallback, classed as local push, requiring WLED 0.14.0 or newer — [WLED integration](https://www.home-assistant.io/integrations/wled/)
- WLED is not among the 36 tools that Home Assistant lists as emitting MQTT discovery — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)

**Shelly stock firmware (Gen2, Gen3, Gen4)**
- RPC channels: HTTP (`/rpc`, `/rpc/<method>`, no notifications), WebSocket (`ws://<device>/rpc`, with notifications), MQTT and UDP (disabled by default); at most 6 simultaneous non-persistent channels — [Shelly RPC channels](https://shelly-api-docs.shelly.cloud/gen2/General/RPCChannels)
- MQTT: requests on `<id>/rpc` with replies on `<src>/rpc`, notifications on `<id>/events/rpc`, optional full status on `<prefix>/status/<component:id>`, availability on `<id>/online`, simple control on `<prefix>/command/<component:id>`; an `announce` command publishes device info on `shellies/announce` and `<prefix>/announce`; QoS 1; configuration changes need a reboot; Home Assistant discovery is not mentioned — [Shelly MQTT component](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Mqtt/)
- Scripts: JavaScript runs on the device for local automation with no cloud or external server, on a modified Espruino engine, with RPC calls, event handlers, timers, HTTP requests, MQTT publishing, Bluetooth scanning and virtual components; a script that crashes the device is disabled on reboot — [Shelly Scripts overview](https://shelly-api-docs.shelly.cloud/gen2/Scripts/Overview)
- Conflict on the script engine: an older documentation page describes it as a modified mJS (search summary) — [Shelly script language features](https://shelly-api-docs.shelly.cloud/gen2/Scripts/ShellyScriptLanguageFeatures); the current overview says Espruino — [Shelly Scripts overview](https://shelly-api-docs.shelly.cloud/gen2/Scripts/Overview)
- Home Assistant's integration talks to the device directly, states that no cloud connection is needed, uses CoIoT for Gen1 and RPC for Gen2+ (outbound WebSocket for battery devices), is local push, and supports all four generations — [Shelly integration](https://www.home-assistant.io/integrations/shelly/)
- Gen4: Zigbee and Matter are exposed as RPC components (`Zigbee.SetConfig`, network steering; `Matter.GetSetupCode`, factory reset) — [Shelly Zigbee component](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Zigbee); [Shelly Matter component](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Matter)
- Gen4 product pages state the device can work standalone on local Wi-Fi with an embedded web interface; devices default to Matter firmware and switch to Zigbee with five button presses; the API changelog dated 2026-09-23 lists Zigbee OTA and input support (search summary) — [Shelly Dimmer Gen4](https://www.shelly.com/products/shelly-dimmer-gen4); [CSA listing Shelly 1PM Gen4](https://csa-iot.org/csa_product/shelly-1pm-gen4/); [Shelly API changelog](https://shelly-api-docs.shelly.cloud/gen2/changelog)
- Home Assistant forum users describe Gen4 Zigbee mode as buggy and feature-limited (search summary, undated) — [HA forum: Shelly Gen4 Zigbee mode](https://community.home-assistant.io/t/shelly-gen4-zigbee-mode/888242?page=3)

### Inferences
- For a custom MQTT hub, Tasmota and Shelly need a small per-firmware adapter (Tasmota's own discovery payload, Shelly's announce plus RPC introspection), OpenBeken and ESPHome can be read through a generic Home Assistant-discovery consumer, and WLED is easiest over its JSON/WebSocket API with MQTT as a fallback.
- Shelly's local scripting plus MQTT publishing would allow a script to emit Home Assistant-discovery payloads, but no first-party support was found.
- Sockets, light switches and curtain motors are covered natively by all of Tasmota (relays, shutters), Shelly (switch, cover, light components with MQTT control) and Zigbee devices through Zigbee2MQTT; the choice is about announcement format, not capability.

### Gaps
- The exact schema of Tasmota's `tasmota/discovery/<MAC>/config` and `/sensors` payloads was not retrieved from an official page.
- Shelly script resource limits (number of scripts, timers, subscriptions) for Gen3/Gen4 were not found on the current pages; only third-party and older-version figures surfaced.
- An explicit vendor statement that every Gen4 model can be provisioned with no account was not found; the evidence is the product-page wording and the local API.
- No count of Tasmota-supported device templates was found on the official page.
- WLED's JSON API and UDP sync were not examined in detail.

## 4. Other hubs: openHAB, ioBroker, Domoticz, Gladys Assistant, FHEM, Node-RED-centred setups, and 2025-2026 newcomers

### Takeaway
openHAB 5.2 (July 2026) is the only other hub that shipped an LLM-based interpreter, an MCP server and a matter.js-based Matter client, but its LLM path supported only a cloud provider at release and it has no ESPHome voice support. ioBroker, Domoticz, FHEM, Gladys and Node-RED all released in 2026 and are alive but smaller, and no hub started in 2025-2026 shows measurable adoption.

### Cited Findings

**openHAB**
- openHAB 5.2.0 was released 2026-07-05 (a 5.2.1 patch followed); the newest build is milestone 5.3.0.M5 of 2026-11-18 (GitHub API) — [openHAB 5.2 release post](https://www.openhab.org/blog/2026-07-05-openhab-5-2-release.html); [openhab-distro releases](https://github.com/openhab/openhab-distro/releases)
- 5.2 adds a chat interface in Main UI with LLM-based human language interpreters (only Google Gemini at launch; OpenAI and OpenAI-compatible providers such as Ollama announced as planned), an MCP server exposing items, rules, pages, Things, logs and the REST API, an item permission model for voice (none, read-only, read-write), browser-based voice dialogue, and Matter OTA updates plus a Thread network map — [openHAB 5.2 release post](https://www.openhab.org/blog/2026-07-05-openhab-5-2-release.html)
- Scale of the 5.2 cycle: 141 contributors, 3,082 commits, 43 active maintainers, 19 new add-ons, more than 500 add-ons in total — [openHAB 5.2 release post](https://www.openhab.org/blog/2026-07-05-openhab-5-2-release.html)
- Requirements: a 64-bit OS with a Java 21 JVM; a dedicated Raspberry Pi 4 or newer is recommended; no RAM figure is given — [openHAB installation](https://www.openhab.org/docs/installation/)
- Voice status per the project's own forum (February-July 2026): Piper and Whisper add-ons are considered solid, the dialog pipeline is less flexible than Home Assistant's, there is no ESPHome Voice support, and early testers of the Gemini interpreter hit HTTP 400 errors — [openHAB forum thread](https://community.openhab.org/t/revisiting-local-llm-powered-voice-assistants-in-2026/168251)
- openHAB's Matter client is built on matter.js (listed as fully working since openHAB 5.0), and openHAB is listed as a consumer of Home Assistant MQTT discovery — [matter.js README](https://github.com/matter-js/matter.js); [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)

**ioBroker**
- Open-source, runs locally (Raspberry Pi, mini PC, NAS or server), with more than 800 adapters; remote access is via own VPN or the optional paid ioBroker.pro service, and some visualisation features may need optional licences — [iobroker.com](https://www.iobroker.com/)
- js-controller v7.2.5, 2026-10-05; TypeScript, MIT, last push 2026-11-24 (GitHub API) — [js-controller releases](https://github.com/ioBroker/ioBroker.js-controller/releases)
- ioBroker is one of only three documented clients of the ESPHome native API and is listed as offering Matter devices, bridges and controller on matter.js — [Native API component](https://esphome.io/components/api/); [matter.js README](https://github.com/matter-js/matter.js)
- Forum reports in 2026 mention start-up failures after the 7.2.2 controller update for some Docker users (search summary) — [ioBroker forum](https://forum.iobroker.net/topic/84815/iobroker-startet-nicht-mehr-js-7.2.2-update/11)

**Domoticz**
- Stable release 2026.4 published 2026-10-01; C++, GPL-3.0, 3,844 stars, 7 open issues, last push 2026-11-24 (GitHub API) — [Domoticz release 2026.4](https://github.com/domoticz/domoticz/releases/tag/2026.4)
- Listed as a consumer of Home Assistant MQTT discovery — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)

**Gladys Assistant**
- Gladys Assistant 5 was released 2026-08-27 with a complete interface redesign; it ships as a Docker image for Raspberry Pi, NAS or any Docker host, supports Zigbee (Zigbee2MQTT 2.13), Z-Wave, Matter and MQTT, and has a built-in AI assistant that can see and act on devices; the post does not say whether the assistant's model runs locally — [Gladys Assistant 5 post](https://gladysassistant.com/blog/gladys-assistant-5/)
- Latest release v5.3.5, 2026-11-23; JavaScript, Apache-2.0, 3,284 stars (GitHub API) — [Gladys releases](https://github.com/GladysAssistant/Gladys/releases)
- A third-party page says Google Home and Alexa bridging, the REST API and the MCP server belong to the paid Plus tier (search summary) — [chatgate.ai](https://chatgate.ai/post/gladys-assistant-5)

**FHEM**
- A GPL v2 Perl server with over 430 modules, MQTT with internal or external broker, and continuous development through an update command; latest packaged release 6.5 as of 2026-10-24; hardware requirement is any 24/7 machine with a Perl interpreter — [fhem.de](https://fhem.de/)

**Node-RED-centred setups**
- Node-RED 5.0 was released 2026-06-09 as an editor-focused major release and requires Node.js 22.9.0 or later (search summary); latest is 5.0.10 of 2026-11-17; Apache-2.0, 23,876 stars (GitHub API) — [Node-RED 5.0 post](https://nodered.org/blog/2026/06/09/version-5-0-released); [node-red releases](https://github.com/node-red/node-red/releases)
- A user who ran ESPHome, Mosquitto, Zigbee2MQTT and Node-RED for two years reports stability and Git-tracked configuration, but dislikes visual programming and uses mostly JavaScript function nodes, and names the loss of Home Assistant's integration ecosystem as the main cost (pre-2026 source) — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)

**Newcomers 2025-2026**
- Switchyard: Go, MIT, described as home automation for 2026 with a driver SDK and web UI; 0 stars, last push 2026-05-16 (GitHub API) — [fdatoo/switchyard](https://github.com/fdatoo/switchyard)
- ELARIS: JavaScript, local-first platform on MQTT and ESPHome, v0.6.2, announced 2026-03-24, explicitly not positioned as a Home Assistant replacement; 9 stars, last push 2026-04-16 (GitHub API) — [ELARIS announcement](https://community.home-assistant.io/t/sharing-elaris-a-local-first-automation-platform-under-active-development/998544); [elaris-control/core](https://github.com/elaris-control/core)
- gethome-hub: TypeScript, a local hub for Matter, Zigbee and MQTT devices with an assistant and rules on the user's own AI key; created 2026-07-03, 3 stars, last push 2026-11-20 (GitHub API) — [gethome-inc/gethome-hub](https://github.com/gethome-inc/gethome-hub)

### Inferences
- Outside Home Assistant, local-LLM control is either cloud-first (openHAB 5.2 at launch) or unspecified (Gladys 5); none of these hubs offers a ready local-model voice path comparable to Home Assistant plus Wyoming.
- openHAB and Domoticz consuming Home Assistant MQTT discovery strengthens that format's role as the common self-description layer for any controller, including a custom one.
- The newcomers confirm that people keep attempting lean hubs on MQTT plus ESPHome, but with single-digit star counts they are references to read, not platforms to adopt.

### Gaps
- Whether openHAB 5.3 milestones added the announced OpenAI-compatible or Ollama provider was not checked.
- Domoticz 2026.4 release notes failed to load; no feature or footprint detail was obtained.
- No RAM or CPU footprint numbers were found for any of these hubs.
- Whether Gladys 5's built-in assistant can use a local model, and which features need the paid tier, is only covered by a third-party page.
- Homey Self-Hosted Server and other proprietary self-hosted hubs were excluded as not open source.

## 5. MQTT-centred architectures: Zigbee2MQTT, Homie, Home Assistant MQTT discovery as a de-facto standard, broker choices

### Takeaway
Zigbee2MQTT 2.14.3 puts 5,868 devices from 616 vendors behind a self-describing retained JSON document (`bridge/devices` with `exposes`) and, optionally, Home Assistant discovery messages. Home Assistant MQTT discovery is the de-facto self-description format (36 emitting tools are listed and other hubs consume it), Homie v5 exists but adoption looks thin, Mosquitto 2.1 is the default small-home broker, and EMQX is no longer open source.

### Cited Findings

**Zigbee2MQTT**
- Latest release 2.14.3, published 2026-11-01 (2.14.0 on 1 September, 2.14.1 on 3 September, 2.14.2 on 1 October); TypeScript, GPL-3.0, 15,846 stars, last push 2026-11-24 (GitHub API) — [zigbee2mqtt releases](https://github.com/Koenkk/zigbee2mqtt/releases)
- The supported-devices page states 5,868 devices from 616 vendors (fetched 2026-11-25) — [Supported devices](https://www.zigbee2mqtt.io/supported-devices/)
- Requirements: a Zigbee adapter (zStack and EmberZNet recommended), a host such as a Raspberry Pi or Intel NUC (Linux, Windows, macOS), and an MQTT broker (Mosquitto recommended, others should work); installable on Linux, Docker, as a Home Assistant add-on and others; first run offers a web onboarding on port 8080 — [Getting started](https://www.zigbee2mqtt.io/guide/getting-started/)
- 2.14.3 deprecates 32-bit platforms (support ends with Node.js 22 end-of-life, minimum then Node 24, 32-bit Docker images stop updating); 2.14.0 allows keeping device configuration when removing a device — [zigbee2mqtt releases](https://github.com/Koenkk/zigbee2mqtt/releases)
- Device topics: JSON state on `zigbee2mqtt/<friendly_name>`, commands on `/set`, reads on `/get`, optional `/availability`; bridge topics include retained `bridge/state`, `bridge/devices`, `bridge/groups` and `bridge/definitions`, plus `bridge/info`, `bridge/event`, `bridge/logging` and a request/response API (`bridge/request/...` with transaction IDs) for pairing, renaming, removal, network map and backup — [MQTT topics and messages](https://www.zigbee2mqtt.io/guide/usage/mqtt_topics_and_messages.html)
- Each entry in `bridge/devices` carries identity, power source, interview state, endpoints with clusters and bindings, and a `definition` with vendor, model, options and `exposes` — [MQTT topics and messages](https://www.zigbee2mqtt.io/guide/usage/mqtt_topics_and_messages.html)
- `exposes` uses generic types (binary, numeric, enum, text, composite, list) and specific types (light, switch, fan, cover, lock, climate) with name, label, property, unit, ranges, value lists, a config/diagnostic category and a 3-bit access mask (published in state, settable via `/set`, readable via `/get`) — [Exposes](https://www.zigbee2mqtt.io/guide/usage/exposes.html)
- Home Assistant discovery is an opt-in setting (`homeassistant: enabled: true`); it covers devices and groups of lights, switches, locks and covers, publishes button actions as device triggers, and has an experimental event-entity mode — [Zigbee2MQTT Home Assistant integration](https://www.zigbee2mqtt.io/guide/usage/integrations/home_assistant.html)
- Automations can also run inside Zigbee2MQTT as JavaScript external extensions (search summary) — [dhorpynchenko.com](https://dhorpynchenko.com/posts/zigbee2mqtt-extensions/)

**Home Assistant MQTT discovery as a self-description format**
- Topic format `<prefix>/<component>/[<node_id>/]<object_id>/config`, default prefix `homeassistant`; a newer device-based form publishes all components of a device in one message with mandatory device and origin blocks, components under `cmps` and child devices under `cdevs`, plus a documented migration path from per-entity messages — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)
- 32 platforms are listed (30 entity platforms plus device trigger and tag scanner); the controller announces itself with a birth message on `homeassistant/status`, and devices are expected to re-publish discovery when they see it — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)
- 36 tools are listed as emitting discovery, including ESPHome, Tasmota, Zigbee2MQTT, OpenMQTTGateway, EMS-ESP32, ESPurna, ebusd, ble2mqtt and room-assistant; Domoticz and openHAB are listed as consumers — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/)
- Conflict: Tasmota is on that emitter list (from version 5.11.1e) — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/); Tasmota's own page says Home Assistant MQTT discovery was removed from all builds in favour of its native discovery — [Tasmota Home Assistant page](https://tasmota.github.io/docs/Home-Assistant/)
- OpenBeken emits it for lights, relays and simple sensors — [OpenBeken Home Assistant docs](https://github.com/openshwprojects/OpenBK7231T_App/blob/main/docs/homeAssistant.md)
- OpenMQTTGateway (ESP32/ESP8266 firmware bridging BLE, 433/315/868/915 MHz RF, infrared and LoRa to MQTT, more than 100 BLE sensors) has Home Assistant discovery enabled by default on all builds; v1.9.1, 2026-11-10 (GitHub API) — [OpenMQTTGateway docs](https://docs.openmqttgateway.com/); [OpenMQTTGateway Home Assistant page](https://docs.openmqttgateway.com/integrate/home_assistant.html); [OpenMQTTGateway releases](https://github.com/1technophile/OpenMQTTGateway/releases)
- Theengs Gateway (Python BLE-to-MQTT gateway for Raspberry Pi or PC) uses the same topic structure and payloads as OpenMQTTGateway; v1.8.0, 2026-10-13 (GitHub API) — [Theengs Gateway](https://gateway.theengs.io/); [Theengs Gateway releases](https://github.com/theengs/gateway/releases)
- In 2026 the format kept evolving on the controller side: MQTT `object_id` removed (2026.4), time/date platforms added (2026.5), infrared platform (2026.8), child devices (2026.11) — [2026.4](https://www.home-assistant.io/blog/2026/04/01/release-20264/); [2026.5](https://www.home-assistant.io/blog/2026/05/06/release-20265/); [2026.8](https://www.home-assistant.io/blog/2026/08/05/release-20268/); [2026.11](https://www.home-assistant.io/blog/2026/11/04/release-202611/)

**Homie convention**
- The current specification is v5 (the site's release-date line reads 11 May 2025, pre-2026 source); a develop version is dated 12 February 2026. Root topic is `homie/5/<device>/...`; devices publish `$state` (init, ready, disconnected, sleeping, lost) and a single JSON `$description` document describing nodes and properties (datatype, unit, settable); commands go to `<property>/set`, with `$target` to report intended values, alerts under `$alert`, and root/child device hierarchies; messages are retained by default — [Homie specification](https://homieiot.github.io/specification/); [Homie home page](https://homieiot.github.io/)
- The repository's latest GitHub release object is still v4.0.0 (2019) although a v5.0.0 tag exists; 746 stars, last push 2026-02-12 (GitHub API) — [homieiot/convention](https://github.com/homieiot/convention)
- Implementation signals (search summary): a Rust crate for v5 last released October 2024, and a maintained ESP8266 library that offers a v5 mode but keeps Homie 3.0.1 as default because v5 changes the discovery contract and root topic — [homie5 crate](https://docs.rs/homie5); [homie-esp8266 v4/v5 compatibility](https://labodj.github.io/homie-esp8266/others/homie-v4-v5-compatibility/)

**Brokers for a small home**
- Mosquitto: open source (EPL/EDL), implements MQTT 5.0, 3.1.1 and 3.1, described as lightweight and suitable from low-power single-board computers to servers — [mosquitto.org](https://mosquitto.org/)
- Mosquitto 2.1.0 (2026-01-29) added a built-in WebSocket implementation, a SQLite persistence plugin, more plugin events and an idle-wakeup reduction of about 100x; it removed OpenSSL below 3.0 and set a 2,000,000-byte default maximum packet size; the newest tag is v2.1.4 (GitHub API) — [Mosquitto 2.1.0 announcement](https://mosquitto.org/blog/2026/01/version-2-1-0-released/); [mosquitto tags](https://github.com/eclipse-mosquitto/mosquitto/tags)
- EMQX: from 5.9.0 the single offering is under Business Source License 1.1; any number of independent single-node instances may be run, clustering needs a commercial licence, and each version converts to Apache 2.0 after four years; the newest release is 6.3.3 of 2026-11-11 (GitHub API) — [EMQX licence FAQ](https://www.emqx.com/en/content/license-faq); [emqx releases](https://github.com/emqx/emqx/releases)
- NanoMQ: MIT, pure C on POSIX, supports MQTT 3.1.1 and 5.0, targets embedded and edge use (x86_64 and ARM, with a Zephyr porting guide); v0.25.7, 2026-10-21, 2,667 stars (GitHub API) — [nanomq/nanomq](https://github.com/nanomq/nanomq); [nanomq releases](https://github.com/nanomq/nanomq/releases)
- ESPHome's own documentation names the broker as the single point of failure of an MQTT design — [Native API component](https://esphome.io/components/api/)

### Inferences
- A custom hub can cover most of the device edge by implementing two consumers: Home Assistant discovery (ESPHome, OpenBeken, OpenMQTTGateway, Theengs, optionally Zigbee2MQTT) and Zigbee2MQTT's `bridge/devices` document, which is richer than discovery because it carries value ranges, access bits and the config/diagnostic split.
- Homie v5 is technically close to what a self-describing hub needs (one JSON description per device), but with a 2019 "latest release" object, few v5 implementations and no mainstream firmware emitting it, adopting it would mean writing both ends.
- Because the discovery schema changes with Home Assistant releases, a custom consumer should target the stable core (config topic, device block, state and command topics) and tolerate unknown keys.
- For one household the broker choice is effectively Mosquitto; EMQX fails the open-source criterion and NanoMQ offers no advantage that the sources quantify.

### Gaps
- No RAM or CPU figures for Zigbee2MQTT or for any broker were found in primary sources.
- Other lightweight brokers (for example Rust or Go implementations) were not researched before the search budget ran out.
- Whether Theengs Gateway emits Home Assistant discovery by default was not stated on its home page.
- The total number of Homie-compatible devices and controllers was not found; the implementations page was not read.
- Zigbee2MQTT 2.13.x notes and the device count added per 2.14 release were not retrieved.

## 6. Matter and Thread for a self-built hub in 2026

### Takeaway
A self-built hub can get a serious Matter controller by running the Open Home Foundation's matter.js-based Matter Server standalone and talking to its WebSocket API, and OpenThread Border Router also runs standalone. The hard part without Home Assistant, Apple or Google remains commissioning Thread devices and handling Thread credentials, and IKEA's cheap Matter-over-Thread range has had widely reported pairing and dropout problems through 2026.

### Cited Findings

**Open-source controllers**
- matter.js is a TypeScript implementation of the complete Matter protocol for devices, bridges and controllers, runs on Node.js LTS from 20.x, offers a separate BLE package, and is not itself certified by the standards body; users listed are Home Assistant, Matterbridge, ioBroker and openHAB 5.0 — [matter.js README](https://github.com/matter-js/matter.js)
- matter.js latest release v0.18.0, 2026-10-20; Apache-2.0, 932 stars, last push 2026-11-24 (GitHub API) — [matter.js releases](https://github.com/matter-js/matter.js/releases)
- The Open Home Foundation Matter Server (matterjs-server) is a Matter controller over WebSockets built on matter.js, described as suitable for other projects, a drop-in replacement for the Python Matter Server with the same WebSocket API and automatic data migration; it supports Matter 1.6.0, runs from a container image or npm (Node.js 22.13+, 24.x recommended, or 26.x), listens on port 5580 with `/ws`, and has a dashboard with Thread and Wi-Fi topology graphs — [matterjs-server README](https://github.com/matter-js/matterjs-server)
- Commissioning options in that server: a local Bluetooth adapter (`--bluetooth-adapter`) or a BLE proxy mode with a documented protocol and clients (Home Assistant integration, a Python CLI, a JavaScript client) — [matterjs-server README](https://github.com/matter-js/matterjs-server)
- Internet dependence of that server: certificate checks, vendor lookup and OTA use the standards body's ledger; offline operation relies on bundled seed data and manually supplied files (offline, revocation, certificate and OTA directory options) — [matterjs-server README](https://github.com/matter-js/matterjs-server)
- matterjs-server: Apache-2.0, 440 stars, 44 open issues, last push 2026-11-24 (GitHub API) — [matter-js/matterjs-server](https://github.com/matter-js/matterjs-server)
- The Python Matter Server was archived on 2026-06-23 (GitHub API: archived, last push that day); its final release is reported as 8.1.2 (search summary) — [python-matter-server](https://github.com/matter-js/python-matter-server)
- Home Assistant switched to the new server with Matter Server app 9.0 on 2026-06-23 (Matter 1.5.1, Thread 1.4 border router app, faster start-up and reconnection); uncertified devices with development or test certificates can no longer be added out of the box — [Home Assistant Matter post, 2026-06-23](https://home-assistant.io/blog/2026/06/23/the-matter-upgrade-youve-been-waiting-for)
- Early beta users reported lost nodes after enabling the new server in version 8.2.0 (search summary) — [HA forum: nodes gone](https://community.home-assistant.io/t/matter-server-8-2-0-nodes-gone-in-matter-js/978062)
- The reference C++ SDK (connectedhomeip) is at v1.6.1.0, 2026-09-25; 9,004 stars (GitHub API) — [connectedhomeip releases](https://github.com/project-chip/connectedhomeip/releases)

**Thread border routers and credentials**
- OpenThread Border Router (ot-br-posix) latest release v2026.11.0, 2026-11-01; BSD-3-Clause, 586 stars (GitHub API) — [ot-br-posix releases](https://github.com/openthread/ot-br-posix/releases)
- A May 2026 walkthrough of OTBR on an Orange Pi Zero 2 with a Sonoff MG21 dongle for openHAB reports: the official Docker image worked after IPv6-only mDNS and network fixes; an MG24 dongle crashed the Thread network until replaced; the author still used Home Assistant and its Matter server on a laptop plus the Android app and Google Play Services to commission devices, then shared them to openHAB via multi-admin; the author calls the result inconvenient to set up but liveable, and another user needed three evenings for the chip-tool route — [openHAB forum, 2026-05-23](https://community.openhab.org/t/how-to-use-an-orange-pi-zero-2-running-otbr-with-openhab/169352)
- Home Assistant's own Thread page says Thread is IPv6-only, container hosts may lack the needed IPv6 routing support, vendors form separate networks without roaming, and credentials are not in the local announcements — [Thread integration](https://www.home-assistant.io/integrations/thread/)
- Home Assistant's Matter page says commissioning uses a phone (Google Play Services or iOS 16+), that Matter needs IPv6 and mDNS multicast to flow freely and may not fit VLAN setups, and that a standalone Docker Matter server is unsupported — [Matter integration](https://www.home-assistant.io/integrations/matter/)
- ESPHome 2026.11 can act as an experimental ESP32 OpenThread border router — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/)

**Device firmware that speaks Matter without a vendor cloud**
- Tasmota exposes Matter over Wi-Fi/Ethernet on ESP32 with development vendor IDs; Shelly Gen4 exposes Matter with a setup code retrievable over local RPC — [Tasmota Matter](https://tasmota.github.io/docs/Matter/); [Shelly Matter component](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Matter)
- ESPHome's OpenThread page presents Thread as a transport for its own API, naming Matter and HomeKit only as alternative higher-level protocols — [ESPHome OpenThread](https://esphome.io/components/openthread/)

**Availability of cheap Matter-over-Thread devices (IKEA 2025-2026 range)**
- IKEA announced 21 Matter-compatible products, with roll-out from January 2026 and availability varying by country and product (search summary) — [matteralpha.com](https://www.matteralpha.com/news/ikea-launches-21-new-smart-home-devices-with-native-matter-support); [notebookcheck.net](https://www.notebookcheck.net/New-IKEA-Matter-compatible-smart-home-products-spotted-in-stores.1176464.0.html)
- Prices seen: MYGGSPRAY motion sensor $7.99 in the US and 7.99 EUR in Lithuania, TIMMERFLOTTE temperature/humidity sensor $9.99, KAJPLATS E26 bulb $12.99 (search summary; dates late 2025 to 2026) — [IKEA US product page](https://www.ikea.com/us/en/p/myggspray-wireless-motion-sensor-smart-80619451/); [itechguides.com](https://www.itechguides.com/?p=400214)
- IKEA's product page says the devices use Matter over Thread and need a Thread border router, and that its own hub is not required when another Matter controller and border router exist (search summary) — [IKEA product page](https://www.ikea.com/gb/en/p/-30624695/)
- Reliability reports from Home Assistant users in 2026: commissioning timeouts on some models, devices dropping off Thread several times per hour while other brands stayed stable, failed firmware updates (search summary) — [HA forum: commissioning problems](https://community.home-assistant.io/t/ikea-bilresa-and-timmerflotte-thread-commissioning-problems/962416); [HA forum: frequent unavailability](https://community.home-assistant.io/t/ikea-alpsuga-matter-thread-becomes-unavailable-frequently-long-case-timeout-loops/979203)
- A consumer-tech outlet reported user-cited failure rates of up to 50% and a firmware fix in preparation (search summary, article date not confirmed) — [Tom's Guide](https://www.tomsguide.com/home/smart-home/ikeas-new-matter-smart-home-devices-are-struggling-to-stay-connected-but-a-firmware-fix-is-in-the-works)

### Inferences
- The realistic Matter building block for a custom hub is the standalone Matter Server plus its WebSocket client, not matter.js directly: it brings storage, commissioning, a dashboard and OTA handling, and its API is the one Home Assistant itself uses.
- Running Matter fully offline needs deliberate configuration (offline ledger data, development-certificate acceptance for DIY devices), so "works without internet" is true for operation but not automatically for commissioning and certificate checks.
- Phone-free commissioning is technically available (Bluetooth adapter on the server host), which avoids Google and Apple accounts, but the one detailed 2026 account of doing Thread outside Home Assistant still leaned on a phone ecosystem, so this path is less trodden.
- For a first version, Zigbee through Zigbee2MQTT and Wi-Fi through ESPHome carry less risk than Matter-over-Thread; Thread can be added later as a separate service pair (OTBR plus Matter Server).

### Gaps
- No late-2026 status on whether IKEA's firmware fixes resolved the dropouts was found.
- No end-to-end account of commissioning a Thread device with only the standalone Matter Server, a host Bluetooth adapter and OTBR was found.
- The Matter Server WebSocket API documentation and its Python or JavaScript client libraries were not examined in detail.
- Whether ESPHome offers any Matter device support was not confirmed either way.
- Matterbridge (a matter.js plugin manager, 1,014 stars) was not examined beyond its listing.

## 7. LLM-centred open-source smart-home projects from 2024-2026

### Takeaway
Almost every working LLM-for-the-home project is a layer on Home Assistant (home-llm, OpenAI-compatible agent integrations, MCP servers, agent integrations); the stand-alone voice projects of the previous generation are either absorbed into the Open Home Foundation's voice organisation (Rhasspy to Wyoming), slow-moving (Willow, OpenVoiceOS) or tiny (LLM-native hubs with single-digit stars). What worked is small tool-calling models with a deterministic path in front; what did not is LLM-only control on CPU-class hardware and reasoning models in the voice loop.

### Cited Findings

**home-llm (acon96)**
- A Home Assistant integration (Local LLM) plus fine-tuned "Home" models under 5B parameters; backends are in-process llama.cpp, Ollama, generic OpenAI-compatible APIs, llama.cpp server, and OpenAI Responses and Anthropic Messages style APIs; text-generation-webui support was removed in v0.5.0 — [home-llm](https://github.com/acon96/home-llm)
- Current Home models are a Llama 3.2 3B fine-tune and a FunctionGemma 270M fine-tune; the README says they run on a Raspberry Pi with no GPU; general-purpose models are steered with in-context examples; no benchmark numbers are given — [home-llm](https://github.com/acon96/home-llm)
- v0.4 rewrote tool calling with an agentic tool loop and voice streaming, 0.4.8 added auto-repair of malformed tool-call JSON, v0.5.0 moved conversations onto Home Assistant's chat API (breaking); v0.5.1 was published 2026-11-10 and requires Home Assistant 2026.5.0 or newer; 1,499 stars, 42 open issues, no licence detected (GitHub API) — [home-llm](https://github.com/acon96/home-llm); [home-llm releases](https://github.com/acon96/home-llm/releases)

**Other Home Assistant-side LLM integrations (GitHub API metadata, 2026-11-25)**
- A local OpenAI-compatible LLM integration for llama.cpp and vLLM class servers: 320 stars, Apache-2.0, last push 2026-11-22 — [hass_local_openai_llm](https://github.com/skye-harris/hass_local_openai_llm)
- Extended OpenAI Conversation: 1,463 stars, MIT, last push 2026-10-11 — [extended_openai_conversation](https://github.com/jekalmin/extended_openai_conversation)
- Home Generative Agent (agent with automation creation, camera analysis, cloud or fully local models): 384 stars, MIT, last push 2026-11-24 — [home-generative-agent](https://github.com/goruck/home-generative-agent)
- A community "AI Plugin" conversation agent (September-November 2026, v1.0.0 on 1 November) tuned for 8 GB VRAM routes common commands around the model, keeps Ollama's prefix cache warm and adds memory, timers, music and MCP — [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725)

**Home Assistant MCP servers**
- Official MCP server: see section 1 — [MCP server integration](https://www.home-assistant.io/integrations/mcp_server/)
- ha-mcp: a community MCP server with 87 tools for searching, controlling, and building or debugging configuration (automations, scripts, dashboards, helpers, backups); installable as an in-process integration from the Marketplace (Home Assistant 2026.11.0 or newer), as an app, via Docker or uvx; it reaches everything, whereas the built-in server reaches only entities exposed to Assist; it offers a read-only mode, per-tool switches and approval policies; 8,108 stars, MIT, last push 2026-11-24 (GitHub API) — [ha-mcp](https://github.com/homeassistant-ai/ha-mcp)
- hass-mcp: 353 stars, MIT, last push 2026-11-20 (GitHub API) — [voska/hass-mcp](https://github.com/voska/hass-mcp)
- A 2026 student research project built a local-model agent that reaches Home Assistant over MCP and benchmarked model families from 0.5B to 34B parameters; larger models scored better, a cloud model scored best without a perfect score, and local operation needed high-end consumer hardware with latency remaining high — [OST student project abstract, 2026](https://www.ost.ch/files/daten/abstracts/pdf/3509b232-4aa4-4840-8314-15f71cde761f.pdf)
- An MCP server that bridges LLM agents to Zigbee2MQTT devices directly, without a hub, exists as a listing (search summary) — [mcpmarket.com: Zigbee](https://mcpmarket.com/server/zigbee)

**Willow and Willow Inference Server**
- Willow is ESP-IDF firmware for the ESP32-S3-BOX family (about $50 hardware) with on-device wake word, up to 400 on-device commands or a self-hosted inference server for open speech, and command delivery to Home Assistant, openHAB or any REST endpoint; the site claims 500 ms or less from end of speech to completed action and under 1% failures, on a page carrying a 2023 copyright (pre-2026 source) — [heywillow.io](https://heywillow.io/)
- The repository moved to a new organisation; latest release 0.4.3 on 2026-07-21, last commits in July-August 2026 are CI maintenance, 3,145 stars, Apache-2.0; the inference-server repository has no releases (GitHub API) — [HeyWillow/willow](https://github.com/HeyWillow/willow)

**OpenVoiceOS**
- ovos-core's latest GitHub release is 2.1.1 of 2025-11-05 while PyPI carries alpha builds up to 2.6.0a1; a persona component can route unhandled requests to an LLM, including a local OpenAI-compatible server; it targets Raspberry Pi 3B or better in headless mode; a grant running from October 2025 aims at a first stable release (search summary) — [ovos-core on PyPI](https://pypi.org/project/ovos-core/2.6.0a1/); [NLnet project page](https://nlnet.nl/project/OpenVoiceOS/); [ovos-core releases](https://github.com/OpenVoiceOS/ovos-core/releases)

**Rhasspy and its successors**
- The Rhasspy repository is archived (last push 2025-04-22) and Rhasspy 3 is archived (last push 2023-12-26); the Wyoming protocol, Piper, Faster Whisper server, Speech-to-Phrase, the intent corpus and matcher, and the Linux satellite now live under the OHF-Voice organisation with releases through November 2026 (GitHub API) — [rhasspy/rhasspy](https://github.com/rhasspy/rhasspy); [rhasspy/rhasspy3](https://github.com/rhasspy/rhasspy3); [OHF-Voice/wyoming](https://github.com/OHF-Voice/wyoming)

**LLM-native hubs and custom LLM pipelines**
- gethome-hub (TypeScript, Matter/Zigbee/MQTT, assistant on the user's own AI key): 3 stars, created July 2026 (GitHub API) — [gethome-inc/gethome-hub](https://github.com/gethome-inc/gethome-hub)
- A May 2026 write-up describes a custom Go voice pipeline (ESP32 wake word device, VAD, STT, LLM, retrieval over a 43-million-document index) that uses Home Assistant only through its REST API for device control; measured on a 16 GB GPU, thirteen model configurations reached 100% tool-routing accuracy, the fastest averaging 0.54 s, while a model picked from public rankings averaged 3.22 s and a newer sibling 26.73 s, and several models mis-routed everything or lacked tool support — [bitworking.org, 2026-05-17](https://bitworking.org/news/2026/05/surprising-things-i-learned-putting-together-a-home-brain/)
- openHAB 5.2's LLM interpreter: see section 4 — [openHAB 5.2 release post](https://www.openhab.org/blog/2026-07-05-openhab-5-2-release.html)
- An early detailed build (January 2024) with two 16 GB GPUs, a Mixtral model behind vLLM and a forked OpenAI-style integration had to work around missing function calling by constraining JSON output, fit the whole home state into a 10,800-token context, and was described by its author as a little slow (pre-2026 source) — [johnthenerd.com, 2024-01-13](https://johnthenerd.com/blog/local-llm-assistant/)

### Inferences
- The centre of gravity moved from stand-alone voice assistants (Rhasspy, Willow, Mycroft descendants) to Home Assistant's pipeline plus interchangeable model servers; a custom hub that wants LLM control without Home Assistant has reference code to read (home-llm's prompt and tool handling, the Go pipeline write-up, the Zigbee2MQTT MCP bridge) but no maintained framework to adopt.
- MCP has become the standard way to hand a home to a model, and two different jobs are emerging: narrow device-control tool sets for voice and broad administrative tool sets for configuration agents. A custom hub would need the first and could postpone the second.
- Fine-tuned tiny models (270M to 3B) are the only published route to LLM control on CPU-class hardware, but no accuracy numbers were found for them in 2026 sources.

### Gaps
- No 2026 benchmark numbers for home-llm's fine-tuned models were found; the public Home Assistant model leaderboard page returned no readable content.
- Willow Inference Server's maintenance state is unclear beyond having no releases; no 2026 user reports on Willow were retrieved.
- OpenVoiceOS's Home Assistant plugin and real-world LLM use were not researched.
- The Zigbee2MQTT MCP bridge and gethome-hub were not inspected beyond listings and repository metadata.
- Reddit threads on these projects could not be accessed.

## 8. Lessons reported by people who run a fully local LLM voice assistant at home

### Takeaway
Reported end-to-end response times cluster at 1-2 s on 24 GB GPUs, 1.5-4 s on 16 GB, about 3 s on 8 GB and 10-20 s or worse on CPU or integrated graphics, with deterministic commands answering in well under a second. Nearly every account converges on the same architecture: deterministic intent matching first, a small tool-calling model as fallback, few exposed entities, prompt caching and no reasoning mode.

### Cited Findings

**Latency actually achieved**
- A long-running community thread (opened 2025-10-27, page metadata shows it modified 2026-11-16, text now references 2026 models) lists response times after prompt caching: RTX 3090 and RX 7900 XTX 24 GB 1-2 s, RTX 5060 Ti 16 GB 1.5-3 s, RX 9060 XT 16 GB 1.5-4 s, RTX 3050 8 GB with a 4B model about 3 s; its own benchmark averages 0.98 s and 1.25 s per turn for the two preferred models — [HA forum: journey to a reliable local voice assistant](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860); [same thread, page 15](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860?page=15)
- On an RTX 3060 Ti with 8 GB, a 12B model at 3-bit quantisation had a median near 3.6 s and a 9B model near 2.9 s; commands that bypass the model answer in about 0.05 s; preloading removed a 7-8 s penalty on the first voice turn (September-November 2026) — [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725)
- On a fanless Intel N150 mini PC with 16 GB RAM (Vulkan on the integrated GPU) 4B models produced about 4-5 tokens per second and took roughly 10-21 s per answer; a model with reasoning enabled took over four minutes; the author concluded the box suits asynchronous, pre-generated results and not real-time conversation (April-July 2026) — [HA forum: N150 journey](https://community.home-assistant.io/t/running-a-local-llm-on-an-intel-n150-mini-pc-for-home-assistant-a-practical-journey/1007724)
- Home Assistant's own benchmark: streaming TTS brings time-to-first-audio to about 0.56 s with local Piper versus 5.31 s without streaming (pre-2026 source) — [AI in Home Assistant, 2025-09-11](https://www.home-assistant.io/blog/2025/09/11/ai-in-home-assistant)
- Deterministic path: Speech-to-Phrase transcribes in under a second on a Raspberry Pi 4 and about 150 ms on a Raspberry Pi 5; Whisper needs about 8 s on a Raspberry Pi 4 and under a second on an Intel NUC — [Voice chapter 9](https://www.home-assistant.io/blog/2025/02/13/voice-chapter-9-speech-to-phrase/); [Fully local assistant docs](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/)
- A November 2026 guide sets 750-1100 ms to first audio as an optimised target (speech-to-text 150-250 ms, first LLM token 200-400 ms, first TTS chunk 100-150 ms) and states plainly that this is a target, not a measurement — [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/)
- A community guide author notes that generation speed alone misleads: prompt processing can add seconds before the first token, and voice makes delays obvious (August 2026) — [HA forum: So you want to AI](https://community.home-assistant.io/t/so-you-want-to-ai-in-home-assistant/1021777)

**Common failure modes**
- Default low-bit quantisations fail basic tool calls: one author found default 4B tags unreliable and moved to higher-quant GGUF files; another measured 9 of 33 replayed real requests returning empty at 3-bit versus 0 of 33 at a 4-bit variant — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860); [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725)
- Context overrun and entity count: one user was fine with 32 exposed entities, another with 53 saw unreliable behaviour and wrong-device actions until context was raised or lights were grouped; about 30 entities cost roughly 1,300 tokens; in one tuned setup the system prompt plus tools take about 12,900 of 16,384 tokens — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860); [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/); [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725)
- Tool budgets are finite: OpenAI-style function calling caps a request at 128 active tools and long tool descriptions get truncated — [HA forum: So you want to AI](https://community.home-assistant.io/t/so-you-want-to-ai-in-home-assistant/1021777)
- Hallucinated or wrong actions: a model invented weather values when the tool returned nothing, tried to "turn on" automations instead of triggering them, refused to call tools for certain questions until the prompt forced it, and ended every reply with a question, which caused re-listening loops after false wake-ups — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860); [same thread, page 15](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860?page=15)
- Reasoning modes hurt voice: a current model family emits a reasoning trace by default so tool calls can land in the wrong field unless thinking is disabled; structured output can override the no-thinking setting on a llama.cpp server unless a reasoning budget is set — [maloyan.xyz, 2026-06-17](https://maloyan.xyz/blog/home-assistant-local-voice-stack-2026); [HA forum: journey, page 15](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860?page=15)
- Speech stages: Piper mangles currency amounts, phone numbers and addresses; replies ending in emoji get read aloud; televisions cause false wake-word triggers; Wi-Fi jitter on the satellite caused mid-word pauses until audio was streamed and the device moved to a dedicated network — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860); [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/)
- Local speech quality is a reason some "local LLM" users keep cloud speech: one integration author found local Faster Whisper and Piper not good enough on latency and transcription quality and uses the paid cloud service for audio — [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725)
- Model choice from public rankings misleads: the model that topped rankings was over eight times slower on the author's actual routing task than a smaller one that was also more accurate; public function-calling leaderboards put the best open models near 50% overall — [bitworking.org, 2026-05-17](https://bitworking.org/news/2026/05/surprising-things-i-learned-putting-together-a-home-brain/); [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/)
- Power: adding GPUs raised one user's idle draw by about 100 W; clock and memory limits halved another card's idle draw from 50 W to 25 W — [HA forum: journey, page 15](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860?page=15)

**The role of deterministic intent matching before the LLM**
- Home Assistant handles commands locally first by default and sends only unmatched requests to the model — [AI in Home Assistant, 2025-09-11](https://www.home-assistant.io/blog/2025/09/11/ai-in-home-assistant)
- A June 2026 guide calls preferring local intents the most impactful setting that guides omit; a November 2026 guide says built-in intents resolve in a few hundred milliseconds but require specific phrasing — [maloyan.xyz, 2026-06-17](https://maloyan.xyz/blog/home-assistant-local-voice-stack-2026); [botmonster, 2026-11-08](https://botmonster.com/smart-home/build-private-local-ai-voice-assistant-2026/)
- Users describe the split as what makes the system feel reliable day to day: routine commands are instant on built-in sentences and the model is a second stage for complex questions (a 12B model on a 16 GB card in one case) — [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725); [HN comment, September 2026](https://news.ycombinator.com/item?id=49533300)
- Even with a model available, users add sentence-trigger automations for things the model does unreliably, such as music playback or weather — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860)
- Counterpoint: one builder described the stock deterministic voice path as rigid pattern matching and saw it as the reason to add a model, and reported that moving the stack from a Raspberry Pi to a desktop made a large difference — [bitworking.org, 2026-05-17](https://bitworking.org/news/2026/05/surprising-things-i-learned-putting-together-a-home-brain/)

**Hardware and model guidance from practitioners (2026)**
- VRAM is the main lever; an 8 GB card suits 3B-8B quantised models, 12-16 GB is described as an excellent home AI server, and a 16 GB card at about $400 is suggested as a budget pick with llama.cpp on Vulkan — [HA forum: So you want to AI](https://community.home-assistant.io/t/so-you-want-to-ai-in-home-assistant/1021777); [HA forum: journey, page 15](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860?page=15)
- Practitioners pick models by tool-calling reliability on their own tasks rather than parameter count, keep chat history short, and prefer llama.cpp with explicitly chosen quantisations over default Ollama tags — [maloyan.xyz, 2026-06-17](https://maloyan.xyz/blog/home-assistant-local-voice-stack-2026); [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860)
- The author of the longest-running thread concludes that the goal was met and core tasks are handled reliably, but would not recommend the setup to an average user — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860)

### Inferences
- With an always-on home computer running Gemma or Qwen class models on a GPU of 12 GB or more, roughly 1-3 s for model-handled requests and well under 1 s for rule-handled requests is the realistic expectation; without a discrete GPU the model should be kept out of the interactive loop.
- The model's job in the successful setups is narrow: route to one of a small set of tools with a short, cache-stable prompt. Designs that put the whole home state and all tools into every prompt are the ones reporting context overrun and wrong devices.
- Prompt-cache stability is a first-order design concern (stable ordering of entities and aliases, fixed tool list); Home Assistant itself patched alias ordering for this in 2026.
- A deterministic layer is not only a latency optimisation but the safety net for model unreliability, which matters for the project's condition-based triggers: those belong in rules, with the model limited to voice interpretation and open questions.

### Gaps
- r/homeassistant and r/LocalLLaMA could not be accessed, so Reddit evidence appears only where secondary sources relayed it.
- No source gave a full per-stage latency breakdown measured on one system (wake word, speech-to-text, model, TTS) in 2026; the only per-stage table found is a stated target.
- Wake-word false-accept rates were reported only anecdotally.
- The student project's full benchmark figures were not extracted; only the abstract was readable.
- Most latency numbers are self-reported by enthusiasts without a shared methodology.

## 9. Accounts from people who built their own hub on MQTT instead of using Home Assistant

### Takeaway
First-hand accounts are few and mostly from 2023-2025. Builders gained simplicity, transparency (configuration in Git, automations as code), stability and responsiveness; what they lost or regretted was the integration ecosystem, the UI and mobile app, and the open-ended maintenance of personal software. The people who stayed happy kept Zigbee2MQTT and ESPHome at the edge and wrote only the logic.

### Cited Findings

**What people built and gained**
- A two-year retrospective (June 2024): ESPHome over MQTT, Mosquitto, Zigbee2MQTT, Node-RED function nodes, Prometheus and Grafana, with Apple HomeKit bridged in as the phone UI; gains were everything in Git, a familiar service architecture and two years of stable operation; the author left Home Assistant because GUI-stored configuration and background containers made the system opaque and fragile to experiment with (pre-2026 source) — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)
- Hacker News, June 2023: a small Ruby program holding the Zigbee2MQTT stream; a quick Rust replacement for a broken commercial hub that the household found more reliable and responsive; a roughly 300-line Python program on MQTT with a scheduler library, whose author found code simpler than web UIs or YAML (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- Hacker News, May 2025: one user bypasses Home Assistant and drives Zigbee devices directly over MQTT through Zigbee2MQTT and reports it works very well; another uses Home Assistant only as an API for Z-Wave and keeps rules in own code (pre-2026 source) — [HN thread, 2025-05-17](https://news.ycombinator.com/item?id=44011381)
- Zigbee2MQTT is routinely documented as runnable without Home Assistant, with a script, Node-RED or its own JavaScript extensions as the automation layer (search summary) — [freeCodeCamp guide](https://www.freecodecamp.org/news/how-to-set-up-zigbee2mqtt-with-docker/); [dhorpynchenko.com](https://dhorpynchenko.com/posts/zigbee2mqtt-extensions/)

**What people regretted or found costly**
- Losing the pre-built integration ecosystem is named as the main loss, with Node-RED the next-best source of integrations (pre-2026 source) — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)
- The UI is the hard part: one Zigbee2MQTT plus Node-RED user says the nice UI is what cannot easily be replaced and wishes for a simple MQTT-driven interface for web and mobile (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- A developer who has maintained a custom platform since 2013 now mostly uses a commercial system and concludes that custom software in one's life is more liability than asset; others warn of spending more time on the software than on the automations and that maintenance is an open-ended job (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- Household acceptance and failure cost: a race condition made a hall light strobe while the owner was away; commenters stress that a home is only smart if other occupants can use it unaided (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- A would-be builder who wanted an all-in-one system that just works acknowledged that this is probably what Home Assistant's developers thought before reaching today's complexity (pre-2026 source) — [HN comment, 2025-05-17](https://news.ycombinator.com/item?id=44013793)

**Counter-arguments from Home Assistant users**
- March 2026: a commenter grants that switching lights and reading sensors needs only a tiny executable but argues the value of Home Assistant is being the single connecting point for every protocol and device added later, plus one-click packaging of Zigbee2MQTT, ESPHome and camera software — [HN comment, 2026-03-17](https://news.ycombinator.com/item?id=47406771)
- 2023: users report that the initial learning is worth more than writing one's own, that updates stopped breaking things for them, and that a Raspberry Pi install ran for two years without issues (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- Requests for a long-term-support branch because breaking changes get tiresome appear in the same thread (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)

**A middle path that recurs**
- Several accounts keep Home Assistant or its building blocks as a device layer and put logic elsewhere: custom code against the Home Assistant API, Node-RED plus an external MQTT program, or a custom voice pipeline calling the REST API — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688); [bitworking.org, 2026-05-17](https://bitworking.org/news/2026/05/surprising-things-i-learned-putting-together-a-home-brain/)

### Inferences
- The regrets cluster on exactly the two requirements that a lean hub does not get for free from ESPHome and Zigbee2MQTT: a phone app or UI, and breadth of off-the-shelf device support beyond Zigbee, ESPHome, Tasmota and Shelly.
- Reported satisfaction correlates with a narrow scope (a known set of device families, logic as code) and with reusing the bridges; nobody in these accounts reports writing their own Zigbee stack or device firmware.
- The evidence base is thin and self-selected: people who abandoned a custom hub rarely write retrospectives, and people who succeeded are few.

### Gaps
- No detailed 2026 retrospective by someone who replaced Home Assistant with a custom MQTT hub was found; a Hacker News comment search for 2025-2026 returned almost nothing on point, and Reddit was inaccessible.
- No account was found of a custom hub that also carries local-LLM voice control without Home Assistant; the closest is the Go pipeline that still uses Home Assistant for devices.
- No data on how long custom hubs survive, or on migration back to Home Assistant, was found.

## 10. Evidence bearing on the decision: custom hub on an MQTT bus with ESPHome and Zigbee2MQTT at the edge, versus Home Assistant as the core, versus everything from scratch

### Takeaway
The evidence most strongly rules out "everything from scratch" (the device edge is where thousands of contributor-years sit) and leaves a real trade-off between the other two: Home Assistant delivers voice, phone app, Matter and LLM plumbing today at the price of upgrade churn and a large surface, while a custom MQTT hub reusing ESPHome and Zigbee2MQTT gets self-describing devices almost for free but must build or adapt the UI or phone app, the automation engine, the voice server side and any Matter commissioning itself.

### Cited Findings

**Evidence against building the device edge from scratch**
- Zigbee2MQTT describes 5,868 devices from 616 vendors and exposes each device's capabilities, ranges and access rights in one retained JSON document plus a request/response API for pairing and management — [Supported devices](https://www.zigbee2mqtt.io/supported-devices/); [MQTT topics and messages](https://www.zigbee2mqtt.io/guide/usage/mqtt_topics_and_messages.html); [Exposes](https://www.zigbee2mqtt.io/guide/usage/exposes.html)
- ESPHome's single November 2026 release contained 1,463 pull requests from over 240 contributors and over thirty new components, on top of hundreds of existing ones, with OTA, provisioning (Improv, captive portal), a local web UI and Home Assistant-style MQTT discovery built in — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/); [esphome.io](https://esphome.io/); [MQTT component](https://esphome.io/components/mqtt/)
- No first-hand account found describes writing a Zigbee stack or device firmware for a personal hub; satisfied builders reuse Zigbee2MQTT and ESPHome and write only logic — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/); [HN thread, 2025-05-17](https://news.ycombinator.com/item?id=44011381)
- Long-term custom-platform maintainers report that personal software becomes a liability and that maintenance never ends (pre-2026 source) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)

**Evidence for a custom hub on MQTT that reuses ESPHome and Zigbee2MQTT**
- Self-description is available on the bus without Home Assistant: Home Assistant-style discovery from ESPHome, OpenBeken and OpenMQTTGateway, Zigbee2MQTT's `bridge/devices`, Tasmota's native discovery tree, and Shelly's announce plus RPC; other hubs (openHAB, Domoticz) already consume the discovery format — [MQTT integration](https://www.home-assistant.io/integrations/mqtt/); [MQTT component](https://esphome.io/components/mqtt/); [Tasmota Home Assistant page](https://tasmota.github.io/docs/Home-Assistant/); [Shelly MQTT component](https://shelly-api-docs.shelly.cloud/gen2/ComponentsAndServices/Mqtt/)
- ESPHome without Home Assistant on MQTT is documented (one setting to stop API-timeout reboots) and reported stable over two years — [MQTT component](https://esphome.io/components/mqtt/); [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)
- The voice and model building blocks are separable services: Wyoming speech services, Piper, Faster Whisper, Speech-to-Phrase, the intent corpus and matcher, microWakeWord, and any OpenAI-compatible model server — [OHF-Voice/wyoming README](https://github.com/OHF-Voice/wyoming); [llama.cpp integration](https://www.home-assistant.io/integrations/llama_cpp/)
- The native-API-only ESPHome features can in principle be served by a custom controller, because aioesphomeapi (MIT) exposes voice-assistant, Bluetooth, proxy and entity calls — [aioesphomeapi client.py](https://github.com/esphome/aioesphomeapi/blob/main/aioesphomeapi/client.py)
- A Matter controller can be added later as a standalone service with a WebSocket API — [matterjs-server README](https://github.com/matter-js/matterjs-server)
- The successful local-LLM setups use a narrow tool set, a short cache-stable prompt and deterministic rules in front, which a purpose-built hub can enforce by design; Home Assistant users reach the same shape by restricting exposure and adding sentence triggers — [HA forum: AI Plugin](https://community.home-assistant.io/t/integration-tuned-for-running-assist-on-a-local-llm-in-8-gb-vram/1025725); [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860)
- Builders report simplicity, automations as code under version control, stability and better responsiveness — [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/); [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688)
- Avoiding Home Assistant also avoids its churn: at least 44 backward-incompatible entries in 2026.11 alone and dated deprecations in the LLM API — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/); [LLM API developer docs](https://contributors.home-assistant.io/core/internals/llm-api/)

**Evidence against the custom hub (costs it must absorb)**
- UI and phone app: builders name the UI as the part they cannot easily replace and wish for an MQTT-driven web and mobile interface; one bridged devices into Apple's ecosystem to get a phone UI (pre-2026 sources) — [HN thread, 2023-06-05](https://news.ycombinator.com/item?id=36191688); [sambobb.com, 2024-06-29](https://www.sambobb.com/posts/home-automation-without-home-assistant/)
- Voice server side: ESPHome's voice component is specified as streaming to Home Assistant, and the native API documents only three clients, so a custom hub must implement the pipeline server against aioesphomeapi without reference documentation — [ESPHome voice assistant](https://esphome.io/components/voice_assistant/); [Native API component](https://esphome.io/components/api/)
- openHAB, a mature hub with many contributors, still lacked ESPHome voice support in 2026, which indicates the effort involved — [openHAB forum thread](https://community.openhab.org/t/revisiting-local-llm-powered-voice-assistants-in-2026/168251)
- Thread and Matter outside Home Assistant took one experienced user multiple evenings and still leaned on a phone ecosystem — [openHAB forum, 2026-05-23](https://community.openhab.org/t/how-to-use-an-orange-pi-zero-2-running-otbr-with-openhab/169352)
- ESPHome and the discovery schema keep changing (ESPHome breaking-change lists each release; MQTT option removals on the Home Assistant side), so a custom consumer inherits some churn anyway — [ESPHome 2026.11 post](https://esphome.io/blog/2026/11/18/esphome-2026-11/); [2026.4 release post](https://www.home-assistant.io/blog/2026/04/01/release-20264/)
- New lean hubs on the same idea have not gained users: three 2026 projects sit at 0, 3 and 9 stars (GitHub API) — [fdatoo/switchyard](https://github.com/fdatoo/switchyard); [gethome-inc/gethome-hub](https://github.com/gethome-inc/gethome-hub); [elaris-control/core](https://github.com/elaris-control/core)
- The MQTT broker becomes a single point of failure for the whole device edge — [Native API component](https://esphome.io/components/api/)

**Evidence for Home Assistant as the core**
- It already delivers every listed requirement in one product: voice pipeline with local speech and wake word, local-model agents through Ollama or any OpenAI-compatible server, schedules and condition-based automations, sub-second deterministic voice commands, companion apps, Zigbee, Matter and Thread — [Fully local assistant docs](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/); [Ollama integration](https://www.home-assistant.io/integrations/ollama/); [llama.cpp integration](https://www.home-assistant.io/integrations/llama_cpp/); [Matter integration](https://www.home-assistant.io/integrations/matter/)
- Scale and momentum: 92,492 stars, releases every month with weekly patches, 38 new integrations in one release, and the same foundation now maintains ESPHome, the voice stack and the Matter server (GitHub API) — [home-assistant/core](https://github.com/home-assistant/core); [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/); [Home Assistant Matter post, 2026-06-23](https://home-assistant.io/blog/2026/06/23/the-matter-upgrade-youve-been-waiting-for)
- It is extensible from outside without forking: custom LLM tool sets through an `llm.py` platform, an MCP endpoint per tool set, and the REST and WebSocket APIs that custom pipelines already use — [LLM API developer docs](https://contributors.home-assistant.io/core/internals/llm-api/); [bitworking.org, 2026-05-17](https://bitworking.org/news/2026/05/surprising-things-i-learned-putting-together-a-home-brain/)
- The argument from users: the value is being the one place that connects whatever device arrives next — [HN comment, 2026-03-17](https://news.ycombinator.com/item?id=47406771)
- Modest hardware suffices for the hub itself (2 GB minimum, a 4 GB appliance at about 3 W), leaving the home computer for models — [Installation](https://www.home-assistant.io/installation/); [Home Assistant Green](https://www.home-assistant.io/green/)

**Evidence against Home Assistant as the core**
- Churn and surface: monthly breaking-change lists, removal of two installation methods in 2025, the community store being absorbed in 2026.11, and user reports of upgrade fear and opacity — [2026.11 release post](https://www.home-assistant.io/blog/2026/11/04/release-202611/); [Deprecation post](https://www.home-assistant.io/blog/2025/05/22/deprecating-core-and-supervised-installation-methods-and-32-bit-systems/); [HN thread, 2025-05-17](https://news.ycombinator.com/item?id=44011381)
- Paths that conflict with the no-cloud, no-account constraints unless avoided: Firebase-based push notifications, phone-ecosystem Matter commissioning, subscription remote access — [Companion app notification details](https://companion.home-assistant.io/docs/notifications/notification-details/); [Matter integration](https://www.home-assistant.io/integrations/matter/); [Rename announcement, 2026-10-02](https://home-assistant.io/blog/2026/10/02/big-tech-ruined-the-cloud-so-were-renaming-ours/)
- Even on Home Assistant, dependable local-LLM voice took enthusiasts months of prompt, model and automation tuning, and the most experienced author does not recommend it to average users — [HA forum: journey](https://community.home-assistant.io/t/my-journey-to-a-reliable-and-enjoyable-locally-hosted-voice-assistant/944860)
- The container install lacks apps, so Thread, Z-Wave and the supported Matter server path push towards the appliance-style OS — [Installation](https://www.home-assistant.io/installation/); [Matter integration](https://www.home-assistant.io/integrations/matter/)

### Inferences
- The two viable options differ mainly in who owns the "hub core" (state, automations, UI, voice server): in both, ESPHome, Zigbee2MQTT, the Wyoming speech services, an OpenAI-compatible model server and optionally the standalone Matter server are the same reusable blocks. That makes the decision reversible at the edge: devices that announce themselves over MQTT can be attached to Home Assistant later, or run beside it.
- The custom hub's unavoidable build list, judged from what the sources show is not reusable without Home Assistant: (1) a discovery consumer for Home Assistant-style config messages and Zigbee2MQTT's device document, (2) a rule and schedule engine, (3) a phone-usable UI, (4) the server side of the ESPHome voice protocol or an alternative satellite path such as Wyoming-based or Willow-style endpoints, (5) the tool layer between the model and the bus. Items 3 and 4 are where the accounts report pain and where no small reusable component was found.
- A hybrid deserves explicit consideration because it appears repeatedly in the accounts: keep Home Assistant as a device and app layer (integrations, companion app, Matter) while the project's own service owns voice, model routing and complex condition logic over MQTT, REST or MCP. This avoids rebuilding the UI and commissioning paths but keeps Home Assistant's upgrade churn in the system.
- For the stated feature list, nothing in the evidence requires Matter or Thread in a first version: sockets, switches and curtain motors are covered by Zigbee, ESPHome, Tasmota and Shelly devices, and the complex-condition triggers are ordinary sensor logic once illuminance and sound level arrive as MQTT state.
- The strongest argument for the custom route is not footprint (Home Assistant's is small) but control over the model interface: a hub designed around a narrow, cache-stable tool set and deterministic-first routing matches what the successful local-LLM users converged on.
- The strongest argument against it is breadth over time: every new device family beyond the self-describing ones costs adapter work that Home Assistant users get from monthly releases.

### Gaps
- No evidence was found on the effort, in time, of building a discovery consumer, rule engine and UI of the scope described; the newcomer projects were not inspected for how far they got.
- No source compared a custom MQTT hub and Home Assistant on the same hardware for latency or reliability.
- Open-source phone app options for a custom hub (generic MQTT dashboards, progressive web apps, bridging into Matter or HomeKit) were not researched; note that bridging into a phone vendor's home app would reintroduce a vendor account.
- No documented example was found of a non-Home-Assistant server implementing the ESPHome voice-assistant protocol end to end.
- Reddit could not be accessed and the web-search budget was exhausted before follow-up searches on lightweight brokers, the ESPHome Zigbee release history and late-2026 IKEA firmware status.
