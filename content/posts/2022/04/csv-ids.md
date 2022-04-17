+++
title = "Object Ids to CSV"
date = 2022-04-16T11:15:00+01:00
images = []
tags = ["solidcode"]
categories = ["solidcode"]
draft = false
slug=""
+++

A colleague at work asked me recently how to format a ```QueryString``` to specify multiple ```ids```. I suggested concatenating the ```ids``` as a CSV (comma separated values) ```string``` such as ```?ids=1,2,3```. They then asked me how to create such a ```string```. I was slightly taken aback as it seemed a simple thing to do. 

After pointing them in the direction of ```String.Join()``` I started to wonder if this was the best option. I decided to investigate.


A simplified version of the code is shown below. 

``` csharp {linenos=false}
Console.WriteLine("");
```


My recommendation is to always use braces with <kbd>if</kbd> statements to prevent the above issue.