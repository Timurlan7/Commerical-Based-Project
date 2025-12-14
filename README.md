ASSESSMENT 1 - COMMERCIAL BASED PROJECT
Files required:
1. `project_transactions.csv`
2. `hh_demographic.csv`
3. `product.csv`

Overall Objectives:
1. Read data from CSV files
2. Explore large datasets
3. Create new columns for assisting in analysis
4. Combine multiple DataFrames
5. Filter, sort and aggregate data to pinpoint and summarise information
6. Analyse time series with datetime fields
7. Build plots to communicate key insights
8. Optimise import workflow
9. Summary tables

___
## 1. SETUP & EXPLORATORY
1. Import libraries
2. Load data from `project_transactions.csv`
3. Specify columns to be used in the DataFrame:
    * household_key
    * BASKET_ID
    * DAY
    * PRODUCT_ID
    * QUANTITY
    * SALES_VALUE
4. Cast the datatypes as following:
    * "DAY": "Int16"
    * "QUANTITY": "Int32"
    * "PRODUCT_ID": "Int32"
5. Run .describe().round()
6. Info and memory usage check
7. Run .isna().sum() to check missing value counts
8. Overwrite the DataFrame, t to create a 'DATE' Column and drop the 'DAY' column via .assign()

#1-5
import pandas as pd
import numpy as np   
path = 'project_transactions.csv'
cols = ["household_key", "BASKET_ID", "DAY", "PRODUCT_ID", "QUANTITY", "SALES_VALUE"]
dtypes = {"DAY": "Int16", "QUANTITY": "Int32", "PRODUCT_ID": "Int32"}

t = pd.read_csv(path, usecols=cols, dtype=dtypes)
t.describe().round()

#6
t.info(memory_usage='deep')
<img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/8932301b-dc7b-4fab-9d37-0c025dc99255" />


#7
t.isna().sum()

#8
t = t.assign(DATE = (pd.to_datetime("2016", format='%Y') + pd.to_timedelta(t["DAY"].sub(1).astype(str) + " days"))).drop(["DAY"], axis=1)
t.head()
<img width="1218" height="548" alt="Снимок экрана 2025-12-14 в 3 51 15 PM" src="https://github.com/user-attachments/assets/d94ea71d-5afd-44e8-b54e-ae7817f6dccf" />


___
## 2. TIME BASED ANALYSIS
1. Question: Are sales growing over time?
    * Plot the sum of sales by month:
        * Set a date index
        * Use values from the 'SALES_VALUE' column
        * Calculate a monthly sum via .resample()
        * Plot the default line graph
2. Plot the same series for the period from 2017 January to latest:
    * Filter above plot to specified date range with row slice in .loc[]
3. Plot the sum of sales 2016 vs 2017 sales:
    * Calculate a monthly sum via .resample()
    * Create a new column 's2016' with .assign()
        * The new column 's2016' holds values of monthly sales shifted a year (12 rows/months) --> .shift(12)
4. Plot total sales by day of week:
    * Groupby those transactions via .dt.dayofweek
    * Calculate sum
    * Plot a bar chart

#1 Are sales growing over time?
t.set_index('DATE').loc[:, 'SALES_VALUE'].resample('ME').sum().plot()
<Axes: xlabel='DATE'>
<img width="578" height="449" alt="image" src="https://github.com/user-attachments/assets/2656f77b-7621-4ffb-aa2f-dffb524357d3" />


#2 Plot the same series for the period from 2017 January to latest
t.set_index('DATE').loc['2017':, 'SALES_VALUE'].resample('ME').sum().plot()
<Axes: xlabel='DATE'>
<img width="591" height="449" alt="image" src="https://github.com/user-attachments/assets/37e2eaa3-498b-47e8-89ea-d6e0c9ab41cc" />


#3 Plot the sum of sales 2016 vs 2017 sales
(t.set_index('DATE').loc[:, ['SALES_VALUE']].resample('ME').sum().assign(s2016=lambda x: x['SALES_VALUE'].shift(12)).loc['2017'].plot())
<Axes: xlabel='DATE'>
<img width="591" height="449" alt="image" src="https://github.com/user-attachments/assets/dfcf49bb-4459-4f51-8b3a-0adb25427251" />


