
# WeatherPy Analysis

### Summary of Data and Rationale

For this project, 3 different sets of data were pulled into one data frame:
+ the current weather at time of request, 
+ the weather at the highest temperature forecasted at three hour intervals in the next 24 hours from time of request, 
+ and the average of the selected weather parameters forecasted in intervals of 3 hours for the next 5 days (120 hours) from time of request

Initially, the hypotheses was that current weather would give you skewed information based on longitude and time of day, while the forecasted weather may not be as accurate.  The belief was that pulling information only from the forecasted maximized temperature over a 24 hours period would minimize this effect, but also might skew additional parameters like cloud cover and wind speed.  Surprisingly, no noticeable difference was experienced between these two groups.  

The average values over the next 5 days/120h hours did yield slightly different results, but might not allow us to inspect addition relationships accurately.  For example, to inspect temperature versus wind speed exact data points may be more beneficial than the average values.  This information, however, may not be as accurate as current data or a 24 hours forecast.
________

### Analysis

+ Using 24 hour data, we can see that cities on or very near the equator did not experience the highest temperatures, although high temperatures were more consistent in that area and there was slightly less variability.  The highest temperatures where instead experienced closer to the Tropics of Cancer and Capricorn, located at 23.5 and -23.5 degrees latitude respectively.  This phenomenon and variability may be due to the differences in the number of cities that fall along these latitudes as well as the geographic features.  For example, the Sahara Desert falls along the Tropic of Cancer (see graphs at the end of document).   Instead, it is safer to conclude that the highest temperatures were experienced between latitudes of -30 and 30 degrees, although with generally more variability as you increase distance from 0 degrees latitude.  Using the average temperature parameter, we can see more consistency between -20 and 20 degrees latitude.  
+ Percent humidity doesn't seem to have much correlation to latitude.  Several varying latitudes have % humidity ranging from 40 to 100%.  However, based on this small date sampling, humidity does experience variation as we approach the Tropic of Cancer and Capricorn with some falling below 40% humidity.  The cities on the equator experienced the 40% to 100% range.  
+ No significant correlation was found between latitude and cloud cover or wind speed.  Frequency of wind speed, however, can be observed.  The most common observation was speeds between 0 and 10 mph.  Observations decrease in frequency as you increase wind speed with rare occurrences of wind speeds greater than 30 mph.

### Additional Notes

By plotting latitude versus longitude, we can observe addition patterns in weather by location.  For example, local patterns are observed by geographic regions.  For example, humidity and wind speed appear to be generally higher in coastal regions when compared to inland regions.

This dataset is limited by the sampling time span.  Historical data might give a better picture of trends and allow you to explore additional relationships.   



```python
# Dependencies
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from citipy import citipy
import requests as req
import unidecode
import time
from math import sqrt
from datetime import datetime
```


```python
# Build data frame of randomly generated lat and long
location_data = pd.DataFrame()
location_data['rand_lat'] = [np.random.uniform(-90,90) for x in range(1500)]
location_data['rand_lng'] = [np.random.uniform(-180, 180) for x in range(1500)]

# add closest city and country column
location_data['closest_city'] = ""
location_data['country'] = ""

#find and add closest city and country code
for index, row in location_data.iterrows():
    lat = row['rand_lat']
    lng = row['rand_lng']
    location_data.set_value(index, 'closest_city', citipy.nearest_city(lat, lng).city_name)
    location_data.set_value(index, 'country', citipy.nearest_city(lat, lng).country_code)
    

```


```python
# delete repeated cities and find unique city count
location_data = location_data.drop_duplicates(['closest_city', 'country'])
location_data = location_data.dropna()
len(location_data['closest_city'].value_counts())
```




    606




```python
# USE BELOW VALUE FOR COUNT MATCHING 
rec_check = len(location_data['closest_city'])  #Difference is due to same city name but different country
rec_check
```




    609




```python
#preview data
location_data.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rand_lat</th>
      <th>rand_lng</th>
      <th>closest_city</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>50.231246</td>
      <td>-69.057679</td>
      <td>chute-aux-outardes</td>
      <td>ca</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-12.656156</td>
      <td>-7.599494</td>
      <td>jamestown</td>
      <td>sh</td>
    </tr>
    <tr>
      <th>2</th>
      <td>32.585848</td>
      <td>112.093323</td>
      <td>dengzhou</td>
      <td>cn</td>
    </tr>
    <tr>
      <th>3</th>
      <td>51.315598</td>
      <td>53.432239</td>
      <td>ilek</td>
      <td>ru</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15.492748</td>
      <td>147.185004</td>
      <td>airai</td>
      <td>pw</td>
    </tr>
  </tbody>
</table>
</div>




```python
# keep only city and country
# random lats and lngs no longer needed
location_data = location_data[['closest_city', 'country']]

#rename columnds for later merging
location_data = location_data.rename(columns = {'closest_city': 'city'})
```


```python
# read in open weather map's country Id json
# downloaded from https://openweathermap.org/appid#work
# done because of open weather maps documentation suggestion

api_city_data = pd.read_json('city.list.json')

for index, row in api_city_data.iterrows():
    lower_city = row['name'].lower() #make all city name lowercase
    unaccented = unidecode.unidecode(lower_city) # strip accents from city name
    lower_country = row['country'].lower() # make all two digit county 
    api_city_data.set_value(index, 'name', unaccented) # reset the value of name (city) to stripped down version
    api_city_data.set_value(index, 'country', lower_country) # reset the value of country to lower case
    
api_city_data = api_city_data.rename(columns = {'name': 'city'}) # rename for merge 
```


```python
#merge with random cities from location_data

merged_df = location_data.merge(api_city_data, how = 'left', on = ('city', 'country'))
merged_df = merged_df.drop_duplicates(['city', 'country'])

# to verify with number below
len(merged_df)
```




    609




```python
#check with above
rec_check
```




    609




```python
#preview merged_df
merged_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>chute-aux-outardes</td>
      <td>ca</td>
      <td>{'lon': -68.398956, 'lat': 49.116791}</td>
      <td>5922281.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>dengzhou</td>
      <td>cn</td>
      <td>{'lon': 112.08194, 'lat': 32.68222}</td>
      <td>1812990.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ilek</td>
      <td>ru</td>
      <td>{'lon': 53.38306, 'lat': 51.527088}</td>
      <td>557340.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>airai</td>
      <td>pw</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
#clean-up
merged_df['coord'] = merged_df['coord'].fillna('') #fill na cells with emplty string for coordinates
merged_df['id'] = merged_df['id'].fillna(0) # fill na with 0 for id in order to change to int64
merged_df['id'] = merged_df['id'].astype(dtype = 'int64') # cast id column as type int64 to remove floating .0
merged_df['id'].dtype #check type of id
```




    dtype('int64')




```python
#check how many returned valid ids
#merged_df['id'].value_counts()      
```


```python
# check which countries did not find ids
no_id = merged_df[merged_df['id'] == 0]
no_id.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>airai</td>
      <td>pw</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>illoqqortoormiut</td>
      <td>gl</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>samusu</td>
      <td>ws</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>belushya guba</td>
      <td>ru</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>barentsburg</td>
      <td>sj</td>
      <td></td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check how many without ids
len(no_id)
```




    106




```python
# Google Api key
gkey = 'AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis'
```


```python
#trying to find lat and lng for cities missing ids from google geocoding api
g_url = 'https://maps.googleapis.com/maps/api/geocode/json?address='

counter = 0 #for check of all cities
for index,row in merged_df.iterrows():
    if row['id'] == 0:
        city = row['city']
        country = row['country']
        print('Now retrieving coordinates for city #%s: %s, %s' %(index, city, country))
        target_url = '%s%s,+%s&key=%s' % (g_url, city, country, gkey)
        print(target_url)
        try:
            response = req.get(target_url).json()
            response_path = response['results'][0]['geometry']['location']
            merged_df.set_value(index, 'coord', {'lon': response_path['lng'], 'lat': response_path['lat']})
        except:
            print('Missing Data for city #%s: %s,%s' %(index, city, country))
        counter += 1
#     if counter == 10:
#             break

