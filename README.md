# nifty-sma-strategy
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt

# Download NIFTY data
nifty = yf.download("^NSEI", start="2020-01-01", end="2024-12-31")

print(nifty.head())
# Calculate moving averages
nifty['SMA_20'] = nifty['Close'].rolling(window=20).mean()
nifty['SMA_50'] = nifty['Close'].rolling(window=50).mean()
nifty['Signal'] = 0

# Buy signal
nifty.loc[nifty['SMA_20'] > nifty['SMA_50'], 'Signal'] = 1

# Sell signal
nifty.loc[nifty['SMA_20'] < nifty['SMA_50'], 'Signal'] = -1
nifty['Position'] = nifty['Signal'].diff()
plt.figure(figsize=(14,7))

plt.plot(nifty['Close'], label='NIFTY Close', alpha=0.7)
plt.plot(nifty['SMA_20'], label='SMA 20', color='green')
plt.plot(nifty['SMA_50'], label='SMA 50', color='red')

# Buy signals
plt.scatter(
    nifty[nifty['Position'] == 1].index,
    nifty['SMA_20'][nifty['Position'] == 1],
    marker='^', color='blue', label='Buy', s=100
)

# Sell signals
plt.scatter(
    nifty[nifty['Position'] == -1].index,
    nifty['SMA_20'][nifty['Position'] == -1],
    marker='v', color='black', label='Sell', s=100
)

plt.title("NIFTY 50 SMA Crossover Strategy")
plt.legend()
plt.grid()
plt.show()
nifty['Market_Return'] = nifty['Close'].pct_change()
nifty['Strategy_Return'] = nifty['Market_Return'] * nifty['Signal'].shift(1)

# Cumulative returns (compounded)
(1 + nifty[['Market_Return', 'Strategy_Return']]).cumprod().plot(figsize=(12,6))
plt.title("Market vs SMA Strategy Cumulative Returns")
plt.show()
