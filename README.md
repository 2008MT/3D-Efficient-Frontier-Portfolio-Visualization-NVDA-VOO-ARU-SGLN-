# 3D Portfolio Optimization and Efficient Surface Analysis

Hi! I'm Mael, a 17-year-old incoming high school senior from the South of France, passionate about finance, quantitative analysis, investing, and the intersection of technology and markets. I intend to pursue studies and a career in hedge funds in the United States. My experiences include a work experience placement at Morgan Stanley, a two-week research placement at the University of Montpellier exploring quantitative and computational finance tools, co-founding and hosting two podcasts, award-winning public speaking and debate achievements, and being an ardent golfer. This coding project represents my exploration into quantitative finance using Python—as I continue learning programming, I want to be transparent that I utilized AI tools including Claude, GitHub Copilot, and ChatGPT to assist in development, helping accelerate my learning while I build coding fundamentals.

## Purpose and Motivation

This project demonstrates advanced 3D visualization techniques for Modern Portfolio Theory (MPT) optimization, addressing a common problem in portfolio analysis: **how to meaningfully visualize multi-dimensional portfolio optimization in 3D space**.

Traditional portfolio optimization often relies on 2D efficient frontier plots, but these can obscure important relationships between asset allocations, risk, return, and optimization metrics. This project explores multiple 3D visualization approaches to provide deeper insights into portfolio construction and the geometric properties of the efficient frontier.

**Key Research Questions:**
- How can we effectively visualize 4+ asset portfolios in 3D space?
- What insights are revealed when we separate risk-return relationships from asset allocation patterns?
- How do different visualization approaches highlight different aspects of portfolio optimization?

## Theoretical Foundation

### Modern Portfolio Theory (Markowitz Framework)

This analysis is built upon Harry Markowitz's foundational Modern Portfolio Theory (1952), which established the mathematical foundation for portfolio optimization. Modern portfolio theory (MPT), or mean-variance analysis, is a mathematical framework for assembling a portfolio of assets such that the expected return is maximized for a given level of risk. It is a formalization and extension of diversification in investing, the idea that owning different kinds of financial assets is less risky than owning only one type. Its key insight is that an asset's risk and return should not be assessed by itself, but by how it contributes to a portfolio's overall risk and return.

#### Core Principles

**1. Expected Portfolio Return:**

$$
E[R_p] = \sum_{i=1}^N w_i \cdot E[R_i]
$$

Where:
- $w_i$ : Weight of asset $i$
- $E[R_i]$ : Expected (annualized) return of asset $i$
- $N$ : Number of assets

**2. Portfolio Variance (Risk):**

$$
\sigma_p = \sqrt{ \sum_{i=1}^n \sum_{j=1}^n w_iw_jCov(R_i; R_j) }
= \sqrt{ \mathbf{w}^T \Sigma \mathbf{w} }
$$

Where:
- $\mathbf{w}$ : Vector of asset weights
- $\Sigma$ : Covariance matrix of asset returns

**3. Sharpe Ratio (Risk-Adjusted Return):**

$$
Sharpe = \frac{E[R_i]-R_f}{\sigma_p}
$$

Where:
- $R_f$ = Risk-free rate (assumed to be 0 in this analysis)

#### The Efficient Frontier

The efficient frontier represents the set of optimal portfolios offering the highest expected return for each level of risk. Mathematically, this is a constrained optimization problem:

**Minimize:** $\sigma_p$ (portfolio standard deviation)
**Subject to:**
- $\Sigma w_i = 1$ (weights sum to 1)
- $w_i ≥ 0$ (no short selling)
- $E[R_p]=$ target return (for each target return level)

#### Multi-Dimensional Challenge

With 4 assets, we have a 4-dimensional optimization space constrained to a 3-dimensional simplex (due to the weight constraint). This creates visualization challenges that this project addresses through multiple complementary 3D representations.

## Methodology

### Asset Selection Strategy

Our portfolio consists of four strategically selected assets:

