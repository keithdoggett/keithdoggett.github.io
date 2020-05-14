---
layout: post
title: "Spatial Stats"
date: 2020-05-14 15:22:00 -0500
categories: spatial projects
img: projects/spatial_stats/pitt.png
---
For the past few months, I've been working on a ruby/rails gem called Spatial Stats. The goal of this project is to bring an optimized spatial statistics library to the ruby community and I'm proud to say that the gem is ready for use.

![PA Map](/assets/images/projects/spatial_stats/pitt.png "PA Map")

<a href="https://www.flickr.com/photos/sk53_osm/22832538622/in/album-72157660934333051/" rel="nofollow" target="_blank">SK53 OSM</a>

While it is not fully featured, the [package](https://www.github.com/keithdoggett/spatial_stats) is fast and includes a lot of commonly used measurements. Currently, global spatial autocorrelation is possible with `moran` and `bivariate_moran` classes. Local spatial autocorrelation is possible with `moran`, `bivariate_moran`, `getis_ord`, `geary`, and `multivariate_geary`. Further, `ActiveRecord` utilities exist to automatically query spatial databases and create weight matrices so that data can easily go from database to the stat classes.