---
title: Momentum Trading in Cryptocurrencies
author: Domenico Sauta
date: 2021-10-09 21:52:00 +0800
categories: [Algorithmic Trading]
tags: [crypto, trading]
math: true
mermaid: true
---

Momentum trading is a technical trading strategy where you analyse assets in the short-term, buy assets that have been showing up-trends, and close the position when the trend starts to lose momentum. In this strategy, the Exponential Moving Average of price in a given period is used as  a technical indicators to provide trading signals.

---

## Exponential Moving Average

When looking backwards at historical prices, traders have a range of different moving averages to choose from. The most common of which is a simple moving average (SMA). The SMA is, as the name suggests, is an equally weighted average calculation.

The primary drawback of using a simple moving average is that new information is considered to be equally as important as older information when generating a trading signal. In reality, this is not the case and it is likely, by the efficient market hypothesis, that the most recent price (excluding any noise) contains the most information and should therefore be considered the most important. To capture this idea, we employ an exponential moving average as defined below:

$$
\text{EMA}_t(P, \alpha) =
\begin{cases}
P_0, & t = 0 \\t
\alpha \cdot P_t + (1 - \alpha) \cdot \text{EMA}_{t-1} (P, \alpha), & t > 0
\end{cases}
$$

For this bot, the tuning paramter $\alpha$ was defined using the its centre of mass definition:

$$\alpha = \frac{1}{1 + \text{com}}$$

Below, we highlight the differences in the EMAs when changing the COM hyper-parameter from 8 to 24. Notice the differences in reactivity to movements in price.

![ema_demo](/assets/2021-10-10/ema_demo.PNG)

## Generating a Trading signal

Now that we have the baseline for our signal, the signal is then normalised and created as follows:

```python
ema_l = lookback['close'].ewm(com = n_l, adjust = False).mean()[today]
ema_s = lookback['close'].ewm(com = n_s, adjust = False).mean()[today]
sigma_s = lookback['close'].rolling(window = sigma_s_lookback).std()[today]

y = (ema_s - ema_l) / sigma_s
signal_values.append(y)
z = y / np.std(signal_values)
```

In essence, the difference of the long and short term EMA is being used to determine potential entrance and exit points. This difference is then normalised using the volatility of the prices itself, before being further normalised using the standard deviation of the signal itself. The outputted *z* value however is not standardised and is therefore not particularly useful to us. Let us then standardise *z* such that it will take values between -1 and 1 inclusive.   

```python
u = (z * np.exp(-z**2  /4)) / (np.sqrt(2) * np.exp(-1/2))
```
Notice that was actually transform the signal twice- the first time with a short term volatility measure of the underlying, and the second time with a longer term volatility measure of the signal itself. Due to this we actually lose a lot of trading signals in the warm-up period while we wait for the data to accumulate. This can be seen in the plot below. After all of our transformations, we can make use of the signal detailed below. It is standardised to be between -1 and 1, where positive values indicate a long position and negative numbers indicate a short position, and the magnitude represents the relative strength of that signal. This algorithm however, only assumes long positions.

![signal](/assets/2021-10-10/signal.PNG)

Now, it is important to construct a threshold for which we buy and sell. This should be subject to optimisation, but I chose to bin the signal into 3 equally sized responses- "long", "short" and "neutral". The signal outputs a long position when the signal is greater than 0.33, and short position when the signal is less than 0.33. Let us define this threshold as $\eta$. Note that in order to make the algorithm more aggressive, one would simply need to widen the window for buying and selling. We will explore this idea below.

## Testing the Algorithm

Let us examine the results of the algorithm on Cardano (ADA) using high frequency 30-minute price data, and start tweaking some of the algorithm hyper-parameters. Firstly, let's use default values as defined below. Note that these values all have units of '30 minute periods', except $\eta$, has no unit as it represents the signal.

|$n_s$|$n_l$|$\hat \sigma(P)$|$\hat \sigma(y_k)$|$\eta$|
|---|---|---|---|---|
|8   | 24  | 12 | 168  | 0.33   |

Executing the back-test over a one-year period:

![default_ada](/assets/2021-10-10/default_ada.PNG)

Notice the high degree of correlation between price and return- this is slightly worrying for all-weather performance. The high degree of correlation may suggest that this algorithm will out-perform when the underlying is out-performing, but under-perform in a bear market setting.

Let's now explore a more aggressive algorithm with a $\eta$ of 0.1 for demonstration purposes.

![low_eta](/assets/2021-10-10/low_eta.PNG)

Notice that this strategy performs a lot worse- likely a result of more false trading signals. We also trade 326 times- much more than our previous strategy. Maintaining all other variables as constant and increasing our $\eta$ value to 0.5 will yield opposite results:

![high_eta](/assets/2021-10-10/high_eta.PNG)

Changing the $\eta$ value to 0.5, we can observe that the algorithm makes 220 trades. In general, it seems that the more conservative $\eta$ value seems to yield better trading decisions, but may not exit positions fast enough to secure profits.

## Opportunities for Further Exploration

The keen readers may have noticed that the chosen hyper-parameter values for $n_s$ and $n_l$ did not receive proper justification. One may choose to fit these values to historical back-test results in the form of a multivariate grid search or alike. However, this poses a rather significant risk of over-fitting to historical conditions. If irrespective of this, we did optimise the values, what we would optimise for would pose an interesting problem. Optimising for return alone would likely lead to a highly risk tolerant algorithm. Sharpe ratio may be a more appropriate proxy for performance.

The primary issue with this algorithm is the coin-specific risk as a result of trading a single Crytpo- in our case Cardano. This should be dealt with by implementing a portfolio of coins. Fortunately, this strategy does scale well to that idea and signals can be generated simultaneously on multiple coins if desired, with cash allocation simply evenly split between assets.
