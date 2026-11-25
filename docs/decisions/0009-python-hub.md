# ADR-0009: The hub is written in Python

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

The hub is the always-on service: device registry, state, scheduler, rule engine, API for the app, agent runtime and voice orchestration. It is I/O-bound; the heavy computation (language and speech models) runs in separate server processes.

## Options considered

| Option | For | Against |
|---|---|---|
| **Python (chosen)** | ESPHome's own client library (aioesphomeapi) and the speech and machine-learning tooling are Python-first; one language for the hub and the AI glue; fastest to iterate | Long-running reliability needs care; higher memory use than a compiled service |
| TypeScript (Node.js) | One language for hub and app | Speech and ESPHome voice would need a Python sidecar |
| Go | Small static binary, easy concurrency | Same Python sidecar |
| Rust | Leanest and most robust at runtime | Slowest to develop; same Python sidecar |

## Decision

- Python (current stable release), asynchronous throughout, as one process: a modular monolith.
- Managed with `uv`; formatted and linted with `ruff`; type-checked in strict mode; tested with `pytest`.
- Default libraries, each replaceable behind a thin interface: `aiomqtt` (bus), `FastAPI` with `uvicorn` (HTTP and WebSocket API), `pydantic` (typed models and schemas), SQLite through `aiosqlite` (state, history, rules), `aioesphomeapi` (voice satellites), `astral` (sunrise and sunset).

## Consequences

- No sidecar process is needed for ESPHome voice or for speech tooling.
- Reliability is engineered rather than assumed: a system service that restarts on failure, strict typing, structured logs, and no blocking call inside the event loop.
- Model servers are separate processes, so a crashed or overloaded model cannot take schedules and rules down with it ([ADR-0016](0016-deterministic-core.md)).
- Memory use of the hub is small next to the models and is acceptable on 16 GB.

## Revisit when

Profiling shows the hub core itself is a bottleneck. This is unlikely, because its work is waiting on the network.
