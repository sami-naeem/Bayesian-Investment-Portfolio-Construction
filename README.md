<p align="center">
  <img src="Pics/header.png" alt="Bayesian Investment Portfolio Construction" />
</p>


# Background


# Data


**Data Source:** Yahoo Finance API

| **Phase**         | **Period**                  | **Duration** |
|-------------------|-----------------------------|--------------|
| Data History      | 01/01/2013 – 12/31/2024     | 12 years     |
| Model Training    | 01/01/2013 – 12/31/2018     | 6 years      |
| Model Testing     | 01/01/2019 – 12/31/2024     | 6 years      |



**Limitations (data history):** 
- Limited data availability prior to 2010
- Extended history needs to be analyzed to model different market cycles


#### Asset Classes


| **Asset Class**             | **Benchmark Index**                          | **ETF Name**                                       | **Ticker** |
|:----------------------------|:---------------------------------------------|:---------------------------------------------------|:----------:|
| **Large Cap**               | S&P 500 Index                                 | iShares Core S&P 500 ETF                           | `IVV`      |
| **Mid Cap**                 | S&P MidCap 400 Index                          | iShares S&P MidCap 400 ETF                         | `IJH`      |
| **Small Cap**               | S&P SmallCap 600 Index                        | iShares S&P SmallCap 600 ETF                       | `IJR`      |
| **Intl. Dev. Equities**     | MSCI EAFE Index                               | Vanguard FTSE Developed Markets ETF                | `VEA`      |
| **Emerging Market Equities**| MSCI Emerging Markets Index                  | Vanguard FTSE Emerging Markets ETF                 | `VWO`      |
| **Intermediate Bonds**      | Bloomberg U.S. Aggregate Bond Index           | Vanguard Total Bond Market ETF                     | `BND`      |
| **T-Bills**                 | ICE BofA 3-Month U.S. Treasury Bill Index     | SPDR Bloomberg 1–3 Month T-Bill ETF                | `BIL`      |
| **REITs**                   | FTSE Nareit All Equity REITs Index           | Vanguard Real Estate ETF                           | `VNQ`      |
| **Commodities**             | S&P GSCI Total Return Index                   | iShares S&P GSCI Commodity-Indexed Trust           | `GSG`      |


# Optimization Functions

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

- Takes point estimates of expected returns (μ) and the covariance matrix (Σ), computes excess returns over a risk-free rate, then finds weights w ∝ Σ⁻¹(μ – rₙ).
- Clips negative weights to zero (no shorting) and normalizes so that weights sum to one.
- Allocates more to assets with high expected return relative to their contribution to overall volatility.



### 2. Maximum Sharpe‐Ratio


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

- Solves a constrained optimization to maximize Sharpe ratio.
- Chooses the portfolio on the efficient frontier that gives the highest reward per unit of risk


### 3. Minimum CVaR Portfolio (at level α)



How it works: 

- Given a matrix of simulated returns (shape = sims × assets), defines portfolio return in each simulation and computes the Conditional Value at Risk (average of worst α-percentile losses)
- Directly targets tail-risk: ensures that, in the worst α*100% of scenarios, the average loss is as small as possible

# Investment Portfolio Generator Models

## 1. Static-Weights Portfolios 

**Morningstar’s Target Allocation Index** used to determine the static portfolios:  

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



## 2. Monte Carlo Sampling & Distribution Metrics


xxx asset class sampled 

**Model Details:** 
- MC simulations: 10 000 draws

- Risk‐free rate (for MC and MV): default 0% (set to 1% in model)
- Hierarchical/Multivariate Models: Used to map the relationship between the different asset classes, while minimizing overfitting.


How it works:
- Estimate μ and Σ from historical returns.
- Simulate thousands of return scenarios from 𝒩(μ,Σ).
- Compute distribution metrics (mean, cov, VaR, CVaR).
- Apply each of the three optimizers (mean-variance, max-Sharpe, min-CVaR) to those metrics.
- By simulating, you capture how random fluctuations might play out and build portfolios based on the full distribution—not just point estimates.





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
\mu_i^{\mathrm{sim}}
= \frac{1}{S}\sum_{s=1}^S R^{(s)}_i,
\quad
\Sigma_{ij}^{\mathrm{sim}}
= \frac{1}{S-1}\sum_{s=1}^S
  \bigl(R_i^{(s)}-\mu_i^{\mathrm{sim}}\bigr)
  \bigl(R_j^{(s)}-\mu_j^{\mathrm{sim}}\bigr).
