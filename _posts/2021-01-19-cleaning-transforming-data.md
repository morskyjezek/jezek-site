---
title: 'Wrangling Humanities Data: Cleaning and Transforming Data'
date: 2021-01-19
permalink: /posts/2021/cleaning-transforming-data/
excerpt: 'This post continues the "Wrangling NEH Data" series. This installment cleans and transforms the original data to create a geospatial dataset.'
header:
  overlay_image: "binary-1280w.jpg"
  overlay_filter: 0.3
  image_description: "Rows and columns of 0 and 1 with a blue background."
  caption: "Image by [Gerd Altmann](https://pixabay.com/illustrations/binary-digitization-null-one-pay-1377017/) on Pixabay"
  og_image: "binary-1280w.jpg"
  teaser: "binary-th.jpg"
read_time: false
categories:
  - mapping humanities
  - data curation
tags:
  - python
  - neh 
  - geospatial
  - humanities data
---

This post continues "Wrangling Humanities Data," a series that walks through the design of a data curation process using publicly-available grant data provided by the National Endowment for the Humanities (NEH). In this installment, I draw on some of the files, with which I [previously demonstrated the creation of an archival data package]({% post_url 2020-12-20-finding-describing-data %}), and explore the data, check on the quality and reliability of the data, and then use the data to create a geospatial dataset. The geospatial data will be the basis for a future post that will undertake more mapping activities based on the data. To do this analysis, I use the pandas data library, which is supported in a Python environment and Jupyter notebook.

At a high level, the notebook (code and descriptions below) takes these steps:

1. Set up the programming environment (import useful Python modules, then input the original NEH grant data information).
1. Explore the data to check data quality for inconsistencies, missing or inaccurate data, and other useful information about the information.
1. Clean the data by removing incorrect values, and enchance and transform the data by adding or making the data more consistent. In this case, this involves providing the geographic coordinates for the awards.
1. Finally, export the data to geojson, a portable and lightweight format that can be used for more advanced mapping projects in later steps.

