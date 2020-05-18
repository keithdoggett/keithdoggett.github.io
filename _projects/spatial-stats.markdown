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

📸: <a href="https://www.flickr.com/photos/sk53_osm/22832538622/in/album-72157660934333051/" rel="nofollow" target="_blank">SK53 OSM</a>

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

### Technical Challenges
Two major challenges[^1] arose during the initial development of `spatial_stats`. The first is a lack of consistency among different GIS programs/packages, the second is the lack of a complete scientific computing library in Ruby.

#### Lack of Consistency
While developing modules, I reference well supported GIS libraries/programs to check that my results are consistent with established results. The problem is that the results across these programs are not always consistent. This was not much of a problem for computing the actual statistic but was a much bigger issue when implementing permutation testing.

Two of the main programs I reference are [GeoDa](https://geodacenter.github.io/) and [ESDA](https://github.com/pysal/esda). GeoDa comes with significant amounts of documentation, including formulas and a glossary of terms, so most of my implementations are based off of those definitions. The two packages produce consistent results for the statistical computations, but the issue is that the results of permutation testing between GeoDa and ESDA are inconsistent.

The issue comes down to the way a "more extreme" permutation is defined. In GeoDa's documentation, it says that if a value is greater than or equal to the original, it is more extreme (or less than or equal to if the original is negative), but this produces inconsistent results with how ESDA implements more extreme values. This caused me to spend a lot of time reasoning what the values should be, which was further confounded by the fact that my test data, a checkerboard of 1s and 0s, exacerbated the differences in their methodolgies. Eventually, I settled on following GeoDa's methodology becasue that is consistent with the probabilities I computed[^2].

To make matters more confusing, the above method only works for Moran's I and other methods have to be used for "more extreme" in Geary's C and Getis-Ord, but I followed GeoDa's implementation for them to maintain consistency.

#### Insufficient Scientific Library

First off, this is not an insult to the people working on Numo or SciRuby, they are both great projects, but for this particular use case, some pieces are missing.

The first issue I ran into came while implementing permutation testing. `Numo::NArray` is missing a `shuffle/sample` method, which means that in order to perform the conditional randomization, I perform shuffling in a Ruby array then cast the result to a Numo array. This is not a huge deal, but it could become a bottleneck in large datasets.

The second, bigger issue, is that there is not a well supported sparse matrix implementation in the Numo or SciRuby libraries. To solve this, I created a C Extension within the project that implements a slimmed down CSR Matrix. I'm actually happy that this happened because it was a great experience to learn how to work with C Extensions in Ruby, but it still posed a technical obstacle to overcome.

Ultimately, these issues did not prevent the library from being built, but they did increase development time. Further, they provided some insight into what pieces are lacking in the Numo and SciRuby packages. When I have more free time I might see if there's a way I can try to resolve some of these and make some contributions to those projects.

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

### Footnotes

[^1]: In reality, these weren't enormous roadblocks that nearly made me scrap the project, but they were the issues I spent around 80% of my time dealing with, so I figure they're worth documenting.

[^2]: In my checkerboard setup, this is the probability that if an observation's neighbors are sampled without replacement, they are all the opposite value of itself.