---
layout: post
title: "Spatial Stats"
description: "Spatial Stats Ruby/Rails Gem project page. Overview of features, technical challenges, and future work."
date: 2020-05-14 15:22:00 -0500
categories: spatial projects
img: projects/spatial_stats/pitt.png
---
For the past few months, I've been working on a Ruby/Rails gem called Spatial Stats. The goal of this project is to bring an optimized spatial statistics library to the Ruby community and I'm happy to say that the gem is ready for use.

![PA Map](/assets/images/projects/spatial_stats/pitt.png "PA Map")

📸: <a href="https://www.flickr.com/photos/sk53_osm/22995995762/" rel="nofollow" target="_blank">SK53 OSM</a>

While it is not fully featured, the [package](https://www.github.com/keithdoggett/spatial_stats) is performant and includes a lot of commonly used measurements for both distance-based and contiguous geometries. Additionally, it includes helpful utilities and `ActiveRecord` extensions to increase useability.

### Feature List
1. Global Measurements
  * `Moran`'s I
  * `BivariateMoran`'s I
2. Local Measurements
  * `Moran`'s I
  * `BivariateMoran`'s I
  * `Geary`'s C
  * `MultivariateGeary`'s C
  * `GetisOrd` G and G*
3. Utilities
  * `WeightsMatrix` class built from a dictionary of keys (DOK)
  * `CSRMatrix`, a slim implementation of a compressed sparse row matrix (aka Yale Matrix)
  * `SpatialLag`, computations for different weight layouts
4. ActiveRecord Extensions
  * `Rook` weight calculation for contiguous geometries
  * `Queen` weight calculation for contiguous geometries
  * `InverseDistanceWeight` weight calculations for distance-based geometries
  * `KNN` weight calculations for distance-based geometries
5. Extensions
  * `standardize` method on enumerable classes
  * `row_standardize` and `window` methods on `Numo::NArray`


### Path Forward

I'm happy that the project has come together nicely so far, but there is definitely still work that needs to be done on it. A variety of improvements could be made from adding features, adding utilities to increase usability, splitting it up into different gems to reduce dependencies, and other smaller things. I'm going to list some of the things that come to mind below, but I intend on making this a living document that will be updated as the gem progresses. If you want to make a feature suggestion or contribution feel free to open up an issue/pull request on the Github page.

1. Global Measurements
  * `Geary`'s C
  * `GetisOrd`
2. Local Measurements
  * `Join Count`
3. Utilities
  * Add support for .gal/.swm file imports
  * Add support for Rate variables
  * Add support for Bayes smoothing
  * Add an additional gem `spatial_stats-geojson` that will depend on `RGeo`, but can produce weights matrices from geojson inputs.
4. ActiveRecord Extensions
  * Break queries into a seperate `spatial_stats-activerecord` gem that will contain the queries to interface with PostGIS. Will remove core dependency on `Rails`.
5. General
  * Add point pattern analysis module
  * Add regression module

