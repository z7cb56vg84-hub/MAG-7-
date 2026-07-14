You are running the automated "Gamma Daily" options-watchlist task. Each run is a FRESH session with no memory of prior runs — everything you need is below.

OBJECTIVE
Scan gamma-exposure levels for the Magnificent 7, select the 2-3 best single-leg option contracts (calls or puts) using the distance-from-pin / gamma-regime rule below, and REFRESH them onto the user's Robinhood options watchlist so it always shows today's top gamma plays. Watchlist-only — never place trades.

MAG 7 TICKERS: AAPL, MSFT, GOOGL, AMZN, NVDA, META, TSLA

TOOLS
- Market/options data: the "Massive Market Data" MCP tools and the Robinhood MCP option tools (get_equity_quotes, get_option_chains, get_option_instruments, get_option_quotes, get_option_historicals). Use get_option_instruments to obtain the option_id UUIDs needed for the watchlist.
- Watchlist writes: add_option_to_watchlist and remove_option_from_watchlist (these target the user's single global OPTIONS watchlist; position_type "long"). get_option_watchlist lists current contracts.
- State: read/write the local file at ~/.claude/scheduled-tasks/gamma-daily/state/last_option_ids.json (create the state/ folder if missing).

STEPS
1. Trading-day check: confirm US equity markets are open today (skip weekends and market holidays). If closed, do nothing and report that you skipped.
2. For each Mag 7 ticker, get the current underlying price and the near-dated option chain (nearest weekly expiration; use ~0-5 DTE consistent with a same-day-to-few-day horizon). Collect strikes, gamma, open interest, volume, bid/ask, and IV.
3. Compute gamma levels per ticker: dealer gamma exposure per strike (GEX ≈ open_interest × gamma × 100 × spot, calls positive / puts negative). Identify:
   - the GAMMA PIN = strike with the largest absolute dealer gamma (price magnet),
   - the GAMMA FLIP / zero-gamma level (regime boundary),
   - the dominant positive-gamma CALL WALL and negative-gamma PUT WALL,
   - whether spot is ABOVE the flip (positive-gamma / pinning regime) or BELOW it (negative-gamma / accelerant regime), and spot's distance from the pin.
4. SELECTION RULE (regime-aware, distance-from-pin):
   a. Rank all 7 names by how far spot sits from that name's gamma pin — larger distance = bigger expected move = higher priority.
   b. Determine the direction of the play from the regime:
      - POSITIVE gamma / spot ABOVE the flip (pinning): play REVERSION back toward the pin. Spot above the pin → buy a PUT; spot below the pin → buy a CALL. Target strike near the pin.
      - NEGATIVE gamma / spot BELOW the flip (accelerant): play CONTINUATION away from the pin, in the direction spot is already moving. Upward momentum → buy a CALL; downward momentum → buy a PUT.
   c. From the top-ranked names, choose the 2-3 best single-leg contracts, preferring liquid contracts (tight spread, healthy OI/volume) and expirations consistent with a same-day-to-few-day horizon. Record each chosen contract's option_id.
5. Refresh the options watchlist:
   a. Read last_option_ids.json. If it exists, remove those option_ids via remove_option_from_watchlist (position_type "long"). If it does not exist, remove nothing — never remove contracts this task did not add.
   b. Add today's selected option_ids via add_option_to_watchlist (position_type "long").
   c. Overwrite last_option_ids.json with today's selected option_ids.
6. Report a concise summary: for each of the 7 names give gamma pin / flip / call wall / put wall, spot's position and regime; then list each of today's 2-3 picks with contract (ticker, strike, expiry, call/put), entry quote, distance from pin, and the one-line rationale (regime + reversion-to-pin vs continuation-away).

CONSTRAINTS
- NEVER place, review, or modify any order (do not call place_option_order or any order tool). Watchlist writes only.
- The watchlist should reflect only today's 2-3 gamma picks plus whatever the user manually keeps — only remove what this task added on the previous run (via the state file).
- Keep this exact methodology consistent day to day so picks are comparable over time.