print(counter) #to check for same number of records as no_id
```

    Now retrieving coordinates for city #4: airai, pw
    https://maps.googleapis.com/maps/api/geocode/json?address=airai,+pw&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #19: illoqqortoormiut, gl
    https://maps.googleapis.com/maps/api/geocode/json?address=illoqqortoormiut,+gl&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #20: samusu, ws
    https://maps.googleapis.com/maps/api/geocode/json?address=samusu,+ws&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #30: belushya guba, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=belushya guba,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #34: barentsburg, sj
    https://maps.googleapis.com/maps/api/geocode/json?address=barentsburg,+sj&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #38: fevralsk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=fevralsk,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #42: mataura, pf
    https://maps.googleapis.com/maps/api/geocode/json?address=mataura,+pf&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #53: sorvag, fo
    https://maps.googleapis.com/maps/api/geocode/json?address=sorvag,+fo&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #57: bud, no
    https://maps.googleapis.com/maps/api/geocode/json?address=bud,+no&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #60: alekseyevsk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=alekseyevsk,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #88: taolanaro, mg
    https://maps.googleapis.com/maps/api/geocode/json?address=taolanaro,+mg&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #92: gulshat, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=gulshat,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #93: sohag, eg
    https://maps.googleapis.com/maps/api/geocode/json?address=sohag,+eg&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #95: amderma, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=amderma,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #96: meyungs, pw
    https://maps.googleapis.com/maps/api/geocode/json?address=meyungs,+pw&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #97: nizhneyansk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=nizhneyansk,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #104: grand river south east, mu
    https://maps.googleapis.com/maps/api/geocode/json?address=grand river south east,+mu&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #127: litoral del san juan, co
    https://maps.googleapis.com/maps/api/geocode/json?address=litoral del san juan,+co&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #128: itarema, br
    https://maps.googleapis.com/maps/api/geocode/json?address=itarema,+br&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #140: chabahar, ir
    https://maps.googleapis.com/maps/api/geocode/json?address=chabahar,+ir&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #141: yomitan, jp
    https://maps.googleapis.com/maps/api/geocode/json?address=yomitan,+jp&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #146: palabuhanratu, id
    https://maps.googleapis.com/maps/api/geocode/json?address=palabuhanratu,+id&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #157: saleaula, ws
    https://maps.googleapis.com/maps/api/geocode/json?address=saleaula,+ws&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #158: tumannyy, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=tumannyy,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #159: nikolskoye, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=nikolskoye,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #170: kazalinsk, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=kazalinsk,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #179: vaitupu, wf
    https://maps.googleapis.com/maps/api/geocode/json?address=vaitupu,+wf&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #184: lasa, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=lasa,+cn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #185: biak, id
    https://maps.googleapis.com/maps/api/geocode/json?address=biak,+id&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #186: bac lieu, vn
    https://maps.googleapis.com/maps/api/geocode/json?address=bac lieu,+vn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #198: san jeronimo, mx
    https://maps.googleapis.com/maps/api/geocode/json?address=san jeronimo,+mx&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #199: scottsburgh, za
    https://maps.googleapis.com/maps/api/geocode/json?address=scottsburgh,+za&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #205: constitucion, mx
    https://maps.googleapis.com/maps/api/geocode/json?address=constitucion,+mx&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #207: raudeberg, no
    https://maps.googleapis.com/maps/api/geocode/json?address=raudeberg,+no&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #219: krasnoselkup, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=krasnoselkup,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #223: fort saint james, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=fort saint james,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #225: codrington, ag
    https://maps.googleapis.com/maps/api/geocode/json?address=codrington,+ag&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #229: babanusah, sd
    https://maps.googleapis.com/maps/api/geocode/json?address=babanusah,+sd&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #260: sentyabrskiy, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=sentyabrskiy,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #263: rodrigues alves, br
    https://maps.googleapis.com/maps/api/geocode/json?address=rodrigues alves,+br&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #273: burica, pa
    https://maps.googleapis.com/maps/api/geocode/json?address=burica,+pa&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #283: lushunkou, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=lushunkou,+cn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #295: katobu, id
    https://maps.googleapis.com/maps/api/geocode/json?address=katobu,+id&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #296: umzimvubu, za
    https://maps.googleapis.com/maps/api/geocode/json?address=umzimvubu,+za&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #297: bereda, so
    https://maps.googleapis.com/maps/api/geocode/json?address=bereda,+so&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #300: yambio, sd
    https://maps.googleapis.com/maps/api/geocode/json?address=yambio,+sd&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Missing Data for city #300: yambio,sd
    Now retrieving coordinates for city #306: katsiveli, ua
    https://maps.googleapis.com/maps/api/geocode/json?address=katsiveli,+ua&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #312: jalu, ly
    https://maps.googleapis.com/maps/api/geocode/json?address=jalu,+ly&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #327: mys shmidta, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=mys shmidta,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #360: makat, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=makat,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #365: alekseyevka, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=alekseyevka,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #382: ahumada, mx
    https://maps.googleapis.com/maps/api/geocode/json?address=ahumada,+mx&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #384: avera, pf
    https://maps.googleapis.com/maps/api/geocode/json?address=avera,+pf&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #389: bairiki, ki
    https://maps.googleapis.com/maps/api/geocode/json?address=bairiki,+ki&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #393: qui nhon, vn
    https://maps.googleapis.com/maps/api/geocode/json?address=qui nhon,+vn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #399: attawapiskat, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=attawapiskat,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #407: ust-kamchatsk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=ust-kamchatsk,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #420: tsihombe, mg
    https://maps.googleapis.com/maps/api/geocode/json?address=tsihombe,+mg&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #426: khonuu, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=khonuu,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #441: potgietersrus, za
    https://maps.googleapis.com/maps/api/geocode/json?address=potgietersrus,+za&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #461: longlac, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=longlac,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #470: dinsor, so
    https://maps.googleapis.com/maps/api/geocode/json?address=dinsor,+so&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #472: mutsamudu, km
    https://maps.googleapis.com/maps/api/geocode/json?address=mutsamudu,+km&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #473: fort saint john, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=fort saint john,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #475: aras, no
    https://maps.googleapis.com/maps/api/geocode/json?address=aras,+no&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #476: mahon, es
    https://maps.googleapis.com/maps/api/geocode/json?address=mahon,+es&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #481: seara, br
    https://maps.googleapis.com/maps/api/geocode/json?address=seara,+br&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #485: balkhash, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=balkhash,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #486: khani, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=khani,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #499: jiddah, sa
    https://maps.googleapis.com/maps/api/geocode/json?address=jiddah,+sa&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #505: artyk, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=artyk,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #511: kaitangata, nz
    https://maps.googleapis.com/maps/api/geocode/json?address=kaitangata,+nz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #512: boralday, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=boralday,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #515: odweyne, so
    https://maps.googleapis.com/maps/api/geocode/json?address=odweyne,+so&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #525: erenhot, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=erenhot,+cn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #526: faya, td
    https://maps.googleapis.com/maps/api/geocode/json?address=faya,+td&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #540: dzhusaly, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=dzhusaly,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #541: saint anthony, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=saint anthony,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #542: cheuskiny, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=cheuskiny,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Missing Data for city #542: cheuskiny,ru
    Now retrieving coordinates for city #563: macaboboni, ph
    https://maps.googleapis.com/maps/api/geocode/json?address=macaboboni,+ph&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #568: yanan, cn
    https://maps.googleapis.com/maps/api/geocode/json?address=yanan,+cn&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #574: grand centre, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=grand centre,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #575: doha, kw
    https://maps.googleapis.com/maps/api/geocode/json?address=doha,+kw&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #584: tarudant, ma
    https://maps.googleapis.com/maps/api/geocode/json?address=tarudant,+ma&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #590: kerteh, my
    https://maps.googleapis.com/maps/api/geocode/json?address=kerteh,+my&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #605: karaul, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=karaul,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #607: marcona, pe
    https://maps.googleapis.com/maps/api/geocode/json?address=marcona,+pe&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #614: zachagansk, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=zachagansk,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Missing Data for city #614: zachagansk,kz
    Now retrieving coordinates for city #619: tunduru, tz
    https://maps.googleapis.com/maps/api/geocode/json?address=tunduru,+tz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #620: louisbourg, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=louisbourg,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #625: ekhabi, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=ekhabi,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #635: bosaso, so
    https://maps.googleapis.com/maps/api/geocode/json?address=bosaso,+so&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #639: asau, tv
    https://maps.googleapis.com/maps/api/geocode/json?address=asau,+tv&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #640: pakwach, ug
    https://maps.googleapis.com/maps/api/geocode/json?address=pakwach,+ug&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #645: zhezkazgan, kz
    https://maps.googleapis.com/maps/api/geocode/json?address=zhezkazgan,+kz&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #648: lolua, tv
    https://maps.googleapis.com/maps/api/geocode/json?address=lolua,+tv&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Missing Data for city #648: lolua,tv
    Now retrieving coordinates for city #681: chagda, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=chagda,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #682: sobolevo, ru
    https://maps.googleapis.com/maps/api/geocode/json?address=sobolevo,+ru&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #690: wuchi, tw
    https://maps.googleapis.com/maps/api/geocode/json?address=wuchi,+tw&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #718: abu samrah, qa
    https://maps.googleapis.com/maps/api/geocode/json?address=abu samrah,+qa&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #719: mizpe ramon, il
    https://maps.googleapis.com/maps/api/geocode/json?address=mizpe ramon,+il&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #729: saint-georges, gf
    https://maps.googleapis.com/maps/api/geocode/json?address=saint-georges,+gf&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #733: wajid, so
    https://maps.googleapis.com/maps/api/geocode/json?address=wajid,+so&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #736: cabo rojo, us
    https://maps.googleapis.com/maps/api/geocode/json?address=cabo rojo,+us&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #739: moose factory, ca
    https://maps.googleapis.com/maps/api/geocode/json?address=moose factory,+ca&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    Now retrieving coordinates for city #743: lata, sb
    https://maps.googleapis.com/maps/api/geocode/json?address=lata,+sb&key=AIzaSyDehYwx_mkgNA8qUnS6a4XqALdqaChXwis
    106



```python
#preview merged_df
merged_df.head(30)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>chute-aux-outardes</td>
      <td>ca</td>
      <td>{'lon': -68.398956, 'lat': 49.116791}</td>
      <td>5922281</td>
    </tr>
    <tr>
      <th>1</th>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903</td>
    </tr>
    <tr>
      <th>2</th>
      <td>dengzhou</td>
      <td>cn</td>
      <td>{'lon': 112.08194, 'lat': 32.68222}</td>
      <td>1812990</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ilek</td>
      <td>ru</td>
      <td>{'lon': 53.38306, 'lat': 51.527088}</td>
      <td>557340</td>
    </tr>
    <tr>
      <th>4</th>
      <td>airai</td>
      <td>pw</td>
      <td>{'lon': 134.5690225, 'lat': 7.396611799999999}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>cape town</td>
      <td>za</td>
      <td>{'lon': 18.42322, 'lat': -33.925838}</td>
      <td>3369157</td>
    </tr>
    <tr>
      <th>6</th>
      <td>dunshaughlin</td>
      <td>ie</td>
      <td>{'lon': -6.54, 'lat': 53.512501}</td>
      <td>2964472</td>
    </tr>
    <tr>
      <th>7</th>
      <td>methoni</td>
      <td>gr</td>
      <td>{'lon': 21.704861, 'lat': 36.819729}</td>
      <td>257122</td>
    </tr>
    <tr>
      <th>8</th>
      <td>chuy</td>
      <td>uy</td>
      <td>{'lon': -53.46162, 'lat': -33.697071}</td>
      <td>3443061</td>
    </tr>
    <tr>
      <th>9</th>
      <td>hobart</td>
      <td>au</td>
      <td>{'lon': 147.329407, 'lat': -42.87936}</td>
      <td>2163355</td>
    </tr>
    <tr>
      <th>11</th>
      <td>naze</td>
      <td>jp</td>
      <td>{'lon': 129.483337, 'lat': 28.366671}</td>
      <td>1855540</td>
    </tr>
    <tr>
      <th>12</th>
      <td>pevek</td>
      <td>ru</td>
      <td>{'lon': 170.313324, 'lat': 69.700829}</td>
      <td>2122090</td>
    </tr>
    <tr>
      <th>13</th>
      <td>albany</td>
      <td>au</td>
      <td>{'lon': 118.123451, 'lat': -34.7099}</td>
      <td>7839657</td>
    </tr>
    <tr>
      <th>15</th>
      <td>saldanha</td>
      <td>za</td>
      <td>{'lon': 17.944201, 'lat': -33.011669}</td>
      <td>3361934</td>
    </tr>
    <tr>
      <th>16</th>
      <td>hualmay</td>
      <td>pe</td>
      <td>{'lon': -77.613892, 'lat': -11.09639}</td>
      <td>3939761</td>
    </tr>
    <tr>
      <th>17</th>
      <td>hilo</td>
      <td>us</td>
      <td>{'lon': -155.089996, 'lat': 19.729719}</td>
      <td>5855927</td>
    </tr>
    <tr>
      <th>18</th>
      <td>kenai</td>
      <td>us</td>
      <td>{'lon': -151.258331, 'lat': 60.55444}</td>
      <td>5866063</td>
    </tr>
    <tr>
      <th>19</th>
      <td>illoqqortoormiut</td>
      <td>gl</td>
      <td>{'lon': -21.9628757, 'lat': 70.48556909999999}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>samusu</td>
      <td>ws</td>
      <td>{'lon': -171.4299586, 'lat': -14.0056774}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>colares</td>
      <td>pt</td>
      <td>{'lon': -9.46305, 'lat': 38.802059}</td>
      <td>8012548</td>
    </tr>
    <tr>
      <th>23</th>
      <td>busselton</td>
      <td>au</td>
      <td>{'lon': 115.370796, 'lat': -33.684769}</td>
      <td>7839477</td>
    </tr>
    <tr>
      <th>25</th>
      <td>tiznit</td>
      <td>ma</td>
      <td>{'lon': -9.5, 'lat': 29.58333}</td>
      <td>2527087</td>
    </tr>
    <tr>
      <th>28</th>
      <td>kungurtug</td>
      <td>ru</td>
      <td>{'lon': 97.522781, 'lat': 50.599442}</td>
      <td>1501377</td>
    </tr>
    <tr>
      <th>29</th>
      <td>lavrentiya</td>
      <td>ru</td>
      <td>{'lon': -171, 'lat': 65.583328}</td>
      <td>4031637</td>
    </tr>
    <tr>
      <th>30</th>
      <td>belushya guba</td>
      <td>ru</td>
      <td>{'lon': 52.32027799999999, 'lat': 71.545555999...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>zhuhai</td>
      <td>cn</td>
      <td>{'lon': 113.56778, 'lat': 22.276939}</td>
      <td>1790437</td>
    </tr>
    <tr>
      <th>32</th>
      <td>sitka</td>
      <td>us</td>
      <td>{'lon': -135.330002, 'lat': 57.053059}</td>
      <td>5557293</td>
    </tr>
    <tr>
      <th>33</th>
      <td>kerman</td>
      <td>ir</td>
      <td>{'lon': 57.078789, 'lat': 30.283211}</td>
      <td>128234</td>
    </tr>
    <tr>
      <th>34</th>
      <td>barentsburg</td>
      <td>sj</td>
      <td>{'lon': 14.2334597, 'lat': 78.0648475}</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>vaini</td>
      <td>to</td>
      <td>{'lon': -175.199997, 'lat': -21.200001}</td>
      <td>4032243</td>
    </tr>
  </tbody>
</table>
</div>




```python
#check with below
len(merged_df)
```




    609




```python
#check with above
rec_check
```




    609




```python
#check to see how many records with no coordinates
no_coord = merged_df[merged_df['coord'] == ""]
no_coord
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>300</th>
      <td>yambio</td>
      <td>sd</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>542</th>
      <td>cheuskiny</td>
      <td>ru</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>614</th>
      <td>zachagansk</td>
      <td>kz</td>
      <td></td>
      <td>0</td>
    </tr>
    <tr>
      <th>648</th>
      <td>lolua</td>
      <td>tv</td>
      <td></td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# of records without coordinates
len(no_coord)
```




    4




```python
#leave merged_df the same from here on
weather_data = merged_df.copy()
```


```python
# Open weather API key
wkey = '3c995f5860f5c2480a49b210cc497c1e'
```


```python
#preview weather_data to check for success
weather_data.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>chute-aux-outardes</td>
      <td>ca</td>
      <td>{'lon': -68.398956, 'lat': 49.116791}</td>
      <td>5922281</td>
    </tr>
    <tr>
      <th>1</th>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903</td>
    </tr>
    <tr>
      <th>2</th>
      <td>dengzhou</td>
      <td>cn</td>
      <td>{'lon': 112.08194, 'lat': 32.68222}</td>
      <td>1812990</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ilek</td>
      <td>ru</td>
      <td>{'lon': 53.38306, 'lat': 51.527088}</td>
      <td>557340</td>
    </tr>
    <tr>
      <th>4</th>
      <td>airai</td>
      <td>pw</td>
      <td>{'lon': 134.5690225, 'lat': 7.396611799999999}</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Open Weather Maps Api example links for reference
# id url example: 'api.openweathermap.org/data/2.5/weather?id=2172797'
# coord url example: 'api.openweathermap.org/data/2.5/weather?lat=35&lon=139'
# city search use : api.openweathermap.org/data/2.5/weather?q=London,uk
# remember to use &appid= for key

counter = 0 #for breaking and pausing
cur_err_list = [] # for cities without data from current weather
for_err_list = [] # for cities without data from forecast data
cur_errors = 0  #current weather pull errors
for_errors = 0 #forecast weather pull errors

#create additional columns for open weather map data

#make columnds for lat and lng from open weather source
weather_data['lat'] = ""
weather_data['lng'] = ""

#make columns for current weather data (at time of pull)
weather_data['cur_date'] = ""
weather_data['cur_temp'] = ""
weather_data['cur_humidity'] = ""
weather_data['cur_clouds'] = ""
weather_data['cur_wind'] = ""

# make columns for records corresponding to the highest temperature 
# forecasted in the next 24 hours (from time of pull)
weather_data['max_date'] = ""
weather_data['max_temp'] = ""
weather_data['max_temp_humidity'] = ""
weather_data['max_temp_clouds'] = ""
weather_data['max_temp_wind'] = ""

# make columns for records corresponding to the average values
# forecasted in the next 5 days (from time of pull)
weather_data['avg_date0'] = ""
weather_data['avg_date1'] = ""
weather_data['avg_temp'] = ""
weather_data['avg_humidity'] = ""
weather_data['avg_clouds'] = ""
weather_data['avg_wind'] = ""

t0 = time.time() #for pause timer
for index, row in weather_data.iterrows():
    print('Now retrieving data for city #%s: %s, %s' % (index, row['city'], row['country']))
    #uses url believed to be most accurate
    if ((row['id']) == 0) and (row['coord'] != ""): # coordinates if no id, but with coordinates
        lat = row['coord']['lat']
        lon = row['coord']['lon']
        cur_url = 'https://api.openweathermap.org/data/2.5/weather?lat=%s&lon=%s&APPID=%s&units=imperial' % (lat, lon, wkey)  
        for_url = 'https://api.openweathermap.org/data/2.5/forecast?lat=%s&lon=%s&APPID=%s&units=imperial' % (lat, lon, wkey)  
    elif row['id'] != 0: # use if if ID exists
        loc_id = row['id']
        cur_url = 'https://api.openweathermap.org/data/2.5/weather?id=%s&APPID=%s&units=imperial' % (loc_id, wkey)
        for_url = 'https://api.openweathermap.org/data/2.5/forecast?id=%s&APPID=%s&units=imperial' % (loc_id, wkey)
    else: #use city and country if no id AND no coordinates
        city = row['city']
        country = row['country']
        cur_url = 'https://api.openweathermap.org/data/2.5/weather?q=%s,%s&APPID=%s&units=imperial' % (city, country, wkey)
        for_url = 'https://api.openweathermap.org/data/2.5/forecast?q=%s,%s&APPID=%s&units=imperial' % (city, country, wkey)
    print('Current Weather URL:')
    print(cur_url)
    print('Forecast Weather URL:')
    print(for_url)
    #get current weather data
    try:
        cur_response = req.get(cur_url).json()
        weather_data.set_value(index, 'lat', cur_response['coord']['lat'])
        weather_data.set_value(index, 'lng', cur_response['coord']['lon'])
        weather_data.set_value(index, 'cur_date', cur_response['dt'])
        weather_data.set_value(index, 'cur_temp', cur_response['main']['temp'])
        weather_data.set_value(index, 'cur_humidity', cur_response['main']['humidity'])
        weather_data.set_value(index, 'cur_clouds', cur_response['clouds']['all'])
        weather_data.set_value(index, 'cur_wind', cur_response['wind']['speed'])
    except:
        print('Missing Current Weather Info for city #%s: %s, %s' % (index, row['city'], row['country']))
        cur_err_list.append(index)
        cur_errors += 1
    try:
        #get max temperature weather data
        for_response = req.get(for_url).json()
        for_path = for_response['list']
        temps_24h = [] # a list of temp over a 24 hours period forecased every 3 hours
        for n in range(9): # a 24 hour period
            temps_24h.append(for_path[n]['main']['temp_max'])
        max_index = temps_24h.index(max(temps_24h))
        weather_data.set_value(index, 'max_date', for_path[max_index]['dt'])
        weather_data.set_value(index, 'max_temp', for_path[max_index]['main']['temp_max'])
        weather_data.set_value(index, 'max_temp_humidity', for_path[max_index]['main']['humidity'])
        weather_data.set_value(index, 'max_temp_clouds', for_path[max_index]['clouds']['all'])
        weather_data.set_value(index, 'max_temp_wind', for_path[max_index]['wind']['speed'])
        # get avg forecast values 
        #set up blank lists for dates, temperature, clouds, wind, humidity over a 5 day period
        dat = []
        tem = []
        clo = []
        win = []
        hum = []
        for n in for_path: # 5 days worth of forecast
            dat.append(n['dt'])
            tem.append(n['main']['temp'])
            clo.append(n['clouds']['all'])
            win.append(n['wind']['speed'])
            hum.append(n['main']['humidity'])
        weather_data.set_value(index, 'avg_date0', dat[0]) #beginning date
        weather_data.set_value(index, 'avg_date1', dat[-1]) #ending date
        weather_data.set_value(index, 'avg_temp', np.mean(tem)) # mean temp over 5 days
        weather_data.set_value(index, 'avg_humidity', np.mean(hum)) #mean humidity over 5 days
        weather_data.set_value(index, 'avg_clouds', np.mean(clo)) #mean cloud cover over 5 days
        weather_data.set_value(index, 'avg_wind', np.mean(win)) #mean wind speed over 5 days
    except:
        print('Missing Forecast Info for city #%s: %s, %s' % (index, row['city'], row['country']))
        for_err_list.append(index)
        for_errors += 1
    print('---------------------------------------------------------------------------')
    counter +=1
    if counter % 30 == 0: #because two records pulled for each city 
        t1 = time.time() #records time very 30 records
        sl_time = 70 - (t1-t0) # calculates buffer for api pull limit
        print("")
        print('********Sleeping for %s seconds.********' % (sl_time))
        print("")
        time.sleep(sl_time) # pauses for appropraite amount of time
        t0 = time.time() # resets for next 30 pull timer
#     if counter == 5:
#         break

```

    Now retrieving data for city #0: chute-aux-outardes, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5922281&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5922281&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #1: jamestown, sh
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3370903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3370903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #2: dengzhou, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1812990&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1812990&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #3: ilek, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=557340&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=557340&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #4: airai, pw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=7.396611799999999&lon=134.5690225&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=7.396611799999999&lon=134.5690225&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #5: cape town, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3369157&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3369157&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #6: dunshaughlin, ie
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2964472&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2964472&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #7: methoni, gr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=257122&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=257122&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #8: chuy, uy
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3443061&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3443061&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #9: hobart, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2163355&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2163355&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #11: naze, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1855540&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1855540&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #12: pevek, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122090&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2122090&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #13: albany, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839657&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839657&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #15: saldanha, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3361934&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3361934&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #16: hualmay, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3939761&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3939761&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #17: hilo, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5855927&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5855927&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #18: kenai, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5866063&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5866063&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #19: illoqqortoormiut, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=70.48556909999999&lon=-21.9628757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=70.48556909999999&lon=-21.9628757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #20: samusu, ws
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-14.0056774&lon=-171.4299586&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-14.0056774&lon=-171.4299586&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #21: colares, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8012548&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8012548&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #23: busselton, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839477&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839477&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #25: tiznit, ma
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2527087&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2527087&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #28: kungurtug, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1501377&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1501377&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #29: lavrentiya, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4031637&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4031637&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #30: belushya guba, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=71.54555599999999&lon=52.32027799999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=71.54555599999999&lon=52.32027799999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #31: zhuhai, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1790437&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1790437&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #32: sitka, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5557293&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5557293&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #33: kerman, ir
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=128234&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=128234&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #34: barentsburg, sj
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=78.0648475&lon=14.2334597&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=78.0648475&lon=14.2334597&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #35: vaini, to
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4032243&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4032243&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.918968200683594 seconds.********
    
    Now retrieving data for city #36: mlonggo, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1635164&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1635164&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #37: husavik, is
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2629833&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2629833&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #38: fevralsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=52.44872650000001&lon=130.8547973&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=52.44872650000001&lon=130.8547973&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #39: dunedin, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2191562&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2191562&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #40: half moon bay, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5354943&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5354943&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #41: san patricio, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3985168&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3985168&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #42: mataura, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-23.3470634&lon=-149.4850445&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-23.3470634&lon=-149.4850445&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #43: atuona, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4020109&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4020109&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #44: rio gallegos, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3838859&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3838859&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #45: dikson, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1507390&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1507390&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #46: hermanus, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3366880&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3366880&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #47: hithadhoo, mv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1282256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1282256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #48: barrow, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5880054&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5880054&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #49: east london, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1006984&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1006984&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #50: avarua, ck
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4035715&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4035715&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #51: mount isa, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2065594&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2065594&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #53: sorvag, fo
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=62.07129279999999&lon=-7.3058656&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=62.07129279999999&lon=-7.3058656&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #54: almaznyy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027859&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2027859&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #56: sao filipe, cv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3374210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #57: bud, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=59.13605109999999&lon=10.2086682&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=59.13605109999999&lon=10.2086682&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #58: punta arenas, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3874787&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3874787&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #59: port alfred, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=964432&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=964432&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #60: alekseyevsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=50.6256541&lon=38.6978113&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=50.6256541&lon=38.6978113&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #61: meulaboh, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1214488&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1214488&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #62: laguna, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3459094&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3459094&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #63: castro, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3896218&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3896218&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #64: algiers, dz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2507480&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2507480&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #65: bethel, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4182260&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4182260&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #76: saskylakh, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2017155&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2017155&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #77: bardiyah, ly
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=80509&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=80509&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 50.40618109703064 seconds.********
    
    Now retrieving data for city #78: carnarvon, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2074865&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2074865&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #79: banda aceh, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1215501&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1215501&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #81: aklavik, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5882953&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5882953&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #82: tautira, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4033557&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4033557&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #83: bredasdorp, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1015776&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1015776&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #84: emerald, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2167425&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2167425&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #86: abastumani, ge
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=584502&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=584502&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #87: new norfolk, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2155415&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2155415&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #88: taolanaro, mg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-24.5965316&lon=46.9028375&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-24.5965316&lon=46.9028375&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #89: kodiak, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5866583&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5866583&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #90: norman wells, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6089245&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6089245&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #91: pimentel, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3693584&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3693584&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #92: gulshat, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=46.6292771&lon=74.3591879&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=46.6292771&lon=74.3591879&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #93: sohag, eg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=26.5590737&lon=31.6956705&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=26.5590737&lon=31.6956705&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #94: tuktoyaktuk, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6170031&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6170031&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #95: amderma, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=69.751221&lon=61.6636961&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=69.751221&lon=61.6636961&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #96: meyungs, pw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=7.352715099999999&lon=134.4488579&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=7.352715099999999&lon=134.4488579&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #97: nizhneyansk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=71.450058&lon=136.1122279&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=71.450058&lon=136.1122279&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #98: victoria, sc
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=241131&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=241131&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #99: les ponts-de-ce, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2999908&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2999908&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #100: asino, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1511309&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1511309&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #101: fortuna, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5563839&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5563839&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #102: rikitea, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4030556&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4030556&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #103: roald, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3141667&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3141667&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #104: grand river south east, mu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-20.2888094&lon=57.78141199999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-20.2888094&lon=57.78141199999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #105: ushuaia, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3833367&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3833367&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #106: abha, sa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=110690&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=110690&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #107: motygino, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1498314&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1498314&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #108: mar del plata, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3430863&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3430863&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #109: alofi, nu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4036284&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4036284&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 48.82935404777527 seconds.********
    
    Now retrieving data for city #110: ust-maya, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2013918&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2013918&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #111: mountain home, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4123037&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4123037&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #114: lieksa, fi
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=648091&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=648091&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #116: geraldton, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5960603&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5960603&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #117: marrakesh, ma
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2542997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2542997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #118: qaanaaq, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3831208&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3831208&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #119: dalvik, is
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2632287&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2632287&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #120: severo-kurilsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2121385&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2121385&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #121: bukama, cd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=217834&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=217834&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #122: provideniya, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4031574&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4031574&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #123: paraiso, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3521912&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3521912&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #124: sinfra, ci
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2281606&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2281606&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #125: nanakuli, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5851349&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5851349&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #126: tahta, eg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=347634&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=347634&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #127: litoral del san juan, co
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=4.2006777&lon=-76.7336521&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=4.2006777&lon=-76.7336521&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #128: itarema, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-2.9209642&lon=-39.91673979999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-2.9209642&lon=-39.91673979999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #129: port hardy, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6111862&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6111862&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #130: ialibu, pg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2095925&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2095925&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #131: summerside, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6159244&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6159244&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #132: luderitz, na
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3355672&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3355672&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #133: chissamba, ao
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3349580&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3349580&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #134: saint-philippe, re
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690301&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6690301&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #136: bengkulu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1649150&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1649150&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #137: turukhansk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1488903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1488903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #138: leningradskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2123814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2123814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #139: sandakan, my
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1734052&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1734052&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #140: chabahar, ir
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=25.296878&lon=60.6459285&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=25.296878&lon=60.6459285&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #141: yomitan, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=26.3960554&lon=127.7442772&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=26.3960554&lon=127.7442772&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #142: outjo, na
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3353715&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3353715&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #143: bambous virieux, mu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1106677&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1106677&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 51.49172306060791 seconds.********
    
    Now retrieving data for city #144: ribeira grande, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8010689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #146: palabuhanratu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-6.9852341&lon=106.5475399&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-6.9852341&lon=106.5475399&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #147: wuwei, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1803936&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1803936&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #148: prince george, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6113365&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6113365&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #149: chokurdakh, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126123&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2126123&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #150: port elizabeth, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=964420&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=964420&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #151: ust-koksa, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1488167&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1488167&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #152: harper, lr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2276492&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2276492&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #153: comodoro rivadavia, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3860443&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3860443&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #154: cabo san lucas, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3985710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3985710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #155: antalaha, mg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1071296&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1071296&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #156: bluff, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206939&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2206939&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #157: saleaula, ws
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-13.4482906&lon=-172.3367114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-13.4482906&lon=-172.3367114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #158: tumannyy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=68.878911&lon=35.6675264&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=68.878911&lon=35.6675264&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #159: nikolskoye, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=55.1981604&lon=166.0015368&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=55.1981604&lon=166.0015368&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #160: broome, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839347&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839347&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #162: sinnamary, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690707&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6690707&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #164: puerto madryn, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3840092&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3840092&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #165: kapaa, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5848280&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5848280&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #166: rajura, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1258786&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1258786&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #167: cockburn town, bs
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3572627&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3572627&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #168: fairbanks, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5861897&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5861897&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #169: isangel, vu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2136825&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2136825&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #170: kazalinsk, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=45.7640559&lon=62.0999125&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=45.7640559&lon=62.0999125&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #171: lippstadt, de
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2876865&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2876865&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #172: georgetown, sh
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2411397&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2411397&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #173: san cristobal, ec
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3651949&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3651949&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #174: kentville, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5991148&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5991148&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #175: sumbe, ao
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3346015&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3346015&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #176: zirandaro, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3979647&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3979647&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 49.66289710998535 seconds.********
    
    Now retrieving data for city #177: cartagena, co
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3687238&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3687238&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #178: kloulklubed, pw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7671223&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7671223&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #179: vaitupu, wf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-13.2308863&lon=-176.1917035&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-13.2308863&lon=-176.1917035&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #180: stromness, gb
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2636638&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2636638&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #181: kushiro, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2129376&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2129376&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #183: rawson, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3839307&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3839307&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #184: lasa, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=29.652491&lon=91.17210999999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=29.652491&lon=91.17210999999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #185: biak, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-1.0381022&lon=135.9800848&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-1.0381022&lon=135.9800848&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #186: bac lieu, vn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=9.2573324&lon=105.7557791&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=9.2573324&lon=105.7557791&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #187: vao, nc
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2137773&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2137773&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #188: geraldton, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2070998&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2070998&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #189: arraial do cabo, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3471451&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3471451&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #190: ovsyanka, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1495797&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1495797&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #192: hastings, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2190224&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2190224&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #193: shimoda, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1852357&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1852357&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #194: businga, cd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=217637&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=217637&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #195: yar-sale, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1486321&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1486321&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #196: yellowknife, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6185377&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6185377&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #197: la asuncion, ve
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3480908&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3480908&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #198: san jeronimo, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=20.4061494&lon=-103.9854708&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=20.4061494&lon=-103.9854708&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #199: scottsburgh, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-30.2862643&lon=30.7551101&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-30.2862643&lon=30.7551101&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #200: vanimo, pg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2084442&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2084442&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #201: cherskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126199&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2126199&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #202: kahului, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5847411&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5847411&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #203: crib point, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2169995&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2169995&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #204: kudahuvadhoo, mv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1337607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1337607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #205: constitucion, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=20.729735&lon=-103.368837&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=20.729735&lon=-103.368837&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #206: pointe michel, dm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3575660&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3575660&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #207: raudeberg, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=61.9829132&lon=5.1351057&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=61.9829132&lon=5.1351057&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #208: tessalit, ml
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2449893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2449893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.627156019210815 seconds.********
    
    Now retrieving data for city #209: mayo, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6068416&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6068416&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #210: hirara, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1862505&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1862505&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #211: honiara, sb
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2108502&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2108502&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #212: praia da vitoria, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010692&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8010692&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #214: faanui, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4034551&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4034551&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #215: mana, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690694&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6690694&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #217: bolgatanga, gh
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2302821&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2302821&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #218: tiksi, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2015306&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2015306&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #219: krasnoselkup, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=65.69455599999999&lon=82.525613&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=65.69455599999999&lon=82.525613&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #220: charters towers, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839570&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839570&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #222: viking, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6174254&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6174254&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #223: fort saint james, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=54.4425721&lon=-124.2510474&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=54.4425721&lon=-124.2510474&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #224: teya, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1489656&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1489656&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #225: codrington, ag
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=17.6425736&lon=-61.8204456&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=17.6425736&lon=-61.8204456&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #226: port lincoln, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839452&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839452&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #228: mareeba, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2158767&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2158767&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #229: babanusah, sd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=11.320725&lon=27.8135579&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=11.320725&lon=27.8135579&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #230: nueva guinea, ni
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3617459&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3617459&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #231: flinders, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2166453&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2166453&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #235: dhone, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1272722&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1272722&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #236: nadvoitsy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=523662&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=523662&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #237: souillac, mu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=933995&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=933995&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #238: guerrero negro, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4021858&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4021858&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #239: mahbubnagar, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1264407&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1264407&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #240: ust-kulom, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=478050&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=478050&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #241: sur, om
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=286245&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=286245&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #242: hofn, is
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2630299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2630299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #243: general roca, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3855065&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3855065&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #245: shingu, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1852109&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1852109&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #247: talas, kg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1527299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1527299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 50.19046497344971 seconds.********
    
    Now retrieving data for city #249: neiafu, to
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4032420&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4032420&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #250: san rafael del sur, ni
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3616594&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3616594&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #251: akora, pk
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1184752&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1184752&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #252: lagoa, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010517&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8010517&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #257: altamira, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3472492&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3472492&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #260: sentyabrskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=60.493056&lon=72.19638909999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=60.493056&lon=72.19638909999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #261: torbay, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6167817&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6167817&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #262: nhulunbuy, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2064735&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2064735&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #263: rodrigues alves, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-7.881874000000001&lon=-73.37086959999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-7.881874000000001&lon=-73.37086959999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #264: eureka, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5563397&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5563397&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #270: khatanga, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2022572&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2022572&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #271: waipawa, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206874&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2206874&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #272: kaoma, zm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=913323&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=913323&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #273: burica, pa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=8.9933743&lon=-79.5388265&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=8.9933743&lon=-79.5388265&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #274: ambilobe, mg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1082243&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1082243&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #275: surany, sk
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3057339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3057339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #276: mount gambier, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839440&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839440&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #278: mercedes, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3430708&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3430708&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #280: beloha, mg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1067565&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1067565&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #281: saint george, bm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3573062&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3573062&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #283: lushunkou, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=38.851705&lon=121.261953&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=38.851705&lon=121.261953&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #284: tateyama, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1850523&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1850523&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #285: brae, gb
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2654970&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2654970&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #286: pandan, ph
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1695546&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1695546&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #290: kruisfontein, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=986717&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=986717&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #291: nouadhibou, mr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2377457&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2377457&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #292: agadez, ne
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2448083&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2448083&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #294: puerto ayora, ec
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3652764&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3652764&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #295: katobu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-4.830303199999999&lon=122.7124903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-4.830303199999999&lon=122.7124903&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #296: umzimvubu, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-30.7781755&lon=28.9528645&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-30.7781755&lon=28.9528645&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 50.44405698776245 seconds.********
    
    Now retrieving data for city #297: bereda, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=37.7538318&lon=-122.4206972&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=37.7538318&lon=-122.4206972&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #298: lebu, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3883457&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3883457&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #299: hasaki, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112802&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2112802&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #300: yambio, sd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?q=yambio,sd&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?q=yambio,sd&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #301: clyde river, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5924351&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5924351&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #302: sagua la grande, cu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3541440&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3541440&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #303: hvolsvollur, is
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3415720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3415720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #304: aksu, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529660&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1529660&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #305: namibe, ao
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3347019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3347019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #306: katsiveli, ua
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=44.3934152&lon=33.9715074&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=44.3934152&lon=33.9715074&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #307: butaritari, ki
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7521588&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7521588&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #309: rio grande, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3451138&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3451138&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #310: camacha, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2270385&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2270385&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #312: jalu, ly
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=29.040621&lon=21.4991334&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=29.040621&lon=21.4991334&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #313: radovitskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=503210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=503210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #314: tasiilaq, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3424607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3424607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #315: thompson, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6165406&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6165406&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #316: tchollire, cm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2221607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2221607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #317: tarata, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3927774&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3927774&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #318: gambela, et
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=337405&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=337405&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #319: coahuayana, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4025994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4025994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #321: omsukchan, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122493&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2122493&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #322: muswellbrook, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839754&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839754&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #324: klaksvik, fo
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2618795&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2618795&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #325: arona, es
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6360622&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6360622&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #327: mys shmidta, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=68.884224&lon=-179.4311219&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=68.884224&lon=-179.4311219&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #328: vestmanna, fo
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2610343&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2610343&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #329: te anau, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2181625&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2181625&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #330: erzin, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1506938&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1506938&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #331: haines junction, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5969025&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5969025&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.44250202178955 seconds.********
    
    Now retrieving data for city #332: ponta do sol, cv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374346&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3374346&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #333: beringovskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2126710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #334: westport, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2206900&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2206900&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #335: vestmannaeyjar, is
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3412093&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3412093&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #336: nizhniy tsasuchey, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2019118&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2019118&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #337: olenegorsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=515698&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=515698&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #338: taguatinga, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6319623&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6319623&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #340: sierra vista, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5314328&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5314328&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #341: olmos, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3694256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3694256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #342: coos bay, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5720495&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5720495&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #343: ancud, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3899695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3899695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #344: crixas, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3465145&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3465145&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #345: xining, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1788852&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1788852&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #346: pokhara, np
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1282898&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1282898&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #347: dajal, pk
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1180752&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1180752&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #348: lodja, cd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=211647&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=211647&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #349: oktyabrskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=515879&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=515879&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #352: banjar, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1650232&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1650232&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #356: kaffrine, sn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2251007&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2251007&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #357: pangody, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1495626&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1495626&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #358: kathu, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=991664&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=991664&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #359: chara, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2025630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2025630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #360: makat, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=47.6441255&lon=53.3301869&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=47.6441255&lon=53.3301869&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #361: gueret, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6447740&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6447740&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #363: gasa, bt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7303651&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7303651&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #365: alekseyevka, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=48.4246612&lon=85.7297001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=48.4246612&lon=85.7297001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #366: bafra, tr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=751335&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=751335&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #367: bathsheba, bb
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374083&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3374083&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #368: anloga, gh
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2304548&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2304548&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #369: orange cove, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5379533&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5379533&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 51.73506784439087 seconds.********
    
    Now retrieving data for city #370: cidreira, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3466165&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3466165&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #371: nelson bay, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2155562&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2155562&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #372: santa maria, cv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374218&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3374218&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #373: yingcheng, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1786640&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1786640&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #376: okhotsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122605&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2122605&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #377: bandarbeyla, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=64814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=64814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #378: tura, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2014833&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2014833&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #379: siavonga, zm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=898188&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=898188&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #380: anaconda, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5637146&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5637146&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #381: karlovo, bg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=730565&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=730565&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #382: ahumada, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=30.6149598&lon=-106.5137018&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=30.6149598&lon=-106.5137018&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #383: daru, pg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2098329&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2098329&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #384: avera, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-16.789539&lon=-151.4099893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-16.789539&lon=-151.4099893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #385: deputatskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2028164&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2028164&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #386: matay, eg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=352628&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=352628&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #387: gazimurskiy zavod, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2024122&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2024122&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #388: caloundra, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2172710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2172710&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #389: bairiki, ki
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=1.3290526&lon=172.9790528&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=1.3290526&lon=172.9790528&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #390: kosonsoy, uz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1513714&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1513714&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #391: esperance, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2071860&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2071860&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #392: puerto leguizamo, co
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3671437&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3671437&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #393: qui nhon, vn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=13.7829673&lon=109.2196634&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=13.7829673&lon=109.2196634&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #394: tyulgan, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=479871&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=479871&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #395: mahebourg, mu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=934322&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=934322&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #396: peniche, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010554&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8010554&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #398: tezu, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1254709&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1254709&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #399: attawapiskat, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=52.9258846&lon=-82.42889219999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=52.9258846&lon=-82.42889219999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #400: wolnzach, de
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2806339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2806339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #401: copala, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3530264&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3530264&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #402: makaha, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5850511&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5850511&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.86445426940918 seconds.********
    
    Now retrieving data for city #403: hayrabolu, tr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=745697&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=745697&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #404: abramovka, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=584401&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=584401&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #407: ust-kamchatsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=56.22747&lon=162.469299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=56.22747&lon=162.469299&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #408: suntar, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2015913&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2015913&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #409: ukiah, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5404476&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5404476&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #410: ulaanbaatar, mn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2028462&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2028462&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #411: verkhoyansk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2013465&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2013465&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #412: rosarito, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3988392&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3988392&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #413: sainte-anne-des-monts, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6137749&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6137749&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #414: san quintin, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3984997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3984997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #415: saint-francois, gp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3578441&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3578441&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #416: palmer, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5074769&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5074769&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #420: tsihombe, mg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-25.3168473&lon=45.48630929999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-25.3168473&lon=45.48630929999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #421: lengshuijiang, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1804169&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1804169&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #422: vila velha, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3445026&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3445026&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #423: agnibilekrou, ci
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2293260&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2293260&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #424: soe, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1626703&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1626703&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #425: sarkand, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1519691&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1519691&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #426: khonuu, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=66.455612&lon=143.2225799&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=66.455612&lon=143.2225799&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #427: benguela, ao
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3351663&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3351663&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #428: pangnirtung, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6096551&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6096551&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #429: saint-augustin, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6137462&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6137462&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #430: talnakh, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1490256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1490256&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #431: tazovskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1489853&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1489853&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #432: arica, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3899361&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3899361&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #433: longyearbyen, sj
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2729907&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2729907&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #434: yilan, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2033403&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2033403&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #435: domoni, km
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=921906&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=921906&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #436: zdvinsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1485312&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1485312&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #437: grand gaube, mu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=934479&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=934479&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 53.31172823905945 seconds.********
    
    Now retrieving data for city #438: redmond, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5747882&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5747882&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #441: potgietersrus, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-24.1808673&lon=29.01389979999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-24.1808673&lon=29.01389979999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #442: korla, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529376&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1529376&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #443: dawei, mm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1293625&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1293625&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #444: touros, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3386213&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3386213&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #445: zhigansk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2012530&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2012530&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #446: salina cruz, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3520064&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3520064&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #447: inhambane, mz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1045114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1045114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #448: leh, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1264976&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1264976&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #449: laukaa, fi
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=648739&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=648739&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #451: cayenne, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6690689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #453: pampierstad, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=966380&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=966380&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #454: rio cuarto, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3838874&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3838874&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #455: westport, ie
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2960970&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2960970&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #456: coquimbo, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3893629&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3893629&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #457: lisakovsk, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1521315&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1521315&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #458: vanavara, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2013727&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2013727&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #459: port hedland, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #461: longlac, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=49.78166599999999&lon=-86.534706&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=49.78166599999999&lon=-86.534706&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #462: russkaya polyana, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1493423&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1493423&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #463: richards bay, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=962367&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=962367&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #464: salalah, om
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=286621&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=286621&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #465: sawakin, sd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=367544&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=367544&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #466: kununurra, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2068110&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2068110&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #467: haimen, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1809062&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1809062&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #469: hobyo, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=57000&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=57000&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #470: dinsor, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=2.4074573&lon=42.9708838&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=2.4074573&lon=42.9708838&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #471: upernavik, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3418910&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3418910&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #472: mutsamudu, km
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-12.1695811&lon=44.40047&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-12.1695811&lon=44.40047&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #473: fort saint john, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=56.252423&lon=-120.846409&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=56.252423&lon=-120.846409&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 53.42540216445923 seconds.********
    
    Now retrieving data for city #474: viedma, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3832899&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3832899&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #475: aras, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=60.77731009999999&lon=4.9329066&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=60.77731009999999&lon=4.9329066&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #476: mahon, es
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=39.8873296&lon=4.259619499999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=39.8873296&lon=4.259619499999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #477: basoko, cd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=219414&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=219414&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #478: belyy yar, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1510370&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1510370&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #480: margate, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=978895&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=978895&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #481: seara, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-27.1377719&lon=-52.3400517&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-27.1377719&lon=-52.3400517&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #482: amga, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027786&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2027786&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #483: yuzawa, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2110460&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2110460&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #485: balkhash, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=46.8435126&lon=74.9809805&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=46.8435126&lon=74.9809805&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #486: khani, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=56.933333&lon=119.965&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=56.933333&lon=119.965&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #487: fomboni, km
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=921889&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=921889&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #488: severnoye, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=496381&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=496381&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #491: mezen, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=527321&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=527321&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #492: manjacaze, mz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1040938&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1040938&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #493: marica, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6322035&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6322035&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #495: hope, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5976783&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5976783&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #496: vagur, fo
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2610806&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2610806&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #497: oranjemund, na
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3354071&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3354071&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #498: umm lajj, sa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=100926&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=100926&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #499: jiddah, sa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=21.2854067&lon=39.2375507&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=21.2854067&lon=39.2375507&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #500: narsaq, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421719&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3421719&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #501: ulaangom, mn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1515029&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1515029&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #502: pokaran, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1259460&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1259460&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #503: kapan, am
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=174875&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=174875&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #504: los llanos de aridane, es
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2514651&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2514651&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #505: artyk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=64.17916699999999&lon=145.129167&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=64.17916699999999&lon=145.129167&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #506: ahipara, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2194098&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2194098&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #507: palmerston, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7839407&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7839407&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #509: port-gentil, ga
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2396518&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2396518&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.4963321685791 seconds.********
    
    Now retrieving data for city #510: aykhal, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2027296&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2027296&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #511: kaitangata, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-46.28334&lon=169.8470967&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-46.28334&lon=169.8470967&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #512: boralday, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=43.3585501&lon=76.8674824&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=43.3585501&lon=76.8674824&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #513: necochea, ar
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3430443&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3430443&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #514: kurumkan, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2021188&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2021188&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #515: odweyne, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=9.410198200000002&lon=45.06299019999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=9.410198200000002&lon=45.06299019999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #516: kaeo, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2189343&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2189343&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #517: leshukonskoye, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=535839&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=535839&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #518: itacoatiara, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3397893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3397893&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #519: port keats, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2063039&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2063039&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #520: lompoc, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5367788&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5367788&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #521: iqaluit, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5983720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5983720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #522: vila franca do campo, pt
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=8010690&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=8010690&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #524: yulara, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6355222&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6355222&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #525: erenhot, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=43.653169&lon=111.977943&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=43.653169&lon=111.977943&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #526: faya, td
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=17.9236623&lon=19.1107114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=17.9236623&lon=19.1107114&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #527: edd, er
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=338345&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=338345&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #528: ixtapa, mx
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4004293&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4004293&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #531: muscat, om
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=287286&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=287286&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #532: christchurch, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2192362&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2192362&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #533: iracoubo, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6690691&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6690691&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #535: dondo, mz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1024696&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1024696&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #536: juneau, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5258190&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5258190&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #538: pisco, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3932145&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3932145&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #539: kushima, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1895695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1895695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #540: dzhusaly, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=50.6602258&lon=58.2691484&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=50.6602258&lon=58.2691484&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #541: saint anthony, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=37.6016065&lon=-120.8513913&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=37.6016065&lon=-120.8513913&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #542: cheuskiny, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?q=cheuskiny,ru&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?q=cheuskiny,ru&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #543: kavieng, pg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2094342&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2094342&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #544: cam ranh, vn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1586350&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1586350&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 54.08590078353882 seconds.********
    
    Now retrieving data for city #545: vilcun, cl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3868210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3868210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #546: nesbyen, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3144733&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3144733&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #547: ilulissat, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3423146&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3423146&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #548: nigde, tr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=303826&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=303826&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #550: shegaon, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1256620&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1256620&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #551: ternate, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1624041&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1624041&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #552: bubaque, gw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2374583&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2374583&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #553: sulangan, ph
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1685422&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1685422&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #556: kalmunai, lk
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1242110&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1242110&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #557: okha, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122614&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2122614&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #558: boguchany, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1509844&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1509844&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #559: ambon, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1651531&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1651531&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #560: suna, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=486236&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=486236&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #561: porto novo, cv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3374336&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3374336&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #562: moranbah, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6533368&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6533368&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #563: macaboboni, ph
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=16.1952956&lon=119.7802409&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=16.1952956&lon=119.7802409&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #564: riyadh, sa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=108410&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=108410&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #565: sangar, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2017215&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2017215&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #566: kawalu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1640902&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1640902&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #567: ede, nl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2756429&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2756429&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #568: yanan, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=36.585445&lon=109.489757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=36.585445&lon=109.489757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #569: bemidji, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5017822&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5017822&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #570: mahibadhoo, mv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1337605&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1337605&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #571: la seyne-sur-mer, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3006414&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3006414&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #573: bilibino, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2126682&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2126682&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #574: grand centre, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=54.4131673&lon=-110.2136806&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=54.4131673&lon=-110.2136806&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #575: doha, kw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=29.3155169&lon=47.8155695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=29.3155169&lon=47.8155695&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #576: ulladulla, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2145554&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2145554&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #577: mitu, co
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3674676&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3674676&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #578: maldonado, uy
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3441894&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3441894&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.89018511772156 seconds.********
    
    Now retrieving data for city #579: nizhniy kuranakh, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2019135&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2019135&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #580: azare, ng
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2348595&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2348595&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #581: tromso, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6453316&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6453316&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #583: uk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1488616&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1488616&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #584: tarudant, ma
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=30.4727126&lon=-8.8748765&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=30.4727126&lon=-8.8748765&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #585: izhma, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=554830&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=554830&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #586: genhe, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2037252&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2037252&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #587: hami, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1529484&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1529484&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #588: cabedelo, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3404558&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3404558&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #589: sao gabriel da cachoeira, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3662342&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3662342&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #590: kerteh, my
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=4.507893999999999&lon=103.4430001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=4.507893999999999&lon=103.4430001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #591: piacabucu, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3454005&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3454005&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #592: najran, sa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=103630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=103630&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #593: dali, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1814093&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1814093&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #595: velez-malaga, es
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6359498&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6359498&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #597: winnemucca, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5710360&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5710360&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #598: saint-andre-les-vergers, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6426546&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6426546&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #600: gayny, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=561698&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=561698&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #601: tilichiki, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2120591&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2120591&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #602: tevaitoa, pf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4033375&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4033375&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #603: saint-ambroise, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6137384&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6137384&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #605: karaul, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=70.070114&lon=83.23083489999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=70.070114&lon=83.23083489999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #606: marawi, sd
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=370481&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=370481&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #607: marcona, pe
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-15.3439659&lon=-75.0844757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-15.3439659&lon=-75.0844757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #608: berlin, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5245497&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5245497&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #614: zachagansk, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?q=zachagansk,kz&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?q=zachagansk,kz&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #615: berlevag, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=780686&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=780686&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #617: kavaratti, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1267390&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1267390&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #618: enshi, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1811720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1811720&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #619: tunduru, tz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-10.9737899&lon=37.4681396&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-10.9737899&lon=37.4681396&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 54.26020622253418 seconds.********
    
    Now retrieving data for city #620: louisbourg, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=45.9221352&lon=-59.9713119&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=45.9221352&lon=-59.9713119&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #621: tanout, ne
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2439155&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2439155&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #622: port hueneme, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5384339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5384339&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #623: vardo, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6453313&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6453313&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #625: ekhabi, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=53.50858299999999&lon=142.96743&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=53.50858299999999&lon=142.96743&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #626: yumen, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1528998&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1528998&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #627: katsuura, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2112309&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2112309&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #628: sistranda, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3139597&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3139597&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #629: toppenish, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5813747&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5813747&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #630: ketchikan, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5554428&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5554428&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #631: branford, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5283054&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5283054&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #632: joshimath, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1268814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1268814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #633: silvan, tr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=300796&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=300796&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #634: sarmanovo, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=498537&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=498537&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #635: bosaso, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=11.2755407&lon=49.1878994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=11.2755407&lon=49.1878994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #636: klyuchi, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1503153&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1503153&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #638: bambanglipuro, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1650434&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1650434&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #639: asau, tv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=-7.488197&lon=178.6807179&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=-7.488197&lon=178.6807179&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #640: pakwach, ug
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=2.4607141&lon=31.4941738&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=2.4607141&lon=31.4941738&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #641: yerbogachen, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2012956&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2012956&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #642: pavlivka, ua
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7021056&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7021056&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #644: krasnovishersk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=542184&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=542184&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #645: zhezkazgan, kz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=47.79636559999999&lon=67.7020019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=47.79636559999999&lon=67.7020019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #646: suihua, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2034655&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2034655&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #647: adrar, dz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2508813&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2508813&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #648: lolua, tv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?q=lolua,tv&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?q=lolua,tv&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #649: arklow, ie
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2966883&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2966883&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #650: grenaa, dk
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2621230&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2621230&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #651: hereford, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5523074&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5523074&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #652: warken, lu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2960008&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2960008&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 53.76808977127075 seconds.********
    
    Now retrieving data for city #653: khlevnoye, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=550102&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=550102&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #654: guantanamo, cu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3557689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3557689&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #655: waingapu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1622318&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1622318&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #656: palu, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1633034&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1633034&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #657: anchorage, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5879400&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5879400&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #659: road town, vg
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3577430&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3577430&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #660: sola, vu
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2134814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2134814&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #661: manggar, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1636426&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1636426&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #662: sol-iletsk, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=491019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=491019&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #663: grand-santi, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3381538&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3381538&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #664: ellisras, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1005768&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1005768&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #665: camopi, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3382226&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3382226&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #666: mayahi, ne
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2441194&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2441194&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #667: santa cruz, cr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3621607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3621607&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #668: bonavista, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5905393&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5905393&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #669: chateauroux, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6448591&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6448591&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #671: myitkyina, mm
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1307741&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1307741&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #672: petropavlovsk-kamchatskiy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2122104&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2122104&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #673: kuruman, za
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=986134&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=986134&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #674: tarko-sale, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1490085&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1490085&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #675: portland, au
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2152667&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2152667&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #677: grants, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5469841&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5469841&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #678: iskilip, tr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=745076&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=745076&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #679: dibulla, co
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3685223&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3685223&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #680: sorland, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3137469&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3137469&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #681: chagda, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=58.75177799999999&lon=130.6163939&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=58.75177799999999&lon=130.6163939&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #682: sobolevo, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=54.300693&lon=155.956757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=54.300693&lon=155.956757&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #683: fukue, jp
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1863997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1863997&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #684: naryan-mar, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=523392&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=523392&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #685: maragogi, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3395458&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3395458&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 52.06404709815979 seconds.********
    
    Now retrieving data for city #686: ambagarh chauki, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1278869&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1278869&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #687: etampes, fr
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6446136&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6446136&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #689: bayonet point, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4146901&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4146901&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #690: wuchi, tw
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=24.253032&lon=120.531052&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=24.253032&lon=120.531052&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #691: high level, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5975004&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5975004&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #692: rafsanjan, ir
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=118994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=118994&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #693: birao, cf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=240210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=240210&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #694: buri, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3468802&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3468802&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #696: olinda, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3393536&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3393536&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #697: grenada, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=4428539&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=4428539&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #698: thinadhoo, mv
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1337610&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1337610&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #699: mirnyy, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=526346&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=526346&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #708: kargil, in
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1267776&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1267776&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #709: tual, id
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1623197&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1623197&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #710: mehamn, no
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=778707&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=778707&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #711: zhangye, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1785036&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1785036&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #712: saint george, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5546220&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5546220&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #717: flores, br
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3399539&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3399539&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #718: abu samrah, qa
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=24.7466105&lon=50.8460992&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=24.7466105&lon=50.8460992&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #719: mizpe ramon, il
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=30.610203&lon=34.801898&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=30.610203&lon=34.801898&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #720: paamiut, gl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=3421193&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=3421193&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #721: bulgan, mn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2032201&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2032201&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #726: tabou, ci
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2281120&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2281120&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #727: kuybysheve, ua
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=703498&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=703498&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #729: saint-georges, gf
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=3.8925718&lon=-51.807382&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=3.8925718&lon=-51.807382&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #730: tuatapere, nz
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2180815&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2180815&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #731: kapit, my
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1737185&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1737185&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #732: la ronge, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=6050066&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=6050066&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #733: wajid, so
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=3.809294&lon=43.2461055&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=3.809294&lon=43.2461055&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #734: togur, ru
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1489499&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1489499&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    
    ********Sleeping for 53.011996030807495 seconds.********
    
    Now retrieving data for city #735: lamesa, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=5524849&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=5524849&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #736: cabo rojo, us
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=18.0885957&lon=-67.14924380000001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=18.0885957&lon=-67.14924380000001&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #737: stargard szczecinski, pl
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=7531735&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=7531735&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #739: moose factory, ca
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=51.262486&lon=-80.59296499999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=51.262486&lon=-80.59296499999999&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #740: dingle, ie
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2964782&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2964782&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #741: taoudenni, ml
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=2450173&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=2450173&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #742: hede, cn
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1808744&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1808744&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #743: lata, sb
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?lat=46.5335004&lon=6.554163&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?lat=46.5335004&lon=6.554163&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------
    Now retrieving data for city #744: simbahan, ph
    Current Weather URL:
    https://api.openweathermap.org/data/2.5/weather?id=1695180&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    Forecast Weather URL:
    https://api.openweathermap.org/data/2.5/forecast?id=1695180&APPID=3c995f5860f5c2480a49b210cc497c1e&units=imperial
    ---------------------------------------------------------------------------



```python
# Summary Report
print("Counts and Errors Report")
print('---------------------------------------------------------------------------')
print("Count")
print('----------------')
print(str(counter))
print('---------------------------------------------------------------------------')
print("Errors")
print('----------------')
print('# of Current Weather Errors: ' + str(cur_errors))
print("Current Weather Errors Index List:")

if len(cur_err_list) > 0:
    for n in cur_errors:
        print(n)
else:
    print('None')
print("")
print('# of Forecast Weather Errors:' + str(for_errors))
print("Forecast Weather Errors Index List: ")
if len(for_err_list) > 0:
    for n in for_errors:
        print(n)
else:
    print('None')
print('---------------------------------------------------------------------------')
```

    Counts and Errors Report
    ---------------------------------------------------------------------------
    Count
    ----------------
    609
    ---------------------------------------------------------------------------
    Errors
    ----------------
    # of Current Weather Errors: 0
    Current Weather Errors Index List:
    None
    
    # of Forecast Weather Errors:0
    Forecast Weather Errors Index List: 
    None
    ---------------------------------------------------------------------------



```python
#view columns
weather_data.columns
```




    Index(['city', 'country', 'coord', 'id', 'lat', 'lng', 'cur_date', 'cur_temp',
           'cur_humidity', 'cur_clouds', 'cur_wind', 'max_date', 'max_temp',
           'max_temp_humidity', 'max_temp_clouds', 'max_temp_wind', 'avg_date0',
           'avg_date1', 'avg_temp', 'avg_humidity', 'avg_clouds', 'avg_wind'],
          dtype='object')




```python
# clean up
clean_col = weather_data.columns[4:]
clean_col
```




    Index(['lat', 'lng', 'cur_date', 'cur_temp', 'cur_humidity', 'cur_clouds',
           'cur_wind', 'max_date', 'max_temp', 'max_temp_humidity',
           'max_temp_clouds', 'max_temp_wind', 'avg_date0', 'avg_date1',
           'avg_temp', 'avg_humidity', 'avg_clouds', 'avg_wind'],
          dtype='object')




```python
#loop throught to clean columns to be able to use for graphs
for c in clean_col:
    weather_data[c] = pd.to_numeric(weather_data[c], errors = 'coerce') 
    weather_data = weather_data[weather_data[c].isnull() == False]

len(weather_data)
```




    609




```python
# check weather data types
weather_data.dtypes
```




    city                  object
    country               object
    coord                 object
    id                     int64
    lat                  float64
    lng                  float64
    cur_date               int64
    cur_temp             float64
    cur_humidity           int64
    cur_clouds             int64
    cur_wind             float64
    max_date               int64
    max_temp             float64
    max_temp_humidity      int64
    max_temp_clouds        int64
    max_temp_wind        float64
    avg_date0              int64
    avg_date1              int64
    avg_temp             float64
    avg_humidity         float64
    avg_clouds           float64
    avg_wind             float64
    dtype: object




```python
#Export to csv
weather_data.to_csv('clean_weather_data.csv')
```


```python
weather_data = pd.read_csv('clean_weather_data.csv')
weather_data.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>city</th>
      <th>country</th>
      <th>coord</th>
      <th>id</th>
      <th>lat</th>
      <th>lng</th>
      <th>cur_date</th>
      <th>cur_temp</th>
      <th>cur_humidity</th>
      <th>...</th>
      <th>max_temp</th>
      <th>max_temp_humidity</th>
      <th>max_temp_clouds</th>
      <th>max_temp_wind</th>
      <th>avg_date0</th>
      <th>avg_date1</th>
      <th>avg_temp</th>
      <th>avg_humidity</th>
      <th>avg_clouds</th>
      <th>avg_wind</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>chute-aux-outardes</td>
      <td>ca</td>
      <td>{'lon': -68.398956, 'lat': 49.116791}</td>
      <td>5922281</td>
      <td>49.12</td>
      <td>-68.40</td>
      <td>1505229180</td>
      <td>61.84</td>
      <td>77</td>
      <td>...</td>
      <td>67.81</td>
      <td>68</td>
      <td>0</td>
      <td>9.69</td>
      <td>1505239200</td>
      <td>1505660400</td>
      <td>55.31675</td>
      <td>78.60</td>
      <td>31.7</td>
      <td>5.91425</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>jamestown</td>
      <td>sh</td>
      <td>{'lon': -5.71675, 'lat': -15.93872}</td>
      <td>3370903</td>
      <td>-15.94</td>
      <td>-5.72</td>
      <td>1505230451</td>
      <td>68.02</td>
      <td>100</td>
      <td>...</td>
      <td>67.95</td>
      <td>100</td>
      <td>8</td>
      <td>25.19</td>
      <td>1505239200</td>
      <td>1505660400</td>
      <td>67.11400</td>
      <td>100.00</td>
      <td>41.1</td>
      <td>22.35150</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>dengzhou</td>
      <td>cn</td>
      <td>{'lon': 112.08194, 'lat': 32.68222}</td>
      <td>1812990</td>
      <td>32.68</td>
      <td>112.08</td>
      <td>1505230452</td>
      <td>69.64</td>
      <td>80</td>
      <td>...</td>
      <td>82.00</td>
      <td>66</td>
      <td>88</td>
      <td>7.43</td>
      <td>1505239200</td>
      <td>1505660400</td>
      <td>73.88875</td>
      <td>75.95</td>
      <td>14.5</td>
      <td>5.12375</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>ilek</td>
      <td>ru</td>
      <td>{'lon': 53.38306, 'lat': 51.527088}</td>
      <td>557340</td>
      <td>51.53</td>
      <td>53.38</td>
      <td>1505230223</td>
      <td>68.92</td>
      <td>41</td>
      <td>...</td>
      <td>78.28</td>
      <td>40</td>
      <td>0</td>
      <td>7.74</td>
      <td>1505239200</td>
      <td>1505660400</td>
      <td>66.09225</td>
      <td>62.95</td>
      <td>13.2</td>
      <td>7.05250</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>airai</td>
      <td>pw</td>
      <td>{'lon': 134.5690225, 'lat': 7.396611799999999}</td>
      <td>0</td>
      <td>7.40</td>
      <td>134.57</td>
      <td>1505227920</td>
      <td>78.80</td>
      <td>94</td>
      <td>...</td>
      <td>82.21</td>
      <td>100</td>
      <td>48</td>
      <td>10.47</td>
      <td>1505239200</td>
      <td>1505660400</td>
      <td>82.34075</td>
      <td>100.00</td>
      <td>70.4</td>
      <td>10.47875</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
#date sorting and conversions dictionary for graph labels

dates = {'max_cur': weather_data['cur_date'].max(),
         'min_cur': weather_data['cur_date'].min(),
         'max_max': weather_data['max_date'].max(),
         'min_max': weather_data['max_date'].min(),
         'min_avg': weather_data['avg_date0'].max(),
         'max_avg': weather_data['avg_date1'].min()
        }

for key in dates.keys():
    convert = datetime.utcfromtimestamp(dates[key]).strftime('%Y-%m-%d%, %I:%M:%S %p')
    dates[key] = convert
```


```python
# dictionary for labels
labels_dic = {"cur_temp": "Current Temperature", 
              'max_temp': 'Maximum Temp 24 Hours', 
              'avg_temp': 'Average Forecasted Temp in 5 Days',
             'cur_humidity': 'Current Humidity',
             'max_temp_humidity': "Forecasted Humidity (%) at the Maximum Temperature in 24 Hours",
             'avg_humidity': 'Average Forecasted Humidity (%) over 5 Days',
             'cur_clouds': 'Current Cloud Cover (%)',
             'max_temp_clouds': 'Forecasted Cloud Cover (%) at the Maximum Forecasted Temperature in 24 Hours',
             'avg_clouds': 'Average Forecasted Cloud Cover (%) over 5 Days',
             'cur_wind': 'Current Wind Speed (mph)',
             'max_temp_wind': 'Forecasted Wind Speed (mph) at the Maximum Forecasted Temperature in 24 Hours',
             'avg_wind': 'Average Forecasted Wind Speed (mph) over 5 Days'}
```


```python
# Temp vs Latitude Graphs
temp_list = ['cur_temp', 'max_temp', 'avg_temp']  #would have done only dict but wanted consistent order.

xvals = weather_data['lat']

for temp in temp_list:
    # y values of each item in list for separate graphs
    yvals = weather_data[temp]
    #adds title including title and timestamp range of sample data
    plt.title("%s vs Latitude \n Samples Taken from %s to %s UTC" % (labels_dic[temp], dates['min_' + temp.split('_')[0]],  dates['max_' + temp.split('_')[0]]))
    plt.axvline(0, color = 'black', alpha = .25, label = 'Equator') #adds equator line
    plt.text(1,30,'Equator',rotation=90)
    plt.ylim(15, 120) #to give consistent scale
    plt.xlabel('Latitude')
    plt.ylabel("Temperature (F)")
    plt.scatter(xvals, yvals)
    plt.show()
    plt.savefig("%s vs Latitude" % (labels_dic[temp]))
```


![png](output_35_0.png)



![png](output_35_1.png)



![png](output_35_2.png)



```python
# Humidity vs Latitude Graphs
#see first set of graphs commenting for notes
hum_list = ['cur_humidity', 'max_temp_humidity', 'avg_humidity']  #would have done only dict but wanted consistent order.

xvals = weather_data['lat']

for hum in hum_list:
    yvals = weather_data[hum]
    plt.title("%s vs Latitude \n Samples Taken from %s to %s UTC" % (labels_dic[hum], dates['min_' + hum.split('_')[0]],  dates['max_' + hum.split('_')[0]]))
    plt.xlabel('Latitude')
    plt.ylabel('Humidity (%)')
    plt.axvline(0, color = 'black', alpha = .25, label = 'Equator')
    plt.text(1,20,'Equator',rotation=90)
    plt.scatter(xvals, yvals)
    plt.show()
    plt.savefig("%s vs Latitude" % (labels_dic[hum]))
```


![png](output_36_0.png)



![png](output_36_1.png)



![png](output_36_2.png)



```python
# Cloud Cover vs Latitude Graphs
#see first set of graphs commenting for notes
cloud_list = ['cur_clouds', 'max_temp_clouds', 'avg_clouds']  #would have done only dict but wanted consistent order.

xvals = weather_data['lat']

for clo in cloud_list:
    yvals = weather_data[clo]
    plt.title("%s vs Latitude \n Samples Taken from %s to %s UTC" % (labels_dic[clo], dates['min_' + clo.split('_')[0]],  dates['max_' + clo.split('_')[0]]))
    plt.xlabel('Latitude')
    plt.ylabel('Cloud Cover (%)')
    plt.ylim(-5,105)
    plt.axvline(0, color = 'black', alpha = .25, label = 'Equator')
    plt.text(-5,-20,'Equator')
    plt.scatter(xvals, yvals)
    plt.show()
    plt.savefig("%s vs Latitude" % (labels_dic[clo]))
```


![png](output_37_0.png)



![png](output_37_1.png)



![png](output_37_2.png)



```python
# Wind Speed vs Latitude Graphs
#see first set of graphs commenting for notes
win_list = ['cur_wind', 'max_temp_wind', 'avg_wind']  #would have done only dict but wanted consistent order.

xvals = weather_data['lat']

for win in win_list:
    yvals = weather_data[win]
    plt.title("%s vs Latitude \n Samples Taken from %s to %s UTC" % (labels_dic[win], dates['min_' + win.split('_')[0]],  dates['max_' + win.split('_')[0]]))
    plt.xlabel('Latitude')
    plt.ylabel('Wind Speed (mph))')
    plt.ylim(-5,60)
    plt.axvline(0, color = 'black', alpha = .25, label = 'Equator')
    plt.text(1,35,'Equator',rotation=90)
    plt.scatter(xvals, yvals)
    plt.show()
    plt.savefig("%s vs Latitude" % (labels_dic[win]))
```


![png](output_38_0.png)



![png](output_38_1.png)



![png](output_38_2.png)



```python
#graphs lats vs long, temperature scale from red(hot) to green(cool), and bubble size based on humidity, cloud cover, and wind speed
xvars = weather_data['lng']
yvars = weather_data['lat']
color = weather_data['avg_temp']
size_list = ['avg_humidity', 'avg_clouds', 'avg_wind']

#loops through size list and only changes size of bubbles based on different variables
for measure in size_list:  
    plt.figure(figsize = (18,12))
    plt.xlim(-180,180)
    plt.ylim(-90,90)
    plt.title("Global Temperatures Based on 5 Day Average \n Samples taken from %s to %s UTC \n Note:  Bubble Size Corresponds to %s" % (dates['min_avg'], dates['max_avg'], labels_dic[measure]))
    plt.axhline(0, color = 'black', alpha = .25, label = 'Equator')
    plt.text(-155,3,'Equator')
    size = weather_data[measure]
    plt.scatter(xvars, 
                yvars, 
                c = color, 
                s = size * 6, 
                edgecolor = 'black', 
                linewidth = 1, 
                alpha = .5, 
                cmap=plt.cm.RdYlGn_r)
    plt.show()
    plt.savefig("Global Temperatures Based on 5 Day Average Bubble Plot: %s" % (measure))
```


    <matplotlib.figure.Figure at 0x11ffe4f98>



![png](output_39_1.png)



    <matplotlib.figure.Figure at 0x11fc0c3c8>



![png](output_39_3.png)



    <matplotlib.figure.Figure at 0x12004ee80>



![png](output_39_5.png)



```python

```
