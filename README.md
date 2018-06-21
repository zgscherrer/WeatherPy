
Observations
1. Temperatures follow a smooth arc with the highest temperatures being closest to the equator and dropping predictably at locations further from the equator. 

2. The majority of humidity data points lie between 80-100%, although there is little to no correlation between humidity and proximity to the equator or location in general.

3. There is no significant relationship between proximity to the equator and cloudiness. The two extremes of 0% cloud cover and near 100% cloud cover make up the majority of data points, with the remaining data evenly distributed between both location and percent cloud cover.


```python
# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
import csv
# Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy
api_key = 'e6896c223b1530c896ffbac8d107ceb6'

# Output File (CSV)
output_data_file = "/Users/ZGS/Documents/Data_Bootcamp/output_data/cities.csv"
# Range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)
```


```python
# List for holding lat_lngs and cities
from collections import defaultdict
lat_lng = defaultdict()
# Create a set of random lat and lng combinations
lats = np.random.uniform(low=-90.000, high=90.000, size=1500)
lngs = np.random.uniform(low=-180.000, high=180.000, size=1500)
lat_lngs = zip(lats, lngs)

for x in lat_lngs:
    lat, long = x # tuple unpacking
    city = citipy.nearest_city(lat, long).city_name
    lat_lng[city] = (lat, long)
```


```python
#Create DataFrame
df = pd.DataFrame(list(lat_lng.items()))
df.columns = ['city', 'lat_long']
df['lat'] = df.lat_long.map(lambda x: str(x[0]))
df['lon'] = df.lat_long.map(lambda x: str(x[1]))
print(df.shape)
df.head()
```

    (620, 4)





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
      <th>city</th>
      <th>lat_long</th>
      <th>lat</th>
      <th>lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>barcelona</td>
      <td>(10.970074861224717, -65.06055142188151)</td>
      <td>10.970074861224717</td>
      <td>-65.06055142188151</td>
    </tr>
    <tr>
      <th>1</th>
      <td>taolanaro</td>
      <td>(-38.227634792576694, 53.65818276297165)</td>
      <td>-38.227634792576694</td>
      <td>53.65818276297165</td>
    </tr>
    <tr>
      <th>2</th>
      <td>albany</td>
      <td>(-76.50822434673898, 102.29848696719012)</td>
      <td>-76.50822434673898</td>
      <td>102.29848696719012</td>
    </tr>
    <tr>
      <th>3</th>
      <td>vaini</td>
      <td>(-20.445769720589, -175.06690591300958)</td>
      <td>-20.445769720589</td>
      <td>-175.06690591300958</td>
    </tr>
    <tr>
      <th>4</th>
      <td>rivadavia</td>
      <td>(-33.51900057472405, -68.47414844292278)</td>
      <td>-33.51900057472405</td>
      <td>-68.47414844292278</td>
    </tr>
  </tbody>
</table>
</div>




```python
url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + api_key 
# print(url)
```


```python
def get_current_weather(df_object):
    base_url = "http://api.openweathermap.org/data/2.5/weather"
    params = {
        'APPID': api_key,
        'lat': df_object.lat,
        'lon': df_object.lon,
        'units': 'Imperial'
    }
    data = requests.get(base_url, params=params)
    time.sleep(.50)
    return data.json()
```


```python
sample = df.sample(n=500)
sample['weather_json'] = sample.apply(get_current_weather, axis=1)
sample['temp'] = sample.weather_json.map(lambda x: x.get('main').get('temp'))
sample['humidity'] = sample.weather_json.map(lambda x: x.get('main').get('humidity'))
sample['cloudiness'] = sample.weather_json.map(lambda x: x.get('clouds').get('all'))
sample['wind speed'] = sample.weather_json.map(lambda x: x.get('wind').get('speed'))
sample.head()
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
      <th>city</th>
      <th>lat_long</th>
      <th>lat</th>
      <th>lon</th>
      <th>weather_json</th>
      <th>temp</th>
      <th>humidity</th>
      <th>cloudiness</th>
      <th>wind speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>242</th>
      <td>bathsheba</td>
      <td>(22.254293558441702, -45.658419320585125)</td>
      <td>22.254293558441702</td>
      <td>-45.658419320585125</td>
      <td>{'coord': {'lon': -45.66, 'lat': 22.25}, 'weat...</td>
      <td>76.29</td>
      <td>100</td>
      <td>0</td>
      <td>6.73</td>
    </tr>
    <tr>
      <th>448</th>
      <td>flin flon</td>
      <td>(58.50456773977308, -101.35958686945862)</td>
      <td>58.50456773977308</td>
      <td>-101.35958686945862</td>
      <td>{'coord': {'lon': -101.36, 'lat': 58.5}, 'weat...</td>
      <td>60.72</td>
      <td>67</td>
      <td>20</td>
      <td>8.75</td>
    </tr>
    <tr>
      <th>79</th>
      <td>quatre cocos</td>
      <td>(-18.797853544709326, 61.593589681424675)</td>
      <td>-18.797853544709326</td>
      <td>61.593589681424675</td>
      <td>{'coord': {'lon': 61.59, 'lat': -18.8}, 'weath...</td>
      <td>76.56</td>
      <td>100</td>
      <td>48</td>
      <td>14.79</td>
    </tr>
    <tr>
      <th>408</th>
      <td>bafata</td>
      <td>(12.506574432216894, -14.74646906559326)</td>
      <td>12.506574432216894</td>
      <td>-14.74646906559326</td>
      <td>{'coord': {'lon': -14.75, 'lat': 12.51}, 'weat...</td>
      <td>96.80</td>
      <td>37</td>
      <td>0</td>
      <td>6.93</td>
    </tr>
    <tr>
      <th>196</th>
      <td>vestmanna</td>
      <td>(64.22874525168444, -8.216963637342502)</td>
      <td>64.22874525168444</td>
      <td>-8.216963637342502</td>
      <td>{'coord': {'lon': -8.22, 'lat': 64.23}, 'weath...</td>
      <td>45.15</td>
      <td>100</td>
      <td>92</td>
      <td>28.43</td>
    </tr>
  </tbody>
</table>
</div>




```python
row_count = 0
for index, row in sample.iterrows():
    print("Now retieving city # " + str(row_count))
    print(url)
    row_count += 1
