+++
title = "Json Storage - Serializing Derived Types"
date = 2020-07-19T18:45:15+01:00
images = []
tags = []
categories = []
draft = false
+++

The next Json serialization topic we are going to look at is serializing derived types.

Firstly we will create the base class to create subtypes from.


```csharp {linenostart=1}
public abstract class BaseClass
{
    public int BaseIntField { get; set; }
}
```

Next we will create a couple of sub classes on the base class.

``` csharp {linenostart=1}
public class DerivedClass1 : BaseClass
{
    public string DerivedClass1Field { get; set; }

}
```

``` csharp {linenostart=1}
public class DerivedClass2 :BaseClass
{
    public string DerivedClass2Field { get; set; }

}
```

Then we create a program to create instances of the sub types and serialize them to file.

``` csharp {linenostart=1}
class Program
{
    static async Task Main(string[] args)
    {
        //Create a DerivedClass1 object and serialize to disk
        DerivedClass1 derived1 = new DerivedClass1();
        derived1.BaseIntField = 1;
        derived1.DerivedClass1Field = "Derived Class 1";
        await SaveObject<DerivedClass1>("DerivedClass1.json", derived1);

        //Create a DerivedClass1 object and serialize to disk
        DerivedClass2 derived2 = new DerivedClass2();
        derived2.BaseIntField = 2;
        derived2.DerivedClass2Field = "Derived Class 2";
        await SaveObject<DerivedClass2>("DerivedClass2.json", derived2);
    }

    protected static async Task SaveObject<T>(string fileName, T objToSave)
    {
        JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
        serializerOptions.WriteIndented = true;

        using (FileStream fs = File.Create(fileName))
        {
            await JsonSerializer.SerializeAsync(fs, objToSave, serializerOptions);
        }

    }
}
```

This produces the following files

``` json {linenostart=1}
{
  "DerivedClass1Field": "Derived Class 1",
  "BaseIntField": 1
}
```

``` json {linenostart=1}
{
  "DerivedClass2Field": "Derived Class 2",
  "BaseIntField": 2
}
```

This shows that the two sub types were serialized properly.

The code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_5](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_5) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 

[Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}}) 

[Part 4 - Custom Basic JsonConverter]({{< ref "JsonStorage-Part-4.md" >}}) 

Part 5 - Serializing Derived Types