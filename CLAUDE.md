# Nona

A lean, fully local smart home: a self-built Python hub on an MQTT bus, ESPHome and Zigbee2MQTT at the device edge, open-weight models on a home computer. No cloud, no subscription, no vendor account.

## Working agreements

- **Record every decision.** Any choice that shapes the stack, the hardware or the way of working gets a record in `docs/decisions/` (next free number, template in `docs/decisions/README.md`) and a row in that index. Change a decision by adding a record that supersedes the old one; do not rewrite the old one.
- **Hard constraints.** Nothing may need the internet, a subscription or a vendor account in order to operate (ADR-0001). Mains-voltage parts are bought, certified products only (ADR-0004). The installation must be fully reversible: rented apartment, no rewiring (ADR-0005).
- **Deterministic core.** Manual control, schedules and rules must work with every AI service stopped. A model only proposes: a validated tool call or a rule draft the owner approves (ADR-0016, ADR-0023).
- **Lean order of work.** Follow `docs/roadmap.md`: one thin end-to-end slice at a time, each phase ending in its exit demo. Do not build or buy ahead of the current phase.
- **Both languages.** A voice feature is done only when it works in English and in Russian (ADR-0006).
- **Keep the documents true.** When work changes a fact in `docs/`, update the document in the same change. Unknowns go in `docs/open-questions.md`.

## Where things are

- `docs/project-brief.md`: goals, features, owned hardware, the owner's answers
- `docs/architecture.md`: layers, flows, contracts, failure behaviour
- `docs/roadmap.md`: the phased plan with exit demos
- `docs/bom.md`: what to buy, in tiers
- `docs/decisions/`: decision records, indexed in its README
- `docs/open-questions.md`: what is not decided or not known yet
- `reports/`, `research_notes/`: research behind the decisions (November 2026)
