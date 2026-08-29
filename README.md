# EU ETS Carbon Futures: 40-Strategy Systematic Backtest

## Research Question
Does the illiquidity, regulatory structure, and lack of fundamental anchoring in carbon markets create patterns that wouldn't survive in a liquid equity market?

## Data
- **Instrument:** EU Carbon Emissions Futures, the CFI2 futures contract (ticker: CFI2H6), traded on ICE, in EUR
- **Underlying:** European Union Allowances (EUAs), the core instrument of the EU Emissions Trading System (EU ETS), the world's largest carbon market. Each EUA is the right to emit 1 tonne of CO2 equivalent.
- **Source:** [Investing.com](https://fr.investing.com/commodities/carbon-emissions)
- **Period:** April 2021 to April 2026, about 1,286 daily closing prices
- **Returns:** log returns throughout, since they're additive over time and compound correctly

(Raw price CSV isn't included in this repo, see Data Availability below.)

## Methodology
A shared backtest engine (Cell 5) runs every strategy under the same rules so results are comparable across families. 40 strategies were tested across 8 families:

| Family | Strategies | Core Idea |
|---|---|---|
| 1. Trend Following | S01-S08 | Ride sustained directional moves (SMA/EMA crossovers, MACD, Parabolic SAR, 12-1 and 3-month momentum) |
| 2. Mean Reversion | S09-S15 | Fade extreme moves back toward average (RSI, Bollinger fade/breakout, Z-score reversion, distance from 52-week high) |
| 3. Volatility and Regime | S16-S21 | Adapt to calm vs crisis conditions (low-vol filters, vol contraction breakout, vol targeting, regime-conditioned RSI) |
| 4. Carbon-Specific Calendar | S22-S26 | Exploit EU ETS regulatory calendar effects (compliance deadline, auction-week avoidance, end-of-quarter buying, winter energy premium) |
| 5. Oscillator and Technical | S27-S31 | Classic technical oscillators (Stochastic, Williams %R, Rate of Change, Keltner Channel, CCI) |
| 6. Breakout and Channel | S32-S35 | Price-channel breakouts (Donchian 20d/55d, Inside Day, 52-week high proximity) |
| 7. Adaptive and ML-Lite | S36-S38 | Dynamic/adaptive-threshold versions of core signals |
| 8. Ensemble and Portfolio | S39-S40 | Combine signals: a 4-signal Trend Ensemble and an 8-family Master Ensemble |

Each strategy is benchmarked against Buy and Hold on annualized return, Sharpe ratio, and max drawdown. There's also a full correlation matrix across all 40 strategies to check for genuine diversification vs strategies that are really just placing the same bet twice.

## Key Findings
1. **Trend following dominates.** Carbon prices move in sustained directional arcs driven by regulatory cycles (annual cap reductions, policy announcements). There isn't much arbitrage capital in this market, so trends persist longer than efficient-market theory would predict.
2. **Mean reversion doesn't work well on its own.** Pure RSI-style fades underperform. Carbon can stay "overbought" for months on genuine policy tightening. Pairing mean reversion with a volatility filter helps.
3. **Volatility regime is the strongest filter I found.** Going flat during high-vol regimes avoids uncompensated tail risk without really costing average returns, since vol spikes tend to line up with real repricing events like energy crises or policy shocks.
4. **Calendar effects are real and they persist.** The April compliance deadline, winter energy premium, and end-of-quarter institutional buying all show up as statistically visible patterns. That's because compliance buyers aren't return-maximizers, they buy when they have to.
5. **Transaction costs kill short-horizon edges.** High-frequency oscillators that turn over every 2-3 days lose their edge once realistic trading costs are applied. Strategies holding 10+ days keep theirs.
6. **Ensembles win on a risk-adjusted basis.** The Master Ensemble (S40) and Trend Ensemble (S39) rank near the top on Sharpe and Calmar ratios by combining signals that aren't correlated with each other.
7. **Buy and Hold is a tough benchmark to beat.** The EU cuts the allowance cap every year, which is a structural tailwind, not a random walk. Active strategies have to beat that drift and cover costs.

**Takeaway:** combining a slow trend filter (SMA 50/200 or a triple EMA stack), a volatility regime exit, and the April compliance calendar overlay gave a Sharpe ratio clearly above Buy and Hold, with a much lower max drawdown.

## Tech Stack
- Python: pandas, numpy, matplotlib, scipy.stats, statsmodels (ACF/PACF), quantstats
- Built and run in Google Colab

## Data Availability
Daily EUA futures prices used here come from Investing.com and aren't redistributed in this repo due to licensing terms. To reproduce this, download historical CFI2 futures data from Investing.com (or a similar provider) as a two-column CSV (Date, Price) and run Cell 2 to load and clean it.

## How to Run
This started as a Colab notebook (.ipynb) exported to a .py script, so the `!pip install` and `google.colab.files.upload()` calls are Colab-specific. To run it:
1. Open it in Google Colab, that matches the original setup, or
2. Adapt it for local use: swap `!pip install quantstats statsmodels -q` for a requirements.txt/pip install, and replace the Colab upload cell with a plain `pd.read_csv("your_file.csv")`.

## Limitations
- Backtested performance doesn't guarantee future results. A regulatory shift (EU ETS reform, a change to the cap trajectory) could break these patterns.
- This is one asset class. Findings are specific to EU carbon allowances and may not carry over to other emissions markets like UK ETS or California Cap-and-Trade.
- Transaction cost assumptions are estimated at 2bp per trade, not pulled from an actual broker fee schedule.
