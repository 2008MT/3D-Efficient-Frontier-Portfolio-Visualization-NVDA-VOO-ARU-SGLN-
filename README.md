# Efficient Frontier and Efficient Surface: A Deep Dive into 4-Asset Portfolio Optimization

## 1. The Assets: Choices and Rationale

Our portfolio explores four complementary assets:

- **Nvidia (NVDA):** A leading semiconductor and AI company with high historical returns and volatility. Represents growth and tech innovation.
- **S&P 500 Vanguard ETF (VOO/VUAA):** Broad market exposure, offering stability and diversification within US large caps.
- **Arafura Rare Earths Ltd (ARU):** An Australian rare earths miner, highly volatile and growth-oriented, provides exposure to critical minerals and the green transition.
- **iShares Physical Gold ETF (SGLN):** Gold is a classic defensive asset, acting as a hedge during market stress, with low correlation to equities.

**Interest in this combination:**  
By mixing tech (NVDA), broad market (VOO), critical minerals (ARU), and gold (SGLN), we aim to balance return potential, diversification, and risk. Each asset plays a unique role in the portfolio’s risk-return landscape.

---

## 2. Methodology: Data, Theory, and Modeling

### Data Collection

We fetch daily closing prices for each ticker over ~10 years using [Yahoo Finance](https://finance.yahoo.com/). We use alternative tickers for international listings to ensure data coverage.

```{r}
try:
    import yfinance as yf
except ImportError:
    raise ImportError("yfinance is required to fetch real-world data. Please install it using: pip install yfinance")

warnings.filterwarnings('ignore')

class EfficientSurface3D:
    def __init__(self, tickers, start_date=None, end_date=None):
        """
        Initialize the 3D Efficient Surface visualization
        
        Parameters:
        tickers (list): List of ticker symbols
        start_date (str): Start date for data retrieval (YYYY-MM-DD)
        end_date (str): End date for data retrieval (YYYY-MM-DD)
        """
        self.tickers = tickers
        self.start_date = start_date or (datetime.now() - timedelta(days=10*365)).strftime('%Y-%m-%d')
        self.end_date = end_date or datetime.now().strftime('%Y-%m-%d')
        self.data = None
        self.returns = None
        self.mean_returns = None
        self.cov_matrix = None
        
    def fetch_data_yfinance(self):
        """Fetch data using yfinance (with error handling)"""
        print("Attempting to fetch data from Yahoo Finance...")
        
        # Try different ticker formats for international stocks
        ticker_variants = {
            'NVDA': ['NVDA'],
            'VOO': ['VOO', 'VUAA.L', 'VUAA'],  # Try VOO first, then alternatives
            'ARU': ['ARU.AX', 'ARU'],
            'SGLN': ['SGLN.L', 'IAU', 'GLD']  # Try London listing, then alternatives
        }
        
        final_tickers = []
        for original_ticker in self.tickers:
            success = False
            for variant in ticker_variants.get(original_ticker, [original_ticker]):
                try:
                    test_data = yf.download(variant, start=self.start_date, end=self.end_date, progress=False)
                    if not test_data.empty and len(test_data) > 100:  # Ensure we have sufficient data
                        final_tickers.append(variant)
                        print(f"  ✓ {original_ticker} -> {variant}: {len(test_data)} days")
                        success = True
                        break
                except Exception:
                    continue
            
            if not success:
                raise ValueError(f"Could not fetch data for ticker {original_ticker}. Please check the ticker symbol or availability.")
        
        # Download all data at once
        data = yf.download(final_tickers, start=self.start_date, end=self.end_date, progress=False)
        
        if len(final_tickers) == 1:
            self.data = pd.DataFrame(data['Close'])
            self.data.columns = final_tickers
        else:
            self.data = data['Close']
            
        # Rename columns to original tickers for consistency
        column_mapping = dict(zip(final_tickers, self.tickers))
        self.data = self.data.rename(columns=column_mapping)
        self.data = self.data[self.tickers]  # <-- Ensure column order matches tickers
        
        # Clean data
        self.data = self.data.dropna()
        
        if len(self.data) < 100:
            raise ValueError("Insufficient data points returned. Try a wider date range or check ticker validity.")
            
        print(f"Successfully fetched data: {len(self.data)} trading days")
        return True
```

### Portfolio Construction

- **Weights:** All combinations where weights sum to 1 and each weight is between 0 and 1.
- **Returns:** Annualized mean daily returns.
- **Volatility:** Annualized standard deviation of daily returns.
- **Covariance matrix:** Captures how returns move together, crucial for risk calculation.

---

## 3. How It Works (Python)


### Data Collection

```{r}
try:
    import yfinance as yf
except ImportError:
    raise ImportError("yfinance is required to fetch real-world data. Please install it using: pip install yfinance")

warnings.filterwarnings('ignore')

class EfficientSurface3D:
    def __init__(self, tickers, start_date=None, end_date=None):
        """
        Initialize the 3D Efficient Surface visualization
        
        Parameters:
        tickers (list): List of ticker symbols
        start_date (str): Start date for data retrieval (YYYY-MM-DD)
        end_date (str): End date for data retrieval (YYYY-MM-DD)
        """
        self.tickers = tickers
        self.start_date = start_date or (datetime.now() - timedelta(days=10*365)).strftime('%Y-%m-%d')
        self.end_date = end_date or datetime.now().strftime('%Y-%m-%d')
        self.data = None
        self.returns = None
        self.mean_returns = None
        self.cov_matrix = None
        
    def fetch_data_yfinance(self):
        """Fetch data using yfinance (with error handling)"""
        print("Attempting to fetch data from Yahoo Finance...")
        
        # Try different ticker formats for international stocks
        ticker_variants = {
            'NVDA': ['NVDA'],
            'VOO': ['VOO', 'VUAA.L', 'VUAA'],  # Try VOO first, then alternatives
            'ARU': ['ARU.AX', 'ARU'],
            'SGLN': ['SGLN.L', 'IAU', 'GLD']  # Try London listing, then alternatives
        }
        
        final_tickers = []
        for original_ticker in self.tickers:
            success = False
            for variant in ticker_variants.get(original_ticker, [original_ticker]):
                try:
                    test_data = yf.download(variant, start=self.start_date, end=self.end_date, progress=False)
                    if not test_data.empty and len(test_data) > 100:  # Ensure we have sufficient data
                        final_tickers.append(variant)
                        print(f"  ✓ {original_ticker} -> {variant}: {len(test_data)} days")
                        success = True
                        break
                except Exception:
                    continue
            
            if not success:
                raise ValueError(f"Could not fetch data for ticker {original_ticker}. Please check the ticker symbol or availability.")
        
        # Download all data at once
        data = yf.download(final_tickers, start=self.start_date, end=self.end_date, progress=False)
        
        if len(final_tickers) == 1:
            self.data = pd.DataFrame(data['Close'])
            self.data.columns = final_tickers
        else:
            self.data = data['Close']
            
        # Rename columns to original tickers for consistency
        column_mapping = dict(zip(final_tickers, self.tickers))
        self.data = self.data.rename(columns=column_mapping)
        self.data = self.data[self.tickers]  # <-- Ensure column order matches tickers
        
        # Clean data
        self.data = self.data.dropna()
        
        if len(self.data) < 100:
            raise ValueError("Insufficient data points returned. Try a wider date range or check ticker validity.")
            
        print(f"Successfully fetched data: {len(self.data)} trading days")
        return True
```

### 
