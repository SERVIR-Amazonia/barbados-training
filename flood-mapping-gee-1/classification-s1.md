---
layout: page
title: Mapping Flooded Areas using SAR
parent: Flood Mapping Using Multiple Sensors in GEE 2
nav_order: 2
---

# Overview

*Create a new script for this workflow*

In this session we will train a classifer to predict flooded land covers using Sentinel1 SAR scenes collected in both dry and wet seasons.

**General Supervised Classification Workflow**

Step 1. Load the image or images to be classified 

Step 2. Gather the training data: 
  * Collect training data to teach the classifier 
  * Collect representative samples of backscatter for each landcover class of interest 

Step 3. Create the training dataset: 
  * Overlay the training areas over the images of interest 
  * Extract backscatter for those areas 

Step 4. Train the classifier and run the classification 

Step 5. Validate your results

**Identify Area of Interest**

We will utilize the geometry drawing tools in the **Map** window to draw an area of interest. 

In the top-left of the Map window, select the Polygon draw button, then create your polygon. 

At the top of your script, rename it from 'geometry' to 'aoi'.

<img align="center" src="../images/flood-mapping-gee/07draw.PNG" hspace="15" vspace="10" width="600">

**Prepare SAR Data**

Load the Sentinel-1 C-band SAR Ground Range dataset, filtering it based on several metadata items and your AOI.

```js
var collection = ee.ImageCollection('COPERNICUS/S1_GRD') 
.filter(ee.Filter.eq('instrumentMode', 'IW'))
.filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'))
.filterMetadata('resolution_meters','equals',10)
.filterBounds(aoi)
.select('VV','VH')
```

We want to retrieve one S1 scene each from the wet season and the dry season. For each period, we will define a start and end date with which to filter the whole Sentinel S1 collection, select the first scene from the result and clip it to our AOI.

```js
//collect one observation each of dry and wet period SAR data
var dry = collection.filterDate('2022-03-01','2022-03-30').first().clip(aoi);
var wet = collection.filterDate('2022-08-01','2022-08-30').first().clip(aoi);

// Display on map
var vis = {bands:['VV'],min:-20,max:0};
Map.centerObject(aoi,12);
Map.addLayer(dry,vis,'dry season',false);
Map.addLayer(wet,vis,'wet season ',false);
```

<img align="center" src="../images/flood-mapping-gee/08dryWet.PNG" hspace="15" vspace="10" width="600">

To reduce the salt-and-pepper effect on our eventual product, we apply a smoothing function across each scene with a focal window. Toggle between the original SAR scene and the speckle filtered result to see the effect.

```js
// apply speckle filter
var smoothingRadius = 50;
var dryFiltered = dry.focal_mean(smoothingRadius,'circle','meters');
var wetFiltered = wet.focal_mean(smoothingRadius,'circle','meters');

// display filtered images
Map.addLayer(dryFiltered, vis,'dry season - speckle filter',false);
Map.addLayer(wetFiltered, vis,'wet season - speckle filter',false);
```

<img align="center" src="../images/flood-mapping-gee/09filter.PNG" hspace="15" vspace="10" width="600">

**Creating Reference Data**

To run a supervised classification, we must collect reference data to "train" the classifier. This involves collecting representative samples of SAR backscatter data for each map class of interest.

Using the Satellite basemap and your wet and dry season SAR layers in your map, we will draw representative polygons for several classes: Open Permanent Water, Flooded Vegetation, Urban/Built Area, and Forest.

Below is the workflow for creating reference data directly in Earth Engine. We will use Open Permanent Water as the example.

* In the geometry drawing toolbar (top-left of the **Map** panel), go to Geometry Imports and click 'new layer'.

<img align="center" src="../images/flood-mapping-gee/10geom.PNG" hspace="15" vspace="10" width="600">

* Click on the 'Edit layer properties' button (cog icon next to the geometry's name) to configure the new geometry layer. Name the new layer 'openWater' and change its color. Under 'Import as', change it to 'FeatureCollection'. Finally, click on the '+ Property' button and enter a new property 'landcover' with a value of 0.  

<img align="center" src="../images/flood-mapping-gee/11configure.PNG" hspace="15" vspace="10" width="600">

Each geometry layer, imported to your script now as a `FeatureCollection` will represent one class within your map product. 

* Draw representative polygons on the map. Click again on the geometry layer 'openWater' in the **Geometry Imports** panel and ensure it shows up in bold text. Then click on the polygon icon. Draw your open water polygons. 

<img align="center" src="../images/flood-mapping-gee/12openWater.PNG" hspace="15" vspace="10" width="600">

Repeat these steps for each of the remaining map classes: Flooded Vegetation, Urban/Built Area, and Forest. 

**Remember**: since each geometry layer represents its own map class, the value for the 'landcover' property must change when you are at the configuration step. For example, use landcover value of 1 for the next geometry layer that you create. 

To get through this demonstration, shoot for 3 or 4 large polygons or 7 or 8 smaller ones per class. We can refine the quality of our reference data later if we have time.

**Generate Training and Testing Data**

Now that we have reference polygons for our four map classes, we will merge their `FeatureCollections` into one. Be mindful of the names that you gave to each polygon feature collection for this next line of code. 

```js
// Merge training FeatureCollections
var newFc = openWater.merge(floodedVegetation).merge(urban).merge(forest);
```

Code Checkpoint: [https://code.earthengine.google.com/03c9175f18ca5d47e558e401f5280896](https://code.earthengine.google.com/03c9175f18ca5d47e558e401f5280896)

Next, we will use the merged `FeatureCollection` of reference polygons to extract the SAR backscatter pixel values for each landcover. The polygons within the `newFc` `FeatureCollection` are overlaid on the image, and each pixel is converted to a point containing the image's pixel values and the other properties inherited from the polygon (in our case 'landcover' property). After you run this, note in the **Console** the total size of reference points we now have to train and validate our map. 

```js
// define bands to be used to train the data
var bands = ['dryVV','dryVH','wetVV','wetVH'];

// combine dry and wet season composite SAR data
var final = ee.Image.cat(dryFiltered,wetFiltered)
.rename(['dryVV','dryVH','wetVV','wetVH']);
print('combined dry/wet season SAR data', final);

var samples = final.select(bands).sampleRegions({
  collection:newFc,
  properties:['landcover'],
  scale:30});

print('Sample size: ',samples.size());
```

Next, we want to set aside 80% of the reference points to train our classifier, and use the remaining 20% for validation. We do this simply by adding a new column to the reference point set called 'random', which contains random decimal numbers between 0 and 1, then filtering them into the two groups.

```js
//split sample data into training and testing
var trainTest = samples.randomColumn();
print('First Train/Test Sample', trainTest.first());

var training = trainTest.filter(ee.Filter.lte('random',0.8));
var testing = trainTest.filter(ee.Filter.gt('random',0.8));
print('training size: ', training.size());
print('testing size: ', testing.size());
```

<img align="center" src="../images/flood-mapping-gee/13trainSize.PNG" hspace="15" vspace="10" width="600">

**Create and Apply Classifier**

We choose a classifier algorithm from `ee.Classifier` family of functions, and train it on the `training` `FeatureCollection`. We must specify the `classProperty` to be the property we want to map, or predict, with the model, and the `inputProperties` as the SAR backscatter values defined by the `bands` variable. 

```js
// train the classifier
var classifier = ee.Classifier.smileCart().train({
  features:training,
  classProperty: 'landcover',
  inputProperties:bands
});
```

Running the classification is as easy as selecting the desired bands from the `final` image (recall it is the dry season scene and wet season scene together), and calling `classify()`.

```js
// run the classification
var classified = final.select(bands).classify(classifier)
```

Display the results, modify the color palette if needed. 

```js
// Display the results, adjust colors according to your land cover stratification
var classVis = {min:0,max:3,palette:['blue','cyan','red','green']}
Map.centerObject(classified,12)
Map.addLayer(classified,classVis,'classified')
```

Right away we can pick out where some features are mixed up between classes, and where the classifier excels. Let's assess the classifier's performance quantitatively now.

<img align="center" src="../images/flood-mapping-gee/14classif.PNG" hspace="15" vspace="10" width="600">

**Accuracy Assessment**

First, lets see how the classifier performed on the training data.

```js
// Compute training accuracy 
print('RF error matrix [Training Set]: ', classifier.confusionMatrix());
print('RF accuracy [Training Set]: ', classifier.confusionMatrix().accuracy());
```

<img align="center" src="../images/flood-mapping-gee/15trainAccuracy.PNG" hspace="15" vspace="10" width="600">

100% accuracy on the training set sounds impressive but is not a true representation of the classifier's performance since it is the data the model saw during training.

To represent a truer picture of the classifier's performance, we can use the 20% of the samples witheld for testing. 

```js
// classify test set with classifier trained on training set
var classificationVal = testing.classify(classifier);
print('Classified points', classificationVal.limit(5));

// Create confusion matrix.
var confusionMatrix = classificationVal.errorMatrix({
  actual: 'landcover', 
  predicted: 'classification'
});

// Print Confusion Matrix and accuracies.
print('Confusion matrix [Testing Set]:', confusionMatrix);
print('Overall Accuracy [Testing Set]:', confusionMatrix.accuracy());
print('Producers Accuracy [Testing Set]:', confusionMatrix.producersAccuracy());
print('Users Accuracy [Testing Set]:', confusionMatrix.consumersAccuracy());
```

<img align="center" src="../images/flood-mapping-gee/16testAccuracy.PNG" hspace="15" vspace="10" width="600">

An Overall Accuracy (OA) above 90% is quite good, but note that it is the average of all class-specific accuracies. In our case, the OA is inflated by a very high Open Water class accuracy, effectively masking the much lower accuracies of the other classes.

For this methodology to hold up to scientific scrutiny, we would also want to collect a completely independent set of validation points outside this exercise to assess the resulting map with. A platform like [Collect Earth Online](https://www.collect.earth/) is great for this application. Since this is just a demonstration we will move on. 

**Deploy Classifier to new Observations**

Now that we have a trained model, we could _deploy_ it, or apply it to other Sentinel 1 SAR observations quite easily in Earth Engine. Change the start and end date in the `.filterDate` call below to any date range starting in 2015 (Sentinel1 data availability). Remember that the `first()` call is taking the first SAR image within that date range.

```js
// Classify another Sentinel SAR observation
// one SAR observation from dry season
var newObsDry = collection.filterDate('2018-03-01','2018-03-31').first().clip(aoi);
// one SAR observation from wet season
var newObsWet = collection.filterDate('2018-08-01','2018-08-31').first().clip(aoi);

// combine the two season observations
var newObsFinal = ee.Image.cat(newObsDry,newObsWet).rename(bands);
print('New Obs Final', newObsFinal);

var newObsFiltered = newObsFinal.focal_mean(smoothingRadius,'circle','meters');
var newObsClassified = newObsFiltered.select(bands).classify(classifier)
```

Code Checkpoint: [https://code.earthengine.google.com/8d406f19f6226df89ecbb4b77c88364a](https://code.earthengine.google.com/8d406f19f6226df89ecbb4b77c88364a)

<img align="center" src="../images/flood-mapping-gee/17newObsClassif.PNG" hspace="15" vspace="10" width="600">

A pre-trained model such as this one could be useful in deriving insights from new satellite observations quickly. Flood monitoring applications like [HydraFloods](https://hydrafloods-servir.adpc.net/) works this way, with just a few more bells and whistles. 
