# ADR-0011: Buy in phases; the first round stays within €100

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

The owner set a ceiling of €100 for the first purchase round, not counting any new computer. The project is built one thin slice at a time, and hardware bought ahead of the software that uses it tends to sit in a drawer or turn out to be the wrong choice.

## Decision

- Purchases are grouped in tiers, and each tier belongs to a roadmap phase. Nothing is bought for a phase that has not started.
- Tier 0 is the hardware already owned. The first phase costs nothing.
- Tier 1 is the first purchase round and stays within €100.
- A later tier is bought only after the previous phase has passed its exit demo.
- One unit first; more of the same only after it has proved itself in the home.
- Prefer parts that stay useful across phases over parts that serve a single experiment.

The tiers, with items, prices and alternatives, are in [bom.md](../bom.md).

## Consequences

- The curtain drive and dedicated voice satellites do not fit into the first round and come in later tiers.
- Prices move; the list records the date of each price and is re-checked before every order.
- A dedicated AI computer is outside these tiers and is decided separately ([ADR-0003](0003-laptop-host-tiered-compute.md)).

## Revisit when

The owner changes the budget.
