<!--
*** Thanks for checking out this README Template. If you have a suggestion that would
*** make this better, please fork the repo and create a pull request or simply open
*** an issue with the tag "enhancement".
*** Thanks again! Now go create something AMAZING! :D
-->





<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->



# Team: Aces
## Members: Hepu Jin, Lingxiao Zhang
Using market regimes, changepoints and anomaly detection in QWIM

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
* [Example](#example)
  * [Step 1](#step1)
  * [Step 2](#step2)
  * [Step 3](#step3)
  



<!-- ABOUT THE PROJECT -->
## About The Project
The purpose of this project is to overcome the challenge that changing market conditions present to traditional portfolio optimization. We formulate a Hidden Markov Model to detect market regimes using standardized absorption ratio, a market risk indicator, calculated by 10 MSCI U.S. sector indices. In this way, we can estimate the expected returns and corresponding covariance matrix of assets based on regimes. By design, these two parameters are calibrated to better describe the properties of different market regimes. Then, these regime-based parameters serve as the inputs of different portfolio weight optimizers, thereby constructing regime-dependent portfolios. In an asset universe consisting of U.S. equities, U.S. bonds and commodities, it is shown that regime-based portfolios have better returns, lower volatility and less tail risks compared with other competing portfolios.


<!-- GETTING STARTED -->
## Getting Started

### Prerequisites
* sklearn
```sh
pip install sklearn
```
* hmmlearn
```sh
pip install hmmlearn
```
* PyPortfolioOpt
```sh
pip install PyPortfolioOpt
```



<!-- EXAMPLE -->
## Example
Here is an example demonstrating how we use hidden Markov Model detect market regime and construct portfolios

_For the full example, please refer to the [project code](https://github.com/stanleyjin1996/Aces/blob/master/Code/project%20code.ipynb)_

### Step 1
This step calculates absorption ratio from 10 MSCI U.S. sector indices.
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA

AR = AbsorptionRatio(data,spx)
ar = AR.absorption_ratio(num_pc = 3) #absorption ratio with 3 principal components
delta = AR.delta_ar(ar) #standardized absorption ratio
AR.visualize(ar,spx) #compare S&P500 index with absorption ratio
```

### Step 2
This step implements hidden Markov Model on standardized absorption ratio and identify market regimes.
```python
from hmmlearn.hmm import GaussianHMM
from datetime import datetime

bt = BackTest(delta)
regime = bt.detect_regime(days_to_train = 1500,rebalance=60,min_length=100) #get regime
bt.plot_regime_color(spx) #plot regime
```
### Step 3
This step constructs regime-based portfolios and compare with their counterparts. For each optimizer, two portfolios are constructed at the same time.
One is regime-based portfolio, one is not using regime information obtained from step 2.
```python
from pypfopt import EfficientFrontier
from pypfopt import objective_functions
import pyfolio as pf

asset = pd.read_csv('trading_asset.csv',index_col=0)
price = asset.join(regime,how='inner')
tickers = asset.columns

pfo = Portfolio(price,tickers)

#construct portoflio using a chosen optimizer
pfo.construct_portfolio(method='max return given risk', target_vol = 0.15)
```
After constructing portfolios, we can compare portfolios performance.

```python
p1 = pfo.value_special.copy() #portfolio 1
p2 = pfo.value_base.copy() #portfolio 2
p1.columns = ['special']
p2.columns = ['base']
p = p1.join(p2, how='inner')
bt.plot_regime_color(p)
```
