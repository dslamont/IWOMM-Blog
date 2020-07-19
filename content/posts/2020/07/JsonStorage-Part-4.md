+++
title = "Json Storage - Custom Basic JsonConverter"
date = 2020-07-12T11:25:15+01:00
images = []
tags = []
categories = []
draft = false
+++

The next Json serialization topic we are going to look at is creating a custom JsonConverter. A JsonConverter is used to perform custom Json serialization.

A simple example to look at is the serialization of <kbd>DateTime</kbd> values.

Firstly we will look at the default behaviour of <kbd>DateTime</kbd> serialization. To achieve this we will add a suitable property to our test class.
 
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

This is fine but we might want to serialize to something more human friendly especially if we are creating an editable config file.

Previously, in [Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}}), we saw how to customise how an <kbd>Enum</kbd> was serialized using the in-built <kbd>JsonStringEnumConverter</kbd>. Unfortunately there is no corresponding in-built converter for <kbd>DateTime</kbd> values. This means we need to create our own converter.

``` csharp {hl_lines=[7,30],linenostart=1}
public class DateTimeConverter : JsonConverter<DateTime>
{
    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        DateTime readInValue = DateTime.ParseExact(reader.GetString(), "dd/MM/yyyy", CultureInfo.InvariantCulture);
        return readInValue;
    }

    public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString("dd/MM/yyyy", CultureInfo.InvariantCulture));
    }
}
```

The converter we have created overrides two methods <kbd>Read</kbd> and <kbd>Write</kbd>. When the converter is registered the <kbd>JsonSerialzer</kbd> will call these methods when it encounters <kbd>DateTime</kbd> values during the serialization process.

The overriden <kbd>Write</kbd> method changes the default serialization of a <kbd>DateTime</kbd> to the <kbd>dd//MM/yyyy</kbd> format. The corresponding <kbd>Read</kbd> method parses dates written to the format to create <kbd>DateTime</kbd> values.

For this example we will register the converter decorating the field definition with the <kbd>JsonConverter</kbd> attribute.

```csharp {hl_lines=[5],linenostart=1}
class StorageExampleClass
{
    public string TextField { get; set; }
    public int IntegerField { get; set; }
    [JsonConverter(typeof(DateTimeConverter))]
    public DateTime DateField { get; set; }
}
```

Running the sample program as before produces a file with the <kbd>DateTime</kbd> field in the <kbd>dd/MM/yyyy</kbd> format.

``` json {hl_lines=[4],linenostart=1}
{
  "TextField": "Sample Text",
  "IntegerField": 23,
  "DateField": "01/08/2020"
}
```

The console output produced is the same as before showing the <kbd>DateTime</kbd> was read in properly.

``` text
Text Field = Sample Text
Integer Field = 23
Date Field = 01/08/2020 00:00:00
```

The code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_4](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_4) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 

[Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}}) 

Part 4 - Custom Basic JsonConverter

[Part 5 - Serializing Derived Types]({{< ref "JsonStorage-Part-5.md" >}}) 

