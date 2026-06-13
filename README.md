<p align="center">
  <img src="Pics/header.png" alt="Bayesian Investment Portfolio Construction" />
</p>

# Bayesian MCMC Hierarchical Portfolio Construction Model

This project builds a proprietary Bayesian Hierarchical MCMC portfolio construction model — unavailable in any commercial robo-advisor — that samples the full probability distribution of returns and correlations across 9 asset classes, embedding investment uncertainty directly into the allocation.

Three design choices differentiate it from industry models:
- **Hierarchical asset grouping** — reduces overfitting by sharing information across related asset classes
- **LKJ-Cholesky priors** — encode investor views directly on cross-asset correlations, extending Black-Litterman beyond expected returns into the full covariance structure
- **CVaR optimization** — targets tail-risk directly rather than variance

Backtested against Morningstar's static allocation benchmarks and a Monte Carlo baseline over 6 years (2019–2024).

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![PyMC](https://img.shields.io/badge/PyMC-Bayesian_MCMC-7B68EE)
![ArviZ](https://img.shields.io/badge/ArviZ-Inference-4B8BBE)
![cvxpy](https://img.shields.io/badge/cvxpy-Optimization-FF6F00)
![yfinance](https://img.shields.io/badge/yfinance-Market_Data-235d9f)
![NumPy](https://img.shields.io/badge/NumPy-2.0-013243?logo=numpy&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-Scientific-8CAAE6?logo=scipy&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-11557C)
![License](https://img.shields.io/badge/License-Apache_2.0-brightgreen)

---

## Table of Contents

- [Setup](#setup)
- [Data](#data)
- [Optimization Functions](#optimization-functions)
- [Portfolio Models](#investment-portfolio-generator-models)
  - [Static-Weights](#1-static-weights-portfolios)
  - [Monte Carlo](#2-monte-carlo-sampling--distribution-metric-portfolios)
  - [Bayesian MCMC](#3-bayesian-hierarchical-mcmc-model-pymc)
- [Results](#results)
- [Takeaways & Future Enhancements](#takeaways-and-future-enhancements)

---

## Setup

Install dependencies:

```bash
pip install -r requirements.txt
```

Run `main.ipynb` end-to-end. Note: MCMC sampling (Section 4.3) runs 4 chains × 4,000 iterations and takes approximately 10–20 minutes depending on hardware. Saved trace artifacts are stored in `models/`.

---

## Data

**Source:** Yahoo Finance API

| **Phase**         | **Period**                  | **Duration** |
|-------------------|-----------------------------|--------------|
| Data History      | 01/01/2013 – 12/31/2024     | 12 years     |
| Model Training    | 01/01/2013 – 12/31/2018     | 6 years      |
| Model Testing     | 01/01/2019 – 12/31/2024     | 6 years      |


**Limitations:**
- Limited data availability prior to 2010
- Extended history needed to model different market cycles


#### Asset Classes


| **Asset Class**              | **Benchmark Index**                            | **ETF Name**                                     | **Ticker** |
|:-----------------------------|:-----------------------------------------------|:-------------------------------------------------|:----------:|
| **Large Cap**                | S&P 500 Index                                  | iShares Core S&P 500 ETF                         | `IVV`      |
| **Mid Cap**                  | S&P MidCap 400 Index                           | iShares S&P MidCap 400 ETF                       | `IJH`      |
| **Small Cap**                | S&P SmallCap 600 Index                         | iShares S&P SmallCap 600 ETF                     | `IJR`      |
| **Intl. Dev. Equities**      | MSCI EAFE Index                                | Vanguard FTSE Developed Markets ETF              | `VEA`      |
| **Emerging Market Equities** | MSCI Emerging Markets Index                    | iShares MSCI Emerging Markets ETF                | `EEM`      |
| **Intermediate Bonds**       | ICE U.S. Treasury 7-10 Year Bond Index         | iShares 7-10 Year Treasury Bond ETF              | `IEF`      |
| **T-Bills**                  | ICE Short U.S. Treasury Securities Index       | iShares Short Treasury Bond ETF                  | `SHV`      |
| **REITs**                    | FTSE Nareit All Equity REITs Index             | Vanguard Real Estate ETF                         | `VNQ`      |
| **Commodities**              | DBIQ Optimum Yield Diversified Commodity Index | Invesco DB Commodity Index Tracking Fund         | `DBC`      |

The asset classes above represent the major public markets, helping effectively model public markets while minimizing model dimensionality.


#### Asset Class Grouping

| **Asset Class**            | **Group Index** | **Group Name**       |
|----------------------------|:---------------:|:---------------------|
| **Large Cap**              | 0               | Domestic Equity      |
| **Mid Cap**                | 0               | Domestic Equity      |
| **Small Cap**              | 0               | Domestic Equity      |
| **Intl Dev Equity**        | 1               | International Equity |
| **Emerging Market Equity** | 1               | International Equity |
| **Intermediate Bonds**     | 2               | Fixed Income         |
| **T-Bill**                 | 2               | Fixed Income         |
| **REIT**                   | 3               | Alternatives         |
| **Commodities**            | 3               | Alternatives         |


##### Why group asset classes?
- Assets within the same category (e.g. all domestic equities) often share similar risk-return characteristics.
- Partial pooling of means: instead of giving each asset its own independent prior, a group-level mean is introduced. Enables portfolio optimization using the investment type categories.

---

## Optimization Functions

#### Output: Asset Class Investment Allocation Weights
These weights are used to construct portfolios and test the performance of all portfolios.

### 1. Mean Variance Optimization


$$
w \;\propto\; \Sigma^{-1}\,(\mu - r_f \,\mathbf{1})
$$

Subject to

$$
w_i \ge 0,
\qquad
\sum_{i=1}^n w_i = 1.
$$

How it works:

- Takes point estimates of expected returns (μ) and the covariance matrix (Σ), computes excess returns over a risk-free rate, then finds weights w ∝ Σ⁻¹(μ – rₙ).
- Clips negative weights to zero (no shorting) and normalizes so weights sum to one.
- Allocates more to assets with high expected return relative to their contribution to overall volatility.



### 2. Maximum Sharpe-Ratio


$$
\mathrm{Sharpe}(w)
= \frac{w^\top \mu - r_f}{\sqrt{w^\top \Sigma\,w}}
$$

Maximize Sharpe\,(w) subject to

$$
w_i \ge 0,
\qquad
\sum_{i=1}^n w_i = 1.
$$


How it works:

- Solves a constrained optimization to maximize the Sharpe ratio.
- Chooses the portfolio on the efficient frontier that gives the highest reward per unit of risk.


### 3. Minimum CVaR Portfolio (at level α)


**Portfolio return for scenario (s):**

$$
R_p^{(s)} := \sum_{i=1}^n w_i\,r_i^{(s)}, \quad s = 1,2,\dots,S.
$$


$$
\begin{aligned}
\min_{\mathbf{w},\,\zeta}\quad
& \zeta \;+\; \frac{1}{S\,(1-\alpha)} \sum_{s=1}^S \max\{-R_p^{(s)} - \zeta,\,0\},\\
\text{s.t.}\quad
& \sum_{i=1}^n w_i = 1,\\
& 0 \le w_i \le 1,\quad \forall i.
\end{aligned}
$$


where
- $S$ is the number of simulated scenarios,
- $r_s \in \mathbb{R}^N$ are the asset returns in scenario $s$,
- $\zeta$ is the VaR (i.e. the $\alpha$-quantile loss),
- $\alpha \in (0,1)$ is the tail probability.


How it works:

- Given a matrix of simulated returns (shape = sims × assets), defines portfolio return in each simulation and computes the Conditional Value at Risk (average of worst α-percentile losses).
- Directly targets tail-risk: ensures that, in the worst α×100% of scenarios, the average loss is as small as possible.

---

## Investment Portfolio Generator Models

## 1. Static-Weights Portfolios

**Morningstar's Target Allocation Index** used to determine the static portfolios:

- **Aggressive**: 90% equity, 5% bonds, 5% cash
- **Balanced**: 60% equity, 30% bonds, 10% cash
- **Conservative**: 30% equity, 50% bonds, 20% cash


<p align="center">
  <img src="Pics/static_portfolio_equity_fi_split.png" alt="Static Portfolio Equity/Fixed Income Split" />
</p>


#### Portfolio Allocation


| **Asset Class**            | **Aggressive** | **Balanced** | **Conservative** |
|----------------------------|:--------------:|:------------:|:----------------:|
| **Large Cap**              |      20%       |     15%      |       10%        |
| **Mid Cap**                |      20%       |     10%      |        5%        |
| **Small Cap**              |      15%       |     10%      |        5%        |
| **Intl. Dev Equity**       |      15%       |     10%      |        5%        |
| **Emerging Market Equity** |      10%       |      5%      |        3%        |
| **REIT**                   |       5%       |      5%      |        1%        |
| **Commodities**            |       5%       |      5%      |        1%        |
| **Intermediate Bonds**     |       5%       |     30%      |       50%        |
| **T-Bill**                 |       5%       |     10%      |       20%        |



## 2. Monte Carlo Sampling & Distribution Metric Portfolios


**Model Details:**
- MC simulations: 10,000 draws
- Risk-free rate: 1%
- Hierarchical/Multivariate Models: used to map the relationship between the different asset classes while minimizing overfitting.


How it works:
- Estimate μ and Σ from historical returns.
- Simulate thousands of return scenarios from 𝒩(μ, Σ).
- Compute distribution metrics (mean, cov, VaR, CVaR).
- Apply each of the three optimizers (mean-variance, max-Sharpe, min-CVaR) to those metrics.
- By simulating, you capture how random fluctuations might play out and build portfolios based on the full distribution — not just point estimates.


#### Parameter Estimation


$$
\hat\mu = \frac{1}{T}\sum_{t=1}^T R_t,
\quad
\hat\Sigma = \frac{1}{T-1}\sum_{t=1}^T (R_t - \hat\mu)(R_t - \hat\mu)^\top.
$$

#### Simulation


$$
R^{(s)} \;\sim\; \mathcal{N}\bigl(\hat\mu,\;\hat\Sigma\bigr),
\quad
s=1,\dots,S.
$$


#### Metrics


$$
\mu_i^{(\mathrm{sim})}
\;=\;\frac{1}{S}\sum_{s=1}^S R_i^{(s)},
\quad
\Sigma_{ij}^{(\mathrm{sim})}
\;=\;\frac{1}{S-1}\sum_{s=1}^S
\bigl(R_i^{(s)} - \mu_i^{(\mathrm{sim})}\bigr)
\bigl(R_j^{(s)} - \mu_j^{(\mathrm{sim})}\bigr).
$$


where

- $S$ is the total number of Monte Carlo samples.
- $\mathbf{r}_s = \bigl(r_{1,s},\dots,r_{N,s}\bigr)^\top$ is the vector of simulated returns in iteration $s$.
- $\mu_i^{(\mathrm{sim})}$ is the average simulated return of asset $i$.
- $\Sigma_{i,j}^{(\mathrm{sim})}$ is the sample covariance between assets $i$ and $j$.


#### Asset Class Mapping Distributions

##### Intermediate Bonds (Monte Carlo)
![Intermediate Bonds – Monte Carlo](./Pics/intermediate_bonds_monte.png)


##### Small Cap (Monte Carlo)
![Small Cap – Monte Carlo](./Pics/small_cap_monte.png)


#### Model Weights


| **Asset Class**             | **Mean-Variance** | **Max-Sharpe** | **CVaR (5%)** |
|-----------------------------|------------------:|---------------:|--------------:|
| **Commodities**             |             0.00% |          0.00% |         0.00% |
| **Emerging Market Equity**  |             0.00% |          0.00% |       100.00% |
| **Intermediate Bonds**      |             0.57% |          0.31% |         0.00% |
| **Mid Cap**                 |             0.00% |          0.00% |         0.00% |
| **Small Cap**               |             0.91% |          0.15% |         0.00% |
| **Large Cap**               |             1.66% |          0.48% |         0.00% |
| **T-Bill**                  |            96.72% |         99.06% |         0.00% |
| **Intl Dev Equity**         |             0.13% |          0.00% |         0.00% |
| **REIT**                    |             0.00% |          0.00% |         0.00% |

> **Note:** The CVaR optimizer concentrates the full allocation into Emerging Market Equity. This is a known behavior of linear CVaR minimization: when a single asset dominates the tail-risk frontier across simulations, the optimizer collapses to it. This highlights a limitation of running CVaR on simulated Gaussian draws — it does not penalize concentration risk. A per-asset maximum weight cap is listed as a future enhancement.


## 3. Bayesian Hierarchical MCMC Model (PyMC)

**Model Details:**
- MCMC sampling: 4 chains; 1,000 tune + 3,000 draws; target_accept = 0.95
- Risk-free rate: 1%
- Hierarchical/Multivariate Models: used to map the relationship between the different asset classes while minimizing overfitting.


How it works:

- Specify priors:
  - μₐ ~ Normal(0, 0.1) for each asset a
  - Σ via an LKJ-Cholesky prior for correlations and Half-Normal priors for marginal SDs
- Observe historical returns as a multivariate normal (with the Cholesky factor).
- Sample the joint posterior of (μ, Σ) using PyMC.
- Summarize the posterior: extract posterior means of μ and empirical covariance of all μ-draws.
- Plug into the three optimizer functions to get final weights.
- Fully accounts for uncertainty in estimates — if data are noisy or scarce, posterior spreads will be wider, leading to more conservative allocations.


#### Priors

$$
\mu_i \sim \mathcal{N}\bigl(0,\;0.1^2\bigr),
\quad i = 1, \dots, n.
$$


$$
L \sim \mathrm{LKJCholeskyCov}\bigl(\eta,\;\mathrm{sd\_dist}\bigr),
\quad
\Sigma = L\,L^\top.
$$


**LKJ–Cholesky (model parameter η = 2):**
Rather than sampling a full correlation matrix directly, we sample its Cholesky factor **L** from the LKJ distribution. This ensures positive-definiteness and efficient sampling in PyMC. The shape parameter η controls the prior belief about cross-asset correlations — higher values shrink off-diagonal correlations toward zero. This is a fixed model parameter set at build time, distinct from the per-asset investor tilt described below.

| **η (model)** | **Prior belief on correlations**                            |
|:-------------:|:------------------------------------------------------------|
| 1             | Flat — no preference over correlation structure             |
| **2** (used)  | Mild shrinkage — moderate pull of off-diagonals toward zero |
| ≥ 4           | Strong shrinkage — most correlations pulled near zero       |


#### Likelihood


$$
R_t \sim \mathcal{N}\bigl(\mu,\;\Sigma\bigr),
\quad t = 1, \dots, T.
$$


#### Posterior


$$
p\bigl(\mu,\Sigma \mid R_{1:T}\bigr)
\;\propto\;
\Biggl[\prod_{t=1}^T \mathcal{N}\bigl(R_t \mid \mu,\Sigma\bigr)\Biggr]
\;\times\;
\mathcal{N}\bigl(\mu \mid 0,\,0.1^2\bigr)
\;\times\;
\mathrm{LKJ}\bigl(\Sigma \mid \eta\bigr).
$$


#### Asset Class Mapping Distributions

#### Intermediate Bonds (MCMC)
![Intermediate Bonds – MCMC](./Pics/intermediate_bonds_mcmc.png)


#### Small Cap (MCMC)
![Small Cap – MCMC](./Pics/small_cap_mcmc.png)


### Enhancement — Investor Return Tilt

After MCMC sampling produces posterior mean returns for each asset, the model allows an investor to inject forward-looking views before optimization. This extends the Black-Litterman intuition — where views on expected returns are overlaid on a model-derived baseline — but replaces the equilibrium CAPM prior with a full Bayesian posterior. The investor's view adjusts where the posterior mean sits before the optimizer sees it, producing allocations that reflect both the historical data and the investor's forward-looking conviction.

#### Mechanism

For each asset, the investor specifies an additive tilt η expressed in the same units as the return series (daily log returns). The tilt shifts the posterior mean upward for a bullish view or downward for a bearish view:

$$\tilde{\mu}_i = \mu_i^{\text{post}} + \eta_i$$

The adjusted return vector $\tilde{\mu}$ replaces the raw posterior means as input to the three optimizers (mean-variance, max-Sharpe, min-CVaR).

#### Tilt Direction Guide

| **η value**  | **Investor view**                            | **Effect on allocation**                                        |
|:------------:|:---------------------------------------------|:----------------------------------------------------------------|
| Positive (+) | Bullish — expects this asset to outperform   | Optimizer allocates more to this asset vs. the no-tilt baseline |
| Zero         | Neutral — rely entirely on the posterior     | No adjustment; posterior mean used as-is                        |
| Negative (−) | Bearish — expects this asset to underperform | Optimizer allocates less vs. the no-tilt baseline               |

> **Scale note:** Tilt values are in daily log return units (typical daily magnitude: −0.02 to +0.02). A tilt of +0.001 represents a forward-looking view of approximately +0.1% additional daily return for that asset. Larger values place increasing weight on the investor's view relative to the data-derived posterior.

#### Tilt Values Applied in This Model

Assets not listed carry a neutral tilt of η = 0 and are allocated purely from the posterior.

| **Asset Class** | **Posterior Mean (μ_post)** | **Investor Tilt (η)** | **Adjusted Mean (μ̃)** |
|:----------------|----------------------------:|----------------------:|-----------------------:|
| Large Cap       |                       0.069 |                  +2.0 |                  2.069 |
| Commodities     |                      −0.001 |                  +1.0 |                  0.999 |
| *All others*    |                     various |                   0.0 |              unchanged |


#### Model Weights


| **Asset Class**            | **Mean-Variance** | **Max-Sharpe** | **CVaR (5%)** |
|----------------------------|------------------:|---------------:|--------------:|
| **Commodities**            |             0.00% |          0.00% |         0.00% |
| **Emerging Market Equity** |            14.93% |          0.00% |         0.00% |
| **Intermediate Bonds**     |            69.68% |         99.07% |         0.00% |
| **Mid Cap**                |             0.00% |          0.00% |         0.00% |
| **Small Cap**              |             4.08% |          0.00% |         0.00% |
| **Large Cap**              |             0.00% |          0.00% |         0.00% |
| **T-Bill**                 |            11.21% |          0.00% |         0.00% |
| **Intl Dev Equity**        |             0.10% |          0.93% |         0.00% |
| **REIT**                   |             0.00% |          0.00% |       100.00% |

> **Note:** The Bayesian CVaR portfolio concentrates fully in REIT. As with the Monte Carlo CVaR result, this reflects the optimizer's behavior under Gaussian-simulated scenarios without a concentration constraint. A per-asset maximum weight cap is listed as a future enhancement.

---

## Results

Log returns used to evaluate portfolio performance because:
- Factors in compounding and multi-period aggregation
- Symmetry for gains and losses — gains and losses do not cancel each other out
- Helps capture small changes as compounding is captured over time


$$
r_t = \ln\!\Bigl(\frac{P_t}{P_{t-1}}\Bigr)
$$

> **Key result:** The Bayesian CVaR portfolio returned **26.87%** over the 6-year test period — nearly **4× the Monte Carlo CVaR equivalent (6.95%)** and outperforming Morningstar's Conservative static benchmark (23.89%).

#### Portfolio Cumulative Returns (01/01/2019 - 12/31/2024)

![Portfolio Cumulative Returns](./Pics/portfolio_cumulative_returns.png)

#### Yearly Returns

| **Year** | **Static Aggressive** | **Static Balanced** | **Static Conservative** | **MC Mean-Variance** | **MC Max-Sharpe** | **MC CVaR (5%)** | **Bayes Mean-Variance** | **Bayes Max-Sharpe** | **Bayes CVaR (5%)** |
|:--------:|----------------------:|--------------------:|------------------------:|---------------------:|------------------:|-----------------:|------------------------:|---------------------:|--------------------:|
| **2020** |                 5.47% |               7.17% |                   8.50% |                1.19% |            10.11% |           10.11% |                   9.78% |                9.74% |             −13.26% |
| **2021** |                18.24% |              12.03% |                   4.21% |                0.41% |            −5.16% |           −5.16% |                  −2.03% |               −3.33% |              38.99% |
| **2022** |               −16.18% |             −14.30% |                 −12.80% |                0.35% |           −22.72% |          −22.72% |                 −14.71% |              −15.59% |             −28.47% |
| **2023** |                14.04% |              10.52% |                   7.83% |                5.40% |             7.67% |            7.67% |                   4.78% |                3.32% |               9.64% |
| **2024** |                 9.72% |               6.66% |                   4.43% |                5.37% |             5.58% |            5.58% |                   1.40% |               −0.64% |               2.57% |

> **Note:** 2019 is excluded from the annual returns table as it represents the first year of the test period. The portfolio requires a full prior calendar year to establish a starting NAV for year-over-year comparison.

#### Full-Period Cumulative Returns

| **Portfolio**               | **Cumulative Return** |
|:----------------------------|----------------------:|
| Static – Aggressive         |                59.28% |
| Static – Balanced           |                41.64% |
| Static – Conservative       |                23.89% |
| Monte Carlo – Mean-Variance |                16.51% |
| Monte Carlo – Max-Sharpe    |                 6.95% |
| Monte Carlo – CVaR (5%)     |                 6.95% |
| Bayesian – Mean-Variance    |                 6.35% |
| Bayesian – Max-Sharpe       |                −0.92% |
| Bayesian – CVaR (5%)        |                26.87% |

---

## Takeaways and Future Enhancements

#### Data Modelling:
- Longer data history needed to model different market cycles
  - Limited data history for some investment securities
- Posteriors obtained from recent data history led to higher portfolio performance
  - Explore decay functions to place more importance on recent data
- **Heavy-tailed likelihood:** Replace the Gaussian observation model with a Multivariate Student-t to better capture fat tails during market dislocations (e.g., 2020, 2022 drawdowns)

#### Portfolio Construction Engine:
- The Bayesian Portfolio Construction Engine outputs asset class weights. These weights can be incorporated as an additional input in the Portfolio Construction Engine, with the weights determined by a posterior to automate portfolio rebalancing.

#### Guardrails — Portfolio Construction Engine:
- Asset Allocation Limits: Maximum and minimum limits on asset allocation.
- Asset Allocation Target Deviation Threshold: Specify threshold for triggering a rebalance — e.g., 3% overall deviation from asset class targets.
- New Portfolio Allocation Guardrails: Factor prior portfolio allocation and implement a penalty function for deviating from prior asset classes. Penalize large asset allocation shifts to minimize large turnover.
- **Concentration Constraint:** Add a per-asset maximum weight cap (e.g., 40%) to prevent the CVaR optimizer from collapsing to a single asset.

---

##### Author: Sami Naeem

---

## License

This project is licensed under the Apache 2.0 License. See [LICENSE](LICENSE) for details.
