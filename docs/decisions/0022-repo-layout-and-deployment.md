# ADR-0022: One repository; third-party services in containers, our own services native

**Status:** Proposed · **Date:** 2026-11-25

## Context

The system has a hub, a web app, firmware configurations, deployment files and documents, built by one person with AI assistance. Development happens on the Mac; production is the Linux laptop ([ADR-0003](0003-laptop-host-tiered-compute.md)).

## Decision

**One repository.**

| Folder | Contents |
|---|---|
| `hub/` | The Python hub: core, adapters, agents, voice |
| `app/` | The web app |
| `firmware/esphome/` | One shared base package and one YAML file per node |
| `contracts/` | The schemas the parts agree on: capability model, rule format, MQTT topic conventions |
| `deploy/` | The container file for third-party services, system service units, broker and proxy configuration |
| `tools/` | Device simulator, model benchmark, command-line client |
| `docs/` | Brief, architecture, roadmap, components list, decision records |
| `reports/`, `research_notes/` | Research behind the decisions |

**Tooling.** Python with `uv`, `ruff`, strict type checking and `pytest`. The web app with TypeScript and Vite.

**Running it.**

| Component | How it runs | Why |
|---|---|---|
| Mosquitto, Zigbee2MQTT, the HTTPS reverse proxy (Caddy) | Containers from one file in `deploy/`, identical on the Mac and the laptop | Reproducible installs of software we do not write |
| The hub, the model server, the speech servers | Native system services that restart on failure | Direct access to the CPU, simpler debugging, no container overhead on 16 GB |

- One restart mechanism per component: the container runtime for containers, the system service manager for native services.
- Configuration is one file; secrets are in a separate file that is never committed.
- Data is one SQLite file plus the rule and registry exports, backed up nightly to a second location.

**Development without hardware.** A simulator in `tools/` publishes fake devices in the same self-description format as real ones, so the hub and the app can be built and tested on the Mac with nothing plugged in.

## Consequences

- A change that spans firmware, hub and app is one commit.
- The same container file gives the Mac and the laptop the same broker and bridge versions.
- The Zigbee coordinator must be passed through to its container on the laptop. On the Mac there is no coordinator; Zigbee devices are simulated there.

## Revisit when

The repository becomes awkward for one of its parts, for example if the Android shell needs its own release cycle.

Evidence: [app and security notes](../../research_notes/Local%20smart%20home%20stack%20options/app_remote_security.md), section 10.
