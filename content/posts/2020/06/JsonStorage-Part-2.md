+++
title = "Json Storage - Indenting Json"
date = 2020-06-28T11:25:15+01:00
images = []
tags = []
categories = []
draft = false
+++

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

Previously, in Part 1 of this series, we saw how to serialize a C# object to disk in Json format.

This time we are going to make the Json more readable by indenting it. This is achieved using the `JsonSerializerOptions` class. This is a class that can be used to specify how the serialization to Json is achieved. The option we are interested in is `WriteIndented`.


The modified code is shown below.

``` csharp {hl_lines=["9-10", 16],linenostart=1}
static async Task Main(string[] args)
{
    //Create a sample object
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;

    //Specify that the Json should be indented
    JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
    serializerOptions.WriteIndented = true;

    //Save the object to disk
    string fileName = "SerializedObject.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, exampleObj, serializerOptions);
    }

    //Create a new object from the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        StorageExampleClass readinObj = await JsonSerializer.DeserializeAsync<StorageExampleClass>(fs);
        Console.WriteLine($"Text Field = {readinObj.TextField}");
        Console.WriteLine($"Integer Field = {readinObj.IntegerField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
    }
}

```
The json file produced this time is

``` json
{
  "TextField": "Sample Text",
  "IntegerField": 23
}
```

The full code for this example can be found at 

[github.com/dslamont/JsonStorageExamples/releases/tag/Part_2](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_2) 


