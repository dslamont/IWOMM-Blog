+++
title = "Json Storage - Enum Serialization"
date = 2020-07-03T11:25:15+01:00
images = []
tags = []
categories = []
draft = false
+++

The next Json serialization topic we are going to look at is the serialization of Enums.

Firstly we will add an Enum class to the sample project.


``` csharp {linenostart=1}
public enum TestEnum
{
    One,
    Two,
    Three
}
```

We then add a <tt>TestEnum</tt> property to our test class.
 
```csharp {hl_lines=[5],linenostart=1}
class StorageExampleClass
{
    public string TextField { get; set; }
    public int IntegerField { get; set; }        
    public TestEnum EnumField { get; set; }
}
```

Next we adapt our sample program to add a <tt>TestEnum</tt> value to our sample object.


``` csharp {hl_lines=[7,30],linenostart=1}
static async Task Main(string[] args)
{
    //Create a sample object
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;
    exampleObj.EnumField = TestEnum.Two;

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
        Console.WriteLine($"Enum Field = {readinObj.EnumField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
        //Enum Field = Two
    }
}
```
The json file produced this time is

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "EnumField": 1
}
```

This shows the serialized <tt>TestEnum</tt> created in the file is the <tt>Enum</tt>'s underlying <tt>int</tt> value rather that the label value. In some circumstances, such as editable config files, it would be preferable to have the label value shown in the file. To achieve this we need a converter to serialize the <tt>TestEnum</tt> as the label and read it back in correctly. 

Fortunately, the <tt>System.Text.Json</tt> namespace has the built-in <tt>JsonStringEnumConverter</tt> which meets our needs. This converter can be set up in a few ways. The first way is to specify it in the <tt>JsonSerializerOptions</tt>. The code below shows this. Note: we now also need to supply the options when deserializing so the json value will be read in correctly.

``` csharp {hl_lines=[11,23],linenostart=1}
static async Task Main(string[] args)
{
    //Create a sample object
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;
    exampleObj.EnumField = TestEnum.Two;

    JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
    serializerOptions.WriteIndented = true;
    serializerOptions.Converters.Add(new JsonStringEnumConverter());

    //Save the object to disk
    string fileName = "SerializedObject.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, exampleObj, serializerOptions);
    }

    //Create a new object from the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        StorageExampleClass readinObj = await JsonSerializer.DeserializeAsync<StorageExampleClass>(fs, serializerOptions);
        Console.WriteLine($"Text Field = {readinObj.TextField}");
        Console.WriteLine($"Integer Field = {readinObj.IntegerField}");
        Console.WriteLine($"Enum Field = {readinObj.EnumField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
        //Enum Field = Two
    }
}
```
This produces the following output showing the <tt>TestEnum</tt> in a more human friendly manner.

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "EnumField": "Two"
}
```

Instead of explicitly specifying the converter in the options we can instead register the converter on the <tt>EnumField</tt> property.

``` csharp {hl_lines=["5-6"],linenostart=1}
class StorageExampleClass
{
    public string TextField { get; set; }
    public int IntegerField { get; set; }        
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public TestEnum EnumField { get; set; }
}
```

If we remove the registering of the converter in the options and run the program again we get the same result.

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "EnumField": "Two"
}
```

The third way is to register the converter on the <tt>TestEnum</tt> Type itself.

``` csharp {hl_lines=[1],linenostart=1}
[JsonConverter(typeof(JsonStringEnumConverter))]
public enum TestEnum
{
    One,
    Two,
    Three
}
```
Which way you decide to register the converter depends if you want to change the serialization behaviour globally or only in specific circumstances.

The code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_3](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_3) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 

Part 3 - Enum Serialization 

[Part 4 - Custom Basic JsonConverter]({{< ref "JsonStorage-Part-4.md" >}}) 