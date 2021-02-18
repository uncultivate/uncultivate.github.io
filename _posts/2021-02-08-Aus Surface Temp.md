---
layout: post2
title: It's Getting Hotter in Here
subtitle: Exploring & Visualising Australian Historical Temperature Data in Pandas
cover-img: /assets/img/aus-heat.jpg
thumbnail-img: /assets/img/aus-temp.png
share-img: /assets/img/aus-temp.png
gh-repo: redseraph/Australian-Temperatures
gh-badge: [star, fork, follow]
tags: [data science, visualization, climate change, maps, pandas]
comments: true
---
<h1>Exploring and Visualising Australian Historical Temperature Data in Pandas</h1>

After enduring the hottest Australian summer on record in 2019/20 sheltered inside with air-con and air purifier running as large parts of my home state of New South Wales burned, I decided to run some exploratory analysis on the Bureau of Meteorology's fantastic [ACORN-SAT 2.1 database](http://www.bom.gov.au/climate/data/acorn-sat/). 
>The Australian Climate Observations Reference Network – Surface Air Temperature (ACORN-SAT) is the dataset used by the Bureau of Meteorology to monitor long-term temperature trends in Australia. ACORN-SAT uses observations from 112 weather stations in all corners of Australia, selected for the quality and length of their available temperature data.



<h2>Setup</h2>

Let's get started by importing the required packages


```python
import pandas as pd
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import chart_studio.plotly as py
import plotly.graph_objects as go
init_notebook_mode(connected=False)
import plotly.express as px
import numpy as np
import seaborn as sns
import string
import glob
import plotly.io as pio
```
Some Plotly visualisations require a Mapbox account - you can register for the basic account tier for free.


```python
#You will need your own token
token = open(".mapbox_token").read() 
```


```python
places = pd.read_csv("acorn_sat_v2.1.0_stations.csv")
places.head()
```




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
      <th>stn_num</th>
      <th>stn_name</th>
      <th>lat</th>
      <th>lon</th>
      <th>elevation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1019</td>
      <td>Kalumburu</td>
      <td>-14.30</td>
      <td>126.65</td>
      <td>23</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2079</td>
      <td>Halls Creek</td>
      <td>-18.23</td>
      <td>127.67</td>
      <td>409</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3003</td>
      <td>Broome</td>
      <td>-17.95</td>
      <td>122.24</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4032</td>
      <td>Port Hedland</td>
      <td>-20.37</td>
      <td>118.63</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4106</td>
      <td>Marble Bar</td>
      <td>-21.18</td>
      <td>119.75</td>
      <td>182</td>
    </tr>
  </tbody>
</table>
</div>



We then create a dataframe from a CSV containing a list of weather station names and locations

The dataset of temperature readings comes split into 'max' and 'min' folders containing a CSV for each weather station. The below code loops through each CSV, creating a dataframe for each station and running some pre-processing, before concatenating them together.


```python
#Loop through the weather station CSVs, run some preprocessing and add the dataframes to a temporary list
path = "max"
li = []
 
for count, f in enumerate(glob.glob(path + "/*.csv")):
    df = pd.read_csv(f, index_col=None, header=0)
    df['site number'] = df['site number'][0]
    df['site name'] = string.capwords(df['site name'][0])
    df['lat'] = places['lat'][count]
    df['lon'] = places['lon'][count]
    df = df.drop(df.index[0])
    df['year'] = pd.to_datetime(df['date']).dt.year
    df['long term average'] = df['maximum temperature (degC)'][(df['year']<1980) & (df['year']>1949)].mean()
    li.append(df)

#Concatenate list of dataframes into one large dataframe of nearly 4 million rows. 
df = pd.concat(li, axis=0, ignore_index=True)
```


```python
#Do likewise with the weather station data from the 'min' dataset
path = "min"
li = []

for count, f in enumerate(glob.glob(path + "/*.csv")):
    df_min = pd.read_csv(f, index_col=None, header=0)
    df_min = df_min.drop(df_min.index[0])
    li.append(df_min)

df_min = pd.concat(li, axis=0, ignore_index=True)
```


```python
#Add the minimum temperature data to the first dataframe
df['min'] = df_min['minimum temperature (degC)']
df.rename(columns = {"maximum temperature (degC)": "max"}, inplace=True)

df['date'] = pd.to_datetime(df['date'])
df['year'] = pd.to_datetime(df['date']).dt.year

```

