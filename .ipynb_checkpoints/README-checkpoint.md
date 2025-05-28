# Bayesian-Investment-Portfolio-Construction





XXXXXXXXX pic  XXXXXXXXXXXXX 







## Background

## Model Overview

Three portfolio construction models built: 

1. Static-Weights Portfolios

2. Monte Carlo Sampling & Distribution Metrics

3. Bayesian Hierarchical MCMC Model (PyMC)


Three portfolio optimization functions built:

1. cVar (Conditional Value at Risk) Optimization

2. Mean Variance Optimization

3. Sharpe Ratio Maximization



XXXXXXXXXXXX provide breakdown of each optimizer function ncluding what it calcualtes, business logic behind it, and how it may be better than a traditional portfolio 




## Modelling and Problem Solving Approach


Asset Allocation Data

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



XXXXXXXXXXXX conserv, bala, agg flow chart XXXXXXXXXXXX


### Static-Weights Portfolios 

Based on Morningstar’s Target Allocation Index definitions where: 
“Aggressive” portfolios hold > 90% in equity, 
“Moderate/Balanced” portfolios hold 60 – 75% in equity
“Conservative” portfolios hold < 40% in equity


xxxx diagram xxxxx


xxxx

### Bayesian Optimized Portfolios 

- Hierarchical/Multivariate Models: Used to map the relationship between the different asset classes, while minimizing overfitting.
- 


Date range
Model training:  
Model testing: 

#### Monte Carlo Sampling & Distribution Metrics


xxx asset class sampled 

Modelling details 


xxxx Asset class distribution charts xxxxx



#### Bayesian Hierarchical MCMC Model (PyMC)



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

## Results 

- Log Returns used to evalaute portfolio performance because:
  - Factors in compunding and multi-period aggregation
  - Symmmetry for gains and losses AKA gains and losses don't cancel each other out
  - Helps capture small changes as compunding is captured over time   


## Takeaways and Future Enhancements

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

