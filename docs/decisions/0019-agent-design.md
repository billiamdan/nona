# ADR-0019: A few narrow agents behind a deterministic router, not a swarm

**Status:** Proposed · **Date:** 2026-11-25

## Context

The brief describes a host application that operates "a swarm of local agents and LLMs". The models that fit this hardware are small.

What the research found (November 2026):

- For models under about 10B the best-supported pattern is a single agent with a small, curated tool set and deterministic scaffolding. In the largest 2026 study, multi-agent setups that route between agents used about 1.8 times the tokens of a single agent and completed fewer tasks (72% against 80%).
- Specialisation still helps when the routing is done by something cheap and reliable, such as rules or a small classifier, that picks a narrow prompt per task.
- The framework matters less than the scaffolding. The measurable gains come from validation, retries, step limits and control of the context. A hand-written loop of a few hundred lines is sufficient.
- The Model Context Protocol is a reasonable local tool interface; small models should be shown only a per-request subset of tools.

## Decision

- A deterministic router picks one narrow agent per request. An agent is a prompt, a short tool list and an output schema. All agents share the same model endpoints ([ADR-0018](0018-llm-runtime-and-tiers.md)).

| Agent | Job | Tools | When |
|---|---|---|---|
| `operator` | Device commands and questions about the home's state | Device and state tools, enumerated entities | First model phase |
| `rule-author` | Turns a described automation into a rule draft for approval | None; it outputs a rule | After the rule engine exists |
| `narrator` | Summaries and briefings from the event log | Read-only state and history | Later |
| `perception` | Describes an audio clip or snapshot that a detector flagged | None; fixed output schema | Later |

- Agents do not talk to each other. They coordinate through the hub: device state, the event log and the rule store.
- The loop is hand-written against the OpenAI-compatible endpoint: validate the tool call, retry once with the error, enforce a step limit and a timeout. No agent framework in the first version.
- The same tool layer can later be exposed as a local MCP server, so other local clients can use the home.

## Consequences

- Each prompt stays short and cache-stable, which is what small models and a CPU-only host need.
- Adding a capability means adding one agent module; nothing emergent has to be debugged.
- This is narrower than "swarm" suggests. The brief's intent, several specialised models working for the home, is kept; chatty agent-to-agent delegation is not.

## Revisit when

A capable model is available around the clock. Then one planning agent that directs narrow workers becomes worth testing.

Evidence: [local LLM notes](../../research_notes/Local%20smart%20home%20stack%20options/local_llm_and_runtimes.md), section 8.
