# ASTRA Cloud — Complete Commercial Plan

**Version:** 1.0
**Date:** 2026-10-18
**Author:** OpenHands agent, on behalf of @msy898931-boop
**Status:** PLAN — awaiting approval. No code has been written.

---

## 0. Executive summary

You already own something rare: a project that **measured its own edge and
honestly reported where it has none**. That is the asset. Almost every
competitor in this space is selling a lie; you have a real, reproducible number.

The plan is therefore not "build a trading bot". It is:

> **Sell the one thing that is measurably true — forward volatility is
> predictable — as a multi-tenant SaaS risk filter, from Syria, paid in
> crypto first and via a foreign entity later.**

**The single most important decision in this document** is the product
boundary in §4: we sell a *risk filter*, never a *signal*. That boundary is
what keeps the product legal, sellable on payment platforms, and honest.

| Dimension | Decision |
| --- | --- |
| Product | Volatility risk filter ("stand aside") + risk dashboard |
| Buyer | Retail XAUUSD / indices / crypto traders who already trade |
| Pricing | Free tier + $19/mo + $49/mo |
| Rail (now) | Crypto (USDT) via a gateway with crypto payouts |
| Rail (later) | Turkish/Georgian entity → Paddle or Stripe |
| Legal | No advice, no execution, no managed money → no licence needed |
| Build order | **Quality gates ✅ done · volatility surface ✅ done · mobile ✅ done** → accounts → **payment rail** → legal → launch |

> **Correction, verified 2026-10-18.** Three phases this plan originally
> scheduled as future work are **already built and passing**: the quality gates
> (`ruff`/`mypy`/coverage in CI), the volatility product surface
> (`volatility.py` + `vol_model.json` + engine-parity test), and the real mobile
> experience (offline shell + Web Push). See `STATE_OF_PLAY.md` for the evidence.
>
> The remaining blocker is not code. It is that **`docs/SELLING.md` is built
> around Stripe Payment Links, and Stripe does not serve Syria** — so the
> product is ready to sell and the rail is not. That is now the top of the
> roadmap, not phase 4.

---

## 1. Where we actually are (measured, not assumed)

Everything below was read from `msy898931-boop/asp` in this session.

### 1.1 The scientific asset — this is the real value

From `research/PHASE0_VOLATILITY_DECISION.md`, on a decision-grade corpus
(**113,083 rows**, XAUUSD, 2021–2025, OOS = 2025):

| Horizon | Minutes | Volatility Spearman | **Volatility AUC** | Direction base rate |
| --- | --- | --- | --- | --- |
| 1 | 15 | 0.000 | (undefined) | 0.517 |
| 4 | 60 | 0.388 | **0.688** | 0.532 |
| 16 | 240 | 0.455 | **0.729** | 0.554 |
| 96 | 1440 | 0.619 | **0.794** | 0.590 |

Stability of volatility AUC at horizon 16, by calendar year:

| 2021 | 2022 | 2023 | 2024 | 2025 |
| --- | --- | --- | --- | --- |
| 0.646 | 0.669 | 0.655 | 0.668 | 0.729 |

**Interpretation.** The edge holds in *every* year — it is not a regime
artifact. This is a real, usable signal.

### 1.2 The uncomfortable truth we must build around

The same study is explicit, and we must respect it:

- **Direction has no edge.** Momentum and reversal both sit on the base rate
  (0.49–0.51). The famous "0.517 directional accuracy" was **market drift, not
  skill** — the OOS up-move base rate at horizon 1 is *identical*, 0.517.
- The promotion gate in `docs/research/PLAN_A_FEATURE_STORE_AND_GATE.md`
  evaluated **7,084 resolved trades** and found: **nothing clears the gate**.
  M15 (the live timeframe) is net-losing in both halves (−414R IS / −156R OOS).
- The one surviving lead (`PLAN_C`: H4-aligned + post-news) is **REJECTED** by
  the gate — permutation p = 0.0067 is encouraging but the Wilson lower bound
  does not clear break-even on both halves.

