# 📊 Cumulative Performance Ratio (CPR) for Any WBS
This project contains visualization tools for tracking Cumulative Performance Ratio (CPR) metrics across any Work Breakdown Structure. It uses Python libraries including pandas, matplotlib, and numpy to generate comparative charts and tables showing baseline, actual, and forecast data.

## 🚀 Description
The CPR charts visualize project performance over time, tracking the relationship between planned and actual/forecast completions across any selected WBS. This analysis helps project managers assess efficiency, identify trends, and make data-driven decisions.

## 📑 Notebook Contents
The `CPR (any WBS).ipynb` file includes:

1. **Library Imports** 📦
   ```python
   import pandas as pd
   import matplotlib.pyplot as plt
   import numpy as np
   ```

2. **Excel File Reading** 📥
   ```python
   df = pd.read_excel('CPR (any WBS).xlsx')
   ```

3. **Date Column Conversion** 📅
   ```python
   df['BL Project Finish'] = pd.to_datetime(df['BL Project Finish'], errors='coerce')
   df['Actual Finish'] = pd.to_datetime(df['Actual Finish'], errors='coerce')
   df['Remaining Early Finish'] = pd.to_datetime(df['Remaining Early Finish'], errors='coerce')
   ```

4. **Cumulative Data Preparation** 📈
   ```python
   data_sorted_bl = df.sort_values(by='BL Project Finish').dropna(subset=['BL Project Finish'])
   data_sorted_actual = df.sort_values(by='Actual Finish').dropna(subset=['Actual Finish'])
   data_sorted_forecast = df.sort_values(by='Remaining Early Finish').dropna(subset=['Remaining Early Finish'])

   cumulative_bl = data_sorted_bl.groupby('BL Project Finish').size().cumsum()
   cumulative_actual = data_sorted_actual.groupby('Actual Finish').size().cumsum()
   cumulative_forecast = data_sorted_forecast.groupby('Remaining Early Finish').size().cumsum()
   ```

5. **Forecast Continuity Adjustment** 🔄
   ```python
   if not cumulative_actual.empty:
       last_actual_date = cumulative_actual.index[-1]
       last_actual_value = cumulative_actual.iloc[-1]
       
       # Ensure continuity: forecast begins the day after the last actual point
       next_forecast_date = last_actual_date + pd.Timedelta(days=1)
       cumulative_forecast = cumulative_forecast[cumulative_forecast.index >= next_forecast_date]
       cumulative_forecast = pd.concat(
           [pd.Series({last_actual_date: last_actual_value}),
            cumulative_forecast + last_actual_value]
       )
   ```

6. **Main CPR Chart Creation** 🎨
   ```python
   plt.figure(figsize=(10, 6))
   plt.plot(cumulative_bl.index, cumulative_bl.values, label='Cum Project Baseline', color='gray')
   plt.plot(cumulative_actual.index, cumulative_actual.values, label='Cum Actual', color='blue')
   plt.plot(cumulative_forecast.index, cumulative_forecast.values, label='Cum Forecast', color='green')

   # Add today's vertical line
   today = pd.Timestamp.now()
   plt.axvline(today, color='red', linestyle='--', label='Today')
   ```

7. **Chart Formatting and Saving** 💾
   ```python
   plt.title('Cumulative Performance Ratio for "any WBS"')
   plt.xlabel('Finish Dates')
   plt.ylabel('Activities')
   plt.legend()
   plt.grid()

   plt.savefig('CPR (any WBS).png', format='png', transparent=True, dpi=300)
   ```

8. **Monthly CPR Calculation** 🧮
   ```python
   # Combine Actual and Forecast data
   data_sorted_actual_forecast = pd.concat([
       data_sorted_actual[['Actual Finish']],
       data_sorted_forecast[['Remaining Early Finish']].rename(columns={'Remaining Early Finish': 'Actual Finish'})
   ]).sort_values(by='Actual Finish').dropna(subset=['Actual Finish'])

   # Group by month and calculate cumulative values
   cumulative_actual_forecast_by_month = data_sorted_actual_forecast.groupby(
       data_sorted_actual_forecast['Actual Finish'].dt.to_period('M')
   ).size().cumsum()

   cumulative_bl_by_month = data_sorted_bl.groupby(
       data_sorted_bl['BL Project Finish'].dt.to_period('M')
   ).size().cumsum()
   ```

9. **CPR Table Creation and Export** 📋
   ```python
   cpr_table = pd.DataFrame({
       'Actual/Forecast Completed': cumulative_actual_forecast_by_month,
       'Planned Completed': cumulative_bl_by_month
   }).fillna(0)
   
   cpr_table['CPR'] = (
       cpr_table['Actual/Forecast Completed'] / cpr_table['Planned Completed']
   ).replace([float('inf'), float('nan')], 0)
   cpr_table['CPR'] = cpr_table['CPR'].round(2)
   
   cpr_table.to_excel('CPR_(any_WBS)_Table.xlsx', index=False)
   ```

10. **Data Interpolation for Zero Values** 📉
    ```python
    # Fill zero values with interpolation
    cpr_table['CPR'] = cpr_table['CPR'].replace(0, np.nan)  # Convert 0s to NaN
    cpr_table['CPR'] = cpr_table['CPR'].interpolate(method='linear')  # Interpolate missing values
    ```

11. **Monthly CPR Variation Chart** 📊
    ```python
    plt.figure(figsize=(10, 6))
    plt.plot(cpr_table['Month'].astype(str), cpr_table['CPR'], 
             marker='o', linestyle='-', color='blue', label='CPR')
    
    # Reference lines
    plt.axhline(y=1, color='gray', linestyle='--', label='Target CPR = 1')
    plt.axhline(y=0.8, color='red', linestyle='--', label='Minimum Acceptable CPR = 0.8')
    
    plt.title('Monthly Variation of Cumulative Performance Ratio (CPR)')
    plt.xlabel('Month')
    plt.ylabel('CPR')
    
    plt.savefig('Monthly Variaton of CPR (any WBS).png', 
                format='png', transparent=True, dpi=300)
    ```

## 🛠️ Requirements
Required libraries:
- pandas
- numpy
- matplotlib

Install via pip:
```sh
pip install pandas numpy matplotlib
```

## 📈 Execution
Open `CPR (any WBS).ipynb` in Jupyter Notebook or JupyterLab and run the cells.

## 📃 License
This project is under the Apache License. See LICENSE file for details.
