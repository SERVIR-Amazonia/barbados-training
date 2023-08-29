---
layout: page
title: Map Validation with CEO
parent: "Map Validation with Collect Earth Online"
nav_order: 6
---

# Sample-Based Map Validation and Area Estimation with CEO

## Overview

In this workflow, we will create a stratified random sample of a land cover map, and use to conduct an accuracy assessment and area estimate of the strata in the land cover map.  We will make the stratified random sample in GEE, collect validation data at those points in CEO, and then calculate error matrices and estimate the true area of each land cover stratum in Google Sheets or Microsoft Excel.

# Create Classification Map in GEE

In the [Advanced Google Earth Engine - Change Detection 1](https://servir-amazonia.github.io/suriname-training/change-detection-1) module, we made a random forest land cover classification for two years.  Here, we will write a short snippet of code to export our land cover classification for 2022 (Year 2) as a GEE Asset.

Open your script titled **Change Detection - Two Date**, insert into the script, and run the script.

```javascript
//--------------------------------------------------------------
// Export Classifications
//--------------------------------------------------------------

// Export 2022 classified map as a GEE Asset to use for further analysis
// change the location to your root GEE folder
Export.image.toAsset({
  image: RFclassification_y2,
  description: 'toasset_LCclassification2022_Suriname',
  assetId: 'users/ebihari/LCclassification2022_Suriname',
  region: aoi,
  scale: 30,
  crs:'EPSG:4326',
  maxPixels: 1e13
});
```

Now, when we run our code, this export task should appear in the **Tasks** tab.  

<img align="center" src="../images/ceo/export_task.png" hspace="15" vspace="10" width="400">

Click **Run** on the task, filling out the names and locations you want the file to be put.

<img align="center" src="../images/ceo/export_run.png" hspace="15" vspace="10" width="400">

# Create Validation Points in GEE

In the new GEE script you created called **Map Validation - Sample Design**, import the land cover classification asset.  I have made land cover classification asset public, so you can use mine one `users/ebihari/LCclassification2022_Suriname` or your own.

```javascript
//--------------------------------------------------------------
// Import Classification Map
//--------------------------------------------------------------

// import land cover classification from previous activity 
var LC = ee.Image('users/ebihari/LCclassification2022_Suriname')

// add classified image to map
Map.addLayer(LC, {min: 1, max: 5, 
            palette: ['00a661','00a0cd','a40000', 'd5ff6a','d3d2cb']}, 
            'RF landcover classification');
```

Then, we create a stratified random sample with 10 points in each of the 5 land cover classes.  In a real project, you would want to collect many more validation points for this big of an AOI, but we will keep it simple for this exercise.

```javascript
//--------------------------------------------------------------
// Create Stratified Random Sample
//--------------------------------------------------------------

// create stratified random sample from the land cover classification map
// values in 'class' property: 
// 1: 'forest',
// 2: 'water',
// 3: 'urban',
// 4: 'agriculture',
// 5: 'bare soil'
var samplePts = LC.stratifiedSample({
  // total # points
  numPoints:50, 
  classBand:'classification', 
  region:LC.geometry(), 
  scale:30, 
  projection:'EPSG:4326', 
  seed:1010, 
  classValues:[1,2,3,4,5],
  // # points in each class
  classPoints:[10,10,10,10,10], 
  dropNulls:true, 
  tileScale:2, 
  geometries:true});
print('Sample points:', samplePts);

// Add points to the map
Map.addLayer(samplePts,{},'sample points');
```

<img align="center" src="../images/ceo/validationpoints.png" hspace="15" vspace="10" width="600">

Now, we export these sample points as a Google Drive file in .csv format.  We need to do some extra formatting to get the data in a format that CEO and SEPAL will accept.  We need to add columns called `PLOTID`, `LON`, and `LAT`, `SAMPLEID`, and `classification_readable` (which is just a column for the classification values in a readable format ("forest", "water", etc.), and will come in handy when doing our final analysis).

```javascript
//--------------------------------------------------------------
// Export Stratified Random Sample
//--------------------------------------------------------------

// create a dictionary to store class names
var classLookup = ee.Dictionary({
  1: 'forest',
  2: 'water',
  3: 'urban',
  4: 'agriculture',
  5: 'bare soil'
});

// write a function to rename the columns 
// so that the format is accepted by CEO
function ceoClean(f){
  // LON,LAT,PLOTID,SAMPLEID, readable classification column
  var fid = f.id();
  var coords = f.geometry().coordinates();
  var classValue = f.get('classification');
  var className = classLookup.get(classValue);
  return f.set('LON',coords.get(0),
               'LAT',coords.get(1),
               'PLOTID',fid,
               'SAMPLEID',fid,
               'classification_readable', className);
}
// print("First sample point:",ee.Feature(samplePts.map(ceoClean).first()));

// map that function to the sample points feature collection
var samplePts_CEO = samplePts.map(ceoClean);

print('Sample points with PLOTID and readable class:', samplePts_CEO);

print("Class labels and number of samples:",
      samplePts_CEO.aggregate_histogram('classification_readable'));

// Export points to Google Drive to use in further analysis
Export.table.toDrive({
  collection: samplePts_CEO,
  description: 'todrive_LCsamplepoints2022_Suriname',
  fileNamePrefix: 'LCsamplepoints2022_Suriname',
  selectors: 'LON,LAT,PLOTID,SAMPLEID,classification,classification_readable'
});
```

Additionally, we need to calculate class areas based on the map to use later in our later analysis. This block of code uses the `ee.Reducer.count()` function to count the number of pixels in each class.  It then stores these values in a `List` of `Dictionaries`.  Each `Dictionary` in this `List` has 2 keys: the `class` and the `count`.  Last, it uses this `List` to create a new `FeatureCollection` with 5 empty features in it (one for each class), and attaches the correct `class` and `count` as properties to each feature.  Thus, we end up with a data type that can be exported as a .csv, and it contains a separate row for each class and a separate column containing the pixel counts of those classes.

``` javascript
//--------------------------------------------------------------
// Calculate Class Areas
//--------------------------------------------------------------

// Create an extra image with a value of 1 in every pixel
// covering the same geometry as the land cover map
// (this is just an something needed for the ee.Reducer.group() function 
// because it needs an image with 2 bands)
var constantImage = ee.Image(1).clip(LC.geometry());

// Calculate the number of pixels in each class
var pixelCounts_dict = constantImage.addBands(LC).reduceRegion({
  // Use ee.Reducer.count().group() function
  reducer: ee.Reducer.count().group({
    // Use band "class" for grouping 
    // (which is now band 1 not 0 since we attached the extra image)
    groupField: 1, 
    groupName: 'class',
  }),
  geometry: LC.geometry(), // region of interest
  scale: 30, // Resolution of the image in meters
  maxPixels: 1e50 // Maximum number of pixels in the region
});

// Print the resulting dictionary of a list of dictionaries
print("Pixel Counts as Dictionary", pixelCounts_dict);

// Convert the this dictionary to just a list of dictionaries
// (we extract the list of dictionaries)
var pixelCounts_list = ee.List(pixelCounts_dict.get('groups'));

// Print the resulting list of dictionaries
print("Pixel Counts as List of Dictionaries", pixelCounts_list);

// Create a FeatureCollection from the list of dictionaries
var pixelCounts_fc = ee.FeatureCollection(
  // apply this function to all the dictionaries in the list
  pixelCounts_list.map(function(dict) {
    var dict2 = ee.Dictionary(dict);
    // extract the className from the dictionary
    var className = ee.Number(dict2.get('class'));
    // extract the count from the dictionary
    var count = ee.Number(dict2.get('count'));
    // put these into an empty feature class as new properties
    // (this allows us to export the info as separate columns in a CSV)
    return ee.Feature(null, {
      'class': className,
      'count': count
    });
  })
);

// Print the resulting FeatureCollection
print("Pixel Counts as FeatureCollection", pixelCounts_fc);

//--------------------------------------------------------------
// Export Class Areas
//--------------------------------------------------------------

// Export the FeatureCollection as a CSV to Google Drive for further analysis
Export.table.toDrive({
  collection: pixelCounts_fc,
  description: 'todrive_LCpixelcounts2022_Suriname',
  fileNamePrefix: 'LCpixelcounts2022_Suriname',
  fileFormat: 'CSV'
});
```

Now, when we run our code, these export tasks should appear in the **Tasks** tab.  

<img align="center" src="../images/ceo/export_task3.png" hspace="15" vspace="10" width="400">

Click **Run** on the task, filling out the names and locations you want the files to be put.

<img align="center" src="../images/ceo/export_run2.png" hspace="15" vspace="10" width="400">

Once the tasks have finished running, we can open the exported .csv files in Google Drive by clicking **Open in Drive** in the **Tasks** tab.

<img align="center" src="../images/ceo/export_open2.png" hspace="15" vspace="10" width="400">

From your Google Drive, download these .csv files to your computer.

*Extra: Add legend.*

``` javascript
//--------------------------------------------------------------
// Add Legend
//--------------------------------------------------------------

// Set position of panel
var legend = ui.Panel({
  style: {
    position: 'bottom-right',
    padding: '8px 15px'
  }
});

// Create legend title
var legendTitle = ui.Label({
  value: 'Land Cover Classes - 2022',
  style: {
    fontWeight: 'bold',
    fontSize: '18px',
    margin: '0 0 4px 0',
    padding: '0'
    }
});

// Add title to legend panel
legend.add(legendTitle);
    
// Write a fcuntion that creates and styles 1 row of the legend
var makeRow = function(color, name) {
      
      // Create the label that is actually the colored box
      var colorBox = ui.Label({
        style: {
          backgroundColor: '#' + color,
          // Use padding to give the box height and width
          padding: '8px',
          margin: '0 0 4px 0'
        }
      });
      
      // Create the label that is the description text
      var description = ui.Label({
        value: name,
        style: {margin: '0 0 4px 6px'}
      });
      
      // Return the panel
      return ui.Panel({
        widgets: [colorBox, description],
        layout: ui.Panel.Layout.Flow('horizontal')
      });
};

// list with the class colors
var palette = visParam.palette;

// list with class names
var names = ['forest','water','urban','agriculture','bare soil'];


// Add color and and names to legend panel
for (var i = 0; i < 5; i++) {
  legend.add(makeRow(palette[i], names[i]));
  }  

// Add legend to map (alternatively you can also print the legend to the console)  
Map.add(legend);  
```

Code Checkpoint: [https://code.earthengine.google.com/?scriptPath=users%2Febihari%2FSurinameWS%3AMap%20Validation%20-%20Sample%20Design](https://code.earthengine.google.com/?scriptPath=users%2Febihari%2FSurinameWS%3AMap%20Validation%20-%20Sample%20Design)

# Collect Validation Data in CEO

Log in to CEO.  On the main CEO page, in the search bar at the top left, search for an institution called “Suriname Geospatial Workshop.” Click `Visit`.

<img align="center" src="../images/ceo/CEO_homepage.png" hspace="15" vspace="10" width="600">

## Add Imagery to the CEO Institution

On the institution's home page, click on `Imagery`.

<img align="center" src="../images/ceo/CEO_imagery.png" hspace="15" vspace="10" width="700">

Click on the `edit` button for the last imagery on the page called "Global Mangrove Forests Distribution".  Here, you can see how to add a new type of imagery to a project.  There are some data sets already available in CEO, like Sentinel and Planet, but you can also import any public GEE `Image` or `ImageCollection` or any private GEE asset.  You just need its asset ID, a start and end date, and some parameters for its visualization.

<img align="center" src="../images/ceo/CEO_imagery2.png" hspace="15" vspace="10" width="700">

## Create a CEO Project

On the institution’s home page, go to the `Projects` tab and click `+ Create New Project`.  The workflow for creating a new project should appear. 

<img align="center" src="../images/ceo/CEO_projectpage.png" hspace="15" vspace="10" width="700">

On this first `Project Overview` page, under `Select Template`, select the `Suriname land cover map validation` project that is already present in the institution, and click `Load`.  All of the project parameters should now be identical to the project that has already been created.

You can also create a project from scratch, but for the sake of simplicity, we will use this project template that has already been made for you.  If you want to model a CEO project off of another project but create entirely new plots/samples or survey questions, you can uncheck `Copy Template Plots and Samples` and `Copy Template Widgets`.

Then, **add YOUR NAME to the end of the project name**.  This way, everyone in the workshop will have their own project to work in.  

<img align="center" src="../images/ceo/CEO_newproject.png" hspace="15" vspace="10" width="700">

Click `Next`.

On the `Imagery Selection` page, you can change the imagery that will be available when collecting data.  You will see the default CEO imagery data sets under `Public Imagery`, as well as the imagery data sets you or someone else manually uploaded to your institution under `Private Institution Imagery `.  Here, we have already imported several useful data sets, such as some elevation, PALSAR and Sentinel 1 radar, and Sentinel 2 optical imagery.

<img align="center" src="../images/ceo/CEO_imageryselection.png" hspace="15" vspace="10" width="700">

On the `Plot Design` page, you cannot currently change the parameters because `Copy Template Plots and Samples` was checked on the `Project Overview` page.  There are 50 plots centered on our validation points exported from GEE.  They are square and 30m in width because the Landsat data used for our classification has a resolution of 30m.

<img align="center" src="../images/ceo/CEO_plotdesign.png" hspace="15" vspace="10" width="700">

If `Copy Template Plots and Samples` was not checked, this page would look like this, and you would need to upload the .csv file with the validation points that was exported from GEE.

<img align="center" src="../images/ceo/CEO_plotdesign2.png" hspace="15" vspace="10" width="700">

Click `Next`.

On the `Sample Design` page, you also cannot change the parameters because `Copy Template Plots and Samples` was checked on the `Project Overview` page.  Each plot corresponds to a single sample located in the center of the plot.

<img align="center" src="../images/ceo/CEO_sampledesign.png" hspace="15" vspace="10" width="700">

If `Copy Template Plots and Samples` was not checked, this page would look like this, and you would be able to create multiple samples within each plot.

<img align="center" src="../images/ceo/CEO_sampledesign2.png" hspace="15" vspace="10" width="700">

Click `Next`.

On the `Survey Questions` page, you can create various types of sruvey questions related to your plots and samples. You can create parent and child questions so that certain questions only appear if the parent question was answered in a certain way.  You can also organize your questions into survey cards that are presented separately, which is particularly helpful when looking at land use change for different time periods.

For this exercise, we have two simple survey questions asking about what the land cover type is and what the percentage of that land cover type is.  On the right, you can see an example of what the survey question will look like when collecting data.

<img align="center" src="../images/ceo/CEO_surveyquestions.png" hspace="15" vspace="10" width="700">

<img align="center" src="../images/ceo/CEO_surveyquestions2.png" hspace="15" vspace="10" width="300">

Click `Next`.

On the `Survey Rules` page, you can create rules related to your survey questions.  For this exercise, we have just created 5 rules that prevent the user from answering 0% for any of the possible land cover classes (this is not a very useful rule since the 0%, 25%, and 50% options don't make any sense - assuming that you would need at least 50% coverage for the plot to be classified as that specific land cover type - but it is a good example of the general functionality of rules).  You can also set the rules so that CEO only accepts answers with certain values/strings or does not accept certain answers if the other questions were answered in a certain way.

<img align="center" src="../images/ceo/CEO_surveyrules.png" hspace="15" vspace="10" width="700">

Click `Next`.

On the `Review` page, you can check that everything looks good and create the project.  Check the box agreeing to the terms and conditions, and click `Create Project`.

<img align="center" src="../images/ceo/CEO_review.png" hspace="15" vspace="10" width="700">

At this point, you can still edit the project.  In order to start collecting data, you will need to click `Publish Project` on the next page, but you will now lose your ability to edit the plot and sample design.

<img align="center" src="../images/ceo/CEO_publish.png" hspace="15" vspace="10" width="700">

## Collect Data in the CEO Project

Now that you have published your project, go back to the institution home page and click on your project you just created to start collecting data.  It should be red before you start collecting data, yellow after you start collecting data, and green when you finish collecting data for all plots.

<img align="center" src="../images/ceo/CEO_projectpage2.png" hspace="15" vspace="10" width="700">

Select `Collect` and click `Go to First Plot`.

<img align="center" src="../images/ceo/CEO_collect.png" hspace="15" vspace="10" width="700">

It should take you to the first plot.  Here, you can view the original map classiciations in `Plot Information`, as well as all the imagery that was selected for this project in `Imagery Options`.  

<img align="center" src="../images/ceo/CEO_collect2.png" hspace="15" vspace="10" width="700">

<img align="center" src="../images/ceo/CEO_collectimagery.png" hspace="15" vspace="10" width="400">

If you want some more high resolution imagery to help in your decision, click `Download Plot KML`.  Now, open this file, and Google Earth Pro will open with the plot geometry already loaded in on top of Google Earth imagery.

<img align="center" src="../images/ceo/CEO_downloadKML.png" hspace="15" vspace="10" width="300">

Once you have opened Google Earth Pro, click on the clock icon on the toolbar at the top of the screen.  A bar showing a timeline of dates will appear at the top left corner of the screen.  You can use this bar to look at all historical and current Google Earth imagery available for this location. 

<img align="center" src="../images/ceo/GEP_KML.png" hspace="15" vspace="10" width="600">

Go back to the CEO project.  In `Survey Questions`, select the land cover type of the plot and the percentage of that land cover type within the plot, and click `Save`.  

<img align="center" src="../images/ceo/CEO_collectquestions.png" hspace="15" vspace="10" width="400">

**It is very important to click `Save` after EVERY plot you finish!**  

<img align="center" src="../images/ceo/CEO_collect2.png" hspace="15" vspace="10" width="700">

When you click `Save`, it should take you to the next plot.  Go through all 20 plots and select whether it was mangrove or not.  When you are done (and have saved each plot individually), click `Quit` to exit data collection mode.

Now that you have finished collecting data in your project, go back to the institution home page and click the `S` button to the right of your project.  This will download the data as a .csv file.  The file you download will retain the original columns from the sample points we generated in GEE and uploaded to CEO as a .csv (with `pl_` added to the column name).

<img align="center" src="../images/ceo/CEO_projectpage2.png" hspace="15" vspace="10" width="600">

<img align="center" src="../images/ceo/CEO_download.png" hspace="15" vspace="10" width="500">

# Assess Accuracy and Estimate Area in Google Sheets

For this section, copy the files from the [Suriname workshop Google Drive](https://drive.google.com/drive/u/2/folders/1czeYS5ZdCimR7tlQg-dE7QK8d08mzzIX) to your own Google Drive and work in Google Sheets.  The files we will be using for this section are your CEO validation data (.csv) and your GEE pixel count data (.csv).

 If you want to work on your own computer in Microsoft Excel, make sure you have all relevant files downloaded to your computer. 

## Error Matrix

Open the CEO validation data .csv file in Google Sheets.  Open the sheet called `error matrix empty`.

<img align="center" src="../images/ceo/GS_errormatrixempty.png" hspace="15" vspace="10" width="650">

Paste the following code into the top left cell, `D4`, and click enter.  Then select this cell and drag the blue selection box to cover the 4 cells to the left.

```
=COUNTIFS('ceo-Suriname-land-cover-map-val'!$N$2:$N$51, $C$4, 'ceo-Suriname-land-cover-map-val'!$O$2:$O$51, D3)
```

<img align="center" src="../images/ceo/GS_errormatrixempty1.png" hspace="15" vspace="10" width="650">

Do the same thing for the next cell below the top left cell, `D5`.

```
=COUNTIFS('ceo-Suriname-land-cover-map-val'!$N$2:$N$51, $C$5, 'ceo-Suriname-land-cover-map-val'!$O$2:$O$51, D3)
```

<img align="center" src="../images/ceo/GS_errormatrixempty2.png" hspace="15" vspace="10" width="650">

Then for `D6`.

```
=COUNTIFS('ceo-Suriname-land-cover-map-val'!$N$2:$N$51, $C$6, 'ceo-Suriname-land-cover-map-val'!$O$2:$O$51, D3)
```

<img align="center" src="../images/ceo/GS_errormatrixempty3.png" hspace="15" vspace="10" width="650">

Then for `D7`.

```
=COUNTIFS('ceo-Suriname-land-cover-map-val'!$N$2:$N$51, $C$7, 'ceo-Suriname-land-cover-map-val'!$O$2:$O$51, D3)
```

<img align="center" src="../images/ceo/GS_errormatrixempty4.png" hspace="15" vspace="10" width="650">

Then for `D8`.

```
=COUNTIFS('ceo-Suriname-land-cover-map-val'!$N$2:$N$51, $C$8, 'ceo-Suriname-land-cover-map-val'!$O$2:$O$51, D3)
```

<img align="center" src="../images/ceo/GS_errormatrixempty5.png" hspace="15" vspace="10" width="650">

### **Question 1**
* **What does this code do?**
* **What do the values in these cells mean?**

Now, in all the cells in the `sum` row (`D9:H9`), add up the samples that were categorized in each class by the **CEO validation data**.

<img align="center" src="../images/ceo/GS_errormatrixempty6.png" hspace="15" vspace="10" width="650">

In all the cells in the `sum` column (`I4:I9`), add up the samples that were categorized in each class by the **GEE classification map**.

<img align="center" src="../images/ceo/GS_errormatrixempty7.png" hspace="15" vspace="10" width="650">

*Hint: In cell `D9`, paste the following code.  Then select this cell and drag the blue selection box to cover the 4 cells to the left.*

```
=SUM(D4:D8)
```

*Hint: In cell `I4`, paste the following code.  Then select this cell and drag the blue selection box to cover the 5 cells below it.*

```
=SUM(D4:H4)
```

### **Question 2**
* **What does this code do?**
* **What do the values in these cells mean?**

In all the cells in the `producer's accuracy` column (`D10:H10`), divide the number of correctly categorized samples by the total number of samples in that class from the **CEO validation data**.

<img align="center" src="../images/ceo/GS_errormatrixempty8.png" hspace="15" vspace="10" width="650">

In all the cells in the `user's accuracy` column (`J4:J8`), divide the number of correctly categorized samples by the total number of samples in that class from the **GEE classification map**.

<img align="center" src="../images/ceo/GS_errormatrixempty9.png" hspace="15" vspace="10" width="650">

*Hint: In cell `D10`, paste the following code. You have to manually do this for each cell.*

```
=D4/D9
```

*Hint: In cell `J4`, paste the following code. You have to manually do this for each cell.*

```
=D4/I4
```

### **Question 3**
* **What does this code do?**
* **What do the values in these cells mean?**

In cell `J10`, add up all the correctly categorized samples and divide by the total number of samples.

<img align="center" src="../images/ceo/GS_errormatrixempty10.png" hspace="15" vspace="10" width="650">

*Hint: Paste the following code.*

```
=SUM(D4,E5,F6,G7,H8)/I9
```

### **Question 4**
* **What does this code do?**
* **What do the values in these cells mean?**

You now have a completed error matrix!  You can see what your final error matrix should look like in the `error matrix` sheet.

### **Question 5**
* **Which classes did our random forest classification do well with? Why did the model easily separate out these classes?**
* **Which classes did it struggle with? Why did the model have difficulty separating out these classes?**
* **Which areas were not well sampled in our stratified random sample? (e.g. were all different types of water bodies sampled? were all urban areas sampled well?)**

## Area Estimation

Open the CEO validation data .csv file in Google Sheets.  Open the sheet called `area estimation`.

The first section called `Sample Points Comparison Table` is where you input your data.  In `D4:H8`, you insert your error matrix values, and in `M4:M8` you insert your pixel count values (from your pixel count .csv).  You also calculate strata weights based on the percentage of total pixels that are in each stratum.

<img align="center" src="../images/ceo/GS_areaestimation1.png" hspace="15" vspace="10" width="700">

The second section called `Area Proportions Comparison Table` is where you calculate the "estimated area proportions" (the proportions of "true" area) for each class based on the number of correctly classified samples in that stratum, the total number of samples in that stratum, and the stratum weight.  You do this separately for each possible combination of classes between the classifiction map and validation points, and then add them all up for each validation point class.  The formula for this might change if your sample design is more complicated.  The area proportions should add up to 100%.

<img align="center" src="../images/ceo/GS_areaestimation2.png" hspace="15" vspace="10" width="600">

<img align="center" src="../images/ceo/formula_areaproportions.png" hspace="15" vspace="10" width="200">

<img align="center" src="../images/ceo/formula_stratifiedareaestimator.png" hspace="15" vspace="10" width="300">

*Cochran, W. G. (1977). Sampling Techniques. New York, NY: Wiley*

The third section called `Uncertainty Estimation` is where you calculate measures of uncertainty, such as the variance, standard error, confidence interval, and margin of error. The formula for these might change if your sample design is more complicated.

<img align="center" src="../images/ceo/GS_areaestimation3.png" hspace="15" vspace="10" width="600">

<img align="center" src="../images/ceo/formula_stratifiedvarianceestimator.png" hspace="15" vspace="10" width="500">

*Cochran, W. G. (1977). Sampling Techniques. New York, NY: Wiley*

The fourth section called `Area Estimation` is where you calculate the "true" areas in real-world units.  You multiply the area proportions by the total area of interest and convert it to hectares.  You do the same with the confidence interval, since that is also calculated as a percentage in the previous step.

<img align="center" src="../images/ceo/GS_areaestimation4.png" hspace="15" vspace="10" width="600">

### **Question 6**
* **Do these area estimates make sense?**
* **How do these area estimated compare to the area estimates from the classification map?**
* **Why is using such a small sample size for validation points not very useful?**

In reality, you will probably use these tools with much more complex data, like land cover classifications with many classes or land cover change maps that capture changes over many years.  You may also have data where the strata in the validation data and the classification data are not exactly the same (e.g. there are more substrata captured in the CEO validation data) - and in this case, you will need to do these calculations in Google Sheets or Microsoft Excel.  SEPAL (the tool we mentioned earlier) can only handle data sets in which the strata are identical between them.