- **Nvidia (NVDA):** A leading technology company in AI and semiconductors, offering high growth potential and significant volatility. NVDA is chosen for its representation of innovation and tech sector exposure.
- **S&P 500 Vanguard ETF (VOO/VUAA):** This ETF tracks the S&P 500 index, providing broad exposure to the US equity market with relatively lower volatility. It supplies stability and diversification.
- **Arafura Rare Earths Ltd (ARU):** An Australian mining company focused on rare earths, ARU adds sector-specific risk and the potential for outsized returns due to its exposure to critical minerals needed for renewable energy and tech.
- **iShares Physical Gold ETF (SGLN):** Gold is a classic defensive asset. SGLN brings stability and acts as a hedge against market downturns, with typically low correlation to equities.

**Why this mix?**  
Combining these assets allows us to study the trade-off between risk and return, diversification benefits, and how different sectors and asset classes interact in portfolio construction. This combination provides exposure to growth, value, commodities, and defensive assets - ideal for demonstrating diversification effects.

### Data Processing Pipeline

#### 1. Data Acquisition
```python
def fetch_data(self):
    # Robust ticker resolution with fallbacks
    ticker_variants = {
        'NVDA': ['NVDA'],
        'VOO': ['VOO', 'VUAA.L', 'VUAA'],
        'ARU': ['ARU.AX', 'ARU'],
        'SGLN': ['SGLN.L', 'IAU', 'GLD']
    }
    # Fetches 10 years of historical data
    # Validates data quality (>100 trading days minimum)
```

#### 2. Statistical Computation
```python
def portfolio_metrics(self, weights):
    # Annualized expected return
    portfolio_return = np.sum(weights * self.mean_returns)
    # Portfolio volatility using covariance matrix
    portfolio_volatility = np.sqrt(np.dot(weights.T, np.dot(self.cov_matrix, weights)))
    return portfolio_return, portfolio_volatility
```

#### 3. Optimization Space Generation
```python
def generate_grid_surface(self, n_points=50):
    # Creates n³ weight combinations for first 3 assets
    # 4th asset weight = 1 - sum(other weights)
    # Filters for valid portfolios (all weights ≥ 0, ≤ 1)
    # Computes return, volatility, Sharpe ratio for each
```

## Key Implementation Features

### Multi-Perspective Visualization Architecture

The solution provides **five complementary visualizations** rather than forcing all information into a single 3D plot:

#### 1. Risk-Return-Sharpe 3D Relationship
```python
ax1.scatter(volatilities, returns, sharpes, c=sharpes, cmap='viridis')
ax1.set_xlabel('Portfolio Volatility (Risk)')
ax1.set_ylabel('Portfolio Return')  
ax1.set_zlabel('Sharpe Ratio')
```
**Purpose:** Shows the fundamental MPT relationship between risk, return, and efficiency.

#### 2. Asset Allocation Space Analysis
```python
ax2.scatter(nvda_w, voo_w, aru_w, c=sharpes, cmap='plasma')
ax2.set_xlabel('NVDA Weight')
ax2.set_ylabel('VOO Weight')
ax2.set_zlabel('ARU Weight')
```
**Purpose:** Visualizes how different weight combinations affect portfolio efficiency.

#### 3. Dominant Asset Pattern Recognition
```python
weights_matrix = np.vstack([nvda_w, voo_w, aru_w, sgln_w]).T
dominant_asset = np.argmax(weights_matrix, axis=1)
# Color-codes portfolios by their dominant holding
```
**Purpose:** Reveals which assets drive performance in different regions of the efficient frontier.

#### 4. Enhanced Efficient Frontier (2D)
```python
# Finds upper envelope of risk-return combinations
efficient_points = []
max_return_so_far = -np.inf
for vol, ret in zip(vol_sorted, ret_sorted):
    if ret > max_return_so_far:
        efficient_points.append((vol, ret))
        max_return_so_far = ret
```
**Purpose:** Traditional efficient frontier with proper mathematical derivation.

#### 5. Risk-Sharpe Optimization Surface
```python
ax2.scatter(volatilities, sharpes, c=returns, cmap='plasma')
# Highlights maximum Sharpe ratio portfolio
max_sharpe_idx = np.argmax(sharpes)
ax2.scatter(volatilities[max_sharpe_idx], sharpes[max_sharpe_idx], 
           c='red', s=100, marker='*')
```
**Purpose:** Shows the risk-efficiency tradeoff independent of return levels.

### Advanced Portfolio Analysis

#### Optimal Portfolio Detection
```python
def find_optimal_portfolios(self):
    # Minimum Volatility Portfolio
    min_vol_idx = df['volatility'].idxmin()
    # Maximum Return Portfolio  
    max_return_idx = df['return'].idxmax()
    # Maximum Sharpe Ratio Portfolio
    max_sharpe_idx = df['sharpe'].idxmax()
    # Efficient portfolios at different risk percentiles
    for p in [10, 30, 50, 70, 90]:
        vol_target = np.percentile(volatilities, p)
        # Find highest return at each risk level
```

## Results and Analysis

### Asset Performance Statistics (Annualized)

| Asset | Expected Return | Volatility | Sharpe Ratio |
|-------|----------------|------------|--------------|
| **NVDA** | 73.6% | 50.5% | 1.456 |
| **VOO** | 15.0% | 18.6% | 0.806 |
| **ARU** | 47.9% | 82.8% | 0.578 |
| **SGLN** | 14.0% | 15.1% | 0.927 |

---

### Portfolio Universe Generated: 
22,090 valid portfolios from 125,000 combinations (17.7% efficiency)

- Return Range: 14.0% to 73.6%
- Volatility Range: 11.1% to 82.8%
- Sharpe Range: 0.579 to 1.865

---

### Risk Scaling Patterns

| Risk Level | NVDA | VOO | ARU | SGLN | Return | Volatility | Sharpe |
|------------|------|-----|-----|------|--------|------------|--------|
| 10th %ile | 26.5% | 6.1% | 6.1% | 61.2% | 31.96% | 17.22% | 1.856 |
| 30th %ile | 42.9% | 2.0% | 8.2% | 46.9% | 42.34% | 23.70% | 1.786 |
| 50th %ile | 55.1% | 0.0% | 12.2% | 32.7% | 51.00% | 29.96% | 1.702 |
| 70th %ile | 67.3% | 0.0% | 16.3% | 16.3% | 59.69% | 36.86% | 1.619 |
| 90th %ile | 93.9% | 0.0% | 6.1% | 0.0% | 72.04% | 47.85% | 1.506 |

**Pattern:** As risk tolerance increases, NVDA allocation grows while SGLN decreases, maintaining optimal risk-adjusted returns. We notice a relatively important decrease in sharpe ratio from the 50th %ile.

---

### Data Visualisations Analysis

#### 1. Risk-Return-Sharpe 3D Relationship

<img width="280" height="243" alt="Screenshot 2025-07-29 at 10 27 43" src="https://github.com/user-attachments/assets/1423ea2e-56b5-4bca-bdf0-8999bdbcd02e" />

This fundamental visualization maps the core Modern Portfolio Theory relationship in three dimensions:

- X-axis: Portfolio Volatility (Risk)
- Y-axis: Portfolio Return
- Z-axis: Sharpe Ratio
- Color: Sharpe Ratio intensity

#### Key Interpretations:

- The curved surface represents the feasible portfolio space, showing how risk and return combine to produce different efficiency levels
- Peak efficiency region (bright yellow/green) occurs at moderate risk levels (~17-30% volatility) with Sharpe ratios exceeding 1.6
- Diminishing returns are clearly visible: beyond 40% volatility, Sharpe ratios decline despite higher returns
- The 3D curvature reveals that the traditional 2D efficient frontier is actually a projection of this more complex surface

#### Curated Result Analysis:
The optimal risk-return combination doesn't occur at maximum return but rather where the Sharpe ratio peaks, demonstrating the importance of risk-adjusted thinking.

#### 2. Asset Allocation Space

<img width="268" height="230" alt="Screenshot 2025-07-29 at 10 28 05" src="https://github.com/user-attachments/assets/69074523-21e2-4575-a74e-410fcfb00b6c" />

This visualization maps portfolio weights in 3D space:

- X-axis: NVDA Weight (0-100%)
- Y-axis: VOO Weight (0-100%)
- Z-axis: ARU Weight (0-100%)
- Color: Sharpe Ratio (SGLN weight = 1 - others)

#### Key Interpretations:

- High-efficiency zone (yellow region) concentrates in the low-VOO, moderate-NVDA space
- The constraint surface forms a triangular shape due to the weight sum constraint
- VOO-heavy allocations (high Y-axis) consistently show poor Sharpe ratios (purple/dark colors)
- NVDA-ARU combinations without VOO dominate the efficient region

#### Curated Result Analysis:
This confirms that VOO adds little value to this particular asset mix, being crowded out by more efficient combinations of growth (NVDA) and defensive (SGLN) assets.

#### 3. Risk-Return by Dominant Asset

<img width="409" height="311" alt="Screenshot 2025-07-28 at 17 44 01" src="https://github.com/user-attachments/assets/3dec7497-dc55-46c1-a7d8-ab59645405ee" />

This plot reveals which asset drives performance in different portfolio regions:

Same axes as Plot 1 but colored by dominant holding
- Red: NVDA-dominant portfolios
- Blue: VOO-dominant portfolios
- Green: ARU-dominant portfolios
- Gold: SGLN-dominant portfolios

#### Key Interpretations:

- SGLN dominance (gold) clusters in the low-risk, moderate-return region (defensive zone)
- NVDA dominance (red) occupies the high-risk, very-high-return space (growth zone)
- ARU dominance (green) occupies the high-risk, high-return space (moderate growth zone)
- VOO (blue) barely appears in this mix, surprising as it is usually considered as an important diversifier
- Clear bifurcation between defensive (SGLN) and growth (NVDA) strategies

#### Curated Result Analysis:
The optimal approach is a "barbell strategy" - combine defensive assets (SGLN) for stability with high-growth assets (NVDA) for returns, avoiding the "middle ground" (VOO) assets.

#### 4. Enhanced Efficient Frontier

<img width="445" height="370" alt="Screenshot 2025-07-28 at 17 53 12" src="https://github.com/user-attachments/assets/31cf37b2-89e7-4f69-9d83-ba84c1e00eb3" />

The traditional 2D efficient frontier enhanced with mathematical rigor:

- Scatter points: All feasible portfolios colored by Sharpe ratio
- Red line: Mathematically derived efficient frontier (upper envelope)
- Color gradient: Sharpe ratio from low (purple) to high (yellow)

#### Key Interpretations:

- Classic hyperbolic shape confirms theoretical expectations
- Efficient frontier clearly separates optimal from suboptimal portfolios
- High Sharpe region (yellow/green) forms a narrow band along the frontier
- Dominated portfolios (purple region) show inferior risk-return combinations

#### Validation: 
This plot validates our 3D analysis by showing the traditional efficient frontier, confirming that our multi-dimensional approach captures the same mathematical relationships.

#### 5. Risk vs Sharpe Ratio Analysis

<img width="418" height="358" alt="Screenshot 2025-07-28 at 17 55 17" src="https://github.com/user-attachments/assets/c2bd7344-2c0e-47f8-8870-949a91c85d1a" />

This unique perspective shows the risk-efficiency tradeoff:

- X-axis: Portfolio Volatility
- Y-axis: Sharpe Ratio
- Color: Portfolio Return
- Red star: Maximum Sharpe ratio portfolio

#### Key Interpretations:

- Inverted relationship: Higher risk generally leads to lower Sharpe ratios beyond the optimal point
- Sweet spot at ~17-18% volatility where Sharpe ratio peaks at 1.865
- Return gradient (color) shows that moderate-return portfolios can be more efficient than high-return ones
- Sharp decline in efficiency beyond 30% volatility

#### Risk Management Insight: 
This plot is crucial for risk management, showing that accepting higher volatility doesn't necessarily improve risk-adjusted returns.

---

### Other Key Findings

#### 1. **Optimal Portfolio Composition**

**Maximum Sharpe Ratio Portfolio (1.865):**
- NVDA: 28.6% (controlled high-growth exposure)
- SGLN: 65.3% (heavy defensive weighting)
- ARU: 6.1% (minimal commodity exposure)
- VOO: 0.0% (completely excluded!)

**Interpretation:** The optimization reveals a "barbell strategy" combining high-growth assets with defensive holdings, rather than traditional balanced allocation.

#### 2. **Surprising VOO Exclusion**

Despite being a broad market ETF, VOO receives zero allocation in the optimal portfolio. This occurs because:
- SGLN provides superior diversification benefits during this time period
- NVDA offers better risk-adjusted returns for growth exposure
- The correlation structure makes VOO redundant

#### 3. **ARU Inefficiency**

ARU consistently receives minimal allocations despite decent returns (47.9%) due to:
- Extremely high volatility (82.8%)
- Poor risk-adjusted performance (Sharpe: 0.578)
- Better alternatives available in both growth (NVDA) and defensive (SGLN) categories