<h2>Exploring the Australian Bureau of Meteorology data</h2>

We start by printing some basic statistics describing the dataset


```python
df.head()
```




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
      <th>date</th>
      <th>max</th>
      <th>site number</th>
      <th>site name</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1941-09-01</td>
      <td>31.0</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>20.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1941-09-02</td>
      <td>31.0</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>20.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1941-09-03</td>
      <td>30.5</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>19.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1941-09-04</td>
      <td>38.8</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1941-09-05</td>
      <td>32.1</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>19.6</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.dtypes
```




    date                 datetime64[ns]
    max                         float64
    site number                 float64
    site name                    object
    lat                         float64
    lon                         float64
    year                          int64
    long term average           float64
    min                         float64
    dtype: object




```python
df.describe(include=[np.object])
```




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
      <th>site name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3882887</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>112</td>
    </tr>
    <tr>
      <th>top</th>
      <td>Cairns Aero</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>40177</td>
    </tr>
  </tbody>
</table>
</div>



There are a not insignificant number of missing values in this dataset, however given we are exploring mean temperature values this is not especially critical.


```python
pd.isnull(df['max']).value_counts()
```




    False    3779214
    True      103673
    Name: max, dtype: int64



<h2>Visualising Extreme Heat Events</h2>

These are the days when you just can't afford for your air-con to break. When koalas come down from their gum trees for a swim in the billabong. Using the BoM dataset let's find out where in Australia you would be better off living underground, and whether the occurences of 40°C+ (104 °F) are on the rise. 


```python
dfheat = df.copy()

#Tag days where the temperature reading exceeded 40°C (104 °F)
dfheat['Average Days of 40C+ / Year'] = np.where(dfheat['max'] >= 40, 1, 0)

dfheat.head()
```




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
      <th>date</th>
      <th>max</th>
      <th>site number</th>
      <th>site name</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
      <th>Average Days of 40C+ / Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1941-09-01</td>
      <td>31.0</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>20.5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1941-09-02</td>
      <td>31.0</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>20.5</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1941-09-03</td>
      <td>30.5</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>19.3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1941-09-04</td>
      <td>38.8</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>21.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1941-09-05</td>
      <td>32.1</td>
      <td>1019.0</td>
      <td>Kalumburu</td>
      <td>-14.3</td>
      <td>126.65</td>
      <td>1941</td>
      <td>33.678851</td>
      <td>19.6</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



<h3>Hottest Places in Australia (2019)</h3>
Let's start by finding the Australian locations with the most days of extreme heat (40°C / 104 °F) in 2019 


```python
#Slice dataframe and select the top 10 locations
df_hottest = dfheat[dfheat['year'] == 2019]
df_hottest = df_hottest.groupby(["site name"]).sum()
df_hottest = df_hottest.sort_values('Average Days of 40C+ / Year', ascending=False)
df_hottest = df_hottest.reset_index().loc[:9, :]

