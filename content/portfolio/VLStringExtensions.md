+++
categories = ["plugin"]
coders = []
date = 2020-06-19T23:00:00Z
description = "useful string manipulation functions in VL"
github = []
image = "/images/portfolio/stringextensions_splash.png"
title = "VL.StringExtensions"
type = "post"
+++

VL.StringExtensions adds super-useful string manipulation nodes to VL. You can easily do things such as verify if a string is a correctly formatted email address, append a suffix if it's missing, or remove all special characters from a string.

It simply wraps two .NET string extensions libraries :

- Jeff Klein's [String.Extensions](https://github.com/Jeff-Klein/String.Extensions)
- Timothy Mugayi's [StringExtensions](https://github.com/timothymugayi/StringExtensions)

The plugin also contains simple overview help patches that show all the nodes in action.

### How to get it

Go to vvvv gamma's command line and type

```
nuget install VL.StringExtensions
```

Then, open the help browser (<kbd>F1</kbd>) and look for StringExtensions.

For more information, check the plugin's [Github repo](https://github.com/sebescudie/VL.StringExtensions).