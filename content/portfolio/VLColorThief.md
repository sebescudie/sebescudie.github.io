+++
categories = ["plugin"]
coders = []
date = 2020-06-19T23:00:00Z
description = "color palette extraction from an image"
github = []
image = "/images/portfolio/colorthief_splash.png"
title = "VL.ColorThief"
type = "post"
[[picture]]
url = "/images/portfolio/colorthief_01.png"
alt = "color palette"
+++

This plugin wraps [KSemenenko](https://github.com/KSemenenko)'s [ColorThief](https://github.com/KSemenenko/ColorThief) library in a convenient nodeset. It allows you to extract a color pallette and dominant color from an input image.

Thanks to [sebl](https://github.com/sebllll/), the plugin also works with `Image` and `Bitmap` inputs.

To install it, simply go to VL's command line and type

```
nuget install VL.ColorThief
```

Then, go to the help browser and look for _Color Thief_, you will find two examples :

- How to retrieve color palette and dominant colors from an image
- How to retrieve dominant colors from an image folder

For more details, you can check [the plugin's repo](https://github.com/sebescudie/VL.ColorThief).

The image for the screenshot below was provided by [Alex Le Guillou](https://alexleguillou.com/).