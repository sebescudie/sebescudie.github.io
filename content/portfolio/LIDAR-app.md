+++
categories = ["backend", "hardware", "software"]
coders = []
date = 2020-06-19T23:00:00Z
description = "a standalone app to calibrate multiple SICK LIDARs and perform tracking with them"
github = []
image = "/images/portfolio/lidar_app.png"
title = "LIDAR App"
type = "post"
+++

At [SUPERBIEN](https://www.superbien.studio/), I developed a stadalone application (codename LIDARApp) that allows to calibrate multiple [SICK 2D LIDARs](https://www.sick.com/us/en/detection-and-ranging-solutions/2d-lidar-sensors/c/g91900) and perform blob-tracking with them.

To get a glimpse of what the application does, you can have a look at the recording of the [15th vvvv meetup](https://youtu.be/UhzEwgdCbGo?t=1062) where I made a quick demo of the app.

Right now, the app is being re-written to add more features and update it to the latest versions of vvvv and Elementa. A link to the source code will be available when this is done.

![screenshot](https://sebescudie.github.io/images/portfolio/lidar_app.png)

### Features

- Visualize pointcloud data from many SICK 2D LIDARs (tested with TIM561 and LMS100). There's no limit "per se", just as much as your CPU can handle. We've run an installation for several months without any issues with 6 LIDARS on the same instance.
- Manually align pointcloud data
- Define a tracking area in which the blobs will be tracked (points outside of this zone will be omited)
- In this tracking area, create action zones in which blobs' position will be mapped to a relative coordinates
- For each zone, an OSC message containing the position and size of the blobs is emitted

#### Upcoming

- Simulate some blobs from an external device (such as a touch screen) to quickly test interaction scenarios

### Credits

This projet was made possible thanks to several open-source .NET libraries and vvvv contributions that were wrapped in VL plugins:

- viceroypenguin's [DBSCAN](https://github.com/viceroypenguin/DBSCAN)
- beolab-io's [SICK-Sensor-LMS1XX](https://github.com/beolabs-io/SICK-Sensor-LMS1XX)
- sebl's [Tracker](https://vvvv.org/contribution/tracker)
- dottore's [Model Runtime Editor design pattern tutorial](https://vvvv.org/contribution/model-runtime-editor-design-pattern)