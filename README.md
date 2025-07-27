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
$$
R_p=\sum_{i=1}^n w_iR_i
$$

```{r}
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
from matplotlib.colors import Normalize
from matplotlib.cm import ScalarMappable
import warnings
from datetime import datetime, timedelta
from scipy.optimize import minimize
import itertools
```