$$
$$
\mathrm{VaR}_\alpha(w)
= \mathrm{Quantile}\bigl\{w^\top R^{(s)},\,\alpha\bigr\},
$$
$$
\mathrm{CVaR}_\alpha(w)
= \mathbb{E}\bigl[w^\top R^{(s)} \mid w^\top R^{(s)} \le \mathrm{VaR}_\alpha(w)\bigr].
$$


xxxx Asset class distribution charts xxxxx



## 3. Bayesian Hierarchical MCMC Model (PyMC)

**Model Details:**
- MCMC sampling: 4 chains; 1 000 tune + 3 000 draws; target_accept = 0.95

- Risk‐free rate (for MC and MV): default 0% (set to 1% in model)
- Hierarchical/Multivariate Models: Used to map the relationship between the different asset classes, while minimizing overfitting.


How it works: 

- Specify priors:
  - μₐ ~ Normal(0,0.1) for each asset a
  - Σ via an LKJ-Cholesky prior for correlations and Half-Normal priors for marginal SDs
- Observe historical returns as a multivariate normal (with the Cholesky factor).
- Sample the joint posterior of (μ,Σ) using PyMC’s NUTS.
- Summarize the posterior: extract posterior means of μ and empirical covariance of all μ-draws.
- Plug into the three optimizer functions to get final weights.
- Fully accounts for uncertainty in your estimates—if data are noisy or scarce, your posterior spreads will be wider, leading to more conservative allocations.


#### Priors

\mu_i \;\sim\; \mathcal{N}(0,\,0.1^2),
\quad
i=1,\dots,n.


L \;\sim\; \mathrm{LKJCholeskyCov}\bigl(\eta,\;\mathrm{sd\_dist}\bigr),
\quad
\Sigma = L\,L^\top.


#### Likelihood


R_t \;\sim\; \mathcal{N}(\mu,\,\Sigma),
\quad
t=1,\dots,T.


#### Posterior


p(\mu,\Sigma \mid R_{1:T})
\;\propto\;
\biggl[\prod_{t=1}^T\mathcal{N}(R_t\mid\mu,\Sigma)\biggr]
\times \mathcal{N}(\mu\mid0,0.1^2I)
\times \mathrm{LKJ}(\Sigma).

xxxx Asset class distribution charts xxxxx



#### Additional Model Enhancement

XXXXXXXXXXXX Investr view 


Interpretation & choice of η
	• η = 1
“I have no strong belief about correlations” (flat).
	• η = 2
Mild shrinkage toward zero correlation.
	• η ≥ 4
Strong shrinkage: most off‐diagonals will be near zero.
In high dimensions, choosing η slightly above 1 (e.g.\ 1.5–2) often yields better behaved estimates than a fully uniform prior.


adjusted_mu = mu_post + tilt_vector

# Results 

- Log Returns used to evalaute portfolio performance because:
  - Factors in compunding and multi-period aggregation
  - Symmmetry for gains and losses AKA gains and losses don't cancel each other out
  - Helps capture small changes as compunding is captured over time   


# Takeaways and Future Enhancements

Modelling enhancements: 
- A longer data history needs to be covered to include several market cycles to better model the bheavior of investment securities.
  - Compute limitations resulted in the data history being limited to 2010 and onwards.
  - Limited data history for some investment securities
    - Data history standardization approach needs to be developed to model investment security distribuutions while accounting for varying data histories.  
  

The Bayesian models (Monte Carlo Sampling and Bayesian Hierarchical MCMC models) can be further improve by: 
- Factoring in asset class allocation weights as part of the model. 

- Bayesian Rebalance schedule

Portfolio Construction guardrails: 

- Limits on allocation to a specific asset class
- Threshold specification of deviation from asset class targets that will trigger a rebalance
-  





## Part 2 Get Started



5. **Packages**  
    - quantmod
    - cbw
    - xts
    - coda
    - bayesm
    - ggplot2


##### Author: Sami Naeem

