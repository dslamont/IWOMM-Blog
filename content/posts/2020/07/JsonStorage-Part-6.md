+++
title = "Json Storage - Simple Serialization of Polymorphic Collections"
date = 2020-07-26T20:45:15+01:00
images = []
tags = []
categories = []
draft = false
+++

Previously we saw that serializing individual instances of derived types worked as expected. Now we will look at serializing collections of derived types.

Firstly we create the base class for the derived types. We override the <kbd>ToString()</kbd> method to use for outputting diagnostic messages.

``` csharp
public class BaseClass
{
    public int BaseIntField { get; set; }

    public BaseClass()
    {
        //Default constructor required for deserialization
    }

    public override string ToString()
    {
        return $"BaseClass: BaseIntField = {BaseIntField}";
    }
}
```

``` csharp
public class DerivedClass1 : BaseClass
{
    public string DerivedClass1Field { get; set; }

    public DerivedClass1()
    {
        //Default constructor required for deserialization
    }

    public override string ToString()
    {
        return $"DerivedClass1Field: BaseIntField = {BaseIntField} DerivedClass1Field = {DerivedClass1Field}";
    }
}
```

``` csharp
public class DerivedClass2 :BaseClass
{
    public string DerivedClass2Field { get; set; }
    public DerivedClass2()
    {
        //Default constructor required for deserialization
    }
    public override string ToString()
    {
        return $"DerivedClass2Field: BaseIntField = {BaseIntField} DerivedClass2Field = {DerivedClass2Field}";
    }
}
```

Then we create a program to create a polymorphic collection of the sub types and serialize them to file.

``` csharp {linenostart=1}
static async Task Main(string[] args)
{
    //Create a collection to contain the sample objects
    List<BaseClass> collection = new List<BaseClass>();

    //Create a DerivedClass1 object and add to the collection
    DerivedClass1 derived1 = new DerivedClass1();
    derived1.BaseIntField = 1;
    derived1.DerivedClass1Field = "Derived Class 1";
    collection.Add(derived1);

    //Create a DerivedClass1 object and add to the collection
    DerivedClass2 derived2 = new DerivedClass2();
    derived2.BaseIntField = 2;
    derived2.DerivedClass2Field = "Derived Class 2";
    collection.Add(derived2);

    JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
    serializerOptions.WriteIndented = true;

    string fileName = "DerivedClassCollection.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, collection, serializerOptions);
    }

    //Read the List to read in the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        List<BaseClass> recoveredCollection = await JsonSerializer.DeserializeAsync<List<BaseClass>>(fs, serializerOptions);
        
        foreach(BaseClass item in recoveredCollection)
        {
            Console.WriteLine(item);
        }
     
        //Output
        //BaseClass: BaseIntField = 1
        //BaseClass: BaseIntField = 2
    }
}
```

This produces the following file

``` json {linenostart=1}
[
  {
    "BaseIntField": 1
  },
  {
    "BaseIntField": 2
  }
]
```

This shows that the serialization didn't work as expected. Only the properties on the base class were serialized. 

To overcome this we need to create a <kbd>JsonConverter</kbd> that can distinguish between the derived types.

