---
layout: page
title: Data Analysis and Interpretation
parent: Introduction to Synthetic Aperture Radar (SAR)
nav_order: 4
---

# Data Analysis and Interpretation
Now that we have an analysis ready image, we can begin to do a bit of analysis and interpretation. Remember that radar imagery is not the same as optical imagery – you’re not looking at visible or infrared light, but rather the **amount, or level, of energy that scattered back to the radar system** that is taking the measurement. We went over these ideas in the introduction of this lesson, but I’ll review them again here to better be able to interpret radar imagery. 
* **Water**: Open, generally calm bodies of water (and other smooth surfaces) appear very dark in images due to the specular reflection they cause. 
* **Rough surfaces**: In general, the rougher the surface, the brighter it appears. Wet rough surfaces appear the brightest due to the dielectric properties of water. 
* **Urban areas**: These areas tend to appear brighter in images due to the double-bounce reflective effect that vertical structures cause. Buildings that are perpendicular to the flight path of the satellite will appear the brightest, while buildings that are not perpendicular will appear less so. 
* **Inundation**: Flooded areas appear brighter because of the double-bounce effect caused by the groundwater onto vegetation or other structures.

#### Exercise 3.1 Classify water/non-water in SAR imagery. 
If we inspect the image we corrected in our earlier exercises, we can see pretty clearly the distinction between the dark channels of the river versus the bright white areas of land. Let’s do a quantitative analysis to actually classify which pixels represent water and which ones do not. 

1. **Generate a histogram of the values in the image.** This histogram will help us understand the distribution of the values in our SAR image and thus determine a threshold to separate water from non-water.
    1. Click on the `Sigma0_VH_db` image. This polarization delineates water from non-water slightly better than the `VV` polarization.
    2. Open the `Colour Manipulation` panel. In the upper menu bar, select `View > Tool Windows > Colour Manipulation`. A panel should appear in the lower left corner of the screen. 
    3. In the `Editor` field, click `Sliders`. A histogram should automatically be generated.
    4. You should see two fairly well-defined peaks – the peak near the maximum value on the chart represents the land-based pixels (brighter pixels), while the peak near the minimum value represents the water pixels (darker pixels). 
    5. Move the middle slider to identify the value that separates water from non-water pixels – somewhere between `18.25` and `18.5` (at the start of the non-water peak). 
2. **Set a threshold to separate water and non-water pixels.** We will create a new band that sets any water pixel as `1`, and any non-water pixel as `0`. 
    1. In the upper menu bar, select `Raster > Band Maths…`
    2. In the `Name:` field, name the new band `water_nonwater`. 
    3. Click `Edit Expression…`.
    4. In the `Expression:` box, input the following equation: `255*(Sigma0_VH_db<-18.38)`
    5. Click `OK`. 
    6. Click `OK` again to generate the thresholded image.
3. **Color the image.** Let’s follow our principles of data visualization from the Introduction to Remote Sensing workshop and visualize this data in a slightly more intuitive way. 
    1. Navigate back to the `Color Manipulation` panel.
    2. In the `Editor:` field, click the `Table` option.
    3. Underneath the `Colour` field, click on the black color tile. Since this value represents non-water pixels, primarily land, choose the color green. 
    4. Click on the white color tile. Since this value represents the water pixels, change the color to blue. 

Awesome! Now you have a very clear image that indicates where bodies of water exist. 
