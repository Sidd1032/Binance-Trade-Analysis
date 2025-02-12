Analysis Process and Findings for Binance Trade Data Project
Analysis Process
Data Loading and Preprocessing:

Loaded the dataset containing Port_IDs and Trade_History.
Parsed the Trade_History column to extract trade-level details like timestamp, side, positionSide, quantity, price, and realizedProfit.
Expanded the data such that each trade corresponds to a separate row with its associated Port_ID.
Data Cleaning and Feature Engineering:

Converted timestamps to datetime for proper time-series analysis.
Dropped missing values to ensure accurate calculations.
Classified each trade as long_open, long_close, short_open, or short_close based on the side and positionSide.
Metric Calculations:

Total Positions: Counted the total number of trades per account.
Win Positions: Counted the number of profitable trades (where realizedProfit > 0).
Win Rate: Calculated as (Win Positions / Total Positions) * 100.
ROI (Return on Investment): Estimated as (Total Profit / Total Trade Volume) * 100.
Sharpe Ratio: Measured risk-adjusted returns using avg_daily_return / std_dev_return.
Maximum Drawdown (MDD): Evaluated the largest loss from peak equity to a subsequent low for each account.
Ranking Accounts:

Normalized all metrics to a [0,1] scale for comparability.
Assigned weights to each metric:
ROI (30%), Total Profit (25%), Win Rate (20%), Sharpe Ratio (15%), and MDD (-10%, as a lower MDD is better).
Calculated a weighted score for each account and ranked them based on the final score.
Exporting Results:

Saved the full account metrics in binance_trade_analysis.csv.
Created a list of the top 20 performing accounts based on the ranking, saved as top_20_accounts.csv.
Findings
Top Performers:

Identified 20 accounts with the highest weighted scores. These accounts demonstrated consistent profitability, high ROI, and minimal drawdowns.
Performance Highlights:

Accounts with higher ROI often exhibited higher win rates, showing a positive correlation between trade success and profitability.
The Sharpe Ratio highlighted accounts with stable returns compared to those with volatile profits.
Risk Analysis:

Some accounts had high profits but poor risk management, reflected by large drawdowns and lower Sharpe Ratios.
Balancing ROI with MDD proved crucial in determining the top performers.
Insights for Improvement:

Accounts with lower scores often had low win rates and significant drawdowns, suggesting the need for improved trading strategies or risk management.
Deliverables
Analysis Code: A complete Python script provided for replicating the analysis.
CSV Files:
binance_trade_analysis.csv: Full metrics for all accounts.
top_20_accounts.csv: Ranked list of the top 20 accounts.
Report: Detailed methodology and key findings summarized above.
