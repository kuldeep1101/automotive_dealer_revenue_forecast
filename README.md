# Auto Dealer Revenue Forecasting (Mid-Month)

This project predicts a car dealership network's total revenue for the current month using only the sales data available so far (partway through the month). I built it in Python and tested it over 34 past months, where it landed within about 3.5% of the actual month-end revenue on average.

## Why I built it

Dealers want to know how the month is going to end while it's still running, not after it's over. If you can tell by day 14 roughly where revenue will finish, you can act on inventory, staffing and targets two weeks earlier. So the question I set out to answer was simple: given the revenue booked so far, what's the full month going to look like?

## How it works

The idea behind the model is that revenue builds up through a month in a fairly repeatable shape. So for any past month, I can ask: how much of that month's total had come in by day 14?

```
completion_ratio = revenue_up_to_day_D / full_month_revenue
implied_forecast = current_revenue_so_far / completion_ratio
```

I do this for four reference months and combine them with weights:

| Reference | What it is | Weight | Why |
|-----------|------------|:------:|-----|
| Y-1 | Same month last year | 0.20 | Seasonality |
| M-3 | Three months ago | 0.16 | Recent pace |
| M-2 | Two months ago | 0.24 | Recent pace |
| M-1 | Last month | 0.40 | Recent pace |

The final forecast is just the weighted sum of the four implied forecasts. The recent months get the most weight because they reflect current momentum, and last year's same month is there to catch the seasonal pattern.

One thing that matters: every reference month has to be cut off at the same day number as the current month. If I have data up to day 14 now, I compare against day 14 of every reference month, not the whole month. Otherwise the ratios aren't comparable.

## The data

Daily dealer-level sales from 2021 to 2024. Each row is a transaction with these columns:

`date, dealer_id, brand, model, final_price, units_sold, revenue`

Cleaning was light: cast the text columns to category, parse the dates, and add a month column to group by.

## What the sales trend looks like

Revenue is seasonal, which is exactly why last year's same month is useful as a reference:

- Oct to Dec is peak season, highest sales of the year
- March is a smaller second peak
- Feb, April and Sep tend to dip
- May to July stays fairly flat and steady

## Current month cumulative trend

To sanity-check the forecast visually, I plot the current month's cumulative revenue as it builds up day by day, and overlay two comparison lines: the same month last year, and last month. On top of that I mark two horizontal lines, one for how much revenue has come in so far, and one for the projected month-end total from the model.

This chart is useful because you can see at a glance whether the current month is tracking ahead of or behind its references, and where the projection expects it to finish. If the current-month bars are sitting above last year's line at the same point in the month, you're pacing ahead, and the projection line reflects that.

<img width="1295" height="698" alt="Current Month - Cumulative - Revenue Trend" src="https://github.com/user-attachments/assets/73193be9-6967-4ec8-a784-5434a9c1441d" />


## Results

I back-tested by going through every completed month, pretending I only had data up to day 14, running the model, and comparing to what the month actually finished at.

| | |
|---|---|
| Months tested | 34 |
| MAPE | 3.54% |
| RMSE | ~330M |
| Cutoff day | 14 |

A 3.5% average error halfway through the month is close enough to be genuinely useful for planning. The error is highest in the peak months (Oct to Dec), which makes sense since those months are the most volatile.

## Built with

Python, pandas, NumPy, Matplotlib, Jupyter.

## Files

```
AutoDealer_car_retail_final.ipynb   # the full analysis, model and back-test
car_retail_sales_dataset_modified.csv
README.md
```

Run the notebook top to bottom. It goes: load and clean the data, look at the seasonal trend, define the monthly_sum helper, run the pacing forecast, plot the cumulative chart, then back-test.

## What I'd add next

- A confidence range instead of a single number. The four completion ratios don't all agree, and the spread between the lowest and highest gives a natural low/high band around the forecast.
- Tuning the weights instead of picking them by hand. Right now the weights are a reasonable guess. A grid search that minimises back-test error, or weights that change by season, should bring down the error in the peak months.
- Forecasting per brand or per dealer, not just the network total.
- Checking how the accuracy changes depending on how far into the month you are (day 10 vs 14 vs 20).

## Skills this project shows

Working with time-based sales data in pandas, building a forecasting method from scratch, spotting seasonal patterns, and properly validating the result out-of-sample with MAPE and RMSE rather than just eyeballing it.