df_hottest
```




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
      <th>site name</th>
      <th>max</th>
      <th>site number</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
      <th>Average Days of 40C+ / Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marble Bar</td>
      <td>13335.4</td>
      <td>1498690.0</td>
      <td>-7730.70</td>
      <td>43708.75</td>
      <td>736935</td>
      <td>12923.318800</td>
      <td>7473.3</td>
      <td>154</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Rabbit Flat</td>
      <td>13120.6</td>
      <td>5718090.0</td>
      <td>-7365.70</td>
      <td>47453.65</td>
      <td>736935</td>
      <td>12218.645090</td>
      <td>5216.3</td>
      <td>139</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Karijini North</td>
      <td>12920.3</td>
      <td>1860770.0</td>
      <td>-8139.50</td>
      <td>43234.25</td>
      <td>736935</td>
      <td>11679.412979</td>
      <td>7623.9</td>
      <td>127</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tennant Creek Airport</td>
      <td>12160.2</td>
      <td>5524275.0</td>
      <td>-7168.60</td>
      <td>48975.70</td>
      <td>736935</td>
      <td>11404.563166</td>
      <td>7292.2</td>
      <td>92</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Victoria River Downs</td>
      <td>12802.4</td>
      <td>5411125.0</td>
      <td>-5986.00</td>
      <td>47818.65</td>
      <td>736935</td>
      <td>12676.517644</td>
      <td>7079.4</td>
      <td>89</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Camooweal Township</td>
      <td>12520.6</td>
      <td>13508650.0</td>
      <td>-7270.80</td>
      <td>50413.80</td>
      <td>736935</td>
      <td>11865.371179</td>
      <td>6600.0</td>
      <td>88</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Halls Creek Airport</td>
      <td>12645.8</td>
      <td>758835.0</td>
      <td>-6653.95</td>
      <td>46599.55</td>
      <td>736935</td>
      <td>11876.945578</td>
      <td>6926.7</td>
      <td>77</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Oodnadatta Airport</td>
      <td>11400.5</td>
      <td>6220695.0</td>
      <td>-10059.40</td>
      <td>49439.25</td>
      <td>736935</td>
      <td>10442.994556</td>
      <td>5205.0</td>
      <td>74</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Birdsville Airport</td>
      <td>11622.4</td>
      <td>13879490.0</td>
      <td>-9453.50</td>
      <td>50862.75</td>
      <td>736935</td>
      <td>11004.352290</td>
      <td>5546.8</td>
      <td>74</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Learmonth Airport</td>
      <td>12208.1</td>
      <td>1827555.0</td>
      <td>-8117.60</td>
      <td>41646.50</td>
      <td>736935</td>
      <td>11368.705666</td>
      <td>6463.8</td>
      <td>74</td>
    </tr>
  </tbody>
</table>
</div>



The fact Marble Bar (located in Western Australia) is at the top of this list shouldn't be a surprise. Marble Bar has a hot desert climate with sweltering summers and warm winters. Most of the annual rainfall occurs in the summer. The town set a world record of most consecutive days of 100 °F (37.8 °C) or above, during a period of 160 days from 31 October 1923 to 7 April 1924.


```python
#Create barchart visualisation using Plotly

colors = ['lightsalmon',] * 10
colors[0] = 'indianred'

fig = go.Figure([go.Bar(x=df_hottest['site name'], y=df_hottest["Average Days of 40C+ / Year"], marker_color=colors)])

fig.update_yaxes(title="Count")
fig.update_layout(title_text= "Australia's Hottest Towns: Most Frequent Days Over 40°C (104 °F)", height=450, width=800)


fig.show()

```


{% include figure1.html %}



<h3>Occurence of Extreme Weather Events Over Time</h3>
Let's look at whether the number of days over 40°C (104 °F) has increased over time.


```python
#Group dataframe by year and obtain average of days over 40C across weather stations
dfheat = dfheat.groupby([pd.Grouper(key="date", freq="Y")]).mean()*365
dfheat = dfheat.reset_index()
dfheat['year'] = dfheat['date'].dt.year

#Select records from 1950 to present
dfheat = dfheat[dfheat['year'] >= 1950]

dfheat.head()

```




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
      <th>date</th>
      <th>max</th>
      <th>site number</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
      <th>Average Days of 40C+ / Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>40</th>
      <td>1950-12-31</td>
      <td>8835.971305</td>
      <td>1.525218e+07</td>
      <td>-10921.469084</td>
      <td>50952.185347</td>
      <td>1950</td>
      <td>8957.598089</td>
      <td>4584.059041</td>
      <td>3.966220</td>
    </tr>
    <tr>
      <th>41</th>
      <td>1951-12-31</td>
      <td>9005.318544</td>
      <td>1.511193e+07</td>
      <td>-10866.890937</td>
      <td>50891.841143</td>
      <td>1951</td>
      <td>8978.719144</td>
      <td>4438.756589</td>
      <td>7.076769</td>
    </tr>
    <tr>
      <th>42</th>
      <td>1952-12-31</td>
      <td>8908.120587</td>
      <td>1.514267e+07</td>
      <td>-10879.187456</td>
      <td>50851.405749</td>
      <td>1952</td>
      <td>8981.081792</td>
      <td>4479.180043</td>
      <td>8.558434</td>
    </tr>
    <tr>
      <th>43</th>
      <td>1953-12-31</td>
      <td>8953.312144</td>
      <td>1.520950e+07</td>
      <td>-10895.896354</td>
      <td>50861.229167</td>
      <td>1953</td>
      <td>8971.730128</td>
      <td>4441.370431</td>
      <td>6.604167</td>
    </tr>
    <tr>
      <th>44</th>
      <td>1954-12-31</td>
      <td>8945.124239</td>
      <td>1.520064e+07</td>
      <td>-10886.287193</td>
      <td>50861.239298</td>
      <td>1954</td>
      <td>8985.271339</td>
      <td>4578.838965</td>
      <td>5.784125</td>
    </tr>
  </tbody>
</table>
</div>



Visualising this data, it appears that there is indeed an upward trend showing increasing extreme weather events across Australia


```python
#Visualise the data
fig = px.bar(dfheat, x="year", y="Average Days of 40C+ / Year",
            hover_data=['year', 'Average Days of 40C+ / Year'], 
             color_continuous_scale="hot_r",
             color='Average Days of 40C+ / Year',
             labels={'year':'Year', 'Average Days of 40C+ / Year': "Count"}, height=450)
fig.update_layout(title_text="Australia Average Days of 40°C+ (104°F) / Year")

fig.show()

```


{% include figure2.html %}

                      

<h2>Visualising Temperature Data By Season</h2>
Given that there seems to be an increasing trend of extreme weather events, let's explore the averaged temperature data in more detail and whether there is any discernable difference between Summer & Winter records


```python
#Set dataframe date range and group by 3 months
start = '1950-08-01'
end = '2019-12-31'
mask = (df['date'] > start) & (df['date'] <= end)
df_seasons = (
    df.loc[mask]
    .groupby(pd.Grouper(freq="3M", key="date"))
    .mean()
    .iloc[::2]
    .reset_index()
)
#Label grouped months as seasons
df_seasons.loc[1::2, 'season'] = 'Summer'
df_seasons.loc[::2, 'season'] = 'Winter'

#Create temperature range column & year column
df_seasons.loc[:, 'temp_range'] = df_seasons['max'] - df_seasons['min']
df_seasons.loc[:, 'year'] = pd.DatetimeIndex(df_seasons['date']).year

df_seasons.head()
```




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
      <th>date</th>
      <th>max</th>
      <th>site number</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
      <th>season</th>
      <th>temp_range</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1950-08-31</td>
      <td>19.407288</td>
      <td>41655.182796</td>
      <td>-29.998925</td>
      <td>139.455806</td>
      <td>1950</td>
      <td>24.522322</td>
      <td>11.402815</td>
      <td>Winter</td>
      <td>8.004473</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1951-02-28</td>
      <td>29.599024</td>
      <td>41499.893617</td>
      <td>-29.792340</td>
      <td>139.485957</td>
      <td>1951</td>
      <td>24.579390</td>
      <td>15.553474</td>
      <td>Summer</td>
      <td>14.045550</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1951-08-31</td>
      <td>18.503477</td>
      <td>41499.893617</td>
      <td>-29.792340</td>
      <td>139.485957</td>
      <td>1951</td>
      <td>24.579390</td>
      <td>8.935743</td>
      <td>Winter</td>
      <td>9.567734</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1952-02-29</td>
      <td>30.192992</td>
      <td>41116.715789</td>
      <td>-29.713474</td>
      <td>139.264526</td>
      <td>1952</td>
      <td>24.657485</td>
      <td>15.677567</td>
      <td>Summer</td>
      <td>14.515425</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1952-08-31</td>
      <td>18.738631</td>
      <td>41669.875000</td>
      <td>-29.851771</td>
      <td>139.345833</td>
      <td>1952</td>
      <td>24.580083</td>
      <td>9.362085</td>
      <td>Winter</td>
      <td>9.376546</td>
    </tr>
  </tbody>
</table>
</div>



In addition to the data from the Australian Bureau of Meteorology, I've added a dataframe with cold & warm episodes by season from the National Weather Service to identify when El Nino & La Nina events are occurring and the Oceanic Niño Index (ONI) score.


```python
oni_values = pd.read_csv("ONI Values.csv", skiprows=2)
oni_values.head()
```




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
      <th>Type</th>
      <th>Season</th>
      <th>Unnamed: 2</th>
      <th>Unnamed: 3</th>
      <th>JJA</th>
      <th>JAS</th>
      <th>ASO</th>
      <th>SON</th>
      <th>OND</th>
      <th>NDJ</th>
      <th>DJF</th>
      <th>JFM</th>
      <th>FMA</th>
      <th>MAM</th>
      <th>AMJ</th>
      <th>MJJ</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>1950</td>
      <td>-</td>
      <td>1951</td>
      <td>-0.5</td>
      <td>-0.4</td>
      <td>-0.4</td>
      <td>-0.4</td>
      <td>-0.6</td>
      <td>-0.8</td>
      <td>-0.8</td>
      <td>-0.5</td>
      <td>-0.2</td>
      <td>0.2</td>
      <td>0.4</td>
      <td>0.6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ME</td>
      <td>1951</td>
      <td>-</td>
      <td>1952</td>
      <td>0.7</td>
      <td>0.9</td>
      <td>1.0</td>
      <td>1.2</td>
      <td>1.0</td>
      <td>0.8</td>
      <td>0.5</td>
      <td>0.4</td>
      <td>0.3</td>
      <td>0.3</td>
      <td>0.2</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>WE</td>
      <td>1952</td>
      <td>-</td>
      <td>1953</td>
      <td>-0.1</td>
      <td>0.0</td>
      <td>0.2</td>
      <td>0.1</td>
      <td>0.0</td>
      <td>0.1</td>
      <td>0.4</td>
      <td>0.6</td>
      <td>0.6</td>
      <td>0.7</td>
      <td>0.8</td>
      <td>0.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>WE</td>
      <td>1953</td>
      <td>-</td>
      <td>1954</td>
      <td>0.7</td>
      <td>0.7</td>
      <td>0.8</td>
      <td>0.8</td>
      <td>0.8</td>
      <td>0.8</td>
      <td>0.8</td>
      <td>0.5</td>
      <td>0.0</td>
      <td>-0.4</td>
      <td>-0.5</td>
      <td>-0.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>WL</td>
      <td>1954</td>
      <td>-</td>
      <td>1955</td>
      <td>-0.6</td>
      <td>-0.8</td>
      <td>-0.9</td>
      <td>-0.8</td>
      <td>-0.7</td>
      <td>-0.7</td>
      <td>-0.7</td>
      <td>-0.6</td>
      <td>-0.7</td>
      <td>-0.8</td>
      <td>-0.8</td>
      <td>-0.7</td>
    </tr>
  </tbody>
</table>
</div>




```python
#The ONI scores are now added to the corresponding season in the df_seasons dataframe
for count, f in enumerate(oni_values.values):
    df_seasons.loc[count*2, "oni_value"] = float(f[4])
    df_seasons.loc[count*2+1, "oni_value"] = float(f[10])

df_seasons = df_seasons.dropna()
```

<h3>Visualising Seasonal Average Daily Temperature</h3>
We can now create scatterplots showing average daily temperatures for Summer and for Winter, colour-coded with the ONI data. Red indicates an El Nino event and blue indicates La Nina.


```python
#Create a scatterplot
fig = px.scatter(df_seasons, x="year", y="max", facet_row="season", opacity=1, 
                 labels=dict(year="Year", difference="Average Daily Temperature Range (°C)", oni_value='Oceanic Niño Index'),
                 trendline = "ols", trendline_color_override="red", color="oni_value", color_continuous_scale='bluered')

fig.update_layout(height=600, width=1000, title_text="Australian Seasonal Average Daily Temperature (1950-2020)")

fig.update_xaxes(nticks=10)
fig.update_yaxes(nticks=10, title_text="Temperature (°C)", matches=None)

fig.show()

```

{% include figure3.html %}


Temperatures in Summer & Winter have both increased by approximately 1.5°C since 1950, which is also reflected in the number of extreme heat events in the previous section. The data shows that although there is no apparently correlation between ONI and temperature in Winter, Summer El Nino events appear to be correlated with with higher than average temperatures (record heat in 1973, 1983, 1997 & 2019-20 was driven by strong El Nino events), and likewise La Nina events often bring lower than average temperatures (1974-76, 1984, 2000 & 2011-12).

<h3>Visualising Average Daily Temperature Range</h3>
Using the same dataframe, we can assess whether the increase in temperatures is likely to bring about an increased daily temperature range (i.e. colder nights, hotter days) across the BoM weather station locations.


```python
#Create a scatterplot showing average daily temperature range for Summer and one for Winter
fig = px.scatter(df_seasons, x="year", y="temp_range", facet_row="season", opacity=1, 
                 labels=dict(year="Year", temp_range="Temperature Range (°C)"),
                 trendline = "ols", trendline_color_override="red")

