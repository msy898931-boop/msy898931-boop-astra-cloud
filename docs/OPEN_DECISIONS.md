# Open decisions

Two decisions block everything. The rest are smaller and can wait.

**Corrected 2026-10-18.** Three decisions from the first version are now moot,
because the work they gated is already built. They are recorded at the bottom
under "Closed".

---

## D-A — Payment rail (blocks launch) 🔴

This is the real blocker. `docs/SELLING.md` is written end to end around
**Stripe Payment Links**, and Stripe does not serve Syria. The runbook cannot be
executed as written.

**Verified blocked for Syria:** Stripe, PayPal, Payoneer, Lemon Squeezy,
Gumroad, Payhip. **Uncertain:** Paddle. **Works today:** crypto.

Pick one:

- [ ] **A. Crypto only, direct.** USDT TRC-20 to a wallet; activate manually via
      the existing `/grant <chat_id> pro`. Works today, zero setup, smallest
      market. The code already supports this path.
- [ ] **B. Crypto + a merchant-of-record** that accepts a Syrian operator
      (needs verification — none is confirmed).
- [ ] **C. Foreign entity first**, then Paddle/Stripe. Best long-term, slowest,
      and needs a partner or director in that country.
- [ ] **D. Something else** you have in mind.

**What I need from you:** which rail, and whether you already hold a crypto
wallet you intend to use for business receipts.

**What I will do once you pick:** rewrite `SELLING.md` around it, and make
activation work without any Stripe webhook (the manual `/grant` path already
survives, so this is smaller than it sounds).

---

## D-B — Accounts: build now, or sell on the chat-id gate first? 🟡

Today tiers are keyed to **Telegram chat ids** (`entitlements.py`), and the web
app uses **one static key**. That works for a handful of buyers and does not
scale past one operator.

- [ ] **A. Sell first, build accounts later.** Ship the rail, take the first
      buyers on the chat-id gate, build accounts when volume justifies it.
      *(Fastest to revenue. Honest about the limit.)*
- [ ] **B. Build accounts before selling.** Slower, cleaner, no migration later.
- [ ] **C. Both in parallel.**

**My recommendation: A.** The gate is rate-limited, the product is read-only by
construction, and the first ten buyers do not need a login system. Revenue
first, then the plumbing.

---

## Closed — no decision needed

### D1 — Where the plan lives ✅
Its own repository, `msy898931-boop/msy898931-boop-astra-cloud`, with a mirror
on the `plan/astra-cloud-commercial` branch of `asp`. Nothing in `asp` changed.

### D2 — Product boundary ✅ **decided**
Confirmed: sell the **volatility filter**, never a direction signal. This is
already what the code does — `volatility.py` ships the Arabic line
"لا يتنبّأ بالاتجاه — فقط بحجم الحركة".

### D7 — First thing to build ✅ **already built**
The choice was quality gates. They were built before the decision was made —
`ruff`, `mypy` and coverage run in CI and pass. See `STATE_OF_PLAY.md` §1.

---

## Still open, but not blocking

### D4 — Entity jurisdiction
Turkey · Georgia · UAE · not yet decided. Needs a partner or director.
Only matters once fiat is wanted (Phase E).

### D5 — Language
Arabic-first · English-first · bilingual. The code is already bilingual
(`i18n.js`, and Arabic copy in the volatility alert).

### D6 — Reuse strategy
The engine and research code live in `asp`; new product code should import them
directly rather than duplicate them. Low stakes — decide at build time.

### D8 — Name and domain
`ASTRA` is used throughout. Whether to keep it publicly is your call.

### D9 — First paying-customer channel
Where the first buyers come from. Not a code decision; worth deciding before
Phase A completes.
