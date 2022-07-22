---
title: 'Wrangling Humanities Data: Exploratory Maps of NEH Awards by State'
date: 2021-01-22
permalink: /posts/2021/mapping-neh-data-by-state-1960s/
excerpt: 'This post continues the "Wrangling NEH Data" topic and demonstrates how to map the geographic distribution of NEH funds across different states using grant data from the 1960s.'
header:
  overlay_image: "/assets/images/binary-1280w.jpg"
  overlay_filter: 0.3
  image_description: "Rows and columns of 0 and 1 with a blue background."
  caption: "Photo by [Gerd Altmann](https://pixabay.com/illustrations/binary-digitization-null-one-pay-1377017/) on Pixabay"
  og_image: "/assets/images/wrangling-humanities-data/mapping-by-state/output_27_0.png"
  teaser: "/assets/images/wrangling-humanities-data/mapping-by-state/output_27_0.png"
categories:
  - mapping humanities
  - data curation
tags:
  - python
  - neh 
  - geospatial
  - humanities data
classes: wide
---

{% include toc_mapping_humanities_data.md %}

*This installment uses the [geospatial dataset previously created]({% post_url 2021-01-19-cleaning-transforming-data %}) and uses some of the visualization tools provided by the geopandas librarywhich is supported in a Python environment and Jupyter notebook. You can download [a Jupyter Notebook version of this post](https://github.com/morskyjezek/neh-grant-data-project/blob/main/03a-mapping-by-state.ipynb) from the GitHub repository along with all of the data discussed here. File references discussed below are included in the same [neh-grant-data-project repository](https://github.com/morskyjezek/neh-grant-data-project).*


# Mapping State by State

Now that I have a pretty good set of points, I wanted to visualize these against maps for the states. For example, is it possible to see all of the grants in a given state for a given time period? Could I look at different states? What about all of the continental states? For these kinds of questions, geopandas has a lot of built-in tools for filtering the clean data, as well as for outputting a few initial maps. Below, I walk through a process to develop state-by-state maps. This uses some of the filtering capacities of pandas in combination with the mapping visualization tools of geopandas. Let's go! 

## Set up the environment

This activity will only use two python modules: geopandas and matplotlib. However, it also requires the clean dataset that I developed previously. Since that was saved as a geojson file, it is now reusable and serves as the basis for these examples. The data file is included in the repository as `neh_1960s_grants.geojson`. I am still exploring this data, so rather than looking at the entirety of the NEH data that was available, I continue working with the 1960s decade as before.    


```python
import geopandas as gpd

import matplotlib.pyplot as plt
%matplotlib inline
```

## Start with the geojson

Having the previously created and saved geojson data, which is already cleaned and transformed to include valid POINT coordinates, now I can use the file to load in data rather than going through the cleaning and transformation process again.


```python
gdf_neh_1960s = gpd.read_file('neh_1960s_grants.geojson', driver='GeoJSON')
```


```python
gdf_neh_1960s.head()
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
      <th>ProjectDesc</th>
      <th>ToSupport</th>
      <th>Participants</th>
      <th>Disciplines</th>
      <th>geometry</th>
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
      <td>No description</td>
      <td>No to support statement</td>
      <td>John Elliot [Project Director]</td>
      <td>English</td>
      <td>POINT (-122.26813 37.87029)</td>
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
      <td>No description</td>
      <td>No to support statement</td>
      <td>Steven Matthysse [Project Director]</td>
      <td>History of Religion</td>
      <td>POINT (-117.70701 34.10373)</td>
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
      <td>No description</td>
      <td>No to support statement</td>
      <td>John Staude [Project Director]</td>
      <td>History, General</td>
      <td>POINT (-117.33113 33.97561)</td>
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
      <td>42.33950</td>
      <td>-71.09048</td>
      <td>1967</td>
      <td>Title not available</td>
      <td>Fellowships for Younger Scholars</td>
      <td>Fellowships and Seminars</td>
      <td>8387.0</td>
      <td>No description</td>
      <td>No to support statement</td>
      <td>Thomas Havens [Project Director]</td>
      <td>History, General</td>
      <td>POINT (-71.09048 42.33950)</td>
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
      <td>No description</td>
      <td>No to support statement</td>
      <td>Gresham Riley [Project Director]</td>
      <td>Psychology</td>
      <td>POINT (-75.19276 39.95298)</td>
    </tr>
  </tbody>
</table>
</div>




```python
gdf_neh_1960s.shape
```




    (997, 19)



The data import worked as expected. The first four records appear correctly, and the `.shape` call shows 997 records, which is what it should be (the CSV had 998 rows).

## Mapping State by State

Now I can use geopandas to filter the data to create different kinds of maps,
and to export maps to files that can be reused in reports. 

### Define and use map shapes for US states

The project to clean and check data quality focused on information with single, clear geographic coordinates, aka POINTs. For more complex maps, I need to use some additional geospatial data types. Geospatial data and visualization most frequently uses 2D shapes, called Polygons or Multipolygons, which are used to represent areas on maps like states or countries. In this task to create state maps, I will need to pair the point data with the states' shape data. Rather than going to data.gov, which is where the NEH data can be found, I used the useful [US state shape data that Eric Celeste has made available](http://eric.clst.org/tech/usgeojson), which makes it easy to get geojson and various levels of detail. 

Fortunately, the US state shapes are well defined, and the Census Bureau and others make the information readily available. This section explains how to import the shapes for US states, how to display the shapes, and then how to display the points for each grant on the state shapes.  


```python
# test the states shapefile

us_states = gpd.read_file('gz_2010_us_040_00_5m.json')

us_states.head()
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
      <th>GEO_ID</th>
      <th>STATE</th>
      <th>NAME</th>
      <th>LSAD</th>
      <th>CENSUSAREA</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0400000US01</td>
      <td>01</td>
      <td>Alabama</td>
      <td></td>
      <td>50645.326</td>
      <td>MULTIPOLYGON (((-88.12466 30.28364, -88.08681 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0400000US02</td>
      <td>02</td>
      <td>Alaska</td>
      <td></td>
      <td>570640.950</td>
      <td>MULTIPOLYGON (((-166.10574 53.98861, -166.0752...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0400000US04</td>
      <td>04</td>
      <td>Arizona</td>
      <td></td>
      <td>113594.084</td>
      <td>POLYGON ((-112.53859 37.00067, -112.53454 37.0...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0400000US05</td>
      <td>05</td>
      <td>Arkansas</td>
      <td></td>
      <td>52035.477</td>
      <td>POLYGON ((-94.04296 33.01922, -94.04304 33.079...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0400000US06</td>
      <td>06</td>
      <td>California</td>
      <td></td>
      <td>155779.220</td>
      <td>MULTIPOLYGON (((-122.42144 37.86997, -122.4213...</td>
    </tr>
  </tbody>
</table>
</div>




```python
type(us_states)
```




    geopandas.geodataframe.GeoDataFrame




```python
type(us_states['geometry'])
```




    geopandas.geoseries.GeoSeries



I expected the data to be imported as a GeoDataFrame, which it is, and I confirmed that the `geometry` column is a series data. In the display of the data, also note that the geometry types are all "Polygon" or "Multipolygon". (The latter is a separate datatype that is required when states have non-contiguous parts, like the Hawaiian islands.) Note that the `NAME` column has the full state name, which I will use to reference the shapes.

Next, I want to see how the shapes look using the `.plot()` method:


```python
# nb: the first time I ran this, it required installation of the descartes module
us_states.plot(figsize=(20,20))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa34af34b90>




![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_13_1.png)


The proportions look a bit strange with all of that whitespace on the right, but on a close inspection I can see at the far right that there are a few of the Aleutian Islands flowing across the dateline. See the small blue dots over to the right? Very small, but they are there! Other than that... pretty cool! This is what I wanted!

Now I'll refine this a bit more to show the continental US... (with apologies, but I am leaving the display of Alaska, Hawaii, and Puerto Rico for a future project).


```python
# for illustration, exclude Alaska & Hawaii & Puerto Rico
continental = us_states[us_states['NAME'].isin(['Alaska','Hawaii', 'Puerto Rico']) == False]

continental.plot(figsize=(30,20), color='#d1b26f')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa34af62090>




![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_15_1.png)


That looks good! Note above that I used the geopandas function `.isin()` above to filter out any shapes that did not appear in the list using a boolean filter ("False"). 

Similarly, I can "zoom in" and look at just a single state. Again using the `.isin()` function, but this time the opposite boolean value is set to "True":


```python
# display shape for a single state

hawaii = us_states[us_states['NAME'].isin(['Hawaii']) == True]

hawaii.plot(figsize=(30,20), color='#d1b26f')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7fa34af89a10>




![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_17_1.png)


Yes, that looks like Hawaii!! And note that it's a great example of a "multipolygon" state :smile: 

Now, let's plot the coordinates on these shapes...

## Plot the points in the states

Now that I can draw the states, I want to plot the grant point data! This section begins using the `matplotlib` module for expanded visualization capabilities, such as the inclusion of a title and setting colors.

### Map the grants in Minnesota

For example, to display the grants in one state, I can use filtering to pull grants given to the state of Minnesota, and likewise filter to display the Minnesota state shape. Note I have added the "figure" (figure) and "axis" (ax) components, which the visualization library is using to render different parts of the image. I have also set colors in the arguments provided to the `.plot()` functions.


```python
# set the desired data
state = 'Minnesota'
Minnesota_1960s_grants = gdf_neh_1960s[gdf_neh_1960s['InstState'] == 'MN']
```


```python
# plt to see the points on the map
fig, ax = plt.subplots(1, figsize=(30,20))
base = us_states[us_states['NAME'].isin([state]) == True].plot(ax=ax, color='blue')

# plot the positions
Minnesota_1960s_grants.plot(ax=base, color='yellow')

plt.show()
```


![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_21_0.png)


Woohoo! This is the first thing that really looks like a map! A clear, visually simple, graphic that combines the geographic shape information with the grant data from the NEH to show where the money went!

### Map the grants in any US state

Now, let's make this more dynamic: create options that allow for easier generation of maps for any state. 

First, set map parameters to select a state (lines 2 and 3), filter the data based on the selections (line 6), 
draw out some basic information that can be used in the graphic (lines 8 through 32), then plot the selected data (line 34 and following). I've used pandas filters to draw out various information, like a starting year and ending year, to determine the number of grants that are shown in the image, to sum the dollar amounts of the awards, and to specify colors for the output; most of this information is bundled into the `displayInfo` dictionary.
On line 42, I used python's string `format` substitution methods with some text filtering ([see here for a guide from RealPython](https://realpython.com/python-formatted-output/)), to provide some well-formatted information for the image title and legend.


```python
# set the state info - to map other states, change the next two lines
stateAbbrev = 'MI' #use standardized 2-letter postal abbreviations
state = 'Michigan' #use full name of state written out with spaces and no diacritics

# filter the data
state_1960s_grants = gdf_neh_1960s[gdf_neh_1960s['InstState'] == stateAbbrev]

# get label info
#start year
startYear = state_1960s_grants['YearAwarded'].min()
#end year
endYear = state_1960s_grants['YearAwarded'].max()
#number of awards
numGrants = state_1960s_grants['AppNumber'].count()
#dollars awarded
totalOutright = state_1960s_grants['AwardOutright'].sum()
#map shape color
map_color = '#d1b26f'
#point color
point_color = '#2C4B85'

# create a bundle of info for display rendering
displayInfo = {
    'startYear' : startYear,
    'endYear' : endYear,
    'numGrants' : numGrants,
    'outrightDollars': totalOutright,
    'map_color' : map_color,
    'point_color' : point_color,
    'state' : state,
    'abbrev' : stateAbbrev
}

# plt to see the points on the map
fig, ax = plt.subplots(1, figsize=(30,20))
base = us_states[us_states['NAME'].isin([state]) == True].plot(ax=ax, color=map_color)

# plot the positions
state_1960s_grants.plot(ax=base, color=point_color, legend=True)

# add title and legends
plt.title('NEH Grants awarded in {0[state]}, {0[startYear]}-{0[endYear]} ({0[numGrants]} awards, ${0[outrightDollars]:,.2f})'.format(displayInfo), fontfamily=['Georgia','serif'], fontweight='bold', fontsize='x-large') # title uses some advanced string formatting to display the total $ amounts as a currency display
plt.legend(['Indicates one award (points may overlap)'])
lims = plt.axis('equal') # not sure what this is actually doing, although some states seem less 'squished'

plt.show()
```


![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_24_0.png)


The map looks a bit squashed top to bottom, but aside from that, this is a great start!

### Map the grants in multiple states

Now, what if I want to map the grants from more than one state? One of my goals is to map the points of all the grants in the continental US. Using similar filtering functiosn to those I used above (`.isin()` and boolean filters), I can exclude the data that I don't want. Note the specialized use of the `~` character here, in line 5, which reverses the filter, effectively displaying anything that is "False," that is to say it will return all the values not in the exclude set.

(In this case, my desired visualization focused on many states, so it was easy to filter out a handful. If you are working with a smaller group of states, an inclusive filtering approach might work better.)


```python
exclude = ['Alaska','Hawaii', 'Puerto Rico']
excludeAbbrev = ['AK','HI','PR']

# filter the data
state_grant_info = gdf_neh_1960s[~gdf_neh_1960s.InstState.isin(excludeAbbrev)]

# basic summary information for title
#number of awards
numGrants = state_grant_info['AppNumber'].count()
#dollars awarded
totalOutright = state_grant_info['AwardOutright'].sum()

#plot the map & points
fig, ax = plt.subplots(1, figsize=(30,20))

# filter the base map
base = us_states[us_states['NAME'].isin(exclude) == False].plot(ax=ax, color='#d1b26f')

# plot the positions
state_grant_info.plot(ax=base, color='#2C4B85')

plt.title('NEH Grants awarded in the continental United States, 1966-1969 ({0} awards, ${1:,.2f})'.format(numGrants, totalOutright), fontfamily=['Georgia','serif'], fontweight='bold', fontsize='x-large')
plt.show()
```


![png](/assets/images/wrangling-humanities-data/mapping-by-state/output_27_0.png)


And there we have it! A map of the continental US, with the location of each NEH grant recipient of the 1960s displayed! 

There are still a few more things that might be useful, including drawing in Alaska, Hawaii, and Puerto Rico, which also received grants during this time. Additionally, the NEH does make awards to other entities, including Guam, the US Virgin Islands, and American Samoa, so there could be a lot of additional tweaking necessary. 

Since everyone is looking at things on the web and on their phones these days, an interactive "slippy" web map would also be nice for display. Those are tasks that I may explore in future installments. For the purposes of this demonstration, however, the map is ready!

## Reference list

Credit to the examples in these tutorials and projects (as of January 2021), which were highly informative to the exploratory work outlined above:

* Duong Vu, "[Introduction to Geospatial Data in Python](http://datacamp.com/community/tutorials/geospatial-data-python)" at _Datacamp_, published 24 October 2018.
* Lesley Cordero, "[Getting Started on Geospatial Analysis with Python, GeoJSON and GeoPandas](https://www.twilio.com/blog/2017/08/geospatial-analysis-python-geojson-geopandas.html)" at Twilio, published 14 August 2017.
* Dani Arribas-Bel, "[Mapping in Python with geopandas](http://darribas.org/gds15/content/labs/lab_03.html)", from Geographic Data Science '15. Provides a lot of useful information about creating more complex plots.

See this site for US state shapefile information: 

* Eric Celeste http://eric.clst.org/tech/usgeojson

### Interesting mapping projects with historical data
* Selena Qian, "[Sanborn Maps Navigator](https://github.com/selenaqian/sanborn-maps-navigator)", example of creating a map interface to explore an inventory of the Sanborn maps at the Library of Congress.
* USGS historical atlas explorer https://livingatlas.arcgis.com/topoexplorer/index.html
* "[Keweenaw Time Traveler](https://www.keweenawhistory.com/)," an interactive historical GIS project from the Historic Environments and Spatial Analytics Lab at Michigan Technological University.