fig.update_layout(height=600, width=1000, title_text="Australian Average Daily Temperature Range (1950-2020)")

fig.update_xaxes(nticks=10)
fig.update_yaxes(nticks=10, matches=None)

fig.show()
pio.write_html(fig, file='figure4.html', auto_open=True)
```




{% include figure4.html %}

           


The trendlines show a very slight increase in the average daily temperature range between 1950-2020, leading to approximately 0.5°C greater difference between min / max temperatures in both Summer & Winter.

<h2>Geospatial Historical Temperature Analysis</h2>

Given we have the coordinates of weather stations Australia-wide, let's create an animated map showing temperature changes over time.


```python
#Consolidate dataframe into yearly average temperatures and group by location.
dfyear = df.copy()[df['year'] >= 1950].groupby(['site name', pd.Grouper(key="date", freq="Y")]).mean()

#Using the long term average for each location between 1950-1980, we can calculate the difference in temperature for each year & location.
dfyear = (
    dfyear.dropna(subset=['max']).
    reset_index()
)
dfyear.loc[:,'+/- Long Term Average'] = (dfyear['max']-dfyear['long term average'])
dfyear = dfyear.round(1)

dfyear.head()
```




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
      <th>site name</th>
      <th>date</th>
      <th>max</th>
      <th>site number</th>
      <th>lat</th>
      <th>lon</th>
      <th>year</th>
      <th>long term average</th>
      <th>min</th>
      <th>+/- Long Term Average</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Albany Airport</td>
      <td>1950-12-31</td>
      <td>20.1</td>
      <td>9999.0</td>
      <td>-34.9</td>
      <td>117.8</td>
      <td>1950</td>
      <td>19.8</td>
      <td>10.2</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Albany Airport</td>
      <td>1951-12-31</td>
      <td>19.3</td>
      <td>9999.0</td>
      <td>-34.9</td>
      <td>117.8</td>
      <td>1951</td>
      <td>19.8</td>
      <td>10.1</td>
      <td>-0.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Albany Airport</td>
      <td>1952-12-31</td>
      <td>19.4</td>
      <td>9999.0</td>
      <td>-34.9</td>
      <td>117.8</td>
      <td>1952</td>
      <td>19.8</td>
      <td>9.4</td>
      <td>-0.4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albany Airport</td>
      <td>1953-12-31</td>
      <td>19.5</td>
      <td>9999.0</td>
      <td>-34.9</td>
      <td>117.8</td>
      <td>1953</td>
      <td>19.8</td>
      <td>10.0</td>
      <td>-0.3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albany Airport</td>
      <td>1954-12-31</td>
      <td>19.9</td>
      <td>9999.0</td>
      <td>-34.9</td>
      <td>117.8</td>
      <td>1954</td>
      <td>19.8</td>
      <td>10.0</td>
      <td>0.1</td>
    </tr>
  </tbody>
</table>
</div>



It's easy to create animations with Plotly's Mapbox integration. For some of the Mapbox style types (like "satellite" used here), you'll need a mapbox access token which can be obtained by registering for free. 


```python
fig = px.scatter_mapbox(dfyear, lat="lat", 
                        lon="lon", 
                        size="max",
                        size_max = 20,
                        hover_name="site name", 
                        hover_data=["max","min"],
                        color="+/- Long Term Average",
                        color_continuous_scale="blackbody_r", 
                        range_color =[-2,3],
                        zoom=3.8, height=800, width=1000, 
                        animation_frame="year", 
                        )
fig.update_layout(
    mapbox_style="satellite",
    mapbox_accesstoken=token,
        title={
        'text': 'AUS Surface Temperature 1950-2019<br>Source: <a href="http://www.bom.gov.au/climate/data-services/station-data.shtml">BoM</a>',
        'y':0.3,
        'x':0.05,
        'xanchor': 'left',
        'yanchor': 'top'},
        font=dict(
            color="grey",
        )
)

#Adjust pitch and bearing to adjust the rotation
fig.update_layout(margin={"r":0,"t":20,"l":0,"b":20},
                  mapbox=dict(
                      pitch=40,
                      bearing=0,
                      center=dict(
                      lat=-31,
                      lon = 134),
                  ),


                 )
fig.show()

```




{% include figure5.html %}