#4 Plot total sales by day of week
t.groupby(t['DATE'].dt.dayofweek).agg({'SALES_VALUE': 'sum'}).plot.bar()
<Axes: xlabel='DATE'>
<img width="547" height="442" alt="image" src="https://github.com/user-attachments/assets/00f2e457-4b4b-4b88-aab8-c654c70c4181" />


___
# 3. DEMOGRAPHICS
1. Load data from `hh_demographic.csv`
2. Specify columns to be used in the DataFrame:
    * AGE_DESC
    * INCOME_DESC
    * household_key
    * HH_COMP_DESC
3. Cast datatypes as following:
    * "AGE_DESC": "category"
    * "INCOME_DESC": "category"
    * "HH_COMP_DESC":"category"
4. Info and memory usage check on the DataFrame
5. Show total sales (named 'hhsales') for the household dataframe via a new column 'SALES_VALUE':
    * Groupby transactions table 't' by household_id
    * Calculate the aggregate sum of 'SALES_VALUE' by household
6. Combine household sales (hhsales) to demographics DataFrame (d) via .merge(), inner join & on household_key
    * Name the combined DataFrame as 'hhsales_d'
7. Info and memory usage check on the new DataFrame (hhsales_d)
8. Using the combined DataFrame (hhsales_d), plot a bar chart to show the aggregated sum of sales by age group
    * Note: use the parameter, observed=True within .groupby() to avoid deprecation warning
9. Using the combined DataFrame (hhsales_d), plot a bar chart to show the aggregated sum of sales by income (ordered by magnitude; descending order)
    * Note: use the parameter, observed=True within .groupby() to avoid deprecation warning
10. Question: Which of the demographics has the highest average sales? (The mean household spend by Age Description and HH Composition)
    * Create a heatmap on a pivot table by:
        * Use .pivot_table() on hhsales_d with 'AGE_DESC' as its index
        * Use 'HH_COMP_DESC' for columns
        * Find the aggregated mean for household sales
        * Use the parameter, margins=True
        * Format with a heatmap across all cells via .style.background_gradient(cmap="RdYlGn", axis=None)
11. Delete DataFrames: hhsales & hhsales_d

#1-3
dpath = 'hh_demographic.csv'
dcols = ["AGE_DESC", "INCOME_DESC", "household_key", "HH_COMP_DESC"]
ddtypes = {"AGE_DESC": "category", "INCOME_DESC": "category", "HH_COMP_DESC":"category"}

d = pd.read_csv(dpath, usecols=dcols, dtype=ddtypes)
d.head()


#4
d.info(memory_usage='deep')
<img width="970" height="670" alt="Снимок экрана 2025-12-14 в 3 56 31 PM" src="https://github.com/user-attachments/assets/8f51f06e-9d74-4878-8e99-ef4bad647d85" />


#5 Show total sales (named 'hhsales') for the household dataframe via a new column 'SALES_VALUE'
hhsales = t.groupby('household_key').agg({'SALES_VALUE': 'sum'})
hhsales

#6 Combine household sales (hhsales) to demographics DataFrame (d) via .merge(), inner join & on household_key
<img width="905" height="736" alt="Снимок экрана 2025-12-14 в 3 57 04 PM" src="https://github.com/user-attachments/assets/868d4d7d-ad4a-4cb9-b158-88dad147a1b9" />


hhsales_d = d.merge(hhsales, how='inner', left_on='household_key', right_on='household_key')
hhsales_d.head()

#7 Info and memory usage check on the new DataFrame (hhsales_d)
hhsales_d.info(memory_usage='deep')
<img width="415" height="246" alt="Снимок экрана 2025-12-14 в 3 57 46 PM" src="https://github.com/user-attachments/assets/845b3b08-3af7-4540-b171-e999cfd726e0" />


#8 Using the combined DataFrame (hhsales_d), plot a bar chart to show the aggregated sum of sales by age group
(hhsales_d.groupby(['AGE_DESC']).agg({'SALES_VALUE': 'sum'}).plot.bar())
<Axes: xlabel='AGE_DESC'>
<img width="547" height="474" alt="image" src="https://github.com/user-attachments/assets/5cc1baa1-39ec-4d9b-9d3a-8341d0e8f4af" />


#9 Using the combined DataFrame (hhsales_d), plot a bar chart to show the aggregated sum of sales by income (ordered by magnitude; descending order)
(hhsales_d.groupby(['INCOME_DESC']).agg({'SALES_VALUE': 'sum'}).sort_values('SALES_VALUE', ascending=False).plot.bar())
<Axes: xlabel='INCOME_DESC'>
<img width="578" height="491" alt="image" src="https://github.com/user-attachments/assets/52b70c4d-0d17-4652-9d2a-a73850377dc9" />


#10 Question: Which of the demographics has the highest average sales? (The mean household spend by Age Description and HH Composition)
(hhsales_d.pivot_table(index='AGE_DESC', columns='HH_COMP_DESC', values='SALES_VALUE', aggfunc='mean',margins=True).style.background_gradient(cmap='RdYlGn', axis=None))
<img width="885" height="244" alt="Снимок экрана 2025-12-14 в 3 59 42 PM" src="https://github.com/user-attachments/assets/9e490149-8624-4642-8d69-8e3bdb1b5ada" />


#11 Delete DataFrames: hhsales & hhsales_d
del hhsales
del hhsales_d

___
# 4. PRODUCT DEMOGRAPHICS
1. Load data from `product.csv`
2. Specify columns to be used in the DataFrame:
    * PRODUCT_ID
    * DEPARTMENT
3. Cast datatypes as following:
    * "PRODUCT_ID": "Int32"
    * "DEPARTMENT": "category"
4. Combine three DataFrames (t, d, p) with an inner join:
    * .merge() DataFrame 'd' on household_key
    * .merge() DataFrame 'p' on PRODUCT_ID
5. Info and memory usage check on the new DataFrame (tdp)
6. Question: Which category does the youngest demographic perform well?
    * Create a heatmap on a pivot table by:
        * Use .pivot_table() on tdp with 'DEPARTMENT' as its index
        * Use 'AGE_DESC' for columns
        * Use 'SALES_VALUE' as values
        * Find the aggregated sum of sales
        * Format with a heatmap across all cells via .style.background_gradient(cmap="RdYlGn", axis=1)

#1-3
ppath = 'product.csv'
pcols = ["PRODUCT_ID", "DEPARTMENT"]
pdtypes = {"PRODUCT_ID": "Int32", "DEPARTMENT": "category"}

p = pd.read_csv(ppath, usecols=pcols, dtype=pdtypes)
p.head()

#4 Combine three DataFrames (t, d, p) with an inner join
tdp=t.merge(d, how='inner', left_on='household_key', right_on='household_key').merge(p, how='inner', left_on='PRODUCT_ID', right_on='PRODUCT_ID')
tdp.head()
<img width="1150" height="617" alt="Снимок экрана 2025-12-14 в 4 00 11 PM" src="https://github.com/user-attachments/assets/de53c51b-8391-49f5-bce6-72707a0fddcd" />


#5 Info and memory usage check on the new DataFrame (tdp)
tdp.info(memory_usage='deep')
<img width="553" height="332" alt="Снимок экрана 2025-12-14 в 4 01 00 PM" src="https://github.com/user-attachments/assets/c5d2100c-d20c-43d6-8772-176b828b1ee6" />


#6 Question: Which category does the youngest demographic perform well? (Answer: Alcohol)
(tdp.pivot_table(index='DEPARTMENT', columns='AGE_DESC', values='SALES_VALUE', aggfunc='sum',margins=False).style.background_gradient(cmap='RdYlGn', axis=1))
<img width="820" height="787" alt="Снимок экрана 2025-12-14 в 4 02 17 PM" src="https://github.com/user-attachments/assets/05a80c73-60f0-4218-a505-646a53c50518" />
<img width="800" height="301" alt="Снимок экрана 2025-12-14 в 4 02 33 PM" src="https://github.com/user-attachments/assets/8e53612a-74f3-4b68-8dc7-ba4331b992db" />


___
# 5. EXPORT
Export the pivot table (tdp):
* Excel file name: **cat_sales_dg.xlsx**
* Worksheet name: **sales_pivot**

# Export the pivot table created from above to an excel file
tdp.to_excel('cat_sales_dg.xlsx', sheet_name='sales_pivot')
