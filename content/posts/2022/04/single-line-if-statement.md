+++
title = "Single Line 'if' Statements"
date = 2022-04-10T11:15:00+01:00
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

bool condition1 = true;
Console.WriteLine($" condition1 = {condition1}");
Console.WriteLine("");

if (condition1)
{
    Console.WriteLine($" This line should only appear if 'condition11 is true.");
}

Console.ReadLine();
```

This all worked well.

![](.\single_if_screenshot_1.png)

Then somebody decided to 'improve' it by putting condition onto a single line.

``` csharp {linenos=false}
Console.WriteLine("");

bool condition1 = true;
Console.WriteLine($" condition1 = {condition1}");
Console.WriteLine("");

if (condition1) 
    Console.WriteLine($" This line should only appear if 'condition11 is true.");

Console.ReadLine();
```

This still worked as expected.

![](.\single_if_screenshot_1.png)

The problem occurred when somebody later added another statement thinking it was still dependent on the condition. 

``` csharp {linenos=false}
Console.WriteLine("");

bool condition1 = false;
Console.WriteLine($" condition1 = {condition1}");
Console.WriteLine("");

if (condition1) 
    Console.WriteLine($" This line should only appear if 'condition11 is true.");
    Console.WriteLine($" This other line again should only appear if 'condition11 is true.");

Console.ReadLine();
```

The code worked most of the time until <kbd>condition1</kbd> evaluated to <kbd>false</kbd> and the second statement still executed. 

![](.\single_if_screenshot_2.png)

Despite the indentation the second statement still executed because it isn't covered by the <kbd>if</kbd> statement.

My recommendation is to always use braces with <kbd>if</kbd> statements to prevent the above issue.