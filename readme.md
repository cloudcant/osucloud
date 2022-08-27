<p align="center"><img width=12.5% src="https://github.com/cloudcant/osucloud/blob/main/readme/Untitled.png"></p>
<p align="center"><img width=60% src="https://github.com/cloudcant/osucloud/blob/main/readme/banner.jpg"></p>

## Basic Overview
My edit on osugame.online that adds a darker theme and a diffrent cursor aswell as more in the future such as replays and a pp system.
<p align="center"><img width=95% src="https://user-images.githubusercontent.com/66269103/187049207-bd7d8f31-a679-40a6-80be-2acc09cc4920.png"></p>

<br>

## Gameplay
<p align="center"><img width=95% src="https://user-images.githubusercontent.com/66269103/187049220-5ce21b8d-ca12-4df0-a5ae-fabc8c284572.png"></p>

<br>

## Last Stable Release
```bash
pip install clairvoyant
```

## Latest Development Changes
```bash
python -m pip install git+https://github.com/anfederico/clairvoyant
```

## Backtesting Signal Accuracy
During the testing period, the model signals to buy or sell based on its prediction for price
movement the following day. By putting your trading algorithm aside and testing for signal accuracy
alone, you can rapidly build and test more reliable models.

```python
from clairvoyant.engine import Backtest
import pandas as pd
features  = ["EMA", "SSO"]   # Financial indicators of choice
trainStart = 0               # Start of training period
trainEnd   = 700             # End of training period
testStart  = 701             # Start of testing period
testEnd    = 1000            # End of testing period
buyThreshold  = 0.65         # Confidence threshold for predicting buy (default = 0.65) 
sellThreshold = 0.65         # Confidence threshold for predicting sell (default = 0.65)
continuedTraining = False    # Continue training during testing period? (default = false)
# Initialize backtester
backtest = Backtest(features, trainStart, trainEnd, testStart, testEnd, buyThreshold, sellThreshold, continuedTraining)
# A little bit of pre-processing
data = pd.read_csv("SBUX.csv", date_parser=['date'])
data = data.round(3)                                    
# Start backtesting and optionally modify SVC parameters
# Available paramaters can be found at: http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html
backtest.start(data, kernel='rbf', C=1, gamma=10)
backtest.conditions()
backtest.statistics()  
backtest.visualize('SBUX')
```

#### Output
```text
------------ Data Features ------------
X1: EMA
X2: SSO
---------------------------------------
----------- Model Arguments -----------
kernel: rbf
C: 1
gamma: 10
---------------------------------------
---------  Engine Conditions ----------
Training: 2013-03-01 -- 2015-12-09
Testing:  2015-12-10 -- 2017-02-17
Buy Threshold: 65.0%
Sell Threshold: 65.0%
Continued Training: False
---------------------------------------
------------- Statistics --------------
Total Buys: 170
Buy Accuracy: 68.24%
Total Sells: 54
Sell Accuracy: 59.3%
---------------------------------------
```

<img src="https://github.com/anfederico/clairvoyant/blob/master/media/SBUX.png">

## Simulate a Trading Strategy
Once you've established your model can accurately predict price movement a day in advance, 
simulate a portfolio and test your performance with a particular stock. User defined trading logic
lets you control the flow of your capital based on the model's confidence in its prediction
and the following next day outcome.

```python
def logic(account, today, prediction, confidence):
    
    if prediction == 1:
        Risk         = 0.30
        EntryPrice   = today['close']
        EntryCapital = account.BuyingPower*Risk
        if EntryCapital >= 0:
            account.EnterPosition('Long', EntryCapital, EntryPrice)
    if prediction == -1:
        ExitPrice = today['close']
        for Position in account.Positions:  
            if Position.Type == 'Long':
                account.ClosePosition(Position, 1.0, ExitPrice)
simulation = backtester.Simulation(features, trainStart, trainEnd, testStart, testEnd, buyThreshold, sellThreshold, continuedTraining)
simulation.start(data, 1000, logic, kernel='rbf', C=1, gamma=10)
simulation.statistics()
simulation.chart('SBUX')
```

#### Output

```text
------------- Statistics --------------
Buy and Hold : -6.18%
Net Profit   : -61.84
Strategy     : 5.82%
Net Profit   : 58.21
Longs        : 182
Sells        : 168
Shorts       : 0
Covers       : 0
--------------------
Total Trades : 350
---------------------------------------
```

<img src="https://raw.githubusercontent.com/anfederico/clairvoyant/master/media/Chart.png">

## Other Projects
#### Intensive Backtesting
The primary purpose of this project is to rapidly test datasets on machine learning algorithms (specifically Support Vector Machines). While the Simulation class allows for basic strategy testing, there are other projects more suited for that task. Once you've tested patterns within your data and simulated a basic strategy, I'd recommend taking your model to the next level with:
```text
https://github.com/anfederico/gemini
```

#### Social Sentiment Scores
The examples shown use data derived from a project where we are data mining social media and performing stock sentiment analysis. To get an idea of how we do that, please take a look at:
```text
https://github.com/anfederico/stocktalk
```

## Notes
#### Multivariate Functionality
Remember, more is not always better!
```python
variables = ["SSO"]                            # 1 feature
variables = ["SSO", "SSC"]                     # 2 features
variables = ["SSO", "SSC", "RSI"]              # 3 features
variables = ["SSO", "SSC", "RSI", ... , "Xn"]  # n features
```

## Contributing
Please take a look at our [contributing](https://github.com/anfederico/clairvoyant/blob/master/CONTRIBUTING.md) guidelines if you're interested in helping!
#### Pending Features
- Export model
- Support for multiple sklearn SVM models
- Visualization for models with more than 2 features
