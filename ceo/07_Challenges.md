---
layout: page
title: Challenges 
parent: "Map Validation Using Collect Earth Online"
nav_order: 7
---

# Challenges

Work on whichever challenges are interesting to you.

## Challenge 1

See if you can find additional imagery in GEE (or elsewhere) that can help you in your validation point interpretation in CEO.  Add this imagery to our workshop's institution and symbolize it appropriately.

## Challenge 2

Create a new set of sample points in GEE by changing the number of points in each class. Download them and import them into a new CEO project (templated from the original CEO project we were working in).  Create one new question and one new rule in this CEO project.

*Hint: You will need to change the `numPoints` and `classPoints` parameters in your `stratifiedSample()` function.  You will not need to export the pixel count .csv again.*

*Hint: You will need to unselect `Copy Template Plots and Samples` but keep `Copy Template Widgets`.*

## Challenge 3

Do the interpretation for the new points in your new CEO project.  Download them and create an accuracy assessment and area estimation spreadsheet for them.  How have the results changed?

*Hint: Since we have the same exact classes in both the classification map and validation points, you can copy and paste most parts of the old spreadsheet into your new CEO validation data .csv.*

## Happy Coding!