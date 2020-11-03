+++
categories = ["plugin"]
coders = []
date = 2020-06-19T23:00:00Z
description = "harmonious color palette algorithms"
github = []
image = "/images/portfolio/colorblender_splash.png"
title = "VL.ColorBlender"
type = "post"
+++

This VL plugin wraps the [ColorBlender nuget](https://github.com/wieslawsoltes/ColorBlender), by [Wiesław Šoltés](https://github.com/wieslawsoltes).

Given an input color, it allows you to retrieve a spread of harmonious colors with different algorithms :

- ColorMatch 5K Classic
- ColorExplorer - "Sweet Spot Offset"
- Single Hue
- Complementary
- Split-Complementary
- Analogue
- Triadic
- Square

For each algorithm, you get a process node that takes an `RGBA` and returns a `Spread<RGBA>`. You'll also find the same nodes as static operations if you enable Advanced nodes.

The nuget comes with two help patches that you can find in the help browser.

### How to get it

Open VL's command line and type

```
nuget install VL.ColorBlender
```

You can also check the pugin's [repo](https://github.com/sebescudie/VL.ColorBlender).