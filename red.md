# Data Cleaning: Transforming Messy Sales Data

This document demonstrates how to clean and transform a messy sales dataset using Python and R. The dataset contains order IDs with sales values categorized by customer segments (Consumer, Corporate, Home Office) and shipping modes (First Class, Same Day, Second Class, Standard Class). The goal is to convert the wide, disorganized table into a tidy, long-format structure.

## Example Input Data

Below is a screenshot of the raw data structure from the Excel file "data.xlsx", sheet "Dirty 1". The table has order IDs in the first column, followed by segment and shipping mode combinations, with many unnamed columns.

![Screenshot](https://raw.githubusercontent.com/madhanlibo/data-cleaning/main/pic/Screenshot%202025-03-31%20171953.png)

---

## Python Code

The Python code uses `pandas` to read, clean, and transform the data. Below is the full implementation followed by a step-by-step explanation.

### Python Implementation

```python
import pandas as pd

# Define segments and ship modes
segments = ['consumer', 'corporate', 'home_office']
ship_modes = ['first_class', 'same_day', 'second_class', 'standard_class']

# Generate group shipping names
group_shipping_names = [f"{segment}_{ship_mode}" 
                        for segment in segments 
                        for ship_mode in ship_modes]

# Read the data
df = pd.read_excel("data.xlsx", sheet_name="Dirty 1", skiprows=1)

# Clean and transform the DataFrame
df_cleaned = (df
    .loc[:, ~df.columns.str.startswith('Unnamed')]  # Remove unnamed columns
    .rename(columns={'Ship Mode': 'order_id'})      # Rename first column to "order_id"
    .iloc[1:]                                       # Drop the first row
    .set_axis(['order_id'] + group_shipping_names, axis=1)  # Rename all columns
    .melt(id_vars=['order_id'],                     # Pivot to long format
          var_name='group_shipping', 
          value_name='value')
    .dropna(subset=['value'])                       # Drop rows with NA values
    .assign(segment=lambda x: x['group_shipping'].str.extract(r'(consumer|corporate|home_office)'),
            shipping_mode=lambda x: x['group_shipping'].str.extract(r'_(.+)'))
    .drop(columns=['group_shipping'])               # Remove temporary column
    .sort_values(['segment', 'shipping_mode', 'order_id'])  # Sort the data
)

# Display the result
print(df_cleaned)


