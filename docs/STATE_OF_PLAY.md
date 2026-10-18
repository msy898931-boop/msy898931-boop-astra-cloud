# State of play — verified 2026-10-18

Every line below was checked against the live repository `msy898931-boop/asp`
at commit `b3a1845`, by running the tools rather than reading the docs. Where a
claim could not be verified, it says so.

This document exists because the first version of this plan was written from an
assessment dated **2026-10-16** and from documentation. The repository moved on.
Three of the things that plan proposed as future work are **already built**.

---

## 1. The headline finding

> **The quality gate is already done.** `ruff`, `mypy` and coverage run in CI
> today, with a coverage floor, and all three pass.

The instruction was to build the quality gate first, because it is cheap, high
impact, and makes later refactoring safe. That reasoning was right. The work
was already finished before the instruction was given.

| Claim | Verified how | Result |
| --- | --- | --- |
| `ruff` configured | `ruff.toml` present, 1387 B | ✅ |
| `ruff` passes | `ruff check .` | ✅ **All checks passed** |
| `mypy` configured | `mypy.ini` present | ✅ |
| `mypy` passes | `mypy bot tracker akoan advisor bridge mobile src` | ✅ **no issues in 65 source files** |
| Coverage measured | `coverage run` over 8 suites | ✅ |
| Coverage floor enforced | `coverage report --fail-under=75` | ✅ **78 % measured** |
| Wired into CI | `quality` job in `.github/workflows/ci.yml` | ✅ |
| When it landed | `git log -- ruff.toml` → `98187f1`, `eed7511` | commit *"Quality gate: lint, types and coverage in CI"* |

The rule set is deliberately narrow and the reasons are written in the file
(correctness over style churn; `UP` and `ALL` left off because they would
rewrite hundreds of working lines). That is a defensible engineering choice, not
an oversight.

---

## 2. The volatility product surface already exists

The plan's product thesis — "sell the volatility filter, never a direction
signal" — is **already implemented**, not just proposed.

`bridge/telegram_bot/volatility.py` (verified, read in full) contains:

- A calibrated model loader: standardizer + logistic weights + Platt calibrator.
- Feature recomputation that matches the C++ engine exactly
  (`TRIGGER_WINDOW = 9`, `CONTEXT_BARS = 200`).
- `predict()` returning a calibrated probability, or `None` when there is not
  enough data — it stays silent rather than inventing a number.
- The Arabic alert, which states the product boundary in the user's own words:

  > ⚠️ تقلّب مرتفع متوقّع (N%)
  > …(لا يتنبّأ بالاتجاه — فقط بحجم الحركة.)

| Component | File | State |
| --- | --- | --- |
| Volatility model | `bridge/telegram_bot/volatility.py` | ✅ built |
| Trained artifact | `bridge/telegram_bot/vol_model.json` (1727 B) | ✅ present |
| Engine parity test | `bridge/telegram_bot/test_volatility_parity.py` | ✅ 12 tests, 2 skipped |
| Edge-triggered alert loop | `astra_bot.py` `_check_volatility` | ✅ wired |
| Training script | `scripts/volatility_lab.py` | ✅ present |

---

## 3. The commercial surface is further along than the plan assumed

| Component | File | State |
| --- | --- | --- |
| Free/Pro tiers | `bridge/telegram_bot/entitlements.py` | ✅ built, tested |
| Dormant-until-selling design | `Entitlements.selling` — nobody is gated until `ASTRA_UPGRADE_URL` is set | ✅ |
| Pricing page | `mobile/pricing.html` + `pricing.css` | ✅ built |
| Web Push | `mobile/webpush.py` (17 functions), routes `/push/*` | ✅ built + real crypto round-trip test |
| Offline shell | `mobile/sw.js` | ✅ cache + banner |
| Gate rate limiting | `serve_mobile.py`, `ASTRA_GATE_LOCKOUT_S=900` | ✅ built |
| Secure cookie behind TLS | `d4c9c3f` | ✅ |
| Selling runbook | `docs/SELLING.md` | ✅ written |

The `entitlements.py` design note is worth quoting because it shows the
commercial layer was built carefully:

> when no upgrade URL is configured (`ASTRA_UPGRADE_URL` unset) everyone is
> treated as pro. That keeps the layer dormant until the product is actually on
> sale, so development, CI, and the "not selling yet" state behave exactly as
> before.

---

## 4. Measured state, for the record

Run on 2026-10-18, in this order:

| Check | Command | Result |
| --- | --- | --- |
| Lint | `ruff check .` | All checks passed |
| Types | `mypy bot tracker akoan advisor bridge mobile src` | no issues, 65 files |
| Coverage | `coverage report --fail-under=75` | **78 %** |
| Tests | 8 suites | **972 passing**, 4 skipped |