## Conclusion

This comprehensive 3D portfolio optimization analysis has successfully addressed the fundamental challenges of visualizing multi-dimensional Modern Portfolio Theory relationships, providing insights into asset allocation strategies and revealing the geometric properties of efficient portfolios in ways that traditional 2D approaches cannot capture. We demonstrate that effective 4+ asset portfolio visualization requires a multi-perspective approach rather than attempting to compress all information into a single 3D representation. The solution lies in creating complementary visualizations that each highlight different aspects of the optimization problem: risk-return-Sharpe relationships in natural 3D space to show the fundamental MPT trade-offs, asset allocation weight space to reveal optimal combination patterns, dominant asset analysis to understand which holdings drive performance in different regions, and enhanced 2D projections that maintain mathematical rigor while providing familiar reference points. This approach overcomes the inherent limitation that a 4-asset portfolio optimization problem exists in a 4-dimensional space constrained to a 3-dimensional simplex, which cannot be meaningfully represented in a single 3D visualization without losing critical information.

The separation of risk-return analysis from weight allocation patterns has revealed several counterintuitive findings that challenge conventional portfolio wisdom. Most notably, despite being a broad market ETF traditionally considered a core holding, VOO receives zero allocation in optimal portfolios across all risk levels because SGLN provides superior diversification benefits while NVDA offers better risk-adjusted growth exposure. Rather than the expected balanced allocation across all assets, the optimization consistently produces a "barbell" strategy combining high-growth exposure (NVDA) with defensive positioning (SGLN), while minimizing allocation to low-Sharpe assets (VOO, ARU). This emergent pattern suggests that in certain market conditions, concentrated exposure to complementary extremes may be more efficient than traditional diversification. The analysis reveals that as risk tolerance increases, the optimal strategy involves increasing NVDA allocation while decreasing SGLN, maintaining superior risk-adjusted returns (Sharpe ratios above 1.5) across the entire risk spectrum, demonstrating that the efficient frontier is not just about risk-return trade-offs, but about optimal asset combination at each risk level.

Each visualization technique reveals distinct optimization insights that collectively provide a comprehensive understanding of portfolio construction. The 3D Risk-Return-Sharpe surface reveals the non-linear relationship between risk and efficiency, showing that maximum Sharpe ratios occur at moderate risk levels (~18% volatility) rather than at return extremes, challenging the common misconception that higher returns always justify higher risks. The asset allocation weight space demonstrates that optimal combinations cluster in specific regions of weight space, with high-efficiency portfolios avoiding balanced allocations in favor of concentrated positions in complementary assets. The dominant asset analysis shows clear bifurcation between growth-dominated and defensive-dominated strategies, with minimal presence of "compromise" assets, supporting the "barbell" strategy finding.

The methodological framework starts from Markowitz's fundamental equations to generate 22,090 valid portfolio combinations from 125,000 (50³) possible allocations with 17.7% computational efficiency. The technical implementation leverages Python's scientific computing ecosystem with key innovations including robust data pipeline with automatic ticker resolution, efficient grid generation using vectorized operations, multi-perspective rendering maintaining mathematical consistency, and statistical validation ensuring MPT theoretical compliance. The strategic asset selection (NVDA representing high-growth technology, VOO as broad market exposure, ARU as commodity volatility, and SGLN as defensive positioning) was designed to represent distinct risk-return profiles, with the surprising finding that SGLN's heavy weighting (up to 65.3%) in optimal portfolios reveals the critical importance of low-correlation assets in portfolio construction, while ARU's minimal allocation despite attractive returns demonstrates how extreme volatility can make assets inefficient.

This analysis provides empirical validation of Markowitz's theoretical framework while extending its practical application through visualization, with the mathematically derived efficient frontier matching theoretical expectations and portfolio metrics conforming to expected statistical relationships. The findings have significant implications for portfolio management practice, suggesting that concentrated positions in complementary assets may outperform traditional balanced approaches, particularly in markets with clear growth leaders and defensive alternatives. The emergence of the barbell strategy—combining high-growth technology exposure with defensive gold positioning while avoiding traditional broad market exposure—represents a significant practical finding that reflects the evolving nature of optimal portfolio construction in contemporary markets.