**Consequence for the product:** we cannot sell direction, signals, or "win
rate". Anyone who does is either lying or curve-fitting. We sell **volatility**.

### 1.3 Engineering state (re-measured 2026-10-18, superseding the assessment)

The assessment below was accurate on **2026-10-16**. It is no longer current:
six of its twelve gaps closed within two days. These are the measured figures
from commit `b3a1845`, produced by running the tools.

| Check | Assessment said (16 Oct) | **Measured now (18 Oct)** |
| --- | --- | --- |
| Score | 84.5 / 100 | 84.5 — but 6 gaps since closed |
| Python tests | 597 | **972 passing**, 4 skipped |
| C++ test binaries | 19/19 pass | 19 (unchanged) |
| Committed secrets | none | none |
| Read-only by construction | yes | yes |
| **Linter** | **none** | ✅ **`ruff check .` → all checks passed** |
| **Type checker** | **none** | ✅ **`mypy` → no issues in 65 files** |
| **Coverage** | **none** | ✅ **78 %, floor enforced at 75 %** |
| Offline shell | missing | ✅ built (`sw.js`) |
| Web Push | missing | ✅ built + real crypto round-trip test |
| Gate rate limiting | missing | ✅ built (`ASTRA_GATE_LOCKOUT_S=900`) |
| Oversized `app.js` | 1675 lines | ✅ split into nine scripts (`aa0b6fc`) |
| Public gate | one static key | ⚠️ still one static key — now rate-limited |
| Live usage | 2 APK downloads | not re-measured |

**The corrected verdict:** the engineering gap the assessment identified is
closed. What remains is *productisation* — accounts, a legal surface, Android
hardening, and above all a payment rail that works from Syria.

Test count detail (measured): `bot` 152, `tracker` 44, `research` 50,
`akoan` 59, `advisor` 92, `mobile` 162, `tests` 308, `bridge` 105 = **972**.
The README table says 961; `mobile` and `bridge` are understated by 11.

---

## 2. Market research

### 2.1 Is there demand for a volatility/risk tool?

Yes, and it is a mature category:

- The risk management software market was ~$41.4B in 2024, projected to
  ~$87.2B by 2032 (≈9.75% CAGR).
- PwC's 2026 survey: 54% of financial-services executives name risk management
  as a capability that became *more* critical in volatile markets.
- But trust is the constraint: only **53% of consumers trust AI-powered
  financial tools** during volatility. **This is our opening** — a tool that
  publishes its own AUC and says "no directional edge" is the trust play.

### 2.2 Who is the buyer?

Not "people who want to get rich". That audience buys scams and churns.

The buyer is someone who **already trades** XAUUSD / indices / crypto and has
felt a volatility spike destroy a position. They do not want a signal. They
want to stop walking into spikes. This buyer:

- already has an account somewhere,
- understands drawdown,
- will pay for a tool that stops a specific, felt pain,
- is **not** impressed by "AI robot" marketing — it makes them suspicious.

### 2.3 Competitive positioning

| Competitor type | What they sell | Our difference |
| --- | --- | --- |
| Signal channels / "AI bots" | Direction, win-rate promises | We publish **no directional edge** |
| Charting platforms | Charts + indicators | We sell a **decision to stand aside**, with a measured AUC |
| Risk/portfolio platforms | Institutional, expensive | We are **retail-priced, single-pain, mobile-first** |

Our moat is **credibility**, which is the hardest thing for a competitor to
copy because copying it means admitting their own numbers are bad.

---

## 3. Legal & regulatory research

> **Not legal advice.** Researched from primary sources; verify with a Syrian
> lawyer before trading.

### 3.1 Syria sanctions — lifted

| Date | Action | Source |
| --- | --- | --- |
| 2025-05-13 | US announces lifting of sanctions | OFAC |
| 2025-06-30 | **E.O. 14312** revokes six Syria EOs; national emergency terminated | OFAC |
| 2025-06-30 | Syrian financial institutions (incl. Central Bank) **removed from SDN list** | OFAC FAQ 1221 |
| 2025-08-26 | OFAC removes 31 CFR Part 542 (Syrian Sanctions Regulations) | Federal Register |
| 2025-12-18 | **Caesar Act repealed** (NDAA FY2026) | Thompson Hine |
| 2026-08-24 | State Dept **rescinds Syria's State-Sponsor-of-Terrorism designation** | State Dept |

**Result:** US comprehensive sanctions are gone. Residual **list-based**
sanctions remain on named persons (former Assad associates, human-rights
abusers, Captagon traffickers, ISIS/Al-Qa'ida affiliates). We must screen
against the SDN list — but the country is no longer blocked.

### 3.2 Crypto in Syria

**Legal grey area, improving.** No Syrian law permits or forbids crypto; the
transitional government has not made digital-asset policy a priority. AML/CFT
obligations exist under Legislative Decree No. 33, overseen by the CCMLFT, and
do **not** explicitly cover virtual assets.

**Practical implication:** individual crypto use is not illegal. Operating a
*financial service* (custody, exchange, managing others' money) would attract
scrutiny. **We are neither** — we sell software.

### 3.3 Why our product needs no financial licence

| Activity | Licensed? | Do we do it? |
| --- | --- | --- |
| Giving investment advice / recommendations | Yes | **No** — we describe volatility, not direction |
| Executing trades for clients | Yes | **No** — read-only, shadow mode |
| Managing client money | Yes | **No** — we never touch funds |
| Selling software tools | No | **Yes** — this is all we do |

**The boundary that keeps us legal (§4.1) is also the boundary that keeps us
unlicensed.** They are the same line. Crossing it — even "just adding signals"
— changes our legal category overnight.

---

## 4. Product definition

### 4.1 The product boundary (the most important section)

| ✅ We sell | ❌ We never sell |
| --- | --- |
| "Trailing volatility is in its top band — the next window is likely violent." | "Buy/sell", "CALL/PUT", any direction |
| A measured AUC, published openly | Win-rate claims |
| A risk dashboard: your exposure, your spikes | Signals, "AI robot", auto-trading |
| Alerts to a price *you* set | Predictions of price |
| An honest "no edge measured" where true | "Guaranteed", "risk-free", any performance promise |

**Hard rule:** the app must be able to say *"Edge: none measured"* for
direction and *still be worth paying for* because of volatility. If a feature
cannot survive that rule, it does not ship.

### 4.2 Product surfaces

1. **Volatility filter** — the core. Consumes market data, outputs a band
   (quiet / normal / violent) with a calibrated probability and the trailing
   volatility that produced it.
2. **Risk dashboard** — position sizing helper, exposure summary, spike log.
   Deterministic maths, no prediction.
3. **Alerts** — price levels the user sets, plus volatility-band crossings.
   Delivered by Telegram *and* real Web Push (the gap the assessment named).
4. **Transparency page** — the AUC table, the base rates, the losses. Public.
   This is marketing *and* compliance in one artefact.

### 4.3 Non-goals (explicit)

- No live order execution. Ever.
- No MQL5/EA ingestion.
- No direction prediction.
- No "AI robot" branding.
- No managing or holding user funds.
- No token/credit economy. Ever. (This is the exact mechanism of the scam we
  analysed; it will not appear in our product.)

---

## 5. Payments — the real constraint

### 5.1 What is blocked for Syria (verified this session)

| Provider | Status | Reason |
| --- | --- | --- |
| PayPal | ❌ | Still blocked |
| Payoneer | ❌ | Still blocked |
| Stripe | ❌ | Replied Sept 2026: "remains unsupported, no timeline" |
| Lemon Squeezy | ❌ | Syria on the **unsupported countries** list |
| Gumroad | ❌ | Depends on Stripe Connect payouts |
| Payhip | ❌ | Depends on Stripe/PayPal |
| Paddle | ⚠️ | Registration possible; Syria payout not guaranteed |

**Root cause:** all of them settle through **Stripe Connect**, which does not
pay out to ~30 countries including Syria. Registering is easy; **withdrawing
is the wall.**

### 5.2 Also blocked: our product *category* on those platforms

Independent of Syria, the major MoRs prohibit our category:

> **Lemon Squeezy, prohibited products:** "NFT & Crypto related products",
> "get-rich-quick schemes", "gambling", "business-in-a-box".

**Implication:** if we sold "trading signals", we would be rejected on
category *even from a supported country*. Selling a **risk/volatility tool**
is what makes us acceptable — another reason §4.1 is load-bearing.

### 5.3 The rail we will actually use

**Phase 1 — crypto (works today, from Syria).**

Use a processor that settles to crypto (e.g. a USDT-settling MoR/gateway).
Requirements:
- Settles **USDT on Tron (TRC-20)** — lowest fees, widest acceptance.
- We convert to fiat only when needed.
- Accept the trade-off: smaller addressable market (buyers need crypto) but
  **zero geographic dependency**.

**Phase 2 — foreign entity (the clean long-term rail).**

Register in **Turkey or Georgia** (fast, close, workable banking), then:
- Paddle or Stripe under the company.
- Fiat checkout for the majority of buyers who do not hold crypto.
- Clean tax and accounting.

**Phase 3 — hybrid.** Keep crypto for sanctioned-adjacent buyers, cards for
the rest. Never mix the ledgers.

**Rejected approach:** "use a relative's account in a supported country".
This violates every platform's terms and burns both accounts. Not considered.

---

## 6. Architecture

### 6.1 What changes versus the existing `asp`

| Existing `asp` | ASTRA Cloud |
| --- | --- |
| Single static key | **Multi-tenant accounts** (OAuth/email + sessions) |
| Everyone is Pro | **Plans + entitlements** enforced server-side |
| Telegram-only alerts | **Telegram + real Web Push** |
| One deploy, one user | **Per-tenant rate limits + quotas** |
| No billing | **Billing webhooks → entitlement** |
| No lint/types/coverage | **ruff + mypy + coverage floor in CI** |
| Reuses: engine, science, read-only design | **Keeps: read-only, shadow-mode, honesty rules** |

### 6.2 Component diagram

```
                       ┌──────────────────────────────┐
   phone PWA  ─────────▶│  Edge / reverse proxy (GET)  │
   Telegram bot ───────▶│  auth · rate limit · quotas  │
                       └───────────────┬──────────────┘
                                       │
             ┌─────────────────────────┼─────────────────────────┐
             ▼                         ▼                         ▼
   ┌──────────────────┐   ┌────────────────────────┐   ┌─────────────────┐
   │  Identity &      │   │  Volatility engine     │   │  Billing        │
   │  Entitlements    │   │  (reuse research code) │   │  webhooks →     │
   │  accounts, plans │   │  band + calibration    │   │  entitlements   │
   └────────┬─────────┘   └───────────┬────────────┘   └────────┬────────┘
            │                         │                          │
            └────────────┬────────────┴──────────────┬───────────┘
                         ▼                           ▼
              ┌────────────────────┐      ┌──────────────────────┐
              │  PostgreSQL        │      │  Market-data bridge  │
              │  tenants, subs,    │      │  (loopback, GET only)│
              │  usage, audit      │      │  XAUUSD + crypto     │
              └────────────────────┘      └──────────────────────┘
```

**Invariants carried over (non-negotiable):**
- Read-only by construction: no code path places an order.
- The bridge binds to loopback; the proxy forwards only GET/HEAD.
- Every output that could be read as a recommendation carries the disclaimer.
- The transparency page is generated from the same numbers the engine uses.

### 6.3 Quality gates — ✅ already built (verified)

The plan originally called this "the cheapest, highest-impact fix" and put it
first. That reasoning was correct, and the work is **already done**:

| Gate | File | Verified |
| --- | --- | --- |
| Lint | `ruff.toml` | `ruff check .` → **all checks passed** |
| Types | `mypy.ini` | `mypy bot tracker akoan advisor bridge mobile src` → **no issues, 65 files** |
| Coverage | `requirements-dev.txt` | **78 %**, floor `--fail-under=75` |
| CI wiring | `.github/workflows/ci.yml`, job `quality` | ✅ |

Landed in `98187f1` and `eed7511`. The rule set is deliberately narrow —
correctness and clarity only, with the reasoning written into `ruff.toml`.
`mypy` excludes tests on purpose (duck-typed fakes; annotations there would add
noise without catching real bugs). Both choices are defensible.

**Therefore this phase is closed and needs no work.** The honest next step is
not to rebuild it, but to **raise the floor as coverage grows** and keep the
gate green.

---

## 7. Business model

### 7.1 Pricing

| Plan | Price | Includes |
| --- | --- | --- |
| **Free** | $0 | 1 asset, 1 alert, delayed band, transparency page |
| **Pro** | $19/mo | All assets, unlimited alerts, live band, Web Push, dashboard |
| **Desk** | $49/mo | API access, multiple accounts, CSV export, priority support |

Modelled on the existing `docs/SELLING.md` ($19/mo) so the two tracks agree.

### 7.2 Unit economics (conservative)

| Metric | Value |
| --- | --- |
| Blended ARPU | ~$22/mo |
| Payment fees (crypto rail) | ~1–2% |
| Infra per tenant | < $0.50/mo (shared engine) |
| **Gross margin** | **> 90%** |
| Break-even (infra + tools ~$60/mo) | **~4 paying tenants** |

Software margins, tiny fixed cost. The constraint is **distribution**, not
unit economics.

### 7.3 The honest growth constraint

We must be blunt: this is not a viral product. It is a **credibility product**.
Growth comes from:
1. The transparency page being genuinely convincing.
2. Posting the real numbers where traders argue (forums, not ads).
3. The free tier being useful enough to spread by word of mouth.

Expect **slow, compounding** growth. Plan for 12–18 months to meaningful MRR.

---

## 8. Roadmap (corrected against measured reality)

The original six-phase roadmap assumed three phases were unbuilt. They are not.
**Phases 0, 2 and 3 are done.** What remains is reordered so the actual blocker
comes first.

### ✅ Already complete — no work needed

| Was phase | Item | Evidence |
| --- | --- | --- |
| 0 | Quality gates | `ruff` + `mypy` + coverage in CI, all green (`eed7511`) |
| 2 | Real mobile | offline shell (`sw.js`) + Web Push (`webpush.py`, `e65512e`) |
| 3 | Volatility surface | `volatility.py` + `vol_model.json` + parity test; Arabic copy states the boundary |
| — | Plans/tiers | `entitlements.py`, dormant until `ASTRA_UPGRADE_URL` is set |
| — | Pricing page | `mobile/pricing.html` |
| — | Gate rate limiting | `ASTRA_GATE_LOCKOUT_S=900` (`6af91db`) |

### Phase A — The payment rail (now first; was phase 4)

This is the blocker. `docs/SELLING.md` assumes **Stripe Payment Links**, and
Stripe does not serve Syria — so the runbook cannot be executed as written.

- Rewrite `SELLING.md` around a rail that actually works from Syria.
- Decide the rail (see `OPEN_DECISIONS.md` D4): crypto (USDT TRC-20) direct, or
  a merchant-of-record that accepts the operator.
- Build entitlement activation that does not depend on a Stripe webhook — the
  current design already uses a manual `/grant <chat_id> pro`, which survives
  the change.
- Test the full path: pay → activate → `/plan` reads `pro`.
- **Nothing else ships before this works**, because without it there is no
  revenue and no launch.

### Phase B — Accounts and multi-tenancy (was phase 1)

- Real accounts, sessions, server-side plan enforcement.
- Retire the single static key; keep an operator bypass.
- Per-tenant quotas and rate limits.
- Note: `entitlements.py` keys tiers to Telegram chat ids. That is a working
  bridge, not the end state — it must become login-keyed.

### Phase C — Legal surface (was spread across phases)

- `PRIVACY.md` and `TERMS.md` — verified absent today. Required by stores, by
  payment providers, and by GDPR if EU buyers are accepted.
- A refund/cancellation policy (the payment provider will require a public one).
- Keep the honesty disclaimers on every output.

### Phase D — Hardening (small, specific, verified open)

| Item | Evidence | Fix |
| --- | --- | --- |
| Android cleartext | `usesCleartextTraffic="true"` in `AndroidManifest.xml` | remove, or scope to debug |
| Android backup | `allowBackup="true"` | set false |
| Two bots, one token | both `bot/telegram_bot.py:411` and `astra_bot.py:974` read `TELEGRAM_BOT_TOKEN` | document which owns it (409 otherwise) |
| Oversized files | `terminal.js` 1281, `astra_bot.py` 1066 | split, as `app.js` already was |
| Heavy clone | `.git` 107 MB, `research/data` 51 MB | Git-LFS or on-demand fetch |

### Phase E — Entity + fiat (was phase 5)

- Turkey/Georgia entity; Paddle or Stripe onboarding.
- Migrate willing buyers; keep the crypto rail for the rest.

### Ordering rationale

Phase A first because it is the only thing standing between a finished product
and revenue. Phase B second because the current gate does not scale past one
operator. Phase C before any public launch because a store will reject an app
with no privacy policy. Phase D is cheap and can be done in parallel.

**One correction to the original schedule:** it put quality gates before
everything, "no features until green". That was right, and it is why this
roadmap now starts four phases later than the original — the foundation was
actually poured.

---

## 9. Risk register

| # | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| 1 | Volatility edge decays | Medium | High | Re-measure yearly; publish it; the gate already rejects thin evidence |
| 2 | Category blocked by a payment rail | Medium | High | Sell *risk tool*, not signals; keep crypto rail as fallback |
| 3 | Syria payout disruption | Medium | High | Two rails (crypto + entity); never single-source |
| 4 | Reputational spillover from the scam space | Medium | Medium | Distance explicitly: publish "no directional edge" |
| 5 | Solo-founder bandwidth | High | Medium | Ship in phases; quality gates first so refactors are safe |
| 6 | Legal reclassification (advice) | Low | High | §4.1 boundary; lawyer review before launch |
| 7 | Infra cost creep | Low | Low | Shared engine; per-tenant quotas |
| 8 | Token permission blocks repo creation | **Certain now** | Low | User creates repo or grants scope (§10) |

---

## 10. Where this plan lives

The plan ships in its own repository, **`msy898931-boop/msy898931-boop-astra-cloud`**
(GitHub prefixed the owner name), with the same documents also available on the
branch `plan/astra-cloud-commercial` of `asp`.

| | |
| --- | --- |
| Primary | this repository |
| Mirror | `asp` @ `plan/astra-cloud-commercial` → `docs/astra-cloud/` |
| Effect on `asp` `main` | **none** — additive files only, nothing changed or deleted |

**Tooling note (for the record).** The agent token is a **GitHub App
user-to-server token** (`ghu_…`, app `openhands-ai`) whose installation is
scoped to **selected repositories**. It therefore cannot create repositories,
and until the new repository was added to the installation's selection, pushes
to it were refused (`403`). The token also lacks repository-administration
permission, so it cannot change the repository's visibility.

**Action needed once:** the repository was created **public** and holds
commercial strategy, pricing and a legal plan. Set it to private:
Settings → Change visibility → Make private.

---

## 11. What I have deliberately not done

- **No product code.** The instruction was plan first, approval after.
- **No modification of any existing file in `asp`.** The `asp` mirror branch
  adds one new folder (`docs/astra-cloud/`) and changes nothing else. `main` is
  untouched, and no file was deleted or renamed.
- **No fork.** The existing repo is already a fork of `alimazna/asp` with a
  "never push upstream" constraint. A third layer of forking would be a mess.

---

## 12. The one-sentence version

> Sell the volatility filter — the only thing this project proved — as a
> multi-tenant SaaS from Syria, paid in crypto now and via a foreign entity
> later, with the honesty rules as both the legal boundary and the marketing.

---

_Plan prepared by an AI agent (OpenHands) on behalf of @msy898931-boop._
_All measured figures are sourced in `docs/RESEARCH_SOURCES.md`._
