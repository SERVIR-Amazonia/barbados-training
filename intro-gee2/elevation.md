---
layout: page
title: Retrieving and visualizing elevation data
parent: Introduction to Google Earth Engine 2
nav_order: 5
---

# Retrieving and visualizing elevation data

*Keep the previous script open from 'Processing & Cloud Masking Sentinel', we will continue in this section*

This exercise will walk us to the use of the NASADEM elevation dataset. Let’s remember that each product description in the GEE catalog usually brings a small portion of code to exemplify how to use it. We look for the product:

<img align="center" src="../images/intro-gee-images/24_nasadem.png" hspace="15" vspace="10" width="600">

Figure 24. NASA Digital Elevation Model (DEM) listed options

The first product listed must be imported, and change the name to '*nasadem*'.

<img align="center" src="../images/intro-gee-images/25_nasadem_desc.png" hspace="15" vspace="10" width="600">

Figure 25. NASA DEM description

It’s always important to skim over the dataset product details, such as time availability, spatial scope, significant improvements over other options, and bands.

<img align="center" src="../images/intro-gee-images/26_dem_var.png" hspace="15" vspace="10" width="600">

Figure 26. Product added

```javascript
//	NASA Digital Elevation Model (DEM)  
var elevation = nasadem.select('elevation');
// Set elevation <= 0 as transparent and add to the map.
elevation = elevation.updateMask(elevation.gt(0)).clip(barbados_bou);


// Set elevation visualization properties.
var elevationVis = {
  min: 0,
  max: 400,
};

Map.addLayer(elevation, elevationVis, 'Elevation');
```

Elevations in the interior range from 180 to 240 meters above sea level. Mount Hillaby is the highest point at 340 meters above sea level. [Source](https://en.wikipedia.org/wiki/Geography_of_Barbados)

<img align="center" src="../images/intro-gee-images/mount_hillaby.jpg" hspace="15" vspace="10" width="600">

Figure 27. Mount Hillaby, Barbados. [Source](https://premierhotelier.files.wordpress.com/2014/08/jj2-3.jpg)

```javascript
// Around Mt Hillaby
var p01 = ee.Geometry.Point(-59.58463, 13.2018)

Map.addLayer(p01, {color:'red'}, 'point');
Map.centerObject(p01,12)
```

<img align="center" src="../images/intro-gee-images/28_elev_mount.png" hspace="15" vspace="10" width="600">

Figure 28. DEM layer and the location over El Cerro Del Aripo mountain

Black and gray colors correspond to lowlands and plains, while white and bright pixels correspond to highlands and mountain peaks. We are going to use the Inspector tool.  Click on it, and then we click on the point we added in order to see the elevation value we have there.

<img align="center" src="../images/intro-gee-images/29_insp.png" hspace="15" vspace="10" width="600">

Figure 29. Inspector panel.

Hence, the values from all the add layers will be visible for the specified geographical location.

<img align="center" src="../images/intro-gee-images/30_insp_values.png" hspace="15" vspace="10" width="300">

Figure 30. Location values

In the Inspector panel we will see the values for the layers we have added by code, either activated or deactivated. First, we can identify low surface reflectance values from the Sentinel layer. Second, we can observe that the elevation at this location is 901 meters. Let’s do a quick exercise of visualization. The default view of the map visor 

<img align="center" src="../images/intro-gee-images/31_mode.png" hspace="15" vspace="10" width="300">

Figure 31. Visualization mode.

We can make use of the high-resolution images available in a true color setting to visualize the area and try to identify mountainous zones.

<img align="center" src="../images/intro-gee-images/32_basemap.png" hspace="15" vspace="10" width="600">

Figure 32. Visualization using high-resolution basemaps of GEE

We can also use the Map-terrain view to analyze elevation by contour lines

<img align="center" src="../images/intro-gee-images/33_terrain.png" hspace="15" vspace="10" width="300">

Figure 33. Map terrain view

<img align="center" src="../images/intro-gee-images/34_terrain_view.png" hspace="15" vspace="10" width="600">

Figure 34. Terrain view

We can easily recognize our point of reference located at the 900 m contour line.

Code Checkpoint: [https://code.earthengine.google.com/6e6dfe82c96ad645548d92f3bd09d40f](https://code.earthengine.google.com/6e6dfe82c96ad645548d92f3bd09d40f)