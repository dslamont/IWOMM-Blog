+++
title = "Json Storage - Part 1"
date = 2020-06-25T20:56:15+01:00
images = []
tags = []
categories = []
draft = true
+++


Json (JavaScript Object Notation) is a useful text based format for file storage. 

Below is a simple .Net Console app showing the basics of saving a C# object to disk as a Json file then reloading the file into a new object.

First we need a class to use for the example.

``` csharp
    public class Example1 
    {
        public string TextField { get; set; }
        public int IntegerField { get; set; }

    }
```

Next in Program.cs we create an object based on the class, save it to disk then read it back in to populate a new object.

``` csharp
static async Task Main(string[] args)
{
    //Create a sample object
    Example1 example1Obj = new Example1();
    example1Obj.TextField = "Sample Text";
    example1Obj.IntegerField = 23;

    //Save the object to disk
    string fileName = "Example1.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, example1Obj);
    }

    //Create a new object from the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        Example1 readinObj = await JsonSerializer.DeserializeAsync<Example1>(fs);
        Console.WriteLine($"Text Field = {readinObj.TextField}");
        Console.WriteLine($"Integer Field = {readinObj.IntegerField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
    }
}
```
The json file produced is 

``` json
{"TextField":"Sample Text","IntegerField":23}
```



