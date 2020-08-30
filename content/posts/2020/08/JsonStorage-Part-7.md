+++
title = "Json Storage - Advanced Serialization of Polymorphic Collections"
date = 2020-08-29T09:45:15+01:00
images = []
tags = []
categories = []
draft = false
+++

Last [time]({{< ref "JsonStorage-Part-6.md" >}}) we looked at serializing/deserializing a polymorphic list of derived types. The method we used worked but involved quite a lot of customisation to handle the different object types. With an increase in the number of derived types the customisation required would increase dramatically.

What is needed is a more generic, low maintenance, way to handle collections of derived types. The way we will achieve this when serializing a list of derived types is to wrap the derived types using an outer identifier. Then when reading in from the Json file, the identifier is read first and this can then be used to determine the exact Type of derived type that needs to be read in. This allows us to convert the derived type directly using the standard <kbd>System.Text.Json</kbd> mechanisms.

To demonstrate this technique we will need to create some derived types.  This time we will use Interfaces to implement the polymorphism.

``` csharp {linenos=false}
public interface IBaseInterface
{
    public int BaseIntField { get; set; }
}

public class DerivedClass1 : IBaseInterface    {

    public string DerivedClass1StringField { get; set; }
    public int BaseIntField { get; set; }

    public override string ToString()
    {
        return $"DerivedClass1: BaseIntField = {BaseIntField} DerivedClass1StringField = {DerivedClass1StringField}";
    }
}

public class DerivedClass2 : IBaseInterface
{
    public int DerivedClass2IntField { get; set; }
    public int BaseIntField { get; set; }

    public override string ToString()
    {
        return $"DerivedClass2: BaseIntField = {BaseIntField} DerivedClass2IntField = {DerivedClass2IntField}";
    }
}
```

To handle the serialization/deserialization of a polymorphous collection of these types we need to create a suitable converter.

``` csharp
public class DerivedClassConverter<TItem, TList> : JsonConverter<TList> where TItem : notnull where TList : IList<TItem>, new()
{
    public override bool CanConvert(Type typeToConvert)
    {
        //Ensure the current Type is of interest
        bool canConvert = typeof(TList).IsAssignableFrom(typeToConvert);
    
        return canConvert;
    }


    public override TList Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        if(reader.TokenType != JsonTokenType.StartArray)
        {
            //Not at the start so bomb out
            throw new JsonException();
        }

        //Create the list to be returned
        var results = new TList();

        //Read in the first element
        reader.Read(); 

        while (reader.TokenType == JsonTokenType.StartObject)
        { 
            //Read in the next element (should be the property name)
            reader.Read(); 

            if (reader.TokenType != JsonTokenType.PropertyName)
            {
                //Not a property name so bomb out
                throw new JsonException();
            }

            //Read in the type identifier
            var typeKey = reader.GetString();
            
            //Advance to the next element which should be the start of the object
            reader.Read(); 

            if (reader.TokenType != JsonTokenType.StartObject)
            {
                //Not the start of the object so bomb out
                throw new JsonException();
            }

            //Determine which type if Item we are going to read in
            Type concreteItemType = DerivedTypeLookup.GetTypeByIdentifier(typeKey);
            if (concreteItemType != null)
            {
                //Deserialize the item letting System.Text.Json decide how to achieve this
                var item = (TItem)JsonSerializer.Deserialize(ref reader, concreteItemType, options);
                
                //Add the retrieved item to the list
                results.Add(item);
            }
            else
            {
                //Unknown Item so bomb out
                throw new JsonException();
            }

            //Advance to the end of the wrapper section
            reader.Read(); 
            reader.Read(); 
        }

        if (reader.TokenType != JsonTokenType.EndArray)
        {
            //No End Array so bomb out
            throw new JsonException();
        }

        //Return the retrieved list
        return results;
    }

    public override void Write(Utf8JsonWriter writer, TList items, JsonSerializerOptions options)
    {
        //Start the Json array representing the List
        writer.WriteStartArray();

        //Loop through all the supplied items in the List
        foreach (var item in items)
        {
            //Start the item representation
            writer.WriteStartObject();

            //Determine the Type of the item 
            var itemType = item.GetType();

            //Determine the identifier we will use to identify the item
            string typeKey = DerivedTypeLookup.GetIdentifierByType(itemType);
            if(!String.IsNullOrEmpty(typeKey))
            {
                //Write the item type identifier
                writer.WritePropertyName(typeKey);

                //Output the item letting System.Text.Json decide how to achieve this
                JsonSerializer.Serialize(writer, item, itemType, options);
            }
            else
            {
                //No identifier found so bomb out
                throw new JsonException();
            }

            //Close the item's representation
            writer.WriteEndObject();
        }

        //Close the List representation
        writer.WriteEndArray();
    }

}
```

The converter above makes use of a <kbd>DerivedTypeLookup</kbd> class which is shown below. This class is used to map an object type to a label and the reverse mapping of a label to an object type. 

``` csharp
public class DerivedTypeLookup
{
    private static Dictionary<Type, string> _typesLookup;

    //Returns the text label to identify a Type
    public static string GetIdentifierByType(Type type)
    {
        string identifier = String.Empty;

        //Extract the text label from the lookup
        if (TypesLookup.TryGetValue(type, out var typeId))
        {
            identifier = typeId;
        }

        return identifier;
    }

    //Returns the Type specified by a label
    public static Type GetTypeByIdentifier(string typeId)
    {
        Type type = TypesLookup.FirstOrDefault(t => t.Value == typeId).Key;

        return type;
    }

    //Set up the lookup for Types/Labels
    public static Dictionary<Type, string> TypesLookup
    {
        get
        {
            if (_typesLookup == null)
            {
                _typesLookup = new Dictionary<Type, string>();
                _typesLookup.Add(typeof(DerivedClass1), "Derived Class 1");
                _typesLookup.Add(typeof(DerivedClass2), "Derived Class 2");
            }

            return _typesLookup;
        }

    }

}
```

Next, to demonstrate this, we create create a list of the derived types and serialize them to file then read the file back in to prove it works.

``` csharp {linenostart=1}
static async Task Main(string[] args)
{
    //Create a collection to contain the sample objects
    List<IBaseInterface> collection = new List<IBaseInterface>();

    //Create a DerivedClass1 object and add to the collection
    DerivedClass1 derived1 = new DerivedClass1();
    derived1.BaseIntField = 1;
    derived1.DerivedClass1StringField = "Derived Class 1 Field Text";
    collection.Add(derived1);

    //Create a DerivedClass1 object and add to the collection
    DerivedClass2 derived2 = new DerivedClass2();
    derived2.BaseIntField = 2;
    derived2.DerivedClass2IntField = 44;
    collection.Add(derived2);

    JsonSerializerOptions serializerOptions = new JsonSerializerOptions();
    serializerOptions.WriteIndented = true;
    serializerOptions.Converters.Add(new DerivedClassConverter<IBaseInterface, List<IBaseInterface>>());

    //Save the list to disk
    string fileName = "DerivedClassCollection.json";
    using (FileStream fs = File.Create(fileName))
    {
        await JsonSerializer.SerializeAsync(fs, collection, serializerOptions);
    }

    //Read the List to read in the saved Json
    using (FileStream fs = File.OpenRead(fileName))
    {
        List<IBaseInterface> recoveredCollection = await JsonSerializer.DeserializeAsync<List<IBaseInterface>>(fs, serializerOptions);

        foreach (IBaseInterface item in recoveredCollection)
        {
            Console.WriteLine(item);
        }
        //Output
        //DerivedClass1Field: BaseIntField = 1 DerivedClass1StringField = Derived Class 1 Field Text
        //DerivedClass2Field: BaseIntField = 2 DerivedClass2IntField = 44

    }

    Console.WriteLine("Press any key to exit...");
    Console.In.ReadLine();
}
```

This produces the following <kbd>Json</kbd> file.

``` json
[
  {
    "Derived Class 1": {
      "DerivedClass1StringField": "Derived Class 1 Field Text",
      "BaseIntField": 1
    }
  },
  {
    "Derived Class 2": {
      "DerivedClass2IntField": 44,
      "BaseIntField": 2
    }
  }
]
```

The <kbd>Json</kbd> file produced shows the serialization of the list of derived types. It shows the derived types being 'wrapped' up and identified by the appropriate label value. When deserializing the label value is read in and used to determine the type of object to be read in.

Now, when a new derived type is added the only 

It might be tempting to avoid the lookup class by directly using the string representation of the derived type. However this exposes internal type details which is a potential vulnerability.

The full code for this example can be found at 

[https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_7](https://github.com/dslamont/JsonStorageExamples/releases/tag/Part_7) 

Other posts in this series:

[Part 1 - Basic Example]({{< ref "JsonStorage-Part-1.md" >}}) 

[Part 2 - Indenting Json]({{< ref "JsonStorage-Part-2.md" >}}) 

[Part 3 - Enum Serialization]({{< ref "JsonStorage-Part-3.md" >}}) 

[Part 4 - Custom Basic JsonConverter]({{< ref "JsonStorage-Part-4.md" >}}) 

[Part 5 - Serializing Derived Types]({{< ref "JsonStorage-Part-5.md" >}}) 

[Part 6 - Simple Serialization of Polymorphic Collections]({{< ref "JsonStorage-Part-6.md" >}}) 

Part 7 - Advanced Serialization of Polymorphic Collections"