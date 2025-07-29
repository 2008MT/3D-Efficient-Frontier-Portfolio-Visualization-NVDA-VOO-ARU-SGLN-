import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
from matplotlib.colors import Normalize
from matplotlib.cm import ScalarMappable
import warnings
from datetime import datetime, timedelta
import yfinance as yf

warnings.filterwarnings('ignore')

class EfficientSurface3D:
    def __init__(self, tickers, start_date=None, end_date=None):
        self.tickers = tickers
        self.start_date = start_date or (datetime.now() - timedelta(days=10*365)).strftime('%Y-%m-%d')
        self.end_date = end_date or datetime.now().strftime('%Y-%m-%d')
        self.data = None
        self.returns = None
        self.mean_returns = None
        self.cov_matrix = None
        
    def fetch_data(self):
        print("Fetching data from Yahoo Finance...")
        ticker_variants = {
            'NVDA': ['NVDA'],
            'VOO': ['VOO', 'VUAA.L', 'VUAA'],
            'ARU': ['ARU.AX', 'ARU'],
            'SGLN': ['SGLN.L', 'IAU', 'GLD']
        }
        final_tickers = []
        for original_ticker in self.tickers:
            success = False
            for variant in ticker_variants.get(original_ticker, [original_ticker]):
                try:
                    test_data = yf.download(variant, start=self.start_date, end=self.end_date, progress=False)
                    if not test_data.empty and len(test_data) > 100:
                        final_tickers.append(variant)
                        print(f"  ✓ {original_ticker} -> {variant}: {len(test_data)} days")
                        success = True
                        break
                except Exception:
                    continue
            if not success:
                raise ValueError(f"Could not fetch data for ticker {original_ticker}.")
        data = yf.download(final_tickers, start=self.start_date, end=self.end_date, progress=False)
        if len(final_tickers) == 1:
            self.data = pd.DataFrame(data['Close'])
            self.data.columns = final_tickers
        else:
            self.data = data['Close']
        column_mapping = dict(zip(final_tickers, self.tickers))
        self.data = self.data.rename(columns=column_mapping)
        self.data = self.data[self.tickers]
        self.data = self.data.dropna()
        if len(self.data) < 100:
            raise ValueError("Insufficient data points returned.")
        print(f"Data fetched: {len(self.data)} trading days")
        self.returns = self.data.pct_change().dropna()
        self.mean_returns = self.returns.mean() * 252
        self.cov_matrix = self.returns.cov() * 252
        print("\nAsset Statistics (Annualized):")
        print("-" * 40)
        for ticker in self.tickers:
            ret = self.mean_returns[ticker]
            vol = np.sqrt(self.cov_matrix.loc[ticker, ticker])
            print(f"{ticker:>6}: Return {ret:>7.1%}, Volatility {vol:>6.1%}")

    def portfolio_metrics(self, weights):
        portfolio_return = np.sum(weights * self.mean_returns)
        portfolio_volatility = np.sqrt(np.dot(weights.T, np.dot(self.cov_matrix, weights)))
        return portfolio_return, portfolio_volatility

    def generate_grid_surface(self, n_points=50):  # Reduced for better performance
        print(f"\nGenerating efficient surface data ({n_points}³ combinations)...")
        weight_range = np.linspace(0, 1, n_points)
        nvda_w, voo_w, aru_w = np.meshgrid(weight_range, weight_range, weight_range)
        nvda_w = nvda_w.flatten()
        voo_w = voo_w.flatten()
        aru_w = aru_w.flatten()
        sgln_w = 1 - nvda_w - voo_w - aru_w
        valid = (sgln_w >= 0) & (sgln_w <= 1)
        nvda_w = nvda_w[valid]
        voo_w = voo_w[valid]
        aru_w = aru_w[valid]
        sgln_w = sgln_w[valid]
        weights = np.vstack([nvda_w, voo_w, aru_w, sgln_w]).T
        returns = []
        volatilities = []
        sharpes = []
        for w in weights:
            ret, vol = self.portfolio_metrics(w)
            returns.append(ret)
            volatilities.append(vol)
            sharpes.append(ret/vol if vol > 0 else -999)
        returns = np.array(returns)
        volatilities = np.array(volatilities)
        sharpes = np.array(sharpes)
        print(f"Generated {len(returns)} valid portfolios from {n_points ** 3} combinations")
        print(f"Efficiency: {len(returns) / (n_points ** 3):.1%}")
        return nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes

    def plot_risk_return_surface(self, nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes, figsize=(15, 5)):
        """Create multiple meaningful 3D visualizations"""
        print("\nCreating comprehensive 3D portfolio visualizations...")
        
        fig = plt.figure(figsize=figsize)
        
        # Plot 1: Risk-Return-Sharpe 3D scatter
        ax1 = fig.add_subplot(131, projection='3d')
        scatter = ax1.scatter(volatilities, returns, sharpes, 
                             c=sharpes, cmap='viridis', alpha=0.6, s=20)
        ax1.set_xlabel('Portfolio Volatility (Risk)')
        ax1.set_ylabel('Portfolio Return')
        ax1.set_zlabel('Sharpe Ratio')
        ax1.set_title('Risk-Return-Sharpe\n3D Relationship')
        cbar1 = plt.colorbar(scatter, ax=ax1, shrink=0.5, pad=0.15, label='Sharpe Ratio')
        
        # Plot 2: Weight composition with Sharpe coloring
        ax2 = fig.add_subplot(132, projection='3d')
        scatter2 = ax2.scatter(nvda_w, voo_w, aru_w, 
                              c=sharpes, cmap='plasma', alpha=0.7, s=25)
        ax2.set_xlabel('NVDA Weight')
        ax2.set_ylabel('VOO Weight')
        ax2.set_zlabel('ARU Weight')
        ax2.set_title('Asset Allocation Space\n(Color = Sharpe Ratio)')
        cbar2 = plt.colorbar(scatter2, ax=ax2, shrink=0.5, pad=0.15, label='Sharpe Ratio')
        
        # Plot 3: Risk-Return surface colored by dominant asset
        ax3 = fig.add_subplot(133, projection='3d')
        
        # Determine dominant asset for each portfolio
        weights_matrix = np.vstack([nvda_w, voo_w, aru_w, sgln_w]).T
        dominant_asset = np.argmax(weights_matrix, axis=1)
        colors = ['red', 'blue', 'green', 'gold']  # NVDA, VOO, ARU, SGLN
        asset_names = ['NVDA', 'VOO', 'ARU', 'SGLN']
        
        for i, (color, name) in enumerate(zip(colors, asset_names)):
            mask = dominant_asset == i
            if np.any(mask):
                ax3.scatter(volatilities[mask], returns[mask], sharpes[mask],
                           c=color, alpha=0.6, s=20, label=f'{name} Dominant')
        
        ax3.set_xlabel('Portfolio Volatility')
        ax3.set_ylabel('Portfolio Return')
        ax3.set_zlabel('Sharpe Ratio')
        ax3.set_title('Risk-Return by\nDominant Asset')
        ax3.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
        
        plt.tight_layout()
        return fig, (ax1, ax2, ax3)

    def plot_efficient_frontier_2d(self, returns, volatilities, sharpes, figsize=(12, 5)):
        """Traditional 2D efficient frontier with enhancements"""
        print("Creating enhanced 2D efficient frontier...")
        
        fig, (ax1, ax2) = plt.subplots(1, 2, figsize=figsize)
        
        # Plot 1: Traditional efficient frontier
        scatter1 = ax1.scatter(volatilities, returns, c=sharpes, 
                              cmap='viridis', alpha=0.7, s=30)
        ax1.set_xlabel('Portfolio Volatility (Risk)')
        ax1.set_ylabel('Portfolio Return')
        ax1.set_title('Efficient Frontier\n(Color = Sharpe Ratio)')
        ax1.grid(True, alpha=0.3)
        plt.colorbar(scatter1, ax=ax1, label='Sharpe Ratio')
        
        # Highlight efficient frontier edge
        # Sort by volatility and find upper envelope
        sorted_idx = np.argsort(volatilities)
        vol_sorted = volatilities[sorted_idx]
        ret_sorted = returns[sorted_idx]
        
        # Find efficient frontier (upper envelope)
        efficient_points = []
        max_return_so_far = -np.inf
        for i, (vol, ret) in enumerate(zip(vol_sorted, ret_sorted)):
            if ret > max_return_so_far:
                efficient_points.append((vol, ret))
                max_return_so_far = ret
        
        if efficient_points:
            efficient_vol, efficient_ret = zip(*efficient_points)
            ax1.plot(efficient_vol, efficient_ret, 'r-', linewidth=2, 
                    alpha=0.8, label='Efficient Frontier')
            ax1.legend()
        
        # Plot 2: Sharpe ratio optimization
        scatter2 = ax2.scatter(volatilities, sharpes, c=returns, 
                              cmap='plasma', alpha=0.7, s=30)
        ax2.set_xlabel('Portfolio Volatility')
        ax2.set_ylabel('Sharpe Ratio')
        ax2.set_title('Risk vs Sharpe Ratio\n(Color = Return)')
        ax2.grid(True, alpha=0.3)
        plt.colorbar(scatter2, ax=ax2, label='Portfolio Return')
        
        # Highlight maximum Sharpe ratio
        max_sharpe_idx = np.argmax(sharpes)
        ax2.scatter(volatilities[max_sharpe_idx], sharpes[max_sharpe_idx], 
                   c='red', s=100, marker='*', label='Max Sharpe', zorder=5)
        ax2.legend()
        
        plt.tight_layout()
        return fig, (ax1, ax2)

    def find_optimal_portfolios(self, nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes):
        df = pd.DataFrame({
            'nvda_weight': nvda_w,
            'voo_weight': voo_w,
            'aru_weight': aru_w,
            'sgln_weight': sgln_w,
            'return': returns,
            'volatility': volatilities,
            'sharpe': sharpes
        })
        
        # Find optimal portfolios
        min_vol_idx = df['volatility'].idxmin()
        max_return_idx = df['return'].idxmax()
        max_sharpe_idx = df['sharpe'].idxmax()
        
        # Find efficient frontier points at different risk levels
        vol_percentiles = [10, 30, 50, 70, 90]
        efficient_portfolios = {}
        
        for p in vol_percentiles:
            vol_target = np.percentile(volatilities, p)
            # Find portfolio with highest return at this risk level
            nearby_vol = df[abs(df['volatility'] - vol_target) <= 0.01]
            if len(nearby_vol) > 0:
                best_idx = nearby_vol['return'].idxmax()
                efficient_portfolios[f'{p}th Percentile Risk'] = df.iloc[best_idx]
        
        optimal_portfolios = {
            'Minimum Volatility': df.iloc[min_vol_idx],
            'Maximum Return': df.iloc[max_return_idx],
            'Maximum Sharpe Ratio': df.iloc[max_sharpe_idx],
            **efficient_portfolios
        }
        
        print("\n" + "="*70)
        print("OPTIMAL PORTFOLIO ANALYSIS")
        print("="*70)
        
        for name, portfolio in optimal_portfolios.items():
            print(f"\n{name}:")
            print(f"  NVDA: {portfolio['nvda_weight']:>6.1%}  |  VOO: {portfolio['voo_weight']:>6.1%}")
            print(f"  ARU:  {portfolio['aru_weight']:>6.1%}  |  SGLN: {portfolio['sgln_weight']:>6.1%}")
            print(f"  Expected Return: {portfolio['return']:>6.2%}  |  Volatility: {portfolio['volatility']:>6.2%}")
            print(f"  Sharpe Ratio: {portfolio['sharpe']:>6.3f}")
            
        return optimal_portfolios

def main():
    tickers = ['NVDA', 'VOO', 'ARU', 'SGLN']
    print("="*70)
    print("IMPROVED 3D EFFICIENT SURFACE PORTFOLIO ANALYSIS")
    print("Modern Portfolio Theory Visualization")
    print("="*70)
    
    analyzer = EfficientSurface3D(tickers)
    try:
        analyzer.fetch_data()
        nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes = analyzer.generate_grid_surface(n_points=50)
        
        print("\nVisualization Statistics:")
        print(f"Return range: {returns.min():.1%} to {returns.max():.1%}")
        print(f"Volatility range: {volatilities.min():.1%} to {volatilities.max():.1%}")
        print(f"Sharpe range: {sharpes.min():.3f} to {sharpes.max():.3f}")
        
        # Find optimal portfolios
        optimal_portfolios = analyzer.find_optimal_portfolios(nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes)
        
        # Create visualizations
        fig1, axes1 = analyzer.plot_risk_return_surface(nvda_w, voo_w, aru_w, sgln_w, returns, volatilities, sharpes)
        fig2, axes2 = analyzer.plot_efficient_frontier_2d(returns, volatilities, sharpes)
        
        plt.show()
        
        # Save figures
        try:
            fig1.savefig('portfolio_3d_analysis.png', dpi=300, bbox_inches='tight')
            fig2.savefig('efficient_frontier_2d.png', dpi=300, bbox_inches='tight')
            print(f"\n✓ Visualizations saved as 'portfolio_3d_analysis.png' and 'efficient_frontier_2d.png'")
        except Exception as e:
            print(f"Could not save figures: {e}")
            
        print("\n" + "="*70)
        print("ANALYSIS COMPLETE")
        print("Key Insights:")
        print("- 3D visualizations show risk-return relationships more clearly")
        print("- Asset allocation patterns are visible in weight space")
        print("- Efficient frontier is properly highlighted in 2D")
        print("- Multiple optimization criteria are compared")
        print("="*70)
        
    except Exception as e:
        print(f"\nError in analysis: {e}")
        import traceback
        traceback.print_exc()

if __name__ == "__main__":
    main()
