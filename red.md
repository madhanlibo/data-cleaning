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
```
---

### Step-by-Step Explanation of Python Code

### **1. Import Library**

```python
import pandas as pd
```

This line imports the pandas library, which is essential for data manipulation in Python, similar to R’s tidyverse.

### **2. Define Segments and Ship Modes**

```python
segments = ['consumer', 'corporate', 'home_office']
ship_modes = ['first_class', 'same_day', 'second_class', 'standard_class']
```

Here, two lists are created: one for customer segments and another for shipping modes. These will be used later to generate combined column names.

### **3. Generate Group Shipping Names**

```python
group_shipping_names = [f"{segment}_{ship_mode}" 
                        for segment in segments 
                        for ship_mode in ship_modes]
```

This list comprehension creates 12 combined names (e.g., "consumer_first_class") by iterating through each segment and ship mode.

### **4. Read the Data**

```python
df = pd.read_excel("data.xlsx", sheet_name="Dirty 1", skiprows=1)
```

This line reads data from an Excel file named "data.xlsx", specifically from the sheet "Dirty 1", while skipping the first row which is assumed to be an irrelevant header.

### **5. Remove Unnecessary Columns**

```python
df = df.loc[:, ~df.columns.str.startswith('Unnamed')]
```

This filters out any columns starting with "Unnamed", which are typically artifacts from Excel, ensuring that only relevant data remains.

### **6. Rename First Column**

```python
df = df.rename(columns={'Ship Mode': 'order_id'})
```

The "Ship Mode" column is renamed to "order_id" because it actually contains order IDs rather than shipping modes.

### **7. Drop First Row**

```python
df = df.iloc[1:]
```

This drops the first row (index 0), which contains shipping mode labels that are redundant with the column names.

### **8. Rename Remaining Columns**

```python
df = df.set_axis(['order_id'] + group_shipping_names, axis=1)
```

New column names are assigned: "order_id" for the first column, followed by the 12 generated group shipping names.

### **9. Pivot to Long Format**

```python
df = df.melt(id_vars=['order_id'], var_name='group_shipping', value_name='value')
```

The DataFrame is transformed from a wide format to a long format, creating three columns: order_id, group_shipping, and value.

### **10. Drop NA Values**

```python
df = df.dropna(subset=['value'])
```

This removes any rows where the value column is NA, ensuring that only valid data remains for analysis.

### **11. Split Column Names**

```python
df = df.assign(segment=lambda x: x['group_shipping'].str.extract(r'(consumer|corporate|home_office)'),
               shipping_mode=lambda x: x['group_shipping'].str.extract(r'_(.+)'))
```

Using regular expressions, this extracts the segment and shipping mode from the group_shipping column (e.g., splitting "consumer_first_class").

### **12. Drop Temporary Column**

```python
df = df.drop(columns=['group_shipping'])
```

The temporary group_shipping column is removed after its information has been split into segment and shipping_mode.

### **13. Sort the Data**

```python
df = df.sort_values(['segment', 'shipping_mode', 'order_id'])
```

Finally, the DataFrame is sorted by segment, shipping_mode, and order_id to organize the output neatly.
