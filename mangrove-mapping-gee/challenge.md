---
layout: page
title: Challenges - Make Your Own Model, Make Improvements
parent: Mangrove Mapping Using Multiple Sensors
nav_order: 4
---

# Overview

You've successfully written a remote sensing workflow to train a Random Forest classification model to map Mangrove presence with the Landsat archive. Now make it your own! There are several ways to tweak the original workflow right away to change the objective and make improvments. Here are a few ideas. Collaborate with your colleagues and your instructors. Happy Coding!

## Area of Interest (AOI)

We defined our AOI at the beginning of the script using FAO GAUL data. As discussed previously you can define an AOI in GEE in many other ways. Review Step-Through Part 1 section to get some ideas. 

## Date Range

After merging together our full Landsat `ImageCollection`, we filtered it on a date range (i.e. 2000-2005). Come up with another date range that you'd like to map mangroves for. Remember that we can also specify a Day of Year range in our filters on the `ImageCollection`. This is useful for areas where there is heavy cloud cover during a certain time of year, or the phenomena you want to map exhibits a specific seasonal pattern that we can sense from satellite data.

## Model Structure

There are several ways we could make our RF model more robust... 

The first is in the number of 'trees' in the Random Forest. You can change that in the `ee.Classifier.smileRandomForest()` function and observe whether any of the accuracy metrics have improved. 

## Sample Data

Beyond the model structure itself, we can also provide more and/or better reference data to the model. The first improvement would be to increase the amount of total samples. Try a number between 200 and 500 per class. 

To provide better quality reference data, we can look for another source of Mangrove presence/absence data, or make our own. Making your own will take time and expertise. The Institute of Marine Affairs has produced mangrove extent polygons circa 2014 and 2020. 

<img align="center" src="../images/mangrove-mapping/IMApolysDesc.PNG" hspace="15" vspace="10" width="600">

Paste this code below in a new script to check out the data.

```js
var mangTob2014 = ee.FeatureCollection("projects/caribbean-trainings/assets/trinidad-tobago-2022/vector/mangrovesTobago2014");
var mangTob2020 = ee.FeatureCollection("projects/caribbean-trainings/assets/trinidad-tobago-2022/vector/mangrovesTobago2020");
var mangTrin2014 = ee.FeatureCollection("projects/caribbean-trainings/assets/trinidad-tobago-2022/vector/mangrovesTrinidad2014");
var mangTrin2020 = ee.FeatureCollection("projects/caribbean-trainings/assets/trinidad-tobago-2022/vector/mangrovesTrinidad2020");

Map.centerObject(mangTrin2014,9);
Map.addLayer(mangTob2014, {color:'red'}, 'Mangroves Tobago 2014');
Map.addLayer(mangTob2020, {color:'blue'}, 'Mangroves Tobago 2020');
Map.addLayer(mangTrin2014, {color:'red'}, 'Mangroves Trinidad 2014');
Map.addLayer(mangTrin2020, {color:'blue'}, 'Mangroves Trinidad 2020');
```

How could you replace the Giri et al 2011 data in the workflow with this other data source?

## Happy Coding!
Do some experimentation, collaborate with your colleagues, ask your instructor questions - Good luck!

*Tip*: The Docs tab is your friend. If you see a function is being used in the script, find that function in the Docs to see what the required arguments are. It'll help you understand how to change the pre-existing functionality. 

<img align="center" src="../images/mangrove-mapping/Docs.PNG" hspace="15" vspace="10" width="600">
