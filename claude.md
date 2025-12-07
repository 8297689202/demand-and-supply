# Demand & Supply Zone Scanner - Codebase Documentation

## Overview
This is a Streamlit-based web application that scans Indian Stock Market for demand and supply zones using various time intervals. It identifies patterns like Drop Base Rally (DBR), Rally Base Rally (RBR), Rally Base Drop (RBD), and Drop Base Drop (DBD).

## Tech Stack
- **Framework**: Streamlit
- **Data Source**: TvDatafeed (TradingView)
- **Data Processing**: Pandas
- **Visualization**: Plotly
- **Timezone**: Asia/Kolkata (IST)

## Key Components

### 1. Data Fetching Functions

#### `fetch_stock_data_and_resample()`
- **Location**: [app.py:35-81](app.py#L35-L81)
- **Purpose**: Fetches stock data and resamples it to higher timeframes
- **Parameters**:
  - `symbol`: Stock symbol
  - `exchange`: Exchange name (e.g., NSE, BSE)
  - `n_bars`: Number of bars to fetch
  - `htf_interval`: Higher timeframe interval
  - `interval`: Base interval for fetching
  - `interval_key`: Key for resampling rule
  - `fut_contract`: Futures contract (if applicable)
- **Resampling Rules** (lines 47-55):
  ```python
  'in_10_minute': '10min'
  'in_75_minute': '75min'
  'in_125_minute': '125min'
  'in_5_hour': '5h'
  'in_6_hour': '6h'
  'in_8_hour': '8h'
  'in_10_hour': '10h'
  'in_12_hour': '12h'
  ```

#### `fetch_stock_data()`
- **Location**: [app.py:84-100](app.py#L84-L100)
- **Purpose**: Fetches stock data without resampling
- **Returns**: DataFrame with OHLCV data in IST timezone

### 2. Time Validation

#### `validate_time_condition()`
- **Location**: [app.py:220-240](app.py#L220-L240)
- **Purpose**: Validates if a zone is within the acceptable time window
- **Time Delays by Interval**:
  - '1 Minute': 15 minutes
  - '3 Minutes': 75 minutes
  - '5 Minutes': 75 minutes
  - '10 Minutes': 1 day
  - '15 Minutes': 1 day
  - Default: 7 days

### 3. Interval Configuration

#### Available Intervals
- **Location**: [app.py:1040-1061](app.py#L1040-L1061)
- **Active Intervals**:
  - 1 Minute → `Interval.in_1_minute`
  - 5 Minutes → `Interval.in_5_minute`
  - 15 Minutes → `Interval.in_15_minute`
  - 30 Minutes → `Interval.in_30_minute`
  - 1 Hour → `Interval.in_1_hour`
  - 2 Hours → `Interval.in_2_hour`
  - 1 Day → `Interval.in_daily`
  - 1 Week → `Interval.in_weekly`
  - 1 Month → `Interval.in_monthly`

- **Commented Out** (need resampling):
  - 3, 10, 45, 75, 125 Minutes
  - 3, 4, 5, 6, 8, 10, 12 Hours

#### Higher Timeframe Mapping
- **Location**: [app.py:1090-1100](app.py#L1090-L1100)
- 1 Minute → 15 Minutes HTF
- 3/5 Minutes → 1 Hour HTF
- 10/15 Minutes → Daily HTF
- 1 Hour/75/125 Minutes/2 Hours → Weekly HTF
- 3/4 Hours/1 Day/Week/Month → Monthly HTF

### 4. Zone Detection Logic

The application detects 4 pattern types:
1. **Drop Base Rally (DBR)**: Demand zones
2. **Rally Base Rally (RBR)**: Demand zones
3. **Rally Base Drop (RBD)**: Supply zones
4. **Drop Base Drop (DBD)**: Supply zones

Each zone has a status:
- **Fresh Zone**: Untested zone
- **Target Zone**: Price reached target
- **Stoploss Zone**: Zone invalidated

### 5. UI Configuration

- **Page Layout**: Wide mode
- **Max Base Candles**: 1-6 (user selectable)
- **Zone Distance**: Price to entry distance in % (default: 10%)
- **Timezone**: All times displayed in IST

## File Structure
```
/
├── app.py                              # Main application file
├── before_update_app_dot_py_backup_it.py  # Backup file
└── claude.md                           # This documentation file
```

## Adding New Time Intervals

To add a new time interval that requires resampling (like 75min, 125min, 4h, etc.):

### Step 1: Add to `resample_rules` dictionary
Location: [app.py:47-55](app.py#L47-L55)
```python
'in_125_minute': '125min',  # Already exists
'in_4_hour': '4h',          # Add this
```

### Step 2: Uncomment or add to `interval_options`
Location: [app.py:1040-1061](app.py#L1040-L1061)
```python
'125 Minutes': Interval.in_5_minute,  # Uses 5min base, resamples to 125min
'4 Hours': Interval.in_4_hour,         # Direct interval if available
```

### Step 3: Update HTF mapping
Location: [app.py:1090-1100](app.py#L1090-L1100)
Add the interval to appropriate HTF group

### Step 4: Update `validate_time_condition` (optional)
Location: [app.py:221-227](app.py#L221-L227)
Add time delay for the new interval if needed

## Important Notes

1. **Resampling Strategy**: TvDatafeed doesn't support all intervals natively. For custom intervals (75min, 125min, 4h, etc.), the app fetches smaller interval data and resamples it.

2. **Data Limits**: Be mindful of n_bars - fetching too many bars at small intervals can hit API limits.

3. **Timezone Handling**: All data is converted from UTC to IST (Asia/Kolkata).

4. **Pattern Detection**: The core logic scans for specific candle patterns to identify supply/demand zones based on price action.

5. **Zone Validation**: Zones are validated based on time elapsed since creation using `validate_time_condition()`.
