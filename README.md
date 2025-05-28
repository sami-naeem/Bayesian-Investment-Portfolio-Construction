<p align="center">
  <img src="Pics/header.png" alt="Bayesian Investment Portfolio Construction" />
</p>


# Background


# Data

Yahoo Finance API used to retriieve investment returns data


- **Data History:**   01/01/2013 - 12/31/2024
- **Model training:** 01/01/2013 - 12/31/2018 ~ 6 years 
- **Model testing:**  01/01/2019 - 12/31/2024 ~ 6 years


**Limitations (data history):** 
- Limited data availability prior to 2010
- Extended history needs to be analyzed to model different market cycles


**Asset Classes**


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


### 3. Minimum CVaR Portfolio (at level α)


# Investment Portfolio Generator Models

## 1. Static-Weights Portfolios 

**Morningstar’s Target Allocation Index** used to determine the static portfolios:  

- **Aggressive**: 90% equity, 5% bonds, 5% cash
- **Balanced**: 60% equity, 30% bonds, 10% cash
- **Conservative**: 30% equity, 50% bonds, 20% cash


<p align="center">
  <img src="Pics/static_portfolio_equity_fi_split.png" alt="Static Portfolio Equity/Fixed Income Split" />
</p>


**Portfolio Allocation:**


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


xxxx Asset class distribution charts xxxxx



## 3. Bayesian Hierarchical MCMC Model (PyMC)

**Model Details:**
- MCMC sampling: 4 chains; 1 000 tune + 3 000 draws; target_accept = 0.95

- Risk‐free rate (for MC and MV): default 0% (set to 1% in model)
- Hierarchical/Multivariate Models: Used to map the relationship between the different asset classes, while minimizing overfitting.

xxxx Asset class distribution charts xxxxx



##### Additional Model Enhancement

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