*As in the [previous post]({% post_url 2020-12-20-finding-describing-data %}), you can also find a [Jupyter Notebook version of this post](https://github.com/morskyjezek/neh-grant-data-project/blob/main/02b-transforming-geojson.ipynb), which can be downloaded from the GitHub repository along with all of the data discussed here. File references discussed below indicate files included in the same [neh-grant-data-project repository](https://github.com/morskyjezek/neh-grant-data-project).*

# Cleaning and Transforming to Create a Geospatial Dataset

This notebook demonstrates some of the steps involved in cleaning the NEH grant data, checking on quality and consistency, and then transforming the data into geospatial information that can be the basis of a map. While most of the data is contained within the steps and cells of this notebook, at the end there is a script to use to export the final information as `geojson`, which can be used elsewhere or for other purposes. It is more consistent and portable than the original data.

## Setup

For this activity, we will use some Python modules that may not be in the standard JupyterLab configuration. If you do not have them, you may need to install some of these modules: `geojson` ([here](https://pypi.org/project/geojson/#installation)), `geopandas` ([here](https://pypi.org/project/geopandas/)), `geopy` ([here](https://pypi.org/project/geopy/)), `descartes` ([here](https://pypi.org/project/descartes/)), and `shapely` ([here](https://pypi.org/project/shapely/)).


```python
# if you do not have the modules, uncomment the following line(s) to install
#!pip install geojson
#!pip install geopandas
#!pip install descartes
#!pip install geopy
```


```python
# modules for data cleaning & transformation
import pandas as pd
import geopandas as gpd

# basic visualization
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
# modules for mapping
from shapely.geometry import Point

# may use for additional visualization
import seaborn as sns

# modules for geocoding
from geopy.geocoders import Nominatim
from geopy.extra.rate_limiter import RateLimiter
from time import sleep
```

## Clean and filter the grant data using pandas

First, let's explore the grant data and clean it up so that we can define the points that we will map - there should be one coordinate location for each grant that we want to display. For this task, we will begin using the `pandas` modules for working with data. 


```python
df_grants_1960s = pd.read_csv('neh-grants-data-202012/data/NEH_Grants1960s.csv')

df_grants_1960s.head()
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
      <th>AppNumber</th>
      <th>ApplicantType</th>
      <th>Institution</th>
      <th>OrganizationType</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>...</th>
      <th>EndGrant</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>PrimaryDiscipline</th>
      <th>SupplementCount</th>
      <th>Supplements</th>
      <th>ParticipantCount</th>
      <th>Participants</th>
      <th>DisciplineCount</th>
      <th>Disciplines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FB-10007-68</td>
      <td>2</td>
      <td>Regents of the University of California, Berkeley</td>
      <td>Four-Year College</td>
      <td>Berkeley</td>
      <td>CA</td>
      <td>94704-5940</td>
      <td>USA</td>
      <td>13</td>
      <td>37.87029</td>
      <td>...</td>
      <td>1969-12-31</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>English</td>
      <td>0</td>
      <td>NaN</td>
      <td>1</td>
      <td>John Elliot [Project Director]</td>
      <td>1</td>
      <td>English</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FB-10009-68</td>
      <td>2</td>
      <td>Pitzer College</td>
      <td>Four-Year College</td>
      <td>Claremont</td>
      <td>CA</td>
      <td>91711-6101</td>
      <td>USA</td>
      <td>27</td>
      <td>34.10373</td>
      <td>...</td>
      <td>1969-12-31</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>History of Religion</td>
      <td>0</td>
      <td>NaN</td>
      <td>1</td>
      <td>Steven Matthysse [Project Director]</td>
      <td>1</td>
      <td>History of Religion</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FB-10015-68</td>
      <td>2</td>
      <td>University of California, Riverside</td>
      <td>University</td>
      <td>Riverside</td>
      <td>CA</td>
      <td>92521-0001</td>
      <td>USA</td>
      <td>41</td>
      <td>33.97561</td>
      <td>...</td>
      <td>1969-12-31</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>History, General</td>
      <td>0</td>
      <td>NaN</td>
      <td>1</td>
      <td>John Staude [Project Director]</td>
      <td>1</td>
      <td>History, General</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FB-10019-68</td>
      <td>2</td>
      <td>Northeastern University</td>
      <td>Four-Year College</td>
      <td>Boston</td>
      <td>MA</td>
      <td>02115-5005</td>
      <td>USA</td>
      <td>7</td>
      <td>42.3395</td>
      <td>...</td>
      <td>1969-12-31</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>History, General</td>
      <td>0</td>
      <td>NaN</td>
      <td>1</td>
      <td>Thomas Havens [Project Director]</td>
      <td>1</td>
      <td>History, General</td>
    </tr>
    <tr>
      <th>4</th>
      <td>FB-10023-68</td>
      <td>2</td>
      <td>University of Pennsylvania</td>
      <td>University</td>
      <td>Philadelphia</td>
      <td>PA</td>
      <td>19104-6205</td>
      <td>USA</td>
      <td>3</td>
      <td>39.95298</td>
      <td>...</td>
      <td>1969-12-31</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Psychology</td>
      <td>0</td>
      <td>NaN</td>
      <td>1</td>
      <td>Gresham Riley [Project Director]</td>
      <td>1</td>
      <td>Psychology</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 33 columns</p>
</div>



It's important to explore the dataset a bit first, so we can understand it better, do a sanity check, and identify the information that is needed to create a new set of mappable point data. The pandas library allows to easily do some of this basic work, including characterizing the shape of the data, seeing what datatypes it includes, and whether it is missing values.


```python
df_grants_1960s.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1010 entries, 0 to 1009
    Data columns (total 33 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              1010 non-null   object 
     1   ApplicantType          1010 non-null   int64  
     2   Institution            1010 non-null   object 
     3   OrganizationType       1010 non-null   object 
     4   InstCity               1010 non-null   object 
     5   InstState              1010 non-null   object 
     6   InstPostalCode         1010 non-null   object 
     7   InstCountry            1010 non-null   object 
     8   CongressionalDistrict  1010 non-null   int64  
     9   Latitude               1010 non-null   object 
     10  Longitude              1010 non-null   object 
     11  CouncilDate            1010 non-null   object 
     12  YearAwarded            1010 non-null   int64  
     13  ProjectTitle           1010 non-null   object 
     14  Program                1010 non-null   object 
     15  Division               1010 non-null   object 
     16  ApprovedOutright       1010 non-null   float64
     17  ApprovedMatching       1010 non-null   float64
     18  AwardOutright          1010 non-null   float64
     19  AwardMatching          1010 non-null   float64
     20  OriginalAmount         1010 non-null   float64
     21  SupplementAmount       1010 non-null   float64
     22  BeginGrant             1010 non-null   object 
     23  EndGrant               1010 non-null   object 
     24  ProjectDesc            1010 non-null   object 
     25  ToSupport              1010 non-null   object 
     26  PrimaryDiscipline      1010 non-null   object 
     27  SupplementCount        1010 non-null   int64  
     28  Supplements            0 non-null      float64
     29  ParticipantCount       1010 non-null   int64  
     30  Participants           960 non-null    object 
     31  DisciplineCount        1010 non-null   int64  
     32  Disciplines            1010 non-null   object 
    dtypes: float64(7), int64(6), object(20)
    memory usage: 260.5+ KB


Latitude and Longitude series are listed as data objects, not numeric data. I will convert these into numeric data later since it is necessary to process them into geospatial coordinates as numbers, not strings. On closer inspection, many of these fields include the strings "Unknown", "unknown", or "Un", which I also want to change to null or None values. First, though, I will create a new data frame so this original data can be recovered later, if necessary, and also drop some of the information that I won't need to map the grants. 

### Data quality checking and cleaning

There's a lot of information here that won't help to map the data. Aside from the information for location of the points (that is, `Latitude` and `Longitude`), I will keep some for use in a popup that can be displayed when hovering over a point on the map. 

I regularly use panda's `isnull()` and `info()` functions below to look for blank cells or missing information. 


```python
mappable_grant_info = df_grants_1960s.drop(['ApplicantType','OrganizationType','CouncilDate','ApprovedOutright','ApprovedMatching','OriginalAmount','SupplementAmount','BeginGrant','EndGrant','PrimaryDiscipline','SupplementCount','Supplements','ParticipantCount','DisciplineCount'], axis=1)

mappable_grant_info.head()
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>ProjectTitle</th>
      <th>Program</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>FB-10007-68</td>
      <td>Regents of the University of California, Berkeley</td>
      <td>Berkeley</td>
      <td>CA</td>
      <td>94704-5940</td>
      <td>USA</td>
      <td>13</td>
      <td>37.87029</td>
      <td>-122.26813</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>John Elliot [Project Director]</td>
      <td>English</td>
    </tr>
    <tr>
      <th>1</th>
      <td>FB-10009-68</td>
      <td>Pitzer College</td>
      <td>Claremont</td>
      <td>CA</td>
      <td>91711-6101</td>
      <td>USA</td>
      <td>27</td>
      <td>34.10373</td>
      <td>-117.70701</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Steven Matthysse [Project Director]</td>
      <td>History of Religion</td>
    </tr>
    <tr>
      <th>2</th>
      <td>FB-10015-68</td>
      <td>University of California, Riverside</td>
      <td>Riverside</td>
      <td>CA</td>
      <td>92521-0001</td>
      <td>USA</td>
      <td>41</td>
      <td>33.97561</td>
      <td>-117.33113</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>John Staude [Project Director]</td>
      <td>History, General</td>
    </tr>
    <tr>
      <th>3</th>
      <td>FB-10019-68</td>
      <td>Northeastern University</td>
      <td>Boston</td>
      <td>MA</td>
      <td>02115-5005</td>
      <td>USA</td>
      <td>7</td>
      <td>42.3395</td>
      <td>-71.09048</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Thomas Havens [Project Director]</td>
      <td>History, General</td>
    </tr>
    <tr>
      <th>4</th>
      <td>FB-10023-68</td>
      <td>University of Pennsylvania</td>
      <td>Philadelphia</td>
      <td>PA</td>
      <td>19104-6205</td>
      <td>USA</td>
      <td>3</td>
      <td>39.95298</td>
      <td>-75.19276</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Gresham Riley [Project Director]</td>
      <td>Psychology</td>
    </tr>
  </tbody>
</table>
</div>




```python
mappable_grant_info.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1010 entries, 0 to 1009
    Data columns (total 19 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              1010 non-null   object 
     1   Institution            1010 non-null   object 
     2   InstCity               1010 non-null   object 
     3   InstState              1010 non-null   object 
     4   InstPostalCode         1010 non-null   object 
     5   InstCountry            1010 non-null   object 
     6   CongressionalDistrict  1010 non-null   int64  
     7   Latitude               1010 non-null   object 
     8   Longitude              1010 non-null   object 
     9   YearAwarded            1010 non-null   int64  
     10  ProjectTitle           1010 non-null   object 
     11  Program                1010 non-null   object 
     12  Division               1010 non-null   object 
     13  AwardOutright          1010 non-null   float64
     14  AwardMatching          1010 non-null   float64
     15  ProjectDesc            1010 non-null   object 
     16  ToSupport              1010 non-null   object 
     17  Participants           960 non-null    object 
     18  Disciplines            1010 non-null   object 
    dtypes: float64(2), int64(2), object(15)
    memory usage: 150.0+ KB


While there are entries for each grant, on closer inspection it is clear that there are some data quality issues. For example, a lot of grants seem to be missing `Participants` information, and the latitude and longitude information, which will be the basis for our mapping goal, is not numerical but listed as an object type of data. As I can see in the above cell, much of the information is described as "object," which in these case are mostly string data, which will not be possible to convert into coordinate points. 

This means that there is more data cleaning to do before the data is ready to map. I used the following methods to get more information about specific data fields, including `AppNumber` (the data element that is most like a unique identifier), `Latitude` (a creator-generated field, which is presumably generated by a geocoding algorithm), 
and `InstCountry`.


```python
mappable_grant_info['AppNumber'].value_counts()
```




    RO-10389       1
    EO-10051-68    1
    RO-10259-68    1
    FB-10225-68    1
    FT-10147-67    1
                  ..
    EO-10302-69    1
    EO-10085-69    1
    EO-10242-69    1
    FT-10294-68    1
    FT-10157-67    1
    Name: AppNumber, Length: 1010, dtype: int64




```python
mappable_grant_info['Latitude'].value_counts()
```




    Unknown     90
    41.31003    17
    40.81835    16
    34.07516    16
    37.87029    14
                ..
    40.771       1
    42.82649     1
    40.85069     1
    42.29017     1
    42.35675     1
    Name: Latitude, Length: 382, dtype: int64



In looking closer at `InstCounty`, it is clear that not all of the grants were given to recipients within the United States. As a scoping exericse, I want to remove the locations that are not within the US.


```python
mappable_grant_info['InstCountry'].unique()
```




    array(['USA', 'Unknown', 'Canada', 'United Kingdom'], dtype=object)




```python
mappable_grant_info['InstCountry'].value_counts()
```




    USA               993
    Unknown            14
    Canada              2
    United Kingdom      1
    Name: InstCountry, dtype: int64



Many of the grant entries list `Unknown` or `unknown` in the fields with information about country, institution, and even latitude or longitude. This challenges our mapping project since if we cannot locate a grant, we cannot create a point to place it on the map. To address these challenges, we need first to identify grants withough location information, then we need to get the locations of the places without latitude and longitude. For the latter step, we will use a geocoding module.

For scoping, let's drop the UK and Canada grants, and then see whether we can geolocate the unknowns.


```python
#filter out Canada & UK
mappable_grant_info = mappable_grant_info[mappable_grant_info['InstCountry'].isin(['Canada','United Kingdom']) == False]
```


```python
mappable_grant_info['InstCountry'].value_counts()
```




    USA        993
    Unknown     14
    Name: InstCountry, dtype: int64



My goal is to use information about the city and state of the award to get a geographic coordinate for the 14 listing "Unknown" countries. There remain, however, a few entries that don't appear to have enough information to geocode. 

These can be identified by the value `Un` in the `InstState` field:


```python
mappable_grant_info[mappable_grant_info['InstState'] == 'Un']
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>ProjectTitle</th>
      <th>Program</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>221</th>
      <td>FB-10243-68</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for College Teachers and Independe...</td>
      <td>Research Programs</td>
      <td>8387.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Comparative Literature</td>
    </tr>
    <tr>
      <th>337</th>
      <td>FT-10440-68</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Summer Stipends</td>
      <td>Research Programs</td>
      <td>1500.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>English</td>
    </tr>
    <tr>
      <th>428</th>
      <td>RO-10048-67</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Biography of Theodore Roethke</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>5000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Art History and Criticism</td>
    </tr>
    <tr>
      <th>429</th>
      <td>RO-10050-67</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Symbolic Landscape in Modern Poetry</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>2000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>American Literature</td>
    </tr>
    <tr>
      <th>454</th>
      <td>RO-10166-67</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>History of Book Publishing in America</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>25000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>History, General</td>
    </tr>
    <tr>
      <th>459</th>
      <td>RO-10211-68</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1968</td>
      <td>Biography of Richard Wright</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>8000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Literature, General</td>
    </tr>
    <tr>
      <th>480</th>
      <td>RO-10317-69</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Un</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1968</td>
      <td>Biography of Richard Wright</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>8000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Literature, General; Social Sciences, General</td>
    </tr>
    <tr>
      <th>757</th>
      <td>FT-10257-67</td>
      <td>McMaster University</td>
      <td>Ontario, Canada</td>
      <td>Un</td>
      <td>00000-0000</td>
      <td>USA</td>
      <td>1</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Summer Stipends</td>
      <td>Research Programs</td>
      <td>2000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Chauncey Wood [Project Director]</td>
      <td>British Literature</td>
    </tr>
    <tr>
      <th>774</th>
      <td>FT-10310-68</td>
      <td>Simon Fraser University</td>
      <td>B.C., Canada</td>
      <td>Un</td>
      <td>00000-0000</td>
      <td>USA</td>
      <td>0</td>
      <td>Unknown</td>
      <td>unknown</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Summer Stipends</td>
      <td>Research Programs</td>
      <td>1500.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Jared Curtis [Project Director]</td>
      <td>English; Literature, General</td>
    </tr>
  </tbody>
</table>
</div>



It is clear that most of these do not have enough information to map. The last two are in Canada, and so I don't want them anyway. So I'll remove them from the data:


```python
#filter out unknown state rows
mappable_grant_info = mappable_grant_info[mappable_grant_info['InstState'].isin(['Un']) == False]
```


```python
mappable_grant_info.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 998 entries, 0 to 1009
    Data columns (total 19 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              998 non-null    object 
     1   Institution            998 non-null    object 
     2   InstCity               998 non-null    object 
     3   InstState              998 non-null    object 
     4   InstPostalCode         998 non-null    object 
     5   InstCountry            998 non-null    object 
     6   CongressionalDistrict  998 non-null    int64  
     7   Latitude               998 non-null    object 
     8   Longitude              998 non-null    object 
     9   YearAwarded            998 non-null    int64  
     10  ProjectTitle           998 non-null    object 
     11  Program                998 non-null    object 
     12  Division               998 non-null    object 
     13  AwardOutright          998 non-null    float64
     14  AwardMatching          998 non-null    float64
     15  ProjectDesc            998 non-null    object 
     16  ToSupport              998 non-null    object 
     17  Participants           955 non-null    object 
     18  Disciplines            998 non-null    object 
    dtypes: float64(2), int64(2), object(15)
    memory usage: 155.9+ KB


Let's take a closer look at the latitude and longitude information, which will be the basis for determining each point to map later on. This reveals that there are still at least 78 entries without location information:


```python
mappable_grant_info['Latitude'].describe()
```




    count         998
    unique        382
    top       Unknown
    freq           78
    Name: Latitude, dtype: object




```python
mappable_grant_info['Longitude'].describe()
```




    count         998
    unique        383
    top       unknown
    freq           78
    Name: Longitude, dtype: object



It will not be possible to map these grants without further information. My goal is to use data from the city and state location fields, which I can send to a geocoding tool in order to provide a location coordinate. To begin, I will remove the string values "unknown" or "Unknown" in the dataset and then replace them with null (blank) values, which I can filter more easily. Then, I will use Nominatim, a geocoding tool that will search for coordinate information based on the available place information.


```python
# replace strings with null values
nonvals = ['unknown','Unknown']

mappable_grant_info = mappable_grant_info.replace(nonvals, [None, None])

latlons = mappable_grant_info.loc[:,'Latitude':'Longitude']

print(latlons['Latitude'],'\n')
print('Is it a null value?',type(mappable_grant_info.loc[1009,'Latitude']))
```

    0       37.87029
    1       34.10373
    2       33.97561
    3        42.3395
    4       39.95298
              ...   
    1005    40.74955
    1006    32.88647
    1007    41.02476
    1008    29.94888
    1009        None
    Name: Latitude, Length: 998, dtype: object 
    
    Is it a null value? <class 'NoneType'>



```python
mappable_grant_info.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 998 entries, 0 to 1009
    Data columns (total 19 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              998 non-null    object 
     1   Institution            970 non-null    object 
     2   InstCity               997 non-null    object 
     3   InstState              998 non-null    object 
     4   InstPostalCode         970 non-null    object 
     5   InstCountry            991 non-null    object 
     6   CongressionalDistrict  998 non-null    int64  
     7   Latitude               920 non-null    object 
     8   Longitude              920 non-null    object 
     9   YearAwarded            998 non-null    int64  
     10  ProjectTitle           998 non-null    object 
     11  Program                998 non-null    object 
     12  Division               998 non-null    object 
     13  AwardOutright          998 non-null    float64
     14  AwardMatching          998 non-null    float64
     15  ProjectDesc            998 non-null    object 
     16  ToSupport              998 non-null    object 
     17  Participants           955 non-null    object 
     18  Disciplines            998 non-null    object 
    dtypes: float64(2), int64(2), object(15)
    memory usage: 195.9+ KB


Since there are only 920 Lat & Lon entries, that leaves the 78 unknown (now null) entries wihtout information. While it would be possible to gather this information with a tool like [Get Lat+Lon](http://teczno.com/squares/#6/41.60/-93.66), that would take while. So I will use the information about the recipient's city and state to get a general idea of where the award was given.


```python
# create a geoquery string, which will allow to at least map the city
mappable_grant_info['geoquery'] = mappable_grant_info['InstCity'] + ' ' + mappable_grant_info['InstState']

mappable_grant_info[mappable_grant_info['geoquery'].isnull()]
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>ProjectTitle</th>
      <th>Program</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>168</th>
      <td>FA-10157-68</td>
      <td>None</td>
      <td>None</td>
      <td>NY</td>
      <td>None</td>
      <td>USA</td>
      <td>0</td>
      <td>None</td>
      <td>None</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for University Teachers</td>
      <td>Research Programs</td>
      <td>15520.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>U.S. History</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Now there's only one left that lacks enough information. To check, look at that grant:


```python
mappable_grant_info.loc[168, :]
```




    AppNumber                                        FA-10157-68
    Institution                                             None
    InstCity                                                None
    InstState                                                 NY
    InstPostalCode                                          None
    InstCountry                                              USA
    CongressionalDistrict                                      0
    Latitude                                                None
    Longitude                                               None
    YearAwarded                                             1967
    ProjectTitle                             Title not available
    Program                  Fellowships for University Teachers
    Division                                   Research Programs
    AwardOutright                                          15520
    AwardMatching                                              0
    ProjectDesc                                   No description
    ToSupport                            No to support statement
    Participants                                             NaN
    Disciplines                                     U.S. History
    geoquery                                                 NaN
    Name: 168, dtype: object



There is just not enough information here - the record does not include information about the city or institution that received this award, so remove that one, too.


```python
mappable_grant_info = mappable_grant_info.drop([168])

mappable_grant_info.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 997 entries, 0 to 1009
    Data columns (total 20 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              997 non-null    object 
     1   Institution            970 non-null    object 
     2   InstCity               997 non-null    object 
     3   InstState              997 non-null    object 
     4   InstPostalCode         970 non-null    object 
     5   InstCountry            990 non-null    object 
     6   CongressionalDistrict  997 non-null    int64  
     7   Latitude               920 non-null    object 
     8   Longitude              920 non-null    object 
     9   YearAwarded            997 non-null    int64  
     10  ProjectTitle           997 non-null    object 
     11  Program                997 non-null    object 
     12  Division               997 non-null    object 
     13  AwardOutright          997 non-null    float64
     14  AwardMatching          997 non-null    float64
     15  ProjectDesc            997 non-null    object 
     16  ToSupport              997 non-null    object 
     17  Participants           955 non-null    object 
     18  Disciplines            997 non-null    object 
     19  geoquery               997 non-null    object 
    dtypes: float64(2), int64(2), object(16)
    memory usage: 163.6+ KB


### Geocode grants missing location coordinates

For the remaining 77 grants without location information, try to geocode by city and state information.


```python
# create a subset of the information for geoquerying
geoQuerySet = mappable_grant_info.loc[:, ['AppNumber','InstCity','InstState','Latitude','Longitude','geoquery']]

#pull out the entries without location coordinates
geoQuerySet = geoQuerySet[geoQuerySet['Latitude'].isnull()]

geoQuerySet.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 77 entries, 24 to 1009
    Data columns (total 6 columns):
     #   Column     Non-Null Count  Dtype 
    ---  ------     --------------  ----- 
     0   AppNumber  77 non-null     object
     1   InstCity   77 non-null     object
     2   InstState  77 non-null     object
     3   Latitude   0 non-null      object
     4   Longitude  0 non-null      object
     5   geoquery   77 non-null     object
    dtypes: object(6)
    memory usage: 4.2+ KB



```python
# set up geolocator
# increase timeout to reduce errors 
geolocator = Nominatim(user_agent='neh-grant-points', timeout=10)

# limit to comply with rate limiting requirements
geocode = RateLimiter(geolocator.geocode, min_delay_seconds=1)

# initiate geolocation for the 77 grants
geoQuerySet['location'] = geoQuerySet['geoquery'].apply(geocode)
```


```python
geoQuerySet.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 77 entries, 24 to 1009
    Data columns (total 7 columns):
     #   Column     Non-Null Count  Dtype 
    ---  ------     --------------  ----- 
     0   AppNumber  77 non-null     object
     1   InstCity   77 non-null     object
     2   InstState  77 non-null     object
     3   Latitude   0 non-null      object
     4   Longitude  0 non-null      object
     5   geoquery   77 non-null     object
     6   location   76 non-null     object
    dtypes: object(7)
    memory usage: 4.8+ KB


All but one of the `geoquery` requests returned a location value. To see what's missing, check for the empty values using `isnull()`:


```python
geoQuerySet[geoQuerySet['location'].isnull()]
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
      <th>AppNumber</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>geoquery</th>
      <th>location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>998</th>
      <td>RP-10003-68</td>
      <td>Charlottsvlle</td>
      <td>VA</td>
      <td>None</td>
      <td>None</td>
      <td>Charlottsvlle VA</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



Looks like that one has a typo! _I adapted the fix & replace process that @cduvallet outlined (link to full process at the end of this post)._


```python
fixgeolocate = {
    'Charlottsvlle VA' : 'Charlottesville VA'
}

new_locations = {}
for key, val in fixgeolocate.items():
    loc = geocode(val)
    new_locations[key] = loc
    sleep(1)
print(new_locations)
```

    {'Charlottsvlle VA': Location(The V, 201-213, 15th Street Northwest, Venable, Starr Hill, Charlottesville, Virginia, 22903, United States, (38.0360726, -78.49973472559668, 0.0))}


While here, I will also correct the typo in the dataset.


```python
# update to correct spelling of Charlottesville in cleaned_grants_for_mappable = mappable_grant_info i=998
mappable_grant_info = mappable_grant_info.replace('Charlottsvlle','Charlottesville')
```


```python
# update the geoquery set 
# get indices of rows with no location
null_locs = geoQuerySet[geoQuerySet['location'].isnull()].index

# sub in the new_locs
geoQuerySet.loc[null_locs, 'location'] = geoQuerySet.loc[null_locs, 'geoquery'].map(new_locations)
```


```python
geoQuerySet.head()
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
      <th>AppNumber</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>geoquery</th>
      <th>location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>AO-10025-69</td>
      <td>Westport</td>
      <td>CT</td>
      <td>None</td>
      <td>None</td>
      <td>Westport CT</td>
      <td>(Westport Court, Westport, Columbia County, Ge...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>EH-10038-68</td>
      <td>Evanston</td>
      <td>IL</td>
      <td>None</td>
      <td>None</td>
      <td>Evanston IL</td>
      <td>(Evanston, Evanston Township, Cook County, Ill...</td>
    </tr>
    <tr>
      <th>47</th>
      <td>EH-10058-66</td>
      <td>New York</td>
      <td>NY</td>
      <td>None</td>
      <td>None</td>
      <td>New York NY</td>
      <td>(New York, United States, (40.7127281, -74.006...</td>
    </tr>
    <tr>
      <th>51</th>
      <td>EO-10001-67</td>
      <td>Dover</td>
      <td>DE</td>
      <td>None</td>
      <td>None</td>
      <td>Dover DE</td>
      <td>(Dover, Kent County, Delaware, United States, ...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>EO-10051-68</td>
      <td>University Park</td>
      <td>PA</td>
      <td>None</td>
      <td>None</td>
      <td>University Park PA</td>
      <td>(University Park, College Township, Centre Cou...</td>
    </tr>
  </tbody>
</table>
</div>



Now we have all information to provide a coordinate location for those grants that didn't have the information. What remains is to pull the Lat & Lon information back into those columns.


```python
# pull the lats & lons from the locations - see 
geoQuerySet['Latitude'] = geoQuerySet['location'].apply(lambda x: x.latitude)
geoQuerySet['Longitude'] = geoQuerySet['location'].apply(lambda x: x.longitude)

geoQuerySet.head()
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
      <th>AppNumber</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>geoquery</th>
      <th>location</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>AO-10025-69</td>
      <td>Westport</td>
      <td>CT</td>
      <td>33.554852</td>
      <td>-82.066172</td>
      <td>Westport CT</td>
      <td>(Westport Court, Westport, Columbia County, Ge...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>EH-10038-68</td>
      <td>Evanston</td>
      <td>IL</td>
      <td>42.044739</td>
      <td>-87.693046</td>
      <td>Evanston IL</td>
      <td>(Evanston, Evanston Township, Cook County, Ill...</td>
    </tr>
    <tr>
      <th>47</th>
      <td>EH-10058-66</td>
      <td>New York</td>
      <td>NY</td>
      <td>40.712728</td>
      <td>-74.006015</td>
      <td>New York NY</td>
      <td>(New York, United States, (40.7127281, -74.006...</td>
    </tr>
    <tr>
      <th>51</th>
      <td>EO-10001-67</td>
      <td>Dover</td>
      <td>DE</td>
      <td>39.158168</td>
      <td>-75.524368</td>
      <td>Dover DE</td>
      <td>(Dover, Kent County, Delaware, United States, ...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>EO-10051-68</td>
      <td>University Park</td>
      <td>PA</td>
      <td>40.808749</td>
      <td>-77.858566</td>
      <td>University Park PA</td>
      <td>(University Park, College Township, Centre Cou...</td>
    </tr>
  </tbody>
</table>
</div>




```python
geoQuerySet.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 77 entries, 24 to 1009
    Data columns (total 7 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   AppNumber  77 non-null     object 
     1   InstCity   77 non-null     object 
     2   InstState  77 non-null     object 
     3   Latitude   77 non-null     float64
     4   Longitude  77 non-null     float64
     5   geoquery   77 non-null     object 
     6   location   77 non-null     object 
    dtypes: float64(2), object(5)
    memory usage: 7.3+ KB


### Merge the data

I can see using `info()` that the main dataframe still lacks the latitude and longitude information for the newly points geocoded points. So now, let's merge the locations back into the dataframe for mapping (`cleaned_grants_for_mappable`).


```python
# create a clean dataframe, just in case something happens to remove data
cleaned_grants_for_mappable = mappable_grant_info
```


```python
cleaned_grants_for_mappable.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 997 entries, 0 to 1009
    Data columns (total 20 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              997 non-null    object 
     1   Institution            970 non-null    object 
     2   InstCity               997 non-null    object 
     3   InstState              997 non-null    object 
     4   InstPostalCode         970 non-null    object 
     5   InstCountry            990 non-null    object 
     6   CongressionalDistrict  997 non-null    int64  
     7   Latitude               920 non-null    object 
     8   Longitude              920 non-null    object 
     9   YearAwarded            997 non-null    int64  
     10  ProjectTitle           997 non-null    object 
     11  Program                997 non-null    object 
     12  Division               997 non-null    object 
     13  AwardOutright          997 non-null    float64
     14  AwardMatching          997 non-null    float64
     15  ProjectDesc            997 non-null    object 
     16  ToSupport              997 non-null    object 
     17  Participants           955 non-null    object 
     18  Disciplines            997 non-null    object 
     19  geoquery               997 non-null    object 
    dtypes: float64(2), int64(2), object(16)
    memory usage: 163.6+ KB



```python
# create a list of the indices for matching rows between the data frames
geoInfotoReplaceIndices = geoQuerySet.loc[:,'AppNumber'].index
geoInfotoReplaceIndices
```




    Int64Index([  24,   40,   47,   51,   61,   74,   83,  103,  106,  107,  110,
                 111,  117,  120,  123,  124,  148,  151,  154,  174,  188,  198,
                 209,  341,  359,  378,  386,  387,  388,  398,  402,  412,  430,
                 437,  477,  482,  483,  484,  511,  522,  553,  597,  608,  624,
                 637,  669,  685,  689,  702,  708,  746,  750,  779,  791,  795,
                 802,  855,  856,  857,  859,  860,  861,  866,  871,  872,  885,
                 890,  902,  927,  936,  938,  940,  961,  970,  989,  998, 1009],
               dtype='int64')




```python
# use the row indexes to assign data from the geoQuerySet back to the cleaned map info for Latitude
cleaned_grants_for_mappable.loc[geoInfotoReplaceIndices,['Latitude']] = geoQuerySet.loc[geoInfotoReplaceIndices,['Latitude']]

# use the row indexes to assign data from the geoQuerySet back to the cleaned map info for Longitude
cleaned_grants_for_mappable.loc[geoInfotoReplaceIndices,['Longitude']] = geoQuerySet.loc[geoInfotoReplaceIndices,['Longitude']]
```


```python
cleaned_grants_for_mappable.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 997 entries, 0 to 1009
    Data columns (total 20 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   AppNumber              997 non-null    object 
     1   Institution            970 non-null    object 
     2   InstCity               997 non-null    object 
     3   InstState              997 non-null    object 
     4   InstPostalCode         970 non-null    object 
     5   InstCountry            990 non-null    object 
     6   CongressionalDistrict  997 non-null    int64  
     7   Latitude               997 non-null    object 
     8   Longitude              997 non-null    object 
     9   YearAwarded            997 non-null    int64  
     10  ProjectTitle           997 non-null    object 
     11  Program                997 non-null    object 
     12  Division               997 non-null    object 
     13  AwardOutright          997 non-null    float64
     14  AwardMatching          997 non-null    float64
     15  ProjectDesc            997 non-null    object 
     16  ToSupport              997 non-null    object 
     17  Participants           955 non-null    object 
     18  Disciplines            997 non-null    object 
     19  geoquery               997 non-null    object 
    dtypes: float64(2), int64(2), object(16)
    memory usage: 203.6+ KB



```python
# reassign our complete information to the mappable_grant_info dataframe
mappable_grant_info = cleaned_grants_for_mappable
```

At this point, there are two 'complete' datasets: `cleaned_grants_for_mappable` & `mappable_grant_info`. Below, `mappable_grant_info` is used the main set. 

### Add coordinates and map

Using the filled in latitude and longitude, we can now add in coordinate information to prepare to map each grant as a point on a map.


```python
# convert lat & lon into numeric values
mappable_grant_info.astype({'Latitude':'float64','Longitude':'float64'}).dtypes
```




    AppNumber                 object
    Institution               object
    InstCity                  object
    InstState                 object
    InstPostalCode            object
    InstCountry               object
    CongressionalDistrict      int64
    Latitude                 float64
    Longitude                float64
    YearAwarded                int64
    ProjectTitle              object
    Program                   object
    Division                  object
    AwardOutright            float64
    AwardMatching            float64
    ProjectDesc               object
    ToSupport                 object
    Participants              object
    Disciplines               object
    geoquery                  object
    dtype: object




```python
# combine latitute and longitude to create coordinate information for mapping
mappable_grant_info['coordinates'] = mappable_grant_info[['Longitude','Latitude']].values.tolist()

mappable_grant_info.tail()
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>...</th>
      <th>Program</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
      <th>coordinates</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1005</th>
      <td>RX-10010-69</td>
      <td>American Council of Learned Societies</td>
      <td>New York</td>
      <td>NY</td>
      <td>10017-6706</td>
      <td>USA</td>
      <td>12</td>
      <td>40.74955</td>
      <td>-73.97462</td>
      <td>1969</td>
      <td>...</td>
      <td>Conferences</td>
      <td>Research Programs</td>
      <td>25000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Gordon Turner [Project Director]</td>
      <td>Interdisciplinary Studies, General</td>
      <td>New York NY</td>
      <td>[-73.97462, 40.74955]</td>
    </tr>
    <tr>
      <th>1006</th>
      <td>EO-10125</td>
      <td>Salk Institute for Biological Studies</td>
      <td>La Jolla</td>
      <td>CA</td>
      <td>92037-1002</td>
      <td>USA</td>
      <td>49</td>
      <td>32.88647</td>
      <td>-117.24392</td>
      <td>1969</td>
      <td>...</td>
      <td>Institutional Planning and Development</td>
      <td>Education Programs</td>
      <td>7434.0</td>
      <td>0.0</td>
      <td>To teach a course based on American texts and ...</td>
      <td>No to support statement</td>
      <td>Christine Tracy [Project Director]</td>
      <td>History and Philosophy of Science, Technology,...</td>
      <td>La Jolla CA</td>
      <td>[-117.24392, 32.88647]</td>
    </tr>
    <tr>
      <th>1007</th>
      <td>EO-10225</td>
      <td>Manhattanville College</td>
      <td>Purchase</td>
      <td>NY</td>
      <td>10577-2132</td>
      <td>USA</td>
      <td>17</td>
      <td>41.02476</td>
      <td>-73.71567</td>
      <td>1969</td>
      <td>...</td>
      <td>Institutional Planning and Development</td>
      <td>Education Programs</td>
      <td>28320.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Marcus Lawson [Project Director]</td>
      <td>Interdisciplinary Studies, General</td>
      <td>Purchase NY</td>
      <td>[-73.71567, 41.02476]</td>
    </tr>
    <tr>
      <th>1008</th>
      <td>EO-10231</td>
      <td>Louisiana Endowment for the Humanities</td>
      <td>New Orleans</td>
      <td>LA</td>
      <td>70113-1027</td>
      <td>USA</td>
      <td>2</td>
      <td>29.94888</td>
      <td>-90.07392</td>
      <td>1969</td>
      <td>...</td>
      <td>Institutional Planning and Development</td>
      <td>Education Programs</td>
      <td>10000.0</td>
      <td>0.0</td>
      <td>This is one of seven "summer workshops on Negr...</td>
      <td>No to support statement</td>
      <td>Elton Harrison [Project Director]</td>
      <td>Interdisciplinary Studies, General</td>
      <td>New Orleans LA</td>
      <td>[-90.07392, 29.94888]</td>
    </tr>
    <tr>
      <th>1009</th>
      <td>RO-10389</td>
      <td>California State University, Long Beach</td>
      <td>Long Beach</td>
      <td>CA</td>
      <td>90840-0004</td>
      <td>USA</td>
      <td>48</td>
      <td>33.769</td>
      <td>-118.192</td>
      <td>1969</td>
      <td>...</td>
      <td>Basic Research</td>
      <td>Research Programs</td>
      <td>9990.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Nizan Shaked [Project Director]</td>
      <td>Social Sciences, General</td>
      <td>Long Beach CA</td>
      <td>[-118.191604, 33.7690164]</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
# create shapely geoPoints for a geometry column
mappable_grant_info['geometry'] = gpd.points_from_xy(mappable_grant_info['Longitude'],mappable_grant_info['Latitude'])
```


```python
mappable_grant_info.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 997 entries, 0 to 1009
    Data columns (total 22 columns):
     #   Column                 Non-Null Count  Dtype   
    ---  ------                 --------------  -----   
     0   AppNumber              997 non-null    object  
     1   Institution            970 non-null    object  
     2   InstCity               997 non-null    object  
     3   InstState              997 non-null    object  
     4   InstPostalCode         970 non-null    object  
     5   InstCountry            990 non-null    object  
     6   CongressionalDistrict  997 non-null    int64   
     7   Latitude               997 non-null    object  
     8   Longitude              997 non-null    object  
     9   YearAwarded            997 non-null    int64   
     10  ProjectTitle           997 non-null    object  
     11  Program                997 non-null    object  
     12  Division               997 non-null    object  
     13  AwardOutright          997 non-null    float64 
     14  AwardMatching          997 non-null    float64 
     15  ProjectDesc            997 non-null    object  
     16  ToSupport              997 non-null    object  
     17  Participants           955 non-null    object  
     18  Disciplines            997 non-null    object  
     19  geoquery               997 non-null    object  
     20  coordinates            997 non-null    object  
     21  geometry               997 non-null    geometry
    dtypes: float64(2), geometry(1), int64(2), object(17)
    memory usage: 219.1+ KB



```python
mappable_grant_info['geometry'].describe
```




    <bound method NDFrame.describe of 0       POINT (-122.26813 37.87029)
    1       POINT (-117.70701 34.10373)
    2       POINT (-117.33113 33.97561)
    3        POINT (-71.09048 42.33950)
    4        POINT (-75.19276 39.95298)
                       ...             
    1005     POINT (-73.97462 40.74955)
    1006    POINT (-117.24392 32.88647)
    1007     POINT (-73.71567 41.02476)
    1008     POINT (-90.07392 29.94888)
    1009    POINT (-118.19160 33.76902)
    Name: geometry, Length: 997, dtype: geometry>



The geometry column now contains POINTs, a special datatype that will be necessary for the geospatial dataset.

For the final steps of cleaning and transforming this data to a geospatial dataset, the `geopandas` module provides more options, so I will convert the data to geopandas (aka "gpd") now:


```python
# convert the information to a geopandas dataframe
mappable_gdf = gpd.GeoDataFrame(mappable_grant_info)
```

Using the `type()` function, I can see that this is now a "GeoDataFrame" with "GeoSeries" in some of the fields.


```python
type(mappable_gdf)
```




    geopandas.geodataframe.GeoDataFrame




```python
type(mappable_gdf['geometry'])
```




    geopandas.geoseries.GeoSeries



At this point, the `mappable_gdf` data is ready to plot on a map. For these purposes, the data is cleaned and consistent and contains the necessary geospatial coordinates to create a point for each of the 997 grants made during the 1960s decade for which NEH maintained information about the location of the award, and that were made to the U.S. states. 

## Data cleanup by geography

Now I have relatively clean data, and I can use basic mapping functions of geopandas to get an idea of any egregious outliers. For this task, geopandas supports some basic mapping tools (some of the built-in mapping features [included in geopandas](https://geopandas.readthedocs.io/en/latest/gallery/create_geopandas_from_pandas.html)) to get an idea of additional data errors. Once I have taken these additional steps, I can export this data for use later on to create more detailed maps. 


```python
# create a mapspace for the geodataframe
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))

usa = world.query('name == "United States of America"')

ax = usa.plot(
    color='white', edgecolor='black'
)

mappable_gdf.plot(ax=ax, color='red')

plt.show()
```


![png]({{ site.url }}{{ site.baseurl }}/images/wrangling-humanities-data/output_75_0.png)


The map is small, but this offers an initial data quality check, and I can see that there are a few points that don't look right. There may be some in Europe (or Canada?) and one that appears to be in the Pacific. It's possible that this is a grant to American Samoa or another US territory in the Pacific, but I want to find out. 


```python
# find the distant outlier... 

mappable_gdf.sort_values(by=['Latitude','Longitude']).head(2)
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>...</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
      <th>coordinates</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>151</th>
      <td>FA-10103-67</td>
      <td>None</td>
      <td>Richmond</td>
      <td>VA</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>-32.8652</td>
      <td>151.501</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>13270.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>U.S. History</td>
      <td>Richmond VA</td>
      <td>[151.5012, -32.8652]</td>
      <td>POINT (151.50120 -32.86520)</td>
    </tr>
    <tr>
      <th>702</th>
      <td>FT-10051-67</td>
      <td>Elementary Shakespeare Corp.</td>
      <td>Fort Pierce</td>
      <td>FL</td>
      <td>34982</td>
      <td>USA</td>
      <td>18</td>
      <td>27.4467</td>
      <td>-80.3256</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>2000.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>S. Len Weingart [Project Director]</td>
      <td>British Literature</td>
      <td>Fort Pierce FL</td>
      <td>[-80.3256056, 27.4467056]</td>
      <td>POINT (-80.32561 27.44671)</td>
    </tr>
  </tbody>
</table>
<p>2 rows × 22 columns</p>
</div>



That one appears to have been geocoded as a location in Richmond Vale, New South Wales, Australia, which was an error that I should've corrected in the geocoding step. But let's correct that now: [Get Lat+Lon](http://teczno.com/squares/#14/37.53851/-77.43428) suggests 37.5385087,-77.43428 would be acceptable, and so does Nominatim:


```python
geocode('Richmond Virginia')
```




    Location(Richmond, Richmond City, Virginia, 23298, United States, (37.5385087, -77.43428, 0.0))




```python
# update the coordinate values
mappable_gdf.at[151,'Latitude'] = 37.5385087
mappable_gdf.at[151,'Longitude'] =  -77.43428

mappable_gdf.iloc[151]
```




    AppNumber                                        FA-10103-67
    Institution                                             None
    InstCity                                            Richmond
    InstState                                                 VA
    InstPostalCode                                          None
    InstCountry                                             None
    CongressionalDistrict                                      0
    Latitude                                             37.5385
    Longitude                                           -77.4343
    YearAwarded                                             1967
    ProjectTitle                             Title not available
    Program                  Fellowships for University Teachers
    Division                                   Research Programs
    AwardOutright                                          13270
    AwardMatching                                              0
    ProjectDesc                                   No description
    ToSupport                            No to support statement
    Participants                                             NaN
    Disciplines                                     U.S. History
    geoquery                                         Richmond VA
    coordinates                             [151.5012, -32.8652]
    geometry                           POINT (151.5012 -32.8652)
    Name: 151, dtype: object




```python
# create a Point for the geometry column
mappable_gdf.loc[151,'geometry'] = Point(mappable_gdf.loc[151,'Longitude'], mappable_gdf.loc[151,'Latitude'])
```

Map again... 


```python
# create a mapspace for the geodataframe
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))

usa = world.query('name == "United States of America"')

ax = usa.plot(
    color='white', edgecolor='black'
)

mappable_gdf.plot(ax=ax, color='red')

plt.show()
```


![png]({{ site.url }}{{ site.baseurl }}/images/wrangling-humanities-data/output_83_0.png)


There's still appear to be 3 very far to the east of North America... I can identify thes points by sorting the Longitude in descending order:


```python
# fix a datatype problem for sorting... 
mappable_gdf = mappable_gdf.astype({'Longitude':'float64','Latitude':'float64'})

# find the farthest east... 
mappable_gdf.sort_values(by=['Longitude'],ascending=False).head(5)
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>...</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
      <th>coordinates</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>341</th>
      <td>FT-10462-68</td>
      <td>University of Southern Florida</td>
      <td>Orlando</td>
      <td>DE</td>
      <td>32816</td>
      <td>USA</td>
      <td>7</td>
      <td>52.524225</td>
      <td>13.409345</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>1500.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Richard Lukas [Project Director]</td>
      <td>History, General</td>
      <td>Orlando DE</td>
      <td>[13.4093448, 52.5242253]</td>
      <td>POINT (13.40934 52.52423)</td>
    </tr>
    <tr>
      <th>597</th>
      <td>FA-10060-67</td>
      <td>Unaffiliated Independent Scholar</td>
      <td>Belmont</td>
      <td>MA</td>
      <td>02178-0000</td>
      <td>USA</td>
      <td>0</td>
      <td>48.408657</td>
      <td>7.238676</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>13270.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Einar Haugen [Project Director]</td>
      <td>Literature, Other</td>
      <td>Belmont MA</td>
      <td>[7.2386756, 48.4086571]</td>
      <td>POINT (7.23868 48.40866)</td>
    </tr>
    <tr>
      <th>483</th>
      <td>RO-10335-69</td>
      <td>None</td>
      <td>Dennis</td>
      <td>MA</td>
      <td>None</td>
      <td>USA</td>
      <td>0</td>
      <td>51.519807</td>
      <td>-0.137680</td>
      <td>1968</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>4800.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Art History and Criticism</td>
      <td>Dennis MA</td>
      <td>[-0.13767995578500702, 51.519807150000005]</td>
      <td>POINT (-0.13768 51.51981)</td>
    </tr>
    <tr>
      <th>645</th>
      <td>FB-10135-67</td>
      <td>University of Maine System</td>
      <td>Orono</td>
      <td>ME</td>
      <td>04473-1513</td>
      <td>USA</td>
      <td>2</td>
      <td>44.887320</td>
      <td>-68.680500</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>8140.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Jerome Nadelhaft [Project Director]</td>
      <td>U.S. History</td>
      <td>Orono ME</td>
      <td>[-68.6805, 44.88732]</td>
      <td>POINT (-68.68050 44.88732)</td>
    </tr>
    <tr>
      <th>182</th>
      <td>FB-10085-67</td>
      <td>Colby College</td>
      <td>Waterville</td>
      <td>ME</td>
      <td>04901-8840</td>
      <td>USA</td>
      <td>1</td>
      <td>44.541900</td>
      <td>-69.639730</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>8140.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Dorothy Koonce [Project Director]</td>
      <td>Classics</td>
      <td>Waterville ME</td>
      <td>[-69.63973, 44.5419]</td>
      <td>POINT (-69.63973 44.54190)</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



These were geocoded in Europe. 


```python
#row 341
geocode('Orlando DE')
```




    Location(Orlando, 2, Münzstraße, Scheunenviertel, Mitte, Berlin, 10178, Deutschland, (52.5242253, 13.4093448, 0.0))




```python
# this one appears to be a mistake in the data: Florida, not Delaware
mappable_gdf.loc[341]
```




    AppNumber                                     FT-10462-68
    Institution                University of Southern Florida
    InstCity                                          Orlando
    InstState                                              DE
    InstPostalCode                                      32816
    InstCountry                                           USA
    CongressionalDistrict                                   7
    Latitude                                          52.5242
    Longitude                                         13.4093
    YearAwarded                                          1967
    ProjectTitle                          Title not available
    Program                                   Summer Stipends
    Division                                Research Programs
    AwardOutright                                        1500
    AwardMatching                                           0
    ProjectDesc                                No description
    ToSupport                         No to support statement
    Participants             Richard Lukas [Project Director]
    Disciplines                              History, General
    geoquery                                       Orlando DE
    coordinates                      [13.4093448, 52.5242253]
    geometry                    POINT (13.4093448 52.5242253)
    Name: 341, dtype: object




```python
#row 597
geocode('Belmont MA')
```




    Location(Belmont, Molsheim, Bas-Rhin, Grand Est, France métropolitaine, 67130, France, (48.4086571, 7.2386756, 0.0))




```python
#row 483
geocode('Dennis MA')
```




    Location(Dennis, 30, Cleveland Street, Cavendish Square & Oxford Market (South), Fitzrovia, City of Westminster, London, Greater London, England, W1T 4NG, United Kingdom, (51.519807150000005, -0.13767995578500702, 0.0))



I could've avoided this by telling the geocoder to prefer locations in North America. Nominatim, for example, provides options for `bounded` or `country_codes` to limit results. If I am geocoding a longer listin future, I would consider this as an option. For now, these can be corrected by providing a more specific country string with the request:


```python
#this is better... 
geocode('Dennis MA USA')
```




    Location(Dennis, Barnstable County, Massachusetts, United States, (41.7353872, -70.1939087, 0.0))



Following @cduvallet's example, I'm creating a function to update location records. The function takes a specified dataframe, row index, and Location() information.


```python
def update_location(gdf, rowIndex, location):
    '''
    The function updates a record's latitude, longitude, and adds a geo Point entry
    The To specify the changes: 
        gdf is a specific dataframe
        rowIndex is the numerical index or Name of the row to update
        location is a new geoquery string to call for a Lcoation() object from the (Nominatim) geocoder
    The function requires the geocoder to be already implemented as geocode() and for Point to be imported from shapely. 
    '''
    newloc = geocode(location)
    gdf.at[rowIndex, 'Latitude'] = newloc.latitude
    gdf.at[rowIndex, 'Longitude'] = newloc.longitude
    # create a Point
    gdf.at[rowIndex, 'geometry'] = Point(newloc.longitude,newloc.latitude)
```

I'll test this on item 341. This one had a typo: the state was listed as Delaware, not Florida, so I'll provide the correct geoquery to the function, which will query Nominatim and update the data.


```python
# test the function ... 
#row 341
#geocode('Orlando FL')
update_location(mappable_gdf, 341, 'Orlando FL')

#while we're at it, fix the data typo in the InstState field, also in the previous dataframes
mappable_gdf.at[341,'InstState'] = 'FL'
mappable_grant_info.at[341,'InstState'] = 'FL'
cleaned_grants_for_mappable.at[341,'InstState'] = 'FL'

mappable_gdf.loc[341]
```




    AppNumber                                          FT-10462-68
    Institution                     University of Southern Florida
    InstCity                                               Orlando
    InstState                                                   FL
    InstPostalCode                                           32816
    InstCountry                                                USA
    CongressionalDistrict                                        7
    Latitude                                                28.548
    Longitude                                             -81.4128
    YearAwarded                                               1967
    ProjectTitle                               Title not available
    Program                                        Summer Stipends
    Division                                     Research Programs
    AwardOutright                                             1500
    AwardMatching                                                0
    ProjectDesc                                     No description
    ToSupport                              No to support statement
    Participants                  Richard Lukas [Project Director]
    Disciplines                                   History, General
    geoquery                                            Orlando DE
    coordinates                           [13.4093448, 52.5242253]
    geometry                 POINT (-81.41278418563017 28.5479786)
    Name: 341, dtype: object



That looks good, so now I will fix the remaining two.


```python
# update the other two points
update_location(mappable_gdf, 483, 'Dennis MA USA')
update_location(mappable_gdf, 597, 'Belmont MA USA')
```


```python
# remap... 
ax = usa.plot(
    color='white', edgecolor='black'
)
mappable_gdf.plot(ax=ax, color='red')

plt.show()
```


![png]({{ site.url }}{{ site.baseurl }}/images/wrangling-humanities-data/output_99_0.png)


At least all of the points appear to be in logical places now, even if there may be some underlying noise. This brings the data to a point that is clean enough to map! 
But first, what other cool things can we do with the data using gepandas?

### Geopandas supports various data filtering

Now that the data is more or less in a shape that I want, it is ready for more analysis and visualization. We can count the number of awards by state, see all of the awards in a given state, or tally the amount of money awarded to a particular geographic area. And of course, we can make some more detailed maps! First, try some of the filtering and analyzing of the data that pandas supports. 

For example, how may grants were made to each state? 


```python
# use groupby to count by state
mappable_gdf.groupby('InstState').InstState.count()
```




    InstState
    AK      6
    AL      5
    AR      1
    AZ     13
    CA    111
    CO     10
    CT     28
    DC     44
    DE     12
    FL     18
    GA     11
    HI      8
    IA     12
    ID      1
    IL     36
    IN     23
    KS      7
    KY      7
    LA      8
    MA     76
    MD     29
    ME      3
    MI     31
    MN     14
    MO      9
    MS      7
    MT      3
    NC     33
    ND      6
    NE      7
    NH      1
    NJ     36
    NM      2
    NV      2
    NY    146
    OH     32
    OK     10
    OR      6
    PA     53
    RI      7
    SC      9
    SD      4
    TN     20
    TX     26
    UT      3
    VA     22
    VT      3
    WA     11
    WI     16
    WV      3
    WY      6
    Name: InstState, dtype: int64



Or I can ask how what grants were made to a specific state:


```python
# show grants from Montana
mappable_gdf[mappable_gdf['InstState']=='MT']
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>...</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
      <th>coordinates</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>326</th>
      <td>FT-10374-68</td>
      <td>University of Montana</td>
      <td>Missoula</td>
      <td>MT</td>
      <td>59801-4494</td>
      <td>USA</td>
      <td>1</td>
      <td>46.86494</td>
      <td>-113.98401</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>1500.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Duane Hampton [Project Director]</td>
      <td>U.S. History</td>
      <td>Missoula MT</td>
      <td>[-113.98401, 46.86494]</td>
      <td>POINT (-113.98401 46.86494)</td>
    </tr>
    <tr>
      <th>348</th>
      <td>FT-10738-69</td>
      <td>University of Montana</td>
      <td>Missoula</td>
      <td>MT</td>
      <td>59801-4494</td>
      <td>USA</td>
      <td>1</td>
      <td>46.86494</td>
      <td>-113.98401</td>
      <td>1969</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>1500.0</td>
      <td>0.0</td>
      <td>Study of the concept of family in the politica...</td>
      <td>No to support statement</td>
      <td>Richard Chapman [Project Director]</td>
      <td>Political Science, General</td>
      <td>Missoula MT</td>
      <td>[-113.98401, 46.86494]</td>
      <td>POINT (-113.98401 46.86494)</td>
    </tr>
    <tr>
      <th>921</th>
      <td>RO-10179-67</td>
      <td>University of Montana</td>
      <td>Missoula</td>
      <td>MT</td>
      <td>59801-4494</td>
      <td>USA</td>
      <td>1</td>
      <td>46.86494</td>
      <td>-113.98401</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>1370.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Joseph Mussulman [Project Director]</td>
      <td>History, General</td>
      <td>Missoula MT</td>
      <td>[-113.98401, 46.86494]</td>
      <td>POINT (-113.98401 46.86494)</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 22 columns</p>
</div>




```python
Minnesota_1960s_grants = mappable_gdf[mappable_gdf['InstState'] == 'MN']
```


```python
Minnesota_1960s_grants.head()
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
      <th>AppNumber</th>
      <th>Institution</th>
      <th>InstCity</th>
      <th>InstState</th>
      <th>InstPostalCode</th>
      <th>InstCountry</th>
      <th>CongressionalDistrict</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>YearAwarded</th>
      <th>...</th>
      <th>Division</th>
      <th>AwardOutright</th>
      <th>AwardMatching</th>
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geoquery</th>
      <th>coordinates</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>174</th>
      <td>FA-10546-68</td>
      <td>None</td>
      <td>Minneapolis</td>
      <td>MN</td>
      <td>None</td>
      <td>USA</td>
      <td>0</td>
      <td>44.97730</td>
      <td>-93.265469</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>15520.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>American Literature</td>
      <td>Minneapolis MN</td>
      <td>[-93.2654692, 44.9772995]</td>
      <td>POINT (-93.26547 44.97730)</td>
    </tr>
    <tr>
      <th>188</th>
      <td>FB-10105-67</td>
      <td>None</td>
      <td>Minneapolis</td>
      <td>MN</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>44.97730</td>
      <td>-93.265469</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>8140.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>U.S. History</td>
      <td>Minneapolis MN</td>
      <td>[-93.2654692, 44.9772995]</td>
      <td>POINT (-93.26547 44.97730)</td>
    </tr>
    <tr>
      <th>426</th>
      <td>RO-10036-67</td>
      <td>University of Minnesota, Twin Cities</td>
      <td>Minneapolis</td>
      <td>MN</td>
      <td>55455-0433</td>
      <td>USA</td>
      <td>5</td>
      <td>44.97779</td>
      <td>-93.236240</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>18728.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Harrold Alllen [Project Director]</td>
      <td>Linguistics</td>
      <td>Minneapolis MN</td>
      <td>[-93.23624, 44.97779]</td>
      <td>POINT (-93.23624 44.97779)</td>
    </tr>
    <tr>
      <th>477</th>
      <td>RO-10293-68</td>
      <td>None</td>
      <td>Minneapolis</td>
      <td>MN</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>44.97730</td>
      <td>-93.265469</td>
      <td>1968</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>18730.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>NaN</td>
      <td>Linguistics</td>
      <td>Minneapolis MN</td>
      <td>[-93.2654692, 44.9772995]</td>
      <td>POINT (-93.26547 44.97730)</td>
    </tr>
    <tr>
      <th>650</th>
      <td>FB-10159-67</td>
      <td>University of Minnesota, Twin Cities</td>
      <td>Minneapolis</td>
      <td>MN</td>
      <td>55455-0433</td>
      <td>USA</td>
      <td>5</td>
      <td>44.97779</td>
      <td>-93.236240</td>
      <td>1967</td>
      <td>...</td>
      <td>Research Programs</td>
      <td>8140.0</td>
      <td>0.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Jasper Hopkins [Project Director]</td>
      <td>Philosophy, General</td>
      <td>Minneapolis MN</td>
      <td>[-93.23624, 44.97779]</td>
      <td>POINT (-93.23624 44.97779)</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 22 columns</p>
</div>



And I can map grants in a particular state:


```python
# re map... 
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))

usa = world.query('name == "United States of America"')

ax = usa.plot(
    color='white', edgecolor='black'
)
Minnesota_1960s_grants.plot(ax=ax, color='blue')

plt.show()
```


![png]({{ site.url }}{{ site.baseurl }}/images/wrangling-humanities-data/output_107_0.png)


# Exporting a clean dataset

## Output the data as geojson

Before moving on, I want to conclude this section by creating outputting the data as geojson. This allows us to reuse or share the cleaned data with others, or port it into future visualization tools. For example, the data file can be ported into tools that have increased dynamic mapping capabilities, such as [leaflet.js](https://leafletjs.com/examples/geojson/) or [geojson.io](https://geojson.io/).

Geopandas uses the `fiona` library for this functions, so you may need to install and import it.


```python
#!pip install fiona
#import fiona
```


```python
# get rid of those useless coordinate & geoquery fields
mappable_gdf = mappable_gdf.drop(['coordinates'], axis=1)
mappable_gdf = mappable_gdf.drop(['geoquery'], axis=1)
mappable_gdf = mappable_gdf.drop(['AwardMatching'], axis=1)
```


```python
# output cleaned data to geojson

mappable_gdf.to_file('neh_1960s_grants.geojson', driver='GeoJSON')
```

This notebook used many of the analysis and transformation features in python to analyze, clean, and transform the source dataset of NEH's grants from the 1960s. Now I have a dataset, which I will reuse in the next steps as a basic data for mapping. Stay tuned for the next notebooks to follow this process. 

### Reference list

Credit to the examples in these tutorials (as of January 2021), which were highly informative to the exploratory work outlined above:

* Duong Vu, "[Introduction to Geospatial Data in Python](http://datacamp.com/community/tutorials/geospatial-data-python)" at _Datacamp_, published 24 October 2018.
* "[Using GeoJson with Leaflet](https://leafletjs.com/examples/geojson/)" at _leaflet.js_
* Lesley Cordero, "[Getting Started on Geospatial Analysis with Python, GeoJSON and GeoPandas](https://www.twilio.com/blog/2017/08/geospatial-analysis-python-geojson-geopandas.html)" at Twilio, published 14 August 2017.
* Claire Duvallet, "[Mapping my cross-country road trip with Python](https://cduvallet.github.io/posts/2020/02/road-trip-map)", published 9 February 2020.
* Dani Arribas-Bel, "[Mapping in Python with geopandas](http://darribas.org/gds15/content/labs/lab_03.html)", from Geographic Data Science '15. Provides a lot of useful information about creating more complex plots.
* @carsonfarmer offers a great overview of using python for geospatial data wrangling, including an overview of pandas for data analysis, geopandas, and many other geospatial tools available in the python and Jupyter environment; see https://github.com/carsonfarmer/python_geospatial/tree/master/notebooks 

See these sites for US state shapefile information: 

* Eric Celeste http://eric.clst.org/tech/usgeojson
* US Census provides various geographic files for US states and other entities: https://www.census.gov/geographies/mapping-files/time-series/geo/carto-boundary-file.html