```

    Now retieving city # 0
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 1
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 2
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 3
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 4
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 5
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 6
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 7
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 8
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 9
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 10
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 11
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 12
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 13
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 14
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 15
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 16
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 17
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 18
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 19
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 20
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 21
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 22
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 23
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 24
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 25
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 26
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 27
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 28
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 29
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 30
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 31
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 32
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 33
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 34
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 35
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 36
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 37
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 38
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 39
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 40
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 41
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 42
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 43
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 44
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 45
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 46
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 47
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 48
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 49
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 50
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 51
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 52
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 53
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 54
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 55
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 56
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 57
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 58
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 59
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 60
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 61
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 62
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 63
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 64
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 65
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 66
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 67
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 68
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 69
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 70
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 71
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 72
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 73
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 74
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 75
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 76
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 77
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 78
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 79
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 80
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 81
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 82
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 83
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 84
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 85
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 86
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 87
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 88
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 89
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 90
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 91
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 92
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 93
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 94
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 95
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 96
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 97
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 98
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 99
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 100
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 101
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 102
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 103
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 104
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 105
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 106
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 107
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 108
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 109
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 110
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 111
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 112
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 113
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 114
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 115
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 116
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 117
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 118
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 119
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 120
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 121
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 122
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 123
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 124
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 125
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 126
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 127
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 128
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 129
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 130
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 131
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 132
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 133
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 134
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 135
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 136
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 137
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 138
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 139
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 140
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 141
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 142
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 143
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 144
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 145
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 146
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 147
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 148
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 149
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 150
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 151
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 152
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 153
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 154
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 155
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 156
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 157
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 158
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 159
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 160
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 161
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 162
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 163
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 164
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 165
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 166
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 167
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 168
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 169
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 170
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 171
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 172
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 173
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 174
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 175
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 176
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 177
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 178
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 179
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 180
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 181
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 182
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 183
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 184
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 185
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 186
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 187
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 188
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 189
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 190
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 191
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 192
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 193
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 194
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 195
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 196
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 197
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 198
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 199
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 200
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 201
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 202
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 203
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 204
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 205
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 206
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 207
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 208
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 209
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 210
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 211
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 212
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 213
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 214
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 215
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 216
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 217
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 218
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 219
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 220
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 221
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 222
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 223
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 224
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 225
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 226
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 227
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 228
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 229
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 230
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 231
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 232
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 233
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 234
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 235
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 236
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 237
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 238
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 239
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 240
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 241
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 242
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 243
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 244
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 245
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 246
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 247
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 248
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 249
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 250
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 251
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 252
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 253
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 254
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 255
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 256
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 257
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 258
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 259
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 260
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 261
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 262
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 263
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 264
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 265
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 266
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 267
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 268
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 269
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 270
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 271
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 272
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 273
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 274
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 275
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 276
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 277
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 278
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 279
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 280
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 281
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 282
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 283
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 284
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 285
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 286
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 287
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 288
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 289
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 290
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 291
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 292
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 293
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 294
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 295
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 296
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 297
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 298
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 299
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 300
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 301
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 302
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 303
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 304
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 305
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 306
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 307
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 308
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 309
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 310
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 311
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 312
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 313
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 314
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 315
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 316
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 317
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 318
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 319
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 320
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 321
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 322
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 323
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 324
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 325
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 326
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 327
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 328
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 329
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 330
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 331
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 332
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 333
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 334
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 335
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 336
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 337
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 338
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 339
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 340
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 341
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 342
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 343
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 344
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 345
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 346
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 347
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 348
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 349
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 350
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 351
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 352
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 353
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 354
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 355
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 356
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 357
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 358
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 359
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 360
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 361
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 362
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 363
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 364
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 365
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 366
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 367
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 368
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 369
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 370
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 371
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 372
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 373
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 374
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 375
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 376
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 377
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 378
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 379
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 380
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 381
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 382
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 383
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 384
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 385
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 386
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 387
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 388
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 389
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 390
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 391
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 392
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 393
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 394
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 395
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 396
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 397
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 398
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 399
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 400
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 401
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 402
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 403
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 404
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 405
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 406
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 407
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 408
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 409
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 410
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 411
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 412
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 413
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 414
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 415
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 416
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 417
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 418
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 419
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 420
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 421
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 422
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 423
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 424
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 425
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 426
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 427
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 428
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 429
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 430
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 431
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 432
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 433
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 434
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 435
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 436
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 437
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 438
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 439
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 440
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 441
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 442
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 443
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 444
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 445
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 446
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 447
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 448
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 449
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 450
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 451
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 452
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 453
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 454
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 455
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 456
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 457
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 458
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 459
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 460
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 461
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 462
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 463
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 464
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 465
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 466
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 467
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 468
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 469
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 470
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 471
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 472
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 473
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 474
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 475
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 476
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 477
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 478
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 479
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 480
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 481
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 482
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 483
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 484
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 485
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 486
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 487
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 488
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 489
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 490
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 491
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 492
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 493
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 494
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 495
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 496
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 497
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 498
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6
    Now retieving city # 499
    http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=e6896c223b1530c896ffbac8d107ceb6



```python
sample.to_csv("weatherdata.csv", encoding="utf-8", index=False)
```


```python
# Latitude vs Max Temp
plt.scatter(sample["lat"].astype(float), sample["temp"].astype(float), marker="o", color = 'blue', alpha = 0.75)
#chart labels
plt.title("City Latitude vs Max Temperature")
plt.ylabel("Temperature (F) ")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim(-90,90)

#Save the figure
plt.savefig("Lat_vs_MaxTemp.png")
plt.show()
```


![png](output_9_0.png)



```python
# Latitude vs Humidity
plt.scatter(sample["lat"].astype(float), sample["humidity"].astype(float), marker="o", alpha = 0.75)

# Add chart labels
plt.title("City Latitude vs Humidity")
plt.ylabel("Humidity")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim(-90,90)

# Save the figure
plt.savefig("Lat_vs_Humidity.png")

# Show plot
plt.show()
```


![png](output_10_0.png)



```python
# Latitude vs Cloudiness
plt.scatter(sample["lat"].astype(float), sample["cloudiness"].astype(float), marker="o", alpha = 0.75)

# Add chart labels
plt.title("City Latitude vs Cloudiness")
plt.ylabel("Cloudiness")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim(-90,90)

# Save the figure
plt.savefig("Lat_vs_Cloudiness.png")

# Show plot
plt.show()
```


![png](output_11_0.png)



```python
# Latitude vs Wind Speed
plt.scatter(sample["lat"].astype(float), sample["wind speed"].astype(float), marker="o", alpha = 0.75)

# Add chart labels
plt.title("City Latitude vs Wind Speed")
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim(-90,90)

# Save the figure
plt.savefig("Lat_vs_WindSpeed.png")

# Show plot
plt.show()
```


![png](output_12_0.png)


#### 


```python
#sample.to_csv('sample_data.csv', index=False, header=True)
```
