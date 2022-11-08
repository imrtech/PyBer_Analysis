# PyBer_Analysis

## Project Overview

In this project we analyze and visualize ridesharing data from January through April, 2019 by Pyber, a Python-based ridesharing app company. Our analysis will help the company improve access to ride-sharing services and determine affordability for underserved neighborhoods.

The analysis takes into account the following items based on the different city types: Urban, Suburban and Rural.

- the percentage of total rides,
- the percentage of total drivers,
- the percentage of total fares,
- the average fare per ride and driver,
- the total fare by city type.


## Results

### Load and Read the CSV files

We used Matplotlib to graph our visualizations and used two datasets, city_data.csv and ride_data_csv.

``` 
# Add Matplotlib inline magic command
%matplotlib inline
# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import warnings
warnings.filterwarnings('ignore')

# File to Load (Remember to change these)
city_data_to_load = "Resources/city_data.csv"
ride_data_to_load = "Resources/ride_data.csv"

# Read the City and Ride Data
city_data_df = pd.read_csv(city_data_to_load)
ride_data_df = pd.read_csv(ride_data_to_load)
```

### Merge the DataFrames

We then merged the city and ride dataframes. We merged on the city column since both dataframes have that in common and contain the same data. 

```
# Combine the data into a single dataset
pyber_data_df = pd.merge(ride_data_df, city_data_df, how="left", on=["city", "city"])

# Display the data table for preview
pyber_data_df.head()
```

 ![This is an image](/analysis/Screenshot1.png)


### Deliverable 1: Get a Summary DataFrame

The next steps were to create a series of data to extract the total number of rides, total number of drivers, and the total fares for each city type. 

Using the groupby function we created the ride_count_by_city series to get the total rides for each city type.

```
#  1. Get the total rides for each city type
ride_count_by_city = pyber_data_df.groupby('type').count().ride_id
ride_count_by_city

type
Rural        125
Suburban     625
Urban       1625
Name: ride_id, dtype: int64

```
We created the driver_sum_by_city series, again using the groupby function, to get the total drivers for each city type.
```
# 2. Get the total drivers for each city type
driver_sum_by_city = city_data_df.groupby('type').sum().driver_count
driver_sum_by_city

type
Rural         78
Suburban     490
Urban       2405
Name: driver_count, dtype: int64
```

We created the fare_sum_by_city series, again using the groupby function, to get the total amount of fares for each city type.

```
#  3. Get the total amount of fares for each city type
fare_sum_by_city = pyber_data_df.groupby('type').sum().fare
fare_sum_by_city

type
Rural        4327.93
Suburban    19356.33
Urban       39854.38
Name: fare, dtype: float64
```
``` 
#  4. Get the average fare per ride for each city type. 
avg_fare_per_ride_by_city = pyber_data_df.groupby('type').mean().fare
avg_fare_per_ride_by_city

type
Rural       34.623440
Suburban    30.970128
Urban       24.525772
Name: fare, dtype: float64
```

```
# 5. Get the average fare per driver for each city type. 
avg_fare_per_driver_by_city = pyber_data_df.groupby('type').sum().fare/driver_sum_by_city
avg_fare_per_driver_by_city

type
Rural       55.486282
Suburban    39.502714
Urban       16.571468
dtype: float64

```
Finally, we created a summary DataFrame

```
#  6. Create a PyBer summary DataFrame. 
pyber_summary_df = pd.DataFrame()

pyber_summary_df["Total Rides"] = ride_count_by_city
pyber_summary_df["Total Drivers"] = driver_sum_by_city
pyber_summary_df["Total Fares"] = fare_sum_by_city
pyber_summary_df["Average Fare per Ride"] = avg_fare_per_ride_by_city
pyber_summary_df["Average Fare per Driver"] = avg_fare_per_driver_by_city

pyber_summary_df
```

![This is an image](/analysis/Screenshot2.png)

