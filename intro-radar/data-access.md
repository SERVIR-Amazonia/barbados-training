---
layout: page
title: Data Access
parent: Introduction to Synthetic Aperture Radar (SAR)
nav_order: 2
---

# Data Access
You can access SAR data from a number of different data portals. SAR data is available on both of the platforms we explored in the Introduction to Remote Sensing lesson: EarthExplorer and Copernicus Open Access Hub. In this lesson, however, we will examine two other data sources that are particularly useful for SAR data.

## Earthdata 
NASA’s Earthdata portal allows you to search through its free and open source collection based on keywords or collection type. The platform contains many tools you can use to figure out what type of data is available for your research project and the processing steps you need to use to make the data analysis ready.

#### Exercise 1.1 Access SAR data using Earthdata search.
1. Navigate to the Earthdata search page: [https://search.earthdata.nasa.gov/search](https://search.earthdata.nasa.gov/search)
2. Login to your Earthdata account by clicking on the `Login` button in the upper right corner.
3. In the search bar where it says `Search for collections or topics`, type in `SAR`.
4. Select the `SENTINEL-1A_DUAL_POL_GRD_HIGH_RES` collection.
5. Drag the map and zoom in to Amapá, Brazil (the northeastern most state of Brazil). 
6. In the top panel on the right hand side, click on the point icon to `Search by spatial coordinate`. Select a point near the city Macapá.
7. Select the granule with the following ID: `S1A_IW_GRDH_1SDV_20230120T090602_20230120T090627_046865_059EA0_7322`.
8. To download the file, press the download button next to the green `+` button. Click on the name of the zip file under the `Download Files` tab and let the download complete. Move the zip file into the `intro-radar-data` folder.

## Alaska Satellite Facility DAAC
The Earthdata portal also helps manage NASA’s Distributed Active Archive Centers (DAACs).The DAACs process, archive, document, and distribute data from NASA's past and current Earth-observing satellites and field measurement programs and often have particular types of data or applications associated with them. The DAAC for SAR data is hosted by the Alaska Satellite Facility (ASF). The ASF DAAC is located in the Geophysical Institute at the University of Alaska, Fairbanks and is supported by NASA to acquire, process, archive, and distribute SAR data from polar-orbiting satellite and airborne sensors to advance Earth science research. Most of the datasets available through the ASF DAAC are freely available and open for the public to download.

The ASF DAAC is not limited to data from NASA-led satellite missions. It contains data from a wide range of satellites, including but not limited to: Sentinel-1, RADARSAT-1, European Remote Sensing Satellite-1 and -2 (ERS-1 and -2), ALOS PALSAR, Japanese Earth Resource Satellite-1 (JERS-1), SMAP, and Seasat, in addition to the airborne sensors AirSAR and UAVSAR.

#### Exercise 1.2 Access SAR data using the ASF DAAC.
1. Navigate the ASF Data Portal: [http://vertex.daac.asf.alaska.edu/](http://vertex.daac.asf.alaska.edu/)
2. In the upper search panel, set the `Search Type` to `Geographic Search` and the `Dataset` to `Sentinel-1`. 
3. Click on the `Filters` button.
    1. Under the `Area of Interest` field, input  `POLYGON((-50.985 -0.8565,-50.0179 -0.8565,-50.0179 0.0347,-50.985 0.0347,-50.985 -0.8565))`. Note that as an alternative, you could upload a shapefile for the specific area you are interested in, or draw an area of interest on the map directly.
    2. Under the `Date Filters` field, input `1/1/2023` as the `Start Date` and `1/25/2023` as the `End Date`. 
4. Click `Search`.
5. Select the first image result. Note that this is the same image as the one we found with the Earthdata search (you can check the granule IS). Feel free to explore the other images the search returned by scrolling through the results.
6. In the right-hand `Scene Detail` panel, you can read about the metadata of the image, view the image in the image viewer, or download the image. Click on the icon that looks like an eye to `Open in Image Viewer`. 
7. *Do not complete this step. It is the same file you downloaded in the previous exercise.* Click on the `L1 Detected High-Res Dual-Pol (GRD-HD)` product. Download the image product file.

## Sentinel-1 File Naming
The file names of Sentinel-1 data products follow a specific naming convention that provides information about the product and the acquisition conditions. The format of the Sentinel-1 file names is as follows:

`S1[A/B]_[IW/EW]_[TTTR]_[YYYYMMDD]T[HHMMSS]_[YYYYMMDD]T[HHMMSS]_[OOOOOO]_[DDDDDD]_[CCC].zip`

* `S1[A/B]`: Indicates the Sentinel-1 satellite, where A stands for Sentinel-1A and B for Sentinel-1B.
* `[IW/EW]`: Indicates the product type, where `IW` stands for Interferometric Wide swath and `EW` for Extra-Wide swath.
* `[TTTR]`: Indicates the product type and resolution. The product types can be `RAW`, `SLC`, `GRD`, or `OCN`. The resolution `R` can be `F` (full), `H` (high), or `M` (medium).
* `[LFPP]`: Indicates the processing level L, the product class F, and the polarization PP. L can be `1` or `2`, F can be `S` (standard) or `A` (annotation), and PP can be `SH` (single HH), `SV` (single VV), `DH` (dual HH + HV), or `DV` (dual VV + VH).
* `[YYYYMMDD]T[HHMMSS]`: Indicates the start and stop date and time of the acquisition (respectively), where `YYYY` is the year, `MM` is the month, `DD` is the day, and `HHMMSS` is the hour, minute, and second.
* `[OOOOOO]`: Indicates the absolute orbit number of the product.
* `[DDDDDD]`: Indicates the mission data-take identifier.
* `[CCCC]`: Indicates the product unique identifier.

Example: `S1A_IW_GRDH_1SSH_20201231T155959_20201231T160004_031117_03F1E1.zip`

This file name corresponds to a Level 1 Standard Sentinel-1A product acquired on December 31st, 2020, 15:59:59 UTC, with the Interferometric Wide swath mode, the Ground Range Detected High resolution mode, and with single HH polarization.

It's worth noting that the format of the file name may change depending on the specific mission or the data provider. It is recommended to consult with the data provider or refer to the documentation provided to get a better understanding of the file naming conventions.
