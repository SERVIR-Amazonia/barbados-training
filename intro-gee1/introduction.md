---
layout: page
title: Introduction
parent: Introduction to Google Earth Engine 1
nav_order: 1
---

# Introduction

In order to use Earth Engine well, you will need to develop basic skills in remote sensing and programming. The programming language in our Earth Engine tutorials is JavaScript, and you will begin by learning how to manipulate variables using it. With that base, you’ll learn about viewing individual satellite images, viewing collections of images in Earth Engine, and how common remote sensing terms are referenced and used in Earth Engine. 

*The material for this module was adapted from the EEFA book chapter F1, [https://www.eefabook.org/](https://www.eefabook.org/)*

## Objectives 
1. Successfully navigate within Earth Engine's Code Editor interface
2. Understand the basic building blocks of JavaScript and its syntax
3. Ability to explore and visualize remotely sensed data in Earth Engine


## Requirements
1. You will need in advance to have a GEE account approved. Go the GEE website and sign up for a new user account: [https://earthengine.google.com/signup/](https://earthengine.google.com/signup/).
2. Click this link to accept the caribbean-trainings GEE script repository - [https://code.earthengine.google.com/?accept_repo=users/kwoodward/caribbean-trainings](https://code.earthengine.google.com/?accept_repo=users/kwoodward/caribbean-trainings).


### Introduction to Theory 
The Earth Engine API 
Google Earth Engine is a cloud-based platform for scientific data analysis. It provides ready-to-use, cloud-hosted datasets and a large pool of servers. One feature that makes Earth Engine particularly attractive is the ability to run large computations very fast by distributing them across a large pool of servers. The ability to efficiently use cloud-hosted datasets and computation is enabled by the Earth Engine API.

An API is a way to communicate with Earth Engine servers. It allows you to specify what computation you would like to do, and then to receive the results. The API is designed so that users do not need to worry about how the computation is distributed across a cluster of machines and the results are assembled. Users of the API simply specify what needs to be done. This greatly simplifies the code by hiding the implementation detail from the users. It also makes Earth Engine very approachable for users who are not familiar with writing code.

Earth Engine API is designed to be language agnostic. Google provides official client libraries to use the API from both JavaScript and Python. The API remains largely the same regardless of the programming language you use. The main difference is the syntax used to call the API functions. Once you learn the syntax for programming languages, your code can be adapted easily because they all use the same API functions.

**Why JavaScript?**

JavaScript may not be the first choice of programming language for many researchers and data scientists, and some may be wondering why this book is based on the JavaScript API instead of Python or R.

The Earth Engine JavaScript API is the most mature and easiest to use when getting started. The Earth Engine platform comes with a web-based Code Editor that allows you to start using the Earth Engine JavaScript API without any installation. It also provides additional functionality to display your results on a map, save your scripts, access documentation, manage tasks, and more. It has a one-click mechanism to share your code with other users—allowing for easy reproducibility and collaboration. In addition, the JavaScript API comes with a user interface library, which allows you to create charts and web-based applications with little effort. 

In practice, you do not need to become a JavaScript expert to use Earth Engine. The basic syntax described here should be sufficient. A good tip is that if you find yourself doing something complicated in JavaScript, it might be done much better in Earth Engine. All the important computations in Earth Engine need to use the API functions, and even a basic operation—such as adding two numbers in Earth Engine—should be done using the Earth Engine API. 

