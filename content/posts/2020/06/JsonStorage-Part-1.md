+++
title = "Json Storage - Basic Example"
date = 2020-06-25T20:56:15+01:00
images = []
tags = []
categories = []
draft = false
+++

Often when programming there is a need to represent the state of an object as text for transmission or storage. This process of converting an object to text is called serializing or marshalling.

Frequently, in .Net Core, we use the Json (JavaScript Object Notation) format for serializing. Previously in `.Net` this was done using the `Newtonsoft.Json` package but in `.Net Core` there is now the built in `System.Text.Json` namespace for Json serialization.

Below is a simple .Net Console app showing the basics of saving a C# object to disk as a Json file then reloading the file into a new object.

First we need a class to use for the example.

``` csharp
    class StorageExampleClass
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
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;

    //Save the object to disk
    string fileName = "SerializedObject.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, exampleObj);
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
The json file produced is 

``` json
{"TextField":"Sample Text","IntegerField":23}
```

The full code for this example can be found at [github.com/dslamont/JsonStorageExamples](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_1) 


Other posts in this series:

Part 1 - Basic Example

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}})

[Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}})

[Part 4 - Custom Basic JsonConverter]({{< ref "JsonStorage-Part-4.md" >}}) 