``` csharp
public class DerivedClassConverter : JsonConverter<BaseClass>
{
    enum TypeDiscriminator
    {
        DerivedClass1 = 1,
        DerivedClass2 = 2
    }

    public override bool CanConvert(Type typeToConvert) =>
        typeof(BaseClass).IsAssignableFrom(typeToConvert);

    public override BaseClass Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        if (reader.TokenType != JsonTokenType.StartObject)
        {
            throw new JsonException();
        }

        reader.Read();
        if (reader.TokenType != JsonTokenType.PropertyName)
        {
            throw new JsonException();
        }

        string propertyName = reader.GetString();
        if (propertyName != "TypeDiscriminator")
        {
            throw new JsonException();
        }

        reader.Read();
        if (reader.TokenType != JsonTokenType.Number)
        {
            throw new JsonException();
        }

        TypeDiscriminator typeDiscriminator = (TypeDiscriminator)reader.GetInt32();
        BaseClass BaseClass = typeDiscriminator switch
        {
            TypeDiscriminator.DerivedClass1 => new DerivedClass1(),
            TypeDiscriminator.DerivedClass2 => new DerivedClass2(),
            _ => throw new JsonException()
        };

        while (reader.Read())
        {
            if (reader.TokenType == JsonTokenType.EndObject)
            {
                return BaseClass;
            }

            if (reader.TokenType == JsonTokenType.PropertyName)
            {
                propertyName = reader.GetString();
                reader.Read();
                switch (propertyName)
                {
                    case "DerivedClass1Field":
                        string class1Field = reader.GetString();
                        ((DerivedClass1)BaseClass).DerivedClass1Field = class1Field;
                        break;
                    case "DerivedClass2Field":
                        string class2Field = reader.GetString();
                        ((DerivedClass2)BaseClass).DerivedClass2Field = class2Field;
                        break;
                    case "BaseIntField":
                        int intValue = reader.GetInt32();
                        BaseClass.BaseIntField = intValue;
                        break;
                }
            }
        }

        throw new JsonException();
    }

    public override void Write(Utf8JsonWriter writer, BaseClass BaseClass, JsonSerializerOptions options)
    {
        writer.WriteStartObject();

        if (BaseClass is DerivedClass1 DerivedClass1)
        {
            writer.WriteNumber("TypeDiscriminator", (int)TypeDiscriminator.DerivedClass1);
            writer.WriteString("DerivedClass1Field", DerivedClass1.DerivedClass1Field);
        }
        else if (BaseClass is DerivedClass2 DerivedClass2)
        {
            writer.WriteNumber("TypeDiscriminator", (int)TypeDiscriminator.DerivedClass2);
            writer.WriteString("DerivedClass2Field", DerivedClass2.DerivedClass2Field);
        }

        writer.WriteNumber("BaseIntField", BaseClass.BaseIntField);

        writer.WriteEndObject();
    }
}

```

To ensure that only suitable types are converted the method <kbd>CanConvert</kbd> checks that the current type being serialized is <kbd>BaseClass</kbd> or one of the classes derived from it.

The heart of the mechanism to serializing polymorphic types is the <kbd>Write</kbd> method. This method determines the actual type of the type being serialized and injects an extra value into the output that explicitly identifies the type. Then the other fields specific to that type are output.

During deserialization the <kbd>Read</kbd> method gets called. When deserializing the <kbd>Json</kbd> the method firstly reads in the type identifier. This is used to determine the type that should be created. The required type is then created and the subsequent values read in from the <kbd>Json</kbd> file get assigned to the type.

To call the converter we need to register it as shown in the following example.

``` csharp
static async Task Main(string[] args)
{
    //Create a collection to contain the sample objects
    List<BaseClass> collection = new List<BaseClass>();

    //Create a DerivedClass1 object and add to the collection
    DerivedClass1 derived1 = new DerivedClass1();
    derived1.BaseIntField = 1;
    derived1.DerivedClass1Field = "Derived Class 1";
    collection.Add(derived1);

    //Create a DerivedClass1 object and add to the collection
    DerivedClass2 derived2 = new DerivedClass2();
    derived2.BaseIntField = 2;
    derived2.DerivedClass2Field = "Derived Class 2";
    collection.Add(derived2);

    JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
    serializerOptions.WriteIndented = true;
    serializerOptions.Converters.Add(new DerivedClassConverter());

    //Save the list to disk
    string fileName = "DerivedClassCollection.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, collection, serializerOptions);
    }

    //Read the List to read in the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        List<BaseClass> recoveredCollection = await JsonSerializer.DeserializeAsync<List<BaseClass>>(fs, serializerOptions);

        foreach (BaseClass item in recoveredCollection)
        {
            Console.WriteLine(item);
        }
    }
    //Output
    //DerivedClass1Field: BaseIntField = 1 DerivedClass1Field = Derived Class 1
    //DerivedClass2Field: BaseIntField = 2 DerivedClass2Field = Derived Class 2
}
```

Running the code produces the following <kbd>Json</kbd> file showing the values that identifies the type of object.

``` json
[
  {
    "TypeDiscriminator": 1,
    "DerivedClass1Field": "Derived Class 1",
    "BaseIntField": 1
  },
  {
    "TypeDiscriminator": 2,
    "DerivedClass2Field": "Derived Class 2",
    "BaseIntField": 2
  }
]
```

The full code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_6](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_6) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 

[Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}}) 

[Part 4 - Custom Basic JsonConverter]({{< ref "JsonStorage-Part-4.md" >}}) 

[Part 5 - Serializing Derived Types]({{< ref "JsonStorage-Part-5.md" >}}) 

Part 6 - Simple Serialization of Polymorphic Collections