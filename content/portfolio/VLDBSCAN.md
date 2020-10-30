+++
categories = ["plugin"]
coders = []
date = 2020-06-19T23:00:00Z
description = "2d dbscan algorithm for VL"
github = []
image = "/images/portfolio/dbscan_splash.png"
title = "VL.DBSCAN"
type = "post"
+++

VL.DBSCAN allows to use a 2D DBSCAN algorithm in VL. It allows you to identify and track point clusters in a 2D pointcloud.

This plugin simply wraps Stuart Turner's [DBSCAN .NET library](https://github.com/viceroypenguin/DBSCAN) and provides a few nodes to easily integrate DBSCAN in your projects.

To install the plugin, go to VL's command line and type

```
nuget install VL.DBSCAN
```

This plugin is one of the building blocks of the [LIDAR App](https://sebescudie.github.io/portfolio/lidar-app/).

For more information, head over to the plugin's [repo](https://github.com/sebescudie/VL.DBSCAN).