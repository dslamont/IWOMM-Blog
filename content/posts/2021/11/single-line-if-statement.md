+++
title = "Single Line 'if' Statements"
date = 2021-11-19T20:15:15+01:00
images = []
tags = ["debugging", "solidcode"]
categories = []
draft = false
slug=""
+++

I came across the following issue recently. Originally there was a method containing an <kbd>if</kbd> statement. If a variable wasn't <kbd>null</kbd> then some action is undertaken. This is all straightforward enough stuff.

A simplified version of the code is shown below. 

``` csharp {linenos=false}
Console.WriteLine("");
Console.WriteLine("Single Line 'if' Statement");
Console.WriteLine("");

string condition1 = "Some random text";
Console.WriteLine($"condition1 = {condition1}");

if (condition1 != null)
{
    Console.WriteLine($"This line should only appear if 'condition1; is not null.");
}

Console.WriteLine("");
Console.WriteLine("");
```



Single line <kbd>if</kbd> statements are a bad idea.
