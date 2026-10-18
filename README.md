# ASTRA Cloud — commercial plan (corrected)

**Commercial layer for an honest volatility risk-filter.**

Documentation only — no product code. Every claim here was checked against the
live `asp` repository at commit `b3a1845` by running the tools, not by reading
the docs. Where something could not be verified, it says so.

| | |
| --- | --- |
| **What it sells** | A volatility risk filter: "the next window is likely violent — stand aside." |
| **What it never sells** | Signals, direction, predictions, "profitable bot", managed money. |
| **Scientific base** | Trailing volatility ranks forward volatility, AUC 0.69–0.79 (OOS 2025, 113,083 rows). |
| **Status** | Plan. Awaiting approval on two decisions. |

## ⚠️ Read this first

The first version of this plan was wrong in an important way, and
**`docs/STATE_OF_PLAY.md` documents the correction**. Three things it scheduled
as future work are already built and passing:

| Proposed as future work | Actual status |
| --- | --- |
| Quality gates (`ruff` + `mypy` + coverage) | ✅ **in CI, all green** |
| Volatility product surface | ✅ **`volatility.py` + model + parity test** |
| Real mobile (offline + Web Push) | ✅ **built** |

The real blocker is different from the one the plan assumed:
**`docs/SELLING.md` is built around Stripe Payment Links, and Stripe does not
serve Syria.** The product is ready to sell; the payment rail is not.

## Read in this order

1. **`docs/STATE_OF_PLAY.md`** — verified facts, with evidence, and the
   correction. Start here.
2. **`docs/PLAN.md`** — the commercial plan, corrected against those facts.
3. **`docs/RESEARCH_SOURCES.md`** — every external claim with its source.
4. **`docs/OPEN_DECISIONS.md`** — the two decisions needed before any code.

## Measured state, 2026-10-18

| Check | Command | Result |
| --- | --- | --- |
| Lint | `ruff check .` | all checks passed |
| Types | `mypy bot tracker akoan advisor bridge mobile src` | no issues, 65 files |
| Coverage | `coverage report --fail-under=75` | 78 % |
| Tests | 8 suites | 972 passing, 4 skipped |

## Relationship to `asp`

`asp` holds the science, the engine and 972 passing tests. This repository holds
the commercial plan that consumes them. **Nothing in `asp` is modified by the
work planned here.**

## ⚠️ Visibility

Created **public**; the automation token cannot change that (no repository-
administration permission). It holds commercial strategy and pricing, so set it
private: Settings → Change visibility → Make private.

---

_Plan prepared by an AI agent (OpenHands) on behalf of @msy898931-boop._
