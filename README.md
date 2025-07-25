# 3D Efficient Frontier Portfolio Visualization (NVDA; VUAA; ARU; SGLN)
We create a 3D Efficient Frontier Visualization for a portfolio consisting of Nvidia (NVDA) stock, S&amp;P 500 Vanguard ETF (VUAA), Arafura Rare Earths Ltd (ARU) stock and iShares Physical Gold ETF (SGLN).

$$\ R_p=\sum_{i=1}^n w_iR_i $$

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