Suite breakdown (measured, not copied from the README):

| Suite | Tests |
| --- | --- |
| `bot/` | 152 |
| `tracker/` | 44 |
| `research/` | 50 |
| `akoan/` | 59 |
| `advisor/` | 92 |
| `mobile/` | 162 |
| `tests/` | 308 |
| `bridge/telegram_bot/` | 105 |
| **Total** | **972** |

Plus 19 C++ test binaries under `ctest`.

The README table says **961** (`mobile/` 151, `bridge/` 94). The real figures
are 162 and 105. The README is stale by 11 tests — a small thing, but it is
exactly the class of drift this document exists to catch.

---

## 5. What is genuinely still missing

These are verified gaps, not inherited claims.

### 5.1 The payment rail is the real blocker — and the docs assume Stripe

`docs/SELLING.md` is built end to end around **Stripe Payment Links** (6
mentions). Stripe does not serve Syria. So the selling runbook, as written,
cannot be executed by the operator.

This is the single highest-value finding in this audit: **the product is ready
to sell and the payment rail is not.** Everything else on this list is smaller
than that.

### 5.2 No accounts, no multi-tenancy

Still a single static key (`ASTRA_ACCESS_TOKEN`). `entitlements.py` stores tiers
per Telegram chat id in a JSON file — which works, but is keyed to the bot
channel, not to a login. The assessment called this "fine for one person; not
saleable", and that is still accurate.

### 5.3 No privacy policy or terms

Verified: no file matching `*privacy*`, `*terms*`, `*tos*` anywhere in the tree.
Required by stores, by payment providers, and by GDPR if EU buyers are accepted.

### 5.4 Android is still loose

Verified in `android/app/src/main/AndroidManifest.xml`:

```
android:allowBackup="true"
android:usesCleartextTraffic="true"
```

Cleartext permits plain HTTP; backup can export app data. Acceptable for a
personal build, a hole for wide distribution.

### 5.5 Two bots, one token

Verified: `bot/telegram_bot.py:411` and `bridge/telegram_bot/astra_bot.py:974`
both read `TELEGRAM_BOT_TOKEN`. Run both and Telegram returns `409`.

### 5.6 Oversized files

Measured with `wc -l`:

| File | Lines |
| --- | --- |
| `mobile/terminal.js` | 1281 |
| `bridge/telegram_bot/astra_bot.py` | 1066 |
| `bridge/telegram_bot/test_astra_bot.py` | 751 |

Note: the assessment flagged `app.js` at 1675 lines. It has since been split
into nine ordered scripts (`aa0b6fc`) — **that gap is closed.**

### 5.7 Repository weight

`.git` is **107 MB**; `research/data` is **51 MB**. Clones are heavy. Low
severity, easy fix (Git-LFS or on-demand fetch).

---

## 6. What the assessment got right, and what has moved

| Assessment gap (2026-10-16) | Status now |
| --- | --- |
| No linter / type checker / coverage | ✅ **closed** (`eed7511`) |
| `app.js` oversized (1675 lines) | ✅ **closed** (`aa0b6fc` split into 9 scripts) |
| Offline cache + banner | ✅ **closed** |
| Web Push | ✅ **closed** (`e65512e`) |
| Gate not rate-limited | ✅ **closed** (`6af91db`) |
| README does not mention the gate | ✅ **closed** (`b3a1845`) |
| Static key, no multi-user | ❌ **open** |
| Android cleartext + allowBackup | ❌ **open** |
| Two bots, one token | ❌ **open** |
| No privacy policy / terms | ❌ **open** |
| Oversized `astra_bot.py` / `terminal.js` | ❌ **open** |
| Heavy corpus in git | ❌ **open** |
| **Payment rail unusable from Syria** | ⚠️ **new — highest impact** |

Six of twelve closed in two days. The remaining work is smaller and more
specific than the plan's original roadmap assumed.

---

## 7. Consequence for the plan

The original roadmap had six phases. Measured against reality:

| Phase | Original intent | Actual status |
| --- | --- | --- |
| 0 | Quality gates | ✅ **already done** |
| 1 | Multi-tenancy + plans | ⚠️ plans done; accounts not |
| 2 | Real mobile (offline + push) | ✅ **already done** |
| 3 | Volatility product surface | ✅ **already done** |
| 4 | Billing + launch | ❌ **blocked: no usable rail** |
| 5 | Entity + fiat | ❌ not started |

**Two of five phases are complete and a third is half complete.** The revised
plan in `PLAN.md` reflects this: it drops the finished work and concentrates on
the payment rail, accounts, legal pages, and the two hardening items.
