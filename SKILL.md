---
name: mag7-gex
description: Daily Mag 7 gamma scan; refresh 2-3 toward-the-pin call/put picks on the options watchlist (weekdays 10am ET)
schedule: "0 10 * * 1-5"
connectors: [Robinhood, FMP]
---

You are running the automated "MAG7 GEX" gamma watchlist routine. Each run is a fresh session with no memory of prior runs — everything you need is below. Available connectors: Robinhood (options data + watchlist) and FMP (market data).

OBJECTIVE
Scan gamma-exposure levels for the Magnificent 7, select the 2-3 best single-leg option contracts that play a move TOWARD the gamma pin, and refresh them onto the Robinhood options watchlist. Watchlist-only — NEVER place trades.

MAG 7 TICKERS: AAPL, MSFT, GOOGL, AMZN, NVDA, META, TSLA

STEPS
1. Trading-day check: confirm US equity markets are open today (skip weekends and market holidays). If closed, do nothing and report that you skipped.
2. For each Mag 7 ticker, get the current underlying price (FMP or Robinhood quotes) and the near-dated option chain (nearest weekly expiration; ~0-5 DTE). Collect strikes, gamma, open interest, volume, bid/ask, and IV via the Robinhood option tools (get_option_chains, get_option_instruments, get_option_quotes). Use get_option_instruments to obtain the option_id UUIDs needed for the watchlist. If per-contract gamma is not returned by the connector, compute Black-Scholes gamma from spot, strike, IV, and time-to-expiry.
3. Compute gamma levels per ticker: dealer gamma exposure per strike (GEX ≈ open_interest × gamma × 100 × spot, calls positive / puts negative). Identify:
   - the GAMMA PIN = strike with the largest absolute dealer gamma (the momentum TARGET, not the strike to buy),
   - the GAMMA FLIP / zero-gamma level (report only — see rule below),
   - the dominant positive-gamma CALL WALL and negative-gamma PUT WALL,
   - the gamma regime (positive-gamma-pinning if spot is above the flip, negative-gamma-accelerant if below) — RECORD THIS FOR THE REPORT ONLY.
4. SELECTION RULE — ALWAYS PLAY TOWARD THE PIN:
   a. Rank all 7 names by how far spot sits from that name's gamma pin, as a percentage. Largest distance = biggest expected move = highest priority.
   b. Direction is ALWAYS toward the pin, for EVERY name, REGARDLESS of the gamma regime. Do not use the regime to flip direction. Do not ever select a continuation/away-from-pin play.
      - Spot BELOW the pin → buy a CALL (the pin is the upside target).
      - Spot ABOVE the pin → buy a PUT (the pin is the downside target).
   c. STRIKE = 1 STRIKE OTM FROM SPOT, in the direction of the pin. DO NOT buy the pin strike.
      - CALL: buy the first standard strike ABOVE spot (one increment out-of-the-money).
        Example: spot 95, pin 100, strike increment 2.50 -> buy the 97.50 call, NOT the 100 call.
      - PUT: buy the first standard strike BELOW spot (one increment out-of-the-money).
        Example: spot 105, pin 100, strike increment 2.50 -> buy the 102.50 put.
      - Use the ACTUAL strike increment of that ticker's chain (it varies by name and price level).
      - Rationale: the pin is the momentum magnet, not the target strike. The 1-OTM strike sits BETWEEN spot and the pin, so the move toward the pin carries it in the money. Buying the pin strike only pays if price fully reaches the pin.
   d. GUARD: only select a name if its pin is at least 2 strike increments away from spot, so the 1-OTM strike lies strictly between spot and the pin. If the pin is closer than that, skip that name and move to the next-ranked one.
   e. From the top-ranked qualifying names, pick the 2-3 best contracts, preferring liquid contracts (tight spread, healthy OI/volume).
5. Refresh the options watchlist:
   a. Call get_option_watchlist. Remove any single-leg contract on a Mag 7 ticker with a near-dated (~0-7 DTE) expiration via remove_option_from_watchlist (position_type "long") — these are the routine's prior picks. Leave any non-Mag7 or longer-dated contracts (e.g. SPXW) untouched.
   b. Add today's selected option_ids via add_option_to_watchlist (position_type "long").
6. Report: for each of the 7 names give spot / pin / flip / call wall / put wall / regime / % distance from pin. Then list each of today's 2-3 picks with contract (ticker, strike, expiry, call/put), entry quote (mid), OI, volume, % distance from pin, and the regime it occurred in. Every pick must be a toward-the-pin play — state the pin target for each.

CONSTRAINTS
- NEVER place, review, or modify any order. Watchlist writes only.
- Direction is ALWAYS toward the pin. The gamma regime is recorded as data but must NEVER change the direction of the trade.
- The strike is ALWAYS 1 increment OTM from spot — NEVER the pin strike itself.
- Only touch near-dated Mag 7 single-leg contracts when refreshing; never remove the user's other watchlist items.
- Keep this exact methodology consistent day to day so picks are comparable over time.
