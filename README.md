
# momentum-adx-vwap-btc


## Why I Built It

BTC perpetual futures trend strongly but punish traders who chase entries. Most retail approaches either enter too early (before trend confirmation) or too late (after the move has extended). The gap I wanted to fill: a systematic way to enter *with* the trend, but only at high-probability pullback zones where the market is likely to resume direction.

VWAP represents the fair value anchor institutions use intraday. When a strong trend is confirmed and price retests VWAP, that's not weakness — it's an entry opportunity. The ADX-VWAP Trend Rider is built around that thesis.

---

## Core Logic

**Signals used (technical only):**

- **EMA 50 / EMA 200 crossover** — defines trend direction. Long bias only when EMA50 > EMA200, short bias only when EMA50 < EMA200
- **ADX (14, threshold 20)** — filters out ranging conditions. No trade fires unless the trend has real directional strength
- **VWAP proximity (0.5%)** — entry only triggers when price has pulled back within 0.5% of VWAP, the volume-weighted fair value level
- **Volume spike confirmation (2× 20-bar average)** — filters out low-conviction setups; requires institutional-level participation at the entry bar
- **ATR (14)** — sizes stops dynamically based on current volatility, not fixed pip values

**Decision flow:**
Trend direction → Trend strength → Price at VWAP → Volume confirms → Enter. All five gates must open simultaneously.

**Risk management:**
- 2% risk per trade on a $10,000 margin budget
- Max 50% capital utilization at any time
- 5× leverage
- Stop loss at 1.5× ATR from entry
- First TP at 2× risk (50% of position closed)
- Trailing stop activates at 1× R, trails at 1.5× ATR
- Time-based exit after 30 bars if minimum ATR profit threshold isn't met

---

## Development Challenges

**Challenge 1 — Instant exit bug**
One paper trade entered and exited at the same timestamp with 0 minutes hold time. Root cause: ATR or stop distance calculating to near-zero on the entry bar, causing the stop to fire immediately. Fix in progress: adding a minimum ATR floor before position sizing executes.

**Challenge 2 — Short bias in an uptrend**
Both live paper trades fired as shorts during a period when BTC was trending from ~60K toward 84K. The EMA condition was likely not correctly gating short entries. Investigating a logic inversion in the short entry block.

**Challenge 3 — Limited backtest history**
GetAgent Studio's data depth constrained the backtest window, making it harder to validate across multiple market regimes. Worked around this by focusing parameter optimization on the available May–June 2026 window and keeping parameters conservative.

---

## What's Complete / What's Next

**Completed:**
- Full strategy logic in GetAgent Studio Playbook
- Paper trading live since Jun 23
- Risk management framework (ATR stops, position sizing, time exits, trailing stop)
- Published strategy with public brief

**In progress:**
- Fixing the 0-minute exit bug
- Correcting short entry direction filter
- Raising ADX threshold to 25 to reduce low-quality entries

**Next steps:**
- Extend to ETH and SOL perpetuals
- Add session filter (avoid low-liquidity hours)
- Integrate into CycleMind as one of the agent's strategy modules

---

## Frameworks, Models & APIs

- **Bitget tool used: GetAgent Studio / Playbook** (strategy creation, backtesting, paper trading)
- Strategy logic: Python (Studio-generated)
- Indicators: EMA, ADX, VWAP, ATR, RSI, Volume — all computed natively in Studio
- Asset: BTCUSDT perpetual futures

---

## Experience with Bitget AI Tools

GetAgent Studio lowers the barrier to systematic trading significantly. The Playbook format forces you to think in structured logic blocks — entry conditions, exit conditions, risk rules — which is actually a better development discipline than writing ad hoc scripts.

The main limitation I hit was **backtest data depth**. A trend-following strategy needs to be validated across multiple bull and bear cycles to have any statistical confidence. The available window was too short to do that properly.

Suggestions:
- Extend historical data depth to at least 2–3 years
- Add a "logic trace" view that shows *why* each trade triggered (which condition was the deciding factor)
- Allow multi-asset backtesting in one Playbook run

**On agentic trading broadly:** The shift toward agents that manage full trade lifecycles — not just signal generation — is the right direction. The next frontier is agents that adjust their own parameters based on detected regime changes, not just follow fixed rules. That's where CycleMind is headed.
