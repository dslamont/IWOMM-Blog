+++
title = "Json Storage - Custom Basic JsonConverter"
date = 2020-07-07T11:25:15+01:00
images = []
tags = []
categories = []
draft = true
+++

The next Json serialization topic we are going to look at is using the a custom JsonConverter. A JsonConverter is used to perform custom Json serialization.

A simple example is the serialization of <kbd>DateTime</kbd> values.

Firstly we add a <kbd>DateTime</kbd> property to our test class.
 
```csharp {hl_lines=[5],linenostart=1}
class StorageExampleClass
{
    public string TextField { get; set; }
    public int IntegerField { get; set; }
    public DateTime DateField { get; set; }
}
```

Next we adapt our sample program to populate the <kbd>DateTime</kbd> field with a suitable value.


``` csharp {hl_lines=[7,30],linenostart=1}
static async Task Main(string[] args)
{
    //Create a sample object
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;
    exampleObj.DateField = new DateTime(2020, 8, 1);

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
        Console.WriteLine($"Date Field = {readinObj.DateField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
        //Date Field = 01/08/2020 00:00:00
    }
}
```
This produces the following json.

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "DateField": "2020-08-01T00:00:00"
}
```

The default behaviour is to serialize the <kbd>DateField</kbd> value to the <kbd>ISO 8601-1 219</kbd> format.

This is fine but we might want to serialize to something more human friendly. This might be useful if we are creating an editable config file.

We now adapt the program to populate the <kbd>DateField</kbd> property.

``` csharp {hl_lines=[7,30],linenostart=1}
static async Task Main(string[] args)
{
    //Create a sample object
    StorageExampleClass exampleObj = new StorageExampleClass();
    exampleObj.TextField = "Sample Text";
    exampleObj.IntegerField = 23;
    exampleObj.DateField = new DateTime(2020, 8, 1);

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
        Console.WriteLine($"Date Field = {readinObj.DateField}");

        //Output
        //Text Field = Sample Text
        //Integer Field = 23
        //Date Field = 01/08/2020 00:00:00
    }
}
```


This produces the following output showing the <kbd>DateField</kbd> in a more human friendly manner.

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "DateField": "01/08/2020"
}
```



The code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_4](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_4) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 
