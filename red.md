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

## R Code

The R code uses `tidyverse` to read, clean, and transform the data. Below is the full implementation followed by a step-by-step explanation.

### R Implementation

```r
# Load required libraries
library(readxl)    # For reading Excel files
library(dplyr)     # For data manipulation (e.g., select, rename, etc.)
library(tidyr)     # For reshaping data (pivot_longer)
library(glue)      # For string interpolation
library(janitor)   # For cleaning column names

# Define segments and shipping modes for later use
segments <- c('consumer', 'corporate', 'home_office')
ship_modes <- c('first_class', 'same_day', 'second_class', 'standard_class')

# Create names for each combination of segment and shipping mode
group_shipping_names <- paste0(
  rep(segments, each = length(ship_modes)),  # Repeat each segment for every ship mode
  '_',
  rep(ship_modes, length(segments))          # Repeat each ship mode for every segment
)

# Define a regular expression pattern to extract the segment and shipping mode from the column names
regex <- glue::glue('({paste0(segments, collapse = "|")})_(.*)')

# Read the data from an Excel file (data.xlsx), specifically from the 'Dirty 1' sheet
# The first row will be skipped, and column names will be cleaned
df <- read_xlsx("data.xlsx", sheet = 'Dirty 1',
                skip = 1,            # Skip the first row
                .name_repair = make_clean_names)  # Clean column names to make them valid

# Clean and reshape the dataframe
df_cleaned <- df |>
  # Remove columns starting with "x" (often used to indicate unwanted columns in Excel data)
  select(!starts_with("x")) |>
  
  # Rename the 'ship_mode' column to 'order_id' (likely to match your desired output structure)
  rename(order_id = "ship_mode") |>
  
  # Remove the first row (typically header or metadata that is not part of the data)
  slice(-1) |>
  
  # Rename columns with dynamic names created earlier for segment and shipping modes
  rename_with(~c('order_id', group_shipping_names)) |>
  
  # Reshape the data by pivoting it longer, so each row represents a single 'segment' and 'shipping_mode' combination
  # The regex extracts the 'segment' and 'shipping_mode' from the column names
  pivot_longer(cols = -1, names_pattern = regex, 
               names_to = c('segment', 'shipping_mode'),  # Split column names into segment and shipping_mode
               values_drop_na = TRUE) |>
  
  # Arrange the data by 'segment', 'shipping_mode', and 'order_id' for neat organization
  arrange(segment, shipping_mode, order_id)

# Print the cleaned data
print(df_cleaned)

```

### Step-by-Step Explanation of R Code

### **1. Load Libraries**

```r
library(readxl)
library(dplyr)
library(tidyr)
library(glue)
library(janitor)
```

This code loads the necessary packages:

- **`readxl`**: For reading Excel files.
- **`dplyr`**: For data manipulation.
- **`tidyr`**: For reshaping data.
- **`glue`**: For string operations.
- **`janitor`**: For cleaning column names.


### **2. Define Segments and Ship Modes**

```r
segments &lt;- c('consumer', 'corporate', 'home_office')
ship_modes &lt;- c('first_class', 'same_day', 'second_class', 'standard_class')
```

Defines vectors for segments and shipping modes using the `c()` function:

- `segments`: Represents different customer segments.
- `ship_modes`: Represents various shipping modes.


### **3. Generate Group Shipping Names**

```r
group_shipping_names &lt;- paste0(
  rep(segments, each = length(ship_modes)),
  '_',
  rep(ship_modes, length(segments))
)
```

Combines segment and shipping mode names using `paste0()` and `rep()` to create 12 column names (e.g., `"consumer_first_class"`).

### **4. Create Regex**

```r
regex &lt;- glue::glue('({paste0(segments, collapse = "|")})_(.*)')
```

Constructs a regex pattern (e.g., `(consumer|corporate|home_office)_(.*)`) to split column names later.

### **5. Read the Data**

```r
df &lt;- read_xlsx("data.xlsx", sheet = 'Dirty 1',
                skip = 1,
                .name_repair = make_clean_names)
```

Reads the Excel file named `"data.xlsx"`:

- `sheet = 'Dirty 1'`: Specifies the sheet to read.
- `skip = 1`: Skips the first row, which is assumed to be irrelevant.
- `.name_repair = make_clean_names`: Cleans column names by converting spaces to underscores.


### **6. Remove Unnecessary Columns**

```r
df &lt;- df %&gt;% select(!starts_with("x"))
```

Drops columns starting with `"x"`, which are assumed artifacts from Excel.

### **7. Rename First Column**

```r
df &lt;- df %&gt;% rename(order_id = "ship_mode")
```

Renames the `"ship_mode"` column to `"order_id"` to reflect its actual content, which consists of order IDs.

### **8. Drop First Row**

```r
df &lt;- df %&gt;% slice(-1)
```

Removes the first row, which contains redundant shipping mode labels.

### **9. Rename Remaining Columns**

```r
df &lt;- df %&gt;% rename_with(~c('order_id', group_shipping_names))
```

Assigns new names to all columns:

- `"order_id"` for the first column.
- The 12 generated group shipping names for the remaining columns.


### **10. Pivot to Long Format**

```r
df &lt;- df %&gt;% pivot_longer(cols = -1, names_pattern = regex, 
                          names_to = c('segment', 'shipping_mode'),
                          values_drop_na = TRUE)
```

Transforms the DataFrame into a long format:

- `cols = -1`: Excludes the first column (`order_id`) from pivoting.
- `names_pattern = regex`: Uses the regex pattern to split column names into `segment` and `shipping_mode`.
- `values_drop_na = TRUE`: Drops rows with NA values in the pivoted columns.


### **11. Sort the Data**

```r
df &lt;- df %&gt;% arrange(segment, shipping_mode, order_id)
```

Sorts the data by:

1. `segment`
2. `shipping_mode`
3. `order_id`

missings symbols
---

# \#\#\# Step-by-Step Explanation of R Code

### **1. Load Libraries**

```r
library(readxl)
library(dplyr)
library(tidyr)
library(glue)
library(janitor)
```

This code loads the necessary packages:

- **`readxl`**: For reading Excel files.
- **`dplyr`**: For data manipulation.
- **`tidyr`**: For reshaping data.
- **`glue`**: For string operations.
- **`janitor`**: For cleaning column names.


### **2. Define Segments and Ship Modes**

```r
segments &lt;- c('consumer', 'corporate', 'home_office')
ship_modes &lt;- c('first_class', 'same_day', 'second_class', 'standard_class')
```

Defines vectors for segments and shipping modes using the `c()` function:

- `segments`: Represents different customer segments.
- `ship_modes`: Represents various shipping modes.


### **3. Generate Group Shipping Names**

```r
group_shipping_names &lt;- paste0(
  rep(segments, each = length(ship_modes)),
  '_',
  rep(ship_modes, length(segments))
)
```

Combines segment and shipping mode names using `paste0()` and `rep()` to create 12 column names (e.g., `"consumer_first_class"`).

### **4. Create Regex**

```r
regex &lt;- glue::glue('({paste0(segments, collapse = "|")})_(.*)')
```

Constructs a regex pattern (e.g., `(consumer|corporate|home_office)_(.*)`) to split column names later.

### **5. Read the Data**

```r
df &lt;- read_xlsx("data.xlsx", sheet = 'Dirty 1',
                skip = 1,
                .name_repair = make_clean_names)
```

Reads the Excel file named `"data.xlsx"`:

- `sheet = 'Dirty 1'`: Specifies the sheet to read.
- `skip = 1`: Skips the first row, which is assumed to be irrelevant.
- `.name_repair = make_clean_names`: Cleans column names by converting spaces to underscores.


### **6. Remove Unnecessary Columns**

```r
df &lt;- df %&gt;% select(!starts_with("x"))
```

Drops columns starting with `"x"`, which are assumed artifacts from Excel.

### **7. Rename First Column**

```r
df &lt;- df %&gt;% rename(order_id = "ship_mode")
```

Renames the `"ship_mode"` column to `"order_id"` to reflect its actual content, which consists of order IDs.

### **8. Drop First Row**

```r
df &lt;- df %&gt;% slice(-1)
```

Removes the first row, which contains redundant shipping mode labels.

### **9. Rename Remaining Columns**

```r
df &lt;- df %&gt;% rename_with(~c('order_id', group_shipping_names))
```

Assigns new names to all columns:

- `"order_id"` for the first column.
- The 12 generated group shipping names for the remaining columns.


### **10. Pivot to Long Format**

```r
df &lt;- df %&gt;% pivot_longer(cols = -1, names_pattern = regex, 
                          names_to = c('segment', 'shipping_mode'),
                          values_drop_na = TRUE)
```

Transforms the DataFrame into a long format:

- `cols = -1`: Excludes the first column (`order_id`) from pivoting.
- `names_pattern = regex`: Uses the regex pattern to split column names into `segment` and `shipping_mode`.
- `values_drop_na = TRUE`: Drops rows with NA values in the pivoted columns.


### **11. Sort the Data**

```r
df &lt;- df %&gt;% arrange(segment, shipping_mode, order_id)
```

Sorts the data by:

1. `segment`
2. `shipping_mode`
3. `order_id`

missings symbols

It looks like the code you provided has HTML entities (like `&lt;` and `&gt;`) instead of the actual symbols for less than (`&lt;`) and greater than (`&gt;`). Here's the corrected version of your markdown with the proper symbols:

### Step-by-Step Explanation of R Code

### **1. Load Libraries**

```r
library(readxl)
library(dplyr)
library(tidyr)
library(glue)
library(janitor)
```

This code loads the necessary packages:

- **`readxl`**: For reading Excel files.
- **`dplyr`**: For data manipulation.
- **`tidyr`**: For reshaping data.
- **`glue`**: For string operations.
- **`janitor`**: For cleaning column names.

---

### **2. Define Segments and Ship Modes**

```r
segments &lt;- c('consumer', 'corporate', 'home_office')
ship_modes &lt;- c('first_class', 'same_day', 'second_class', 'standard_class')
```

Defines vectors for segments and shipping modes using the `c()` function:

- `segments`: Represents different customer segments.
- `ship_modes`: Represents various shipping modes.

---

### **3. Generate Group Shipping Names**

```r
group_shipping_names &lt;- paste0(
  rep(segments, each = length(ship_modes)),
  '_',
  rep(ship_modes, length(segments))
)
```

Combines segment and shipping mode names using `paste0()` and `rep()` to create 12 column names (e.g., `"consumer_first_class"`).

---

### **4. Create Regex**

```r
regex &lt;- glue::glue('({paste0(segments, collapse = "|")})_(.*)')
```

Constructs a regex pattern (e.g., `(consumer|corporate|home_office)_(.*)`) to split column names later.

---

### **5. Read the Data**

```r
df &lt;- read_xlsx("data.xlsx", sheet = 'Dirty 1',
                skip = 1,
                .name_repair = make_clean_names)
```

Reads the Excel file named `"data.xlsx"`:

- `sheet = 'Dirty 1'`: Specifies the sheet to read.
- `skip = 1`: Skips the first row, which is assumed to be irrelevant.
- `.name_repair = make_clean_names`: Cleans column names by converting spaces to underscores.

---

### **6. Remove Unnecessary Columns**

```r
df &lt;- df %&gt;% select(!starts_with("x"))
```

Drops columns starting with `"x"`, which are assumed artifacts from Excel.

---

### **7. Rename First Column**

```r
df &lt;- df %&gt;% rename(order_id = "ship_mode")
```

Renames the `"ship_mode"` column to `"order_id"` to reflect its actual content, which consists of order IDs.

---

### **8. Drop First Row**

```r
df &lt;- df %&gt;% slice(-1)
```

Removes the first row, which contains redundant shipping mode labels.

---

### **9. Rename Remaining Columns**

```r
df &lt;- df %&gt;% rename_with(~c('order_id', group_shipping_names))
```

Assigns new names to all columns:

- `"order_id"` for the first column.
- The 12 generated group shipping names for the remaining columns.

---

### **10. Pivot to Long Format**

```r
df &lt;- df %&gt;% pivot_longer(cols = -1, names_pattern = regex, 
                          names_to = c('segment', 'shipping_mode'),
                          values_drop_na = TRUE)
```

Transforms the DataFrame into a long format:

- `cols = -1`: Excludes the first column (`order_id`) from pivoting.
- `names_pattern = regex`: Uses the regex pattern to split column names into `segment` and `shipping_mode`.
- `values_drop_na = TRUE`: Drops rows with NA values in the pivoted columns.

---

### **11. Sort the Data**

```r
df &lt;- df %&gt;% arrange(segment, shipping_mode, order_id)
```

Sorts the data by:

1. `segment`
2. `shipping_mode`
3. `order_id`

---

This ensures that the output is organized in a clear and logical manner.

## Example Output Data

Below is a screenshot of the cleaned Excel file. 

![Screenshot](https://raw.githubusercontent.com/madhanlibo/data-cleaning/main/pic/Screenshot%202025-03-31%20182523.png)



---
