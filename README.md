# MAG7_GEX — Daily Mag 7 Gamma-Exposure Watchlist Bot

Scans dealer gamma exposure (GEX) across the Magnificent 7 every trading morning,
picks the 2–3 best single-leg option plays using a regime-aware, distance-from-pin
rule, and refreshes them onto the Robinhood options watchlist. **Watchlist-only — it
never places trades.**

Mag 7: **AAPL, MSFT, GOOGL, AMZN, NVDA, META, TSLA**

---

## Schedule

| | |
|---|---|
| **When** | Weekdays (Mon–Fri) |
| **Time** | 8:00 AM local  (= 10:00 AM ET on Mountain Time — ~30 min after the open) |
| **Cron** | `0 8 * * 1-5` |
| **Frequency** | Once per trading day |

> Cron runs in the machine's **local** timezone. `0 8 * * 1-5` = 10am ET only if the
> machine is on Mountain Time. On a different zone, change the hour so it lands at
> 10am ET (e.g. Pacific → `0 7 * * 1-5`, Central → `0 9 * * 1-5`, Eastern → `0 10 * * 1-5`).

---

## The strategy (regime-aware, distance-from-pin)

1. **Pin** = strike with the largest absolute dealer gamma (price magnet).
2. **Flip** = zero-gamma level; it splits the two regimes.
3. **Rank** all 7 names by how far spot sits from its pin — bigger distance = bigger
   expected move = higher priority.
4. **Direction depends on the regime:**
   - **Positive gamma / spot ABOVE the flip (pinning):** play **reversion toward the pin**.
     Spot above pin → buy a **PUT**; spot below pin → buy a **CALL**.
   - **Negative gamma / spot BELOW the flip (accelerant):** play **continuation away from
     the pin**, with the momentum. Up → **CALL**; down → **PUT**.
5. Pick the 2–3 best liquid contracts (tight spread, healthy OI/volume), ~0–5 DTE.
6. **Refresh:** remove yesterday's picks (tracked in `state/last_option_ids.json`, so it
   never touches contracts you added yourself) and add today's.

`GEX ≈ open_interest × gamma × 100 × spot`  (calls positive, puts negative)

---

## Files

| File | What it is |
|---|---|
| `SKILL.md` | Drop-in scheduled-task file (frontmatter + full prompt). |
| `prompt.md` | Just the task prompt, for pasting into a new scheduled task. |
| `schedule.txt` | Cron string + timezone conversions. |
| `README.md` | This file. |

---

## How to make it run automatically

The bot runs inside the **Claude app's scheduled-tasks system** — it needs the app open
and the **Robinhood** + **Massive Market Data** connectors authorized. GitHub is for
version control / backup / portability; GitHub itself does **not** execute the task
(it has no access to your brokerage connectors).

**To install on any machine running the Claude app:**

1. Copy `SKILL.md` into a folder named `gamma-daily` under your scheduled-tasks dir:
   ```
   ~/.claude/scheduled-tasks/gamma-daily/SKILL.md
   ```
2. Or just tell Claude: *"Create a scheduled task from the prompt in prompt.md, cron `0 8 * * 1-5`."*
3. Open the **Scheduled** section in the sidebar, click **Run now** once to pre-approve
   the market-data and watchlist tools (otherwise the first real run pauses on prompts).
4. Adjust the cron hour for your timezone if you're not on Mountain Time (see above).

> Scheduled tasks fire only while the Claude app is open. If it's closed when a run is
> due, it runs on next launch.
