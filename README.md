Approach:
Parse Trade_History: Extract details like timestamp, asset, side (BUY/SELL), price, quantity, realizedProfit, and positionSide.
Calculate Financial Metrics for each Port_IDs:
ROI, PnL, Sharpe Ratio, MDD, Win Rate, Win Positions, Total Positions.
Rank Accounts based on performance.
Export Results.
I'll modify the code to extract trade details from Trade_History before computing the metrics.


import pandas as pd
import numpy as np
import ast
# Load the dataset
data = pd.read_csv("TRADES_CopyTr_90D_ROI.csv")
data
Port_IDs	Trade_History
0	3925368433214965504	[{'time': 1718899656000, 'symbol': 'SOLUSDT', ...
1	4002413037164645377	[{'time': 1718980078000, 'symbol': 'NEARUSDT',...
2	3923766029921022977	[{'time': 1718677164000, 'symbol': 'ETHUSDT', ...
3	3994879592543698688	[{'time': 1718678214000, 'symbol': 'ETHUSDT', ...
4	3926423286576838657	[{'time': 1718979615000, 'symbol': 'ETHUSDT', ...
...	...	...
145	4000222729738650369	[{'time': 1718982068000, 'symbol': 'ARKMUSDT',...
146	3998659472131949824	[{'time': 1718979385000, 'symbol': 'BTCUSDT', ...
147	4028701921959171840	[{'time': 1718984241000, 'symbol': 'BTCUSDT', ...
148	4014818740371615232	[{'time': 1718983357000, 'symbol': 'SOLUSDT', ...
149	3768170840939476993	[{'time': 1718977395000, 'symbol': 'BNBUSDT', ...
150 rows × 2 columns

# Function to parse Trade_History
def parse_trade_history(trade_history):
    try:
        trades = ast.literal_eval(trade_history)  # Convert string to list of dictionaries
        return pd.DataFrame(trades)
    except:
        return pd.DataFrame()
# Expand Trade_History into multiple rows
data_expanded = data.apply(lambda row: parse_trade_history(row['Trade_History']).assign(Port_IDs=row['Port_IDs']), axis=1)
data_expanded = pd.concat(data_expanded.values, ignore_index=True)
# Convert timestamp to datetime
data_expanded['time'] = pd.to_datetime(data_expanded['time'])
# Handle missing values (if any)
data_expanded = data_expanded.dropna()
# Classify trades
def classify_trade(row):
    if row['side'] == 'BUY' and row['positionSide'] == 'LONG':
        return 'long_open'
    elif row['side'] == 'SELL' and row['positionSide'] == 'LONG':
        return 'long_close'
    elif row['side'] == 'BUY' and row['positionSide'] == 'SHORT':
        return 'short_close'
    elif row['side'] == 'SELL' and row['positionSide'] == 'SHORT':
        return 'short_open'
    return 'unknown'

data_expanded['trade_type'] = data_expanded.apply(classify_trade, axis=1)
# Aggregate data per account
account_metrics = data_expanded.groupby('Port_IDs').agg(
    total_positions=('trade_type', 'count'),
    win_positions=('realizedProfit', lambda x: (x > 0).sum()),
    total_profit=('realizedProfit', 'sum'),
    avg_daily_return=('realizedProfit', 'mean'),
    std_dev_return=('realizedProfit', 'std')
).reset_index()
# Calculate Win Rate
account_metrics['win_rate'] = (account_metrics['win_positions'] / account_metrics['total_positions']) * 100
# Calculate ROI (assumption: initial investment is total trade volume per account)
total_trade_volume = data_expanded.groupby('Port_IDs')['quantity'].sum()
account_metrics['ROI'] = (account_metrics['total_profit'] / total_trade_volume) * 100
# Calculate Sharpe Ratio (assuming risk-free rate = 0)
account_metrics['sharpe_ratio'] = account_metrics['avg_daily_return'] / account_metrics['std_dev_return']
# Calculate Maximum Drawdown (MDD)
def max_drawdown(profit_series):
    cum_returns = profit_series.cumsum()
    peak = cum_returns.cummax()
    drawdown = (cum_returns - peak) / peak
    return drawdown.min()

mdd_values = data_expanded.groupby('Port_IDs')['realizedProfit'].apply(max_drawdown)
account_metrics['max_drawdown'] = mdd_values.values
# Normalize and Rank Metrics
metrics_weights = {
    'ROI': 0.3,
    'total_profit': 0.25,
    'win_rate': 0.2,
    'sharpe_ratio': 0.15,
    'max_drawdown': -0.1  # Negative weight since lower is better
}

for metric, weight in metrics_weights.items():
    account_metrics[metric + '_normalized'] = (account_metrics[metric] - account_metrics[metric].min()) / (account_metrics[metric].max() - account_metrics[metric].min())

account_metrics['score'] = sum(account_metrics[metric + '_normalized'] * weight for metric, weight in metrics_weights.items())
# Rank accounts based on final score
account_metrics = account_metrics.sort_values(by='score', ascending=False).reset_index(drop=True)
# Select top 20 accounts
top_20_accounts = account_metrics.head(20)
# Export Results
account_metrics.to_csv("binance_trade_analysis.csv", index=False)
top_20_accounts.to_csv("top_20_accounts.csv", index=False)

print("Analysis complete. Results saved.")
Analysis complete. Results saved.
