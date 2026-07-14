# MAG7_GEX — Daily Mag 7 Gamma-Exposure Watchlist Bot

Scans dealer gamma exposure (GEX) across the Magnificent 7 every trading morning,
picks the 2–3 best single-leg option plays that target a move **toward the gamma pin**,
and refreshes them onto the Robinhood options watchlist. **Watchlist-only — it never
places trades.**

Mag 7: **AAPL, MSFT, GOOGL, AMZN, NVDA, META, TSLA**

Runs as a **cloud Routine** in the Claude app, with the **Robinhood** and **FMP**
connectors attached.

---

## Schedule

| | |
|---|---|
| **When** | Weekdays (Mon–Fri) |
| **Time** | 10:00 AM ET — ~30 min after the open |
| **Cron** | `0 10 * * 1-5` (timezone: Eastern) |
| **Frequency** | Once per trading day |

10:00 ET is chosen deliberately: the opening rotation (9:30–9:45) has cleared, spreads
have tightened, and spot has revealed where it's sitting relative to the gamma levels —
but it's still early enough to be in front of the day's move.

See `schedule.txt` for timezone conversions.

---

## The strategy — always toward the pin

1. **Pin** = strike with the largest absolute dealer gamma (the price magnet).
2. **Rank** all 7 names by how far spot sits from its pin (%). Bigger distance =
   bigger expected move = higher priority.
3. **Direction is always toward the pin**, for every name, regardless of gamma regime:
   - Spot **below** the pin → buy a **CALL** (pin is the upside target)
   - Spot **above** the pin → buy a **PUT** (pin is the downside target)
4. **Strike** = the pin strike (or nearest liquid strike to it), ~0–5 DTE, liquid only.
5. **Refresh:** clear the prior day's near-dated Mag 7 picks, add today's 2–3. Non-Mag7
   and longer-dated contracts (e.g. SPXW) are left untouched.

`GEX ≈ open_interest × gamma × 100 × spot`  (calls positive, puts negative)

### On the gamma regime
The **flip** (zero-gamma level) is still computed and **reported** — positive-gamma-pinning
above it, negative-gamma-accelerant below — but it is **recorded as data only**. It does
**not** change trade direction. This keeps the methodology constant day to day, so the
regime column can later be used to test whether pin-reversion pays better in one regime
than the other.

> Earlier versions used a regime-aware rule that played *continuation away* from the pin
> in negative gamma. That was removed — direction is now unconditional.

### Known limitations
- The call-positive / put-negative sign convention assumes dealers are long calls and
  short puts. It's the industry-standard proxy, not a measurement of real inventory.
- Single-name OI is far thinner than index OI, so the regime read is noisier than SPX.
- Open interest updates once each morning, so intraday it is slightly stale.

---

## Files

| File | What it is |
|---|---|
| `prompt.md` | The routine Instructions — paste into the Routine's Instructions box. |
| `SKILL.md` | Same prompt with frontmatter (name, schedule, connectors). |
| `schedule.txt` | Cron string + timezone conversions. |
| `README.md` | This file. |

---

## Setup

1. Claude app → **Routines** → **New routine**.
2. **Name:** `MAG7 GEX — daily gamma watchlist`
3. **Instructions:** paste the contents of `prompt.md`.
4. **Trigger → Schedule:** `0 10 * * 1-5`, timezone **Eastern**.
5. **Connectors:** attach **Robinhood** and **FMP**.
6. Save, then **Run now** once to verify the picks look right.

> GitHub stores this for version control and portability — it does **not** execute the
> routine. Execution happens in the Claude app, which holds the brokerage connectors.
