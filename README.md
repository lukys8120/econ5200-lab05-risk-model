# Diagnosing a Flawed Risk Model — VaR, Expected Shortfall & Monte Carlo

**ECON 5200 · Causal Machine Learning & Applied Analytics · Lab 05**

## Objective

I worked out why a junior analyst's Value at Risk model understated the risk on a $10M portfolio, and compared it with risk measures that do not assume returns are normally distributed.

## Methodology

- Ran the analyst's VaR model, which fits a normal distribution to 2,520 simulated daily returns (about ten years), and checked its assumption against the data's descriptive statistics. The returns have an excess kurtosis of 4.28, where a normal distribution has 0.
- Compared the normal VaR with the historical VaR, the actual percentile of the returns, at both 99% and 95% confidence.
- Computed 99% VaR and Expected Shortfall three ways: historical, normal, and a Student-t fitted by maximum likelihood (fitted df = 4.58).
- Priced a European call option by Monte Carlo, once with independent draws and once with antithetic variates (pairing each draw Z with −Z), and compared the standard errors with each other and the prices with the exact Black-Scholes price.
- Used `risk_metrics.py`, a module with `calculate_var`, `calculate_es` and `mc_var`, and ran its self-tests.
- Had an AI write a VaR backtest function, revised my prompt once to state that VaR is passed in as a positive loss, and checked its output against my own count of breach days.

## Key Findings

**The normal VaR understates the risk at 99%.** It came out at $276,519 against a historical VaR of $316,912, which is 12.7% too small and leaves the portfolio about $40,393 short of the capital it needs. The formula only uses the mean and the standard deviation, so it cannot see the fat tails.

**The error flips at 95%.** There the normal VaR is 3.5% too large, so a backtest at 95% would make the model look conservative. Passing at 95% says nothing about whether it holds at 99%.

**Expected Shortfall shows how bad the bad days are.** At 99% the historical ES was $391,464 and the Student-t ES was $426,060, against $317,193 under the normal. The normal model's average bad day is about where the real bad days only start.

**Antithetic variates cut the Monte Carlo standard error by 1.26x** with the same number of paths. The gain is modest because the call payoff is cut off at zero, so a large payoff on one path is only partly offset by its mirror path.

**The backtest confirms the diagnosis.** The AI's function matched my count exactly. The normal 99% VaR was breached on 1.71% of days (about 43 of 2,520) instead of the 1% it promises, while the historical VaR was breached on 1.03%. The analyst's conclusion that risk was "well within acceptable limits" does not hold.

## Repository Contents

| File | Description |
|---|---|
| `lab-ch05-diagnostic.ipynb` | Full lab notebook with rendered output |
| `risk_metrics.py` | VaR, Expected Shortfall and Monte Carlo VaR functions, with self-tests |

## Data

2,520 simulated daily returns drawn from a Student-t distribution with 5 degrees of freedom, scaled to a mean of 0.04% and a standard deviation of 1.2% a day, with `seed=42`. The t distribution reproduces the fat tails of real equity returns but not their negative skew.
