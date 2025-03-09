# Building a Walmart Retail Data Pipeline

## Introduction
Walmart is the biggest retail store in the United States. Just like others, they have been expanding their e-commerce part of the business. By the end of 2022, e-commerce represented a roaring $80 billion in sales, which is 13% of total sales of Walmart. One of the main factors that affects their sales is public holidays, like the Super Bowl, Labour Day, Thanksgiving, and Christmas.

In this project, the goal was to create a data pipeline for analyzing supply and demand around holidays and conducting a preliminary analysis of the data. The project focused on automating data extraction, transformation, aggregation, and loading processes using Python.

## Technology Used
- **Python**: For scripting and data manipulation.
- **Pandas**: To handle dataframes and perform transformations.
- **Jupyter Notebook**: For interactive data analysis and visualization.
- **Parquet**: For efficient storage and access to complementary data.
- **Pytest**: For testing the data pipeline functions.

## Dataset Used
- **grocery_sales**: A PostgreSQL table containing weekly sales data for Walmart stores.
  - Columns:
    - `index`: Unique ID of the row
    - `Store_ID`: Store number
    - `Date`: The week of sales
    - `Weekly_Sales`: Sales for the given store

- **extra_data.parquet**: A complementary dataset including additional sales-related metrics.
  - Key Columns:
    - `IsHoliday`: Whether the week contains a public holiday (1 if yes, 0 if no)
    - `Temperature`: Temperature on the day of sale
    - `Fuel_Price`: Cost of fuel in the region
    - `CPI`: Consumer Price Index
    - `Unemployment`: The prevailing unemployment rate
    - `MarkDown1`, `MarkDown2`, `MarkDown3`, `MarkDown4`: Promotional markdowns
    - `Dept`: Department number in each store
    - `Size`: Size of the store
    - `Type`: Type of the store (based on size)

## Data Pipeline Components

### 1. Extract
```python
import pandas as pd

def extract(store_data, extra_data):
    extra_df = pd.read_parquet(extra_data)
    merged_df = store_data.merge(extra_df, on="index")
    return merged_df
```

### 2. Transform
```python
def transform(raw_data):
    raw_data.fillna(0, inplace=True)
    raw_data['Date'] = pd.to_datetime(raw_data['Date'], errors='coerce')
    raw_data['Month'] = raw_data['Date'].dt.month
    raw_data = raw_data[raw_data['Weekly_Sales'] > 10000]
    clean_data = raw_data[['Store_ID', 'Month', 'Dept', 'IsHoliday', 'Weekly_Sales', 'CPI', 'Unemployment']]
    return clean_data
```

### 3. Aggregate
```python
def avg_weekly_sales_per_month(clean_data: pd.DataFrame) -> pd.DataFrame:
    agg_data = (clean_data[['Month', 'Weekly_Sales']]
                .groupby('Month')
                .agg({'Weekly_Sales': 'mean'})
                .reset_index()
                .round(2))
    return agg_data
```

### 4. Load
```python
import os

def load(clean_data, agg_data, clean_data_path='clean_data.csv', agg_data_path='agg_data.csv'):
    clean_data.to_csv(clean_data_path, index=False)
    agg_data.to_csv(agg_data_path, index=False)
```

### 5. Validation
```python
def validation(file_path):
    return os.path.isfile(file_path)

# Example
print(validation('clean_data.csv'))  # Expected output: True
print(validation('agg_data.csv'))    # Expected output: True
```

## Installation and Setup
1. **Clone the repository:**
```bash
git clone https://github.com/RajeshShahu1/walmart-retail-data-pipeline.git
```
2. **Navigate to the project directory:**
```bash
cd walmart-retail-data-pipeline
```
3. **Run the Jupyter Notebook:**
```bash
jupyter notebook notebooks/notebook.ipynb
```

## Testing
```python
import pytest

# Example test for validation function
def test_validation():
    assert validation('clean_data.csv')
    assert validation('agg_data.csv')
```
## Key Findings and Insights
- The highest average monthly sales were observed during the holiday season, especially in November and December.
- Strong sales spikes correlated with public holidays, particularly Thanksgiving and Christmas.
- Promotional markdowns significantly boosted sales during high-demand periods.
- Economic indicators such as unemployment rates and CPI showed minimal direct influence on weekly sales.
- These insights could help Walmart optimize stock and marketing strategies around holidays.
