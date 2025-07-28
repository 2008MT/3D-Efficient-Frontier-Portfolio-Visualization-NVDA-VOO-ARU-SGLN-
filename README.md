# Portfolio Optimization and the Efficient Frontier: Theory, Assets, and Interpretation

## Choice of Assets

Our portfolio consists of four strategically selected assets:

- **Nvidia (NVDA):** A leading technology company in AI and semiconductors, offering high growth potential and significant volatility. NVDA is chosen for its representation of innovation and tech sector exposure.
- **S&P 500 Vanguard ETF (VOO/VUAA):** This ETF tracks the S&P 500 index, providing broad exposure to the US equity market with relatively lower volatility. It supplies stability and diversification.
- **Arafura Rare Earths Ltd (ARU):** An Australian mining company focused on rare earths, ARU adds sector-specific risk and the potential for outsized returns due to its exposure to critical minerals needed for renewable energy and tech.
- **iShares Physical Gold ETF (SGLN):** Gold is a classic defensive asset. SGLN brings stability and acts as a hedge against market downturns, with typically low correlation to equities.

**Why this mix?**  
Combining these assets allows us to study the trade-off between risk and return, diversification benefits, and how different sectors and asset classes interact in portfolio construction.

---

## Data Collection

We fetch daily closing prices for each ticker over ~10 years using the Yahoo Finance API in Python, `yfinance`. We use alternative tickers for international listings to ensure data coverage.

---

## Theoretical Backing: Modern Portfolio Theory & Markowitz

The experiment is grounded in **Modern Portfolio Theory (MPT)**, developed by Harry Markowitz in the 1950s. MPT provides a quantitative framework for constructing portfolios that optimize expected return for a given level of risk, or equivalently, minimize risk for a given level of expected return.

**Key Concepts:**
- **Diversification:** Reduces portfolio risk by mixing assets that are not perfectly correlated.
- **Efficient Frontier:** The set of optimal portfolios offering the highest expected return for a defined level of risk.
- **Risk (Volatility):** Measured as the standard deviation of portfolio returns.
- **Return:** The expected annualized return of the portfolio.
- **Covariance/Correlation:** Captures how asset returns move together; crucial for understanding diversification's impact.

---

## Mathematical Framework

### 1. Portfolio Return

$$
E[R_p] = \sum_{i=1}^N w_i \cdot E[R_i]
$$

Where:
- $w_i$ : Weight of asset $i$
- $E[R_i]$ : Expected (annualized) return of asset $i$
- $N$ : Number of assets

### 2. Portfolio Risk (Volatility)

$$
\sigma_p = \sum_{i=1}^n \sum_{j=1}^n w_iw_jCov(R_i; R_j)
= \sqrt{ \mathbf{w}^T \Sigma \mathbf{w} }
$$

Where:
- $\mathbf{w}$ : Vector of asset weights
- $\Sigma$ : Covariance matrix of asset returns

### 3. Efficient Frontier

For every target portfolio return, the efficient frontier identifies the portfolio with the lowest possible volatility. In higher dimensions (more assets), the efficient frontier becomes an *efficient surface*, visualized in 3D.

---

## Interpretation of Results

### Asset Statistics

| Asset | Annualized Return | Volatility |
|-------|-------------------|------------|
| NVDA  | 73.5%             | 50.5%      |
| VOO   | 15.0%             | 18.6%      |
| ARU   | 48.1%             | 82.8%      |
| SGLN  | 14.0%             | 15.1%      |

- **NVDA** provides the highest returns but is very volatile.
- **VOO** and **SGLN** offer lower risk and more modest returns.
- **ARU** is extremely volatile but has the potential for high returns.

### Efficient Frontier and Surface

- **Efficient Frontier (2D):** The curve shows the best achievable risk-return combinations. Most portfolios lie below the frontier, meaning they are suboptimal (either too risky for their return or too low-return for their risk).
- **Efficient Surface (3D):** Illustrates how varying asset weights affects portfolio return and risk, revealing regions of high efficiency (high Sharpe ratio), low risk, and diversification benefits.

### Optimal Portfolios

- **Minimum Volatility Portfolio:** Heavy allocation to VOO and SGLN, almost no NVDA or ARU.  
  _Return: ~14.8%, Volatility: ~11.1%, Sharpe: 1.34_
- **Maximum Return Portfolio:** 100% NVDA.  
  _Return: ~73.5%, Volatility: ~50.5%, Sharpe: 1.46_
- **Maximum Sharpe Ratio Portfolio:** Blend of NVDA, ARU, and SGLN, optimizing risk-adjusted returns.  
  _Return: ~33.3%, Volatility: ~17.8%, Sharpe: 1.87_
- **Most Balanced Portfolio:** Almost equal weights, maximizing diversification.  
  _Return: ~37.3%, Volatility: ~26.7%, Sharpe: 1.40_

### Key Takeaways

- **Diversification works:** Balanced portfolios can offer attractive returns with moderate risk.
- **Risk-return trade-off:** Chasing high returns (e.g., all NVDA) comes with high volatility.
- **Markowitz optimization:** Visualizations help identify portfolios that best match your risk tolerance and investment goals.
