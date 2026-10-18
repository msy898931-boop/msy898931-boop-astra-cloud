# ASTRA Cloud — commercial plan

**Commercial layer for an honest volatility risk-filter.**

This repository holds the plan for turning the validated volatility edge in
[`msy898931-boop/asp`](https://github.com/msy898931-boop/asp) into a sellable,
multi-tenant SaaS. It is documentation only: **no product code yet**.

| | |
| --- | --- |
| **What it sells** | A volatility risk filter: "the next window is likely violent — stand aside." |
| **What it never sells** | Signals, direction, predictions, "profitable bot", managed money. |
| **Scientific base** | Trailing volatility ranks forward volatility with AUC 0.69–0.79 (measured, OOS 2025, 113,083 rows). |
| **Status** | Plan only. Awaiting approval. |

## Read in this order

1. **`docs/PLAN.md`** — the complete, researched plan.
2. **`docs/RESEARCH_SOURCES.md`** — every claim, with its source.
3. **`docs/OPEN_DECISIONS.md`** — what must be decided before any code.

## Relationship to `asp`

`asp` is a single-tenant, keyed, shadow-mode tool scored 84.5/100 as an
*engineering tool* — and explicitly "not yet a commercial product" (static key
gate, no accounts, no billing, thin mobile surface). Commercialising it means
adding a multi-tenant product surface on top of science that is already
validated.

This repository is that commercial layer. It **consumes** `asp`'s engine and
research code; it does not modify or replace it. Nothing in `asp` is changed
by the work planned here.

## ⚠️ Visibility

This repository was created **public** and the automation token cannot change
that (it lacks repository-administration permission). It holds commercial
strategy, pricing and a legal plan, so **make it private**:

> Settings → scroll to the bottom → Change visibility → Make private

---

_Plan prepared by an AI agent (OpenHands) on behalf of @msy898931-boop._