```
#  7. Cleaning up the DataFrame. Delete the index name
pyber_summary_df.index.name = None

```
```
#  8. Format the columns.
pyber_summary_df["Total Rides"] = pyber_summary_df["Total Rides"].map("{:,}".format)
pyber_summary_df["Total Drivers"] = pyber_summary_df["Total Drivers"].map("{:,}".format)
pyber_summary_df["Total Fares"] = pyber_summary_df["Total Fares"].map("${:,.2f}".format)
pyber_summary_df["Average Fare per Ride"] = pyber_summary_df["Average Fare per Ride"].map("${:.2f}".format)
pyber_summary_df["Average Fare per Driver"] = pyber_summary_df["Average Fare per Driver"].map("${:.2f}".format)

pyber_summary_df

```
![This is an image](/analysis/Screenshot3.png)



### Deliverable 2. Create a multiple line plot that shows the total weekly of the fares for each type of city.

With our current analysis we are able to create a multiple-line graph that shows the total fares for each week by city type.
We ran the following code to prepare the data for graphing.

```
# 1. Read the merged DataFrame
pyber_data_df.head()

```

![This is an image](/analysis/Screenshot4.png)


```
# 2. Using groupby() to create a new DataFrame showing the sum of the fares 
#  for each date where the indices are the city type and date.

sum_fare_type_date_df = pyber_data_df.groupby(["type", "date"]).sum()[["fare"]]
sum_fare_type_date_df

```
![This is an image](/analysis/Screenshot5.png)


```
# 3. Reset the index on the DataFrame you created in #1. This is needed to use the 'pivot()' function.
# df = df.reset_index()
sum_fare_type_date_df = sum_fare_type_date_df.reset_index()

```
```
# 4. Create a pivot table with the 'date' as the index, the columns ='type', and values='fare' 
# to get the total fares for each type of city by the date. 
sum_fare_type_date_pivot = sum_fare_type_date_df.pivot(index="date", columns="type", values="fare")
sum_fare_type_date_pivot.head(10)
```
![This is an image](/analysis/Screenshot6.png)

```
# 5. Create a new DataFrame from the pivot table DataFrame using loc on the given dates, '2019-01-01':'2019-04-29'.
sum_fare_type_jan_thru_apr_df = sum_fare_type_date_pivot.loc['2019-01-01': '2019-04-29']
sum_fare_type_jan_thru_apr_df.head(10)
```
![This is an image](/analysis/Screenshot7.png)

```
# 6. Set the "date" index to datetime datatype. This is necessary to use the resample() method in Step 8.
# df.index = pd.to_datetime(df.index)
sum_fare_type_jan_thru_apr_df.index = pd.to_datetime(sum_fare_type_jan_thru_apr_df.index)

```
```
# 7. Check that the datatype for the index is datetime using df.info()
sum_fare_type_jan_thru_apr_df.info()


<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 2196 entries, 2019-01-01 00:08:16 to 2019-04-28 19:35:03
Data columns (total 3 columns):
 #   Column    Non-Null Count  Dtype  
---  ------    --------------  -----  
 0   Rural     114 non-null    float64
 1   Suburban  573 non-null    float64
 2   Urban     1509 non-null   float64
dtypes: float64(3)
memory usage: 68.6 KB

```
```
# 8a. Create a new DataFrame using the "resample()" function by week 'W' and get the sum of the fares for each week.
sum_fare_type_jan_thru_apr_df = sum_fare_type_jan_thru_apr_df.resample('W').sum()
sum_fare_type_jan_thru_apr_df

```

![This is an image](/analysis/Screenshot8.png)

```
# 8b. Using the object-oriented interface method, plot the resample DataFrame using the df.plot() function. 
# Object-oriented method to use fig, ax = plt.subplots(figsize=(w,h))

ax=sum_fare_type_jan_thru_apr_df.plot(figsize=(20, 6))
ax.set(xlabel=None)
ax.set_ylabel("Fare ($USD)") 
ax.set_title("Total Fare by City Type")
ax.legend()

# move legend to the middle of line graph
legend_centered = plt.legend(loc="center", title="type")

# Import the style from Matplotlib.
from matplotlib import style
# Use the graph style fivethirtyeight.
style.use('fivethirtyeight')

# Save the file as PyBer_fare_summary.png
plt.savefig("analysis/PyBer_fare_summary.png")
plt.show()

```

![This is an image](/analysis/PyBer_fare_summary.png)

## Summary
- The code file can be found here:  [filename](/Student_Data_Challenge_Starter_Code.ipynb)

