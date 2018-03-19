
# Analysis findings

+ Based on Gender Demographics, males account for 81.15% of all players that purchase.
+ Based on Purchasing Analysis, males account for the majority (81.68%) of total purchase value.
  Females account for 16.74% of total purchase value.
  This might indicate that there is a correlation between gender and purchases
+ Most players are between 15 and 29 years old (77.83%) while most players are between 20 and 24 years old (45.20%)
+ Players between 20 and 24 spend in total most on purchases (USD 978.77)
  Per player however, players between 20 and 24 spend least on items (USD 3.78).
  Players being 40 years old or older spend least in total, however per player they spend most (USD 4.89)
+ With one exception (USD 17.06), no one spends more then USD 17.00 on items. 
+ There seems to be no correlation between Most Popular Items and Most Profitable Items.
  There's only one item (Retribution Axe) that appears in both top 5 lists.
  


```python
# Import Dependencies
import pandas as pd
import os
from IPython.display import display
```


```python
# Load data into dataframe
input_file = input("Enter the name of the JSON file you want to analyze (without extension): ") + ".JSON"
# Set file path (input file should be located on the same level as the folder raw_data)
filepath = os.path.join('raw_data', input_file)
purchases_df = pd.read_json(filepath)
```

    Enter the name of the JSON file you want to analyze (without extension): purchase_data
    


```python
# To check for NaN values per column, use:
purchases_df.isnull().sum()
# If the above returns NaN values for any item related property (Item ID, Item Name, Price) you might want to consider
# to check whether the missing item data is available in other rows. 
# Use purchases_df.dropna() to delete rows having NaN values
```




    Age          0
    Gender       0
    Item ID      0
    Item Name    0
    Price        0
    SN           0
    Age Group    0
    dtype: int64



## Player Count


```python
# Total Number of Players
tot_players = len(purchases_df["SN"].unique())
tot_players_df = pd.DataFrame({"Total Players":[tot_players]})
```

## Purchasing Analysis (Total)


```python
# Calculations
total_unique_items = len(purchases_df["Item ID"].unique())
avg_purchase_price = purchases_df["Price"].mean()
total_purchases = len(purchases_df)
total_revenue = purchases_df["Price"].sum().mean()
# create dataframe
purchase_analysis_df = pd.DataFrame({"Number of Unique Items":[total_unique_items],
                                     "Average Price":[avg_purchase_price],
                                     "Number of Purchases":[total_purchases],
                                     "Total Revenue":[total_revenue]})
# Apply formatting
purchase_analysis_df["Average Price"] = purchase_analysis_df["Average Price"].map("${0:,.2f}".format)
purchase_analysis_df["Total Revenue"] = purchase_analysis_df["Total Revenue"].map("${0:,.2f}".format)
# Rearrange columns
columnsTitles = ["Number of Unique Items","Average Price","Number of Purchases","Total Revenue"]
purchase_analysis_df = purchase_analysis_df.reindex(columns=columnsTitles)
```

## Gender Demographics


```python
# Calculations
gender_count = purchases_df.groupby(["Gender"])["SN"].nunique()
gender_perc = 100*gender_count/tot_players
# Create dataframe
gender_demographics_df = pd.DataFrame({"Total Count":gender_count,"Percentage of Players":gender_perc})
# Apply formatting
gender_demographics_df["Percentage of Players"] = gender_demographics_df["Percentage of Players"].map("{0:,.2f}".format)
# Rearrange columns
columnsTitles = ["Percentage of Players", "Total Count"]
gender_demographics_df = gender_demographics_df.reindex(columns=columnsTitles)
# Apply sorting
gender_demographics_df= gender_demographics_df.sort_values("Total Count", ascending = False, inplace = False)
# Remove index header name
gender_demographics_df = gender_demographics_df.rename_axis(None)
```

## Purchasing Analysis (Gender)


```python
# Calculations
gender_purchase_total = purchases_df.groupby(["Gender"]).sum()["Price"]
gender_average = purchases_df.groupby(["Gender"]).mean()["Price"]
gender_counts = purchases_df.groupby(["Gender"]).count()["Price"]
normalized_total = gender_purchase_total / gender_demographics_df["Total Count"]
# Generate dataframe
purch_data_gender_df = pd.DataFrame({"Purchase Count": gender_counts,
                                     "Average Purchase Price": gender_average,
                                     "Total Purchase Value": gender_purchase_total,
                                     "Normalized Totals": normalized_total})
# Rearrange columns
columnsTitles = ["Purchase Count", "Average Purchase Price", "Total Purchase Value","Normalized Totals"]
purch_data_gender_df = purch_data_gender_df.reindex(columns=columnsTitles)
# Apply formatting
format_list = ["Average Purchase Price","Normalized Totals","Total Purchase Value"]
key_format = "${0:.2f}"
for key, value in purch_data_gender_df.items():
    if key in format_list:
        purch_data_gender_df[key] = value.map(key_format.format)
```

## Age Demographics


```python
# Define age bins and groups
bins = [0, 10, 15, 20, 25, 30, 35, 40, 150]
group_names = ["<10", "10-14", "15-19", "20-24", "25-29","30-34","35-39", "40+"]
purchases_df["Age Group"] = pd.cut(purchases_df["Age"], bins, right = False, labels=group_names)
# Calculations
age_group_total = purchases_df.groupby(["Age Group"])["SN"].nunique()
age_group_perc = age_group_total/tot_players
# Create dataframe
age_group_df = pd.DataFrame({"Total Count": age_group_total, "Percentage of Players": 100*age_group_perc})
# Apply formatting
age_group_df["Percentage of Players"] = age_group_df["Percentage of Players"].map("{0:,.2f}".format)
# Rearrange columns
columnsTitles = ["Percentage of Players","Total Count"]
age_group_df = age_group_df.reindex(columns=columnsTitles)
# Remove index header name
age_group_df = age_group_df.rename_axis(None)
```

## Purchasing Analysis (Age Group)


```python
# Calculations
unique_persons = purchases_df.groupby(["Age Group"])["SN"].nunique()
group_purch = purchases_df.groupby(["Age Group"])["Item ID"].count()
group_avg_price = purchases_df.groupby(["Age Group"])["Price"].mean().round(2)
group_total = purchases_df.groupby(["Age Group"])["Price"].sum()
normalized_total = group_total/unique_persons
# Create dataframe
purch_analysis_group_df = pd.DataFrame({"Purchase Count":group_purch,
                                        "Average Purchase Price":group_avg_price,
                                        "Total Purchase Value":group_total,
                                        "Normalized Totals":normalized_total})
# Rearrange columns
columnsTitles = ["Purchase Count","Average Purchase Price","Total Purchase Value","Normalized Totals"]
purch_analysis_group_df = purch_analysis_group_df.reindex(columns=columnsTitles)
# Apply formatting
format_list = ["Average Purchase Price","Normalized Totals","Total Purchase Value"]
key_format = "${0:.2f}"
for key, value in purch_analysis_group_df.items():
    if key in format_list:
        purch_analysis_group_df[key] = value.map(key_format.format)
```

## Top Spenders


```python
# Calculations
total_spendings = purchases_df.groupby(["SN"])["Price"].sum()
total_buyers = purchases_df.groupby(["SN"])["Item ID"].count()
avg_spending = total_spendings/total_buyers
# Create dataframe
purch_analysis_df = pd.DataFrame({"Purchase Count":total_buyers,
                                  "Average Purchase Price":avg_spending,
                                  "Total Purchase Value":total_spendings})
# Rearrange columns
columnsTitles=["Purchase Count","Average Purchase Price","Total Purchase Value"]
purch_analysis_df=purch_analysis_df.reindex(columns=columnsTitles)
# Apply sorting
purch_analysis_df= purch_analysis_df.sort_values(["Total Purchase Value","Purchase Count"], ascending = [False, False])
# Apply formatting
purch_analysis_df["Total Purchase Value"] = purch_analysis_df["Total Purchase Value"].map("${0:,.2f}".format)
purch_analysis_df["Average Purchase Price"] = purch_analysis_df["Average Purchase Price"].map("${0:,.2f}".format)
```

## Most Popular Items


```python
# Create dataframe for item data
top_items = purchases_df.groupby(["Item ID", "Item Name","Price"])["Item ID"].count()
top_items_df = pd.DataFrame(top_items)
top_items_df = top_items_df.rename(columns={"Item ID":"Purchase Count"})
top_items_df.reset_index(inplace=True) #reset index as you can't merge on an index
# Create dataframe for purchases (revenue) data
total_revenue = purchases_df.groupby(["Item ID"])["Price"].sum()
total_revenue_df = pd.DataFrame(total_revenue)
total_revenue_df = total_revenue_df.rename(columns={"Price":"Total Purchase Value"})
total_revenue_df.reset_index(inplace=True) #reset index as you can't merge on an index
# Merge dataframes
merge_top_items_df = pd.merge(top_items_df, total_revenue_df, on="Item ID", how="inner")
merge_top_items_df.set_index(["Item ID", "Item Name"], inplace=True)
# Rename/rearrange columns
merge_top_items_df = merge_top_items_df.rename(columns={"Price":"Item Price"})
total_revenue_df.reset_index(inplace=True) 
columnsTitles=["Purchase Count","Item Price","Total Purchase Value"]
merge_top_items_df = merge_top_items_df.reindex(columns=columnsTitles)
# Create new dataframe so we can use current dataframe for Most Profitable Items Analysis
popularity_df = merge_top_items_df.copy(deep = True)
# Apply sorting
popularity_df.sort_values(["Purchase Count","Total Purchase Value"], ascending = [False, False], inplace = True)
# Apply formatting
popularity_df["Item Price"] = popularity_df["Item Price"].map("${0:,.2f}".format)
popularity_df["Total Purchase Value"] = popularity_df["Total Purchase Value"].map("${0:,.2f}".format)
```

## Most Profitable Items


```python
# Apply sorting
merge_top_items_df.sort_values(["Total Purchase Value", "Purchase Count"], ascending = [False, False], inplace = True)
# Apply formatting
merge_top_items_df["Item Price"] = merge_top_items_df["Item Price"].map("${0:,.2f}".format)
merge_top_items_df["Total Purchase Value"] = merge_top_items_df["Total Purchase Value"].map("${0:,.2f}".format)
```


```python
print("ANALYSIS REPORT")
print('='*15)
print("Player Count")
display(tot_players_df)
print("Purchasing Analysis (Total)")
display(purchase_analysis_df)
print("Gender Demographics")
display(gender_demographics_df)
print("Purchasing Analysis (Gender)")
display(purch_data_gender_df)
print("Age Demographics")
display(age_group_df)
print("Purchasing Analysis (Age Group)")
display(purch_analysis_group_df)
print("Top Spenders")
display(purch_analysis_df.head())
print("Most Popular Items")
display(popularity_df.head())
print("Most Profitable Items")
display(merge_top_items_df.head())
```

    ANALYSIS REPORT
    ===============
    Player Count
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>573</td>
    </tr>
  </tbody>
</table>
</div>


    Purchasing Analysis (Total)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Price</th>
      <th>Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>183</td>
      <td>$2.93</td>
      <td>780</td>
      <td>$2,286.33</td>
    </tr>
  </tbody>
</table>
</div>


    Gender Demographics
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>81.15</td>
      <td>465</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>17.45</td>
      <td>100</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>1.40</td>
      <td>8</td>
    </tr>
  </tbody>
</table>
</div>


    Purchasing Analysis (Gender)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>136</td>
      <td>$2.82</td>
      <td>$382.91</td>
      <td>$3.83</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>633</td>
      <td>$2.95</td>
      <td>$1867.68</td>
      <td>$4.02</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>11</td>
      <td>$3.25</td>
      <td>$35.74</td>
      <td>$4.47</td>
    </tr>
  </tbody>
</table>
</div>


    Age Demographics
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>3.32</td>
      <td>19</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>4.01</td>
      <td>23</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>17.45</td>
      <td>100</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>45.20</td>
      <td>259</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>15.18</td>
      <td>87</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>8.20</td>
      <td>47</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>4.71</td>
      <td>27</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>1.92</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>


    Purchasing Analysis (Age Group)
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
      <th>Normalized Totals</th>
    </tr>
    <tr>
      <th>Age Group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;10</th>
      <td>28</td>
      <td>$2.98</td>
      <td>$83.46</td>
      <td>$4.39</td>
    </tr>
    <tr>
      <th>10-14</th>
      <td>35</td>
      <td>$2.77</td>
      <td>$96.95</td>
      <td>$4.22</td>
    </tr>
    <tr>
      <th>15-19</th>
      <td>133</td>
      <td>$2.91</td>
      <td>$386.42</td>
      <td>$3.86</td>
    </tr>
    <tr>
      <th>20-24</th>
      <td>336</td>
      <td>$2.91</td>
      <td>$978.77</td>
      <td>$3.78</td>
    </tr>
    <tr>
      <th>25-29</th>
      <td>125</td>
      <td>$2.96</td>
      <td>$370.33</td>
      <td>$4.26</td>
    </tr>
    <tr>
      <th>30-34</th>
      <td>64</td>
      <td>$3.08</td>
      <td>$197.25</td>
      <td>$4.20</td>
    </tr>
    <tr>
      <th>35-39</th>
      <td>42</td>
      <td>$2.84</td>
      <td>$119.40</td>
      <td>$4.42</td>
    </tr>
    <tr>
      <th>40+</th>
      <td>17</td>
      <td>$3.16</td>
      <td>$53.75</td>
      <td>$4.89</td>
    </tr>
  </tbody>
</table>
</div>


    Top Spenders
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Average Purchase Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>5</td>
      <td>$3.41</td>
      <td>$17.06</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>4</td>
      <td>$3.39</td>
      <td>$13.56</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>4</td>
      <td>$3.18</td>
      <td>$12.74</td>
    </tr>
    <tr>
      <th>Haellysu29</th>
      <td>3</td>
      <td>$4.24</td>
      <td>$12.73</td>
    </tr>
    <tr>
      <th>Eoda93</th>
      <td>3</td>
      <td>$3.86</td>
      <td>$11.58</td>
    </tr>
  </tbody>
</table>
</div>


    Most Popular Items
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>39</th>
      <th>Betrayal, Whisper of Grieving Widows</th>
      <td>11</td>
      <td>$2.35</td>
      <td>$25.85</td>
    </tr>
    <tr>
      <th>84</th>
      <th>Arcane Gem</th>
      <td>11</td>
      <td>$2.23</td>
      <td>$24.53</td>
    </tr>
    <tr>
      <th>34</th>
      <th>Retribution Axe</th>
      <td>9</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
    <tr>
      <th>31</th>
      <th>Trickster</th>
      <td>9</td>
      <td>$2.07</td>
      <td>$18.63</td>
    </tr>
    <tr>
      <th>13</th>
      <th>Serenity</th>
      <td>9</td>
      <td>$1.49</td>
      <td>$13.41</td>
    </tr>
  </tbody>
</table>
</div>


    Most Profitable Items
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Purchase Count</th>
      <th>Item Price</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>34</th>
      <th>Retribution Axe</th>
      <td>9</td>
      <td>$4.14</td>
      <td>$37.26</td>
    </tr>
    <tr>
      <th>115</th>
      <th>Spectral Diamond Doomblade</th>
      <td>7</td>
      <td>$4.25</td>
      <td>$29.75</td>
    </tr>
    <tr>
      <th>32</th>
      <th>Orenmir</th>
      <td>6</td>
      <td>$4.95</td>
      <td>$29.70</td>
    </tr>
    <tr>
      <th>103</th>
      <th>Singed Scalpel</th>
      <td>6</td>
      <td>$4.87</td>
      <td>$29.22</td>
    </tr>
    <tr>
      <th>107</th>
      <th>Splitter, Foe Of Subtlety</th>
      <td>8</td>
      <td>$3.61</td>
      <td>$28.88</td>
    </tr>
  </tbody>
</table>
</div>

