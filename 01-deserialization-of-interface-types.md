# Newtonsoft - Deserialization Involving Interface Types

When we deserialize a json string, yes, we deserialize to an exact object. However, that object may contain
properties (directly children or any descendant) that are typed as an interface. For serialization, this is
not a problem since we're just writing any object to JSON. The problem is when we try to deserialize that
JSON string. When the deserialization comes across an object and sees that it's supposed to be of an interface
type, it wouldn't know which concrete class to instantiate. This becomes a problem especially if there are
2 or more classes that implement a given interface with the same exact set of properties, but whose common
methods are implemented differently.

The approach we chose will not add any metadata to our JSON as that could result in unwanted bloat and reveal
implementation details, which is considered a security issue. We will basically need to write a class that
will answer the question "If I (the deserializer) encounter an object that is of an interface type, which
exact concrete class (that implements that interface) would I need to instantiate?" This way, we will not
see "exact-type-to-deserialize-to" all over our JSON when we serialize it.

# Considerations
There are a few things to take into account when we attempt to answer that question of to which exact type
to deserialize to.

When we write our class that handles deserialization of an interface type, we are handed a `JObject` which
is an object that is contains all of any JSON object's property names and values (including
descendants). The `JObject` can literally contain data from an JSON string, however, working with it
directly in "the meat of our code" is tedious and error-prone. However, at our interpreter class, which
isn't considered "the meat of our code" - more like a utility, we will need to "sniff" presence of
certain properties and/or examine certain values of known properties (properties required by the interface).
This means that we will need to have knowledge of ALL of our classes that implement that particular interface,
because we will deserialize to ALL of them in this class we're about to write.

For all of this to work, though, we need to make sure that the pure JSON alone (without type meta-data),
should have sufficient information to give us a clue on which concrete class to deserialize. We already
mentioned looking for property names and investigating values of properties. If two or more of our distinct
classes have the same exact properties (no more than the other), and there is no way for us to deduce
the exact type from examining those property's values, this won't work. In most cases, though, we will be
able to tell to which concrete class to deserialize a JSON object to.

# Creating an Interface's Custom JsonConverter Class

We will be creating a `JsonConverter` class for a property `List<ISomeInterface>` for an object that
has a property of type `List<ISomeInterface>`. Be aware that the code written here is very general, thus
not a live example. We start with the code that we will always start with:

```csharp
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public class SomeInterfaceConverter : JsonConverter
{
  public override bool CanWrite => false;
  public override bool CanRead => true;

  public override bool CanConvert(Type objectType)
  {
    return objectType == typeof(ISomeInterface);
  }

  public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
  {
    throw new InvalidOperationException("Use default serialization.");
  }

  public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
  {
    /* We will write the interpreter here */
  }
}
```

The above is the skeleton of a custom `JsonConverter` class. We set the `CanWrite` to false because we're not concerned about serialization.
Since we're concerned about deserialization, `CanRead` is set to true, since we will be reading a JSON string. We will always have that
`CanConvert` function as written. We will also override that `WriteJson` method to make it return an exception because we're not actually
intending to write a JSON string. `WriteJson` is read by the serialization process, and when it encounters the exception, it will know to
"serialize as normal".

`ReadJson`, though is where we will write the custom code that interprets that JSON string:

```csharp
  public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
  {
    var someInterfaceList = new List<ISomeInterface>();
    var jArray = JArray.Load(reader);

    foreach(var item in jArray)
    {
	  var jObject = item as JObject;
	  var someInterfaceObject = default(ISomeInterface);

	  if (jObject["propertyOnlyInImplementorA"] != null)
	     someInterfaceObject = new ImplementorA();
	  else if (jObject["propertyOnlyInImplementorB"] != null)
	     someInterfaceObject = new ImplementorB();
	  else if (jObject["propertyOnlyInImplementorC"] != null)
	     someInterfaceObject = new ImplementorC();
	  
	  serializer.Populate(jObject.CreateReader(), someInterfaceObject);
	  someInterfaceList.Add(someInterfaceObject);
    }

    return someInterfaceList;
  }
```

Since we are dealing with an `List<ISomeInterface>` in some class, we will need to unpack the JSON string into a `JArray`:
`var jArray = JArray.Load(reader);`.

The `ReadJson` expects us to return an object, and it will be a `List<ISomeInterface>`, so `var someInterfaceList = new List<ISomeInterface>();`.

Now, we go through each item in the `jArray`. We have to "cast" the item (which is a `JToken`) as a JObject: `var jObject = item as JObject;`.

Then we need to "instantiate" an "instance" of an interface. For now, let's regard that the CLR knows how to create some "phantom" object that implements
an `ISomeInterface` with the `default` operator.

We then examine the `jObject` for the presence of a particular property (in this general example) - a particular property that only one of the classes
that implement `ISomeInterface` has that none of the others have. Depending on the presence of a particular property (by name), we instantiate an
actual concrete class, for example, `someInterfaceObject = new ImplementorA();`.

Once we've figured out a concrete class, we will then need to fill that class with the values in the jObject: `serializer.Populate(jObject.CreateReader(), someInterfaceObject);`

Then, once `someInterfaceObject` is populated, we then add it to `someInterfaceList`.

Finally, we return the entire list, which is still a `List<ISomeInterface>`, but it's now populated by the different class instances!

## JsonConvert Attribute

Now that we have written a custom `JsonConverter`, we can now declare it in one of our classes that has a `List<ISomeInterface>`.

```csharp
public class MyObject 
{
  public string Property1 { get; set; }
  public string Property2 { get; set; }

  [JsonConvert(typeof(SomeInterfaceConverter))]
  public List<ISomeInterface> SomeInterfaceObjects { get; set; }
}
```

We can serialize and deserialize as usual, without the `JsonSerializerSettings` setting that would "pollute" `$type` all over our JSON.

## Interface Property, not List<ISomeInterface>

The general process is the same as the list version explained above. However, now, we want to read and return an object that implements
`ISomeInterface`, not a `List<ISomeInterface>`. The difference is in the `ReadJson` function.

```csharp
  public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
  { 
    var jObject = JObject.Load(reader);
    var someInterfaceObject = default(ISomeInterface);

    if (jObject["propertyOnlyInImplementorA"] != null)
      someInterfaceObject = new ImplementorA();
    else if (jObject["propertyOnlyInImplementorB"] != null)
      someInterfaceObject = new ImplementorB();
    else if (jObject["propertyOnlyInImplementorC"] != null)
      someInterfaceObject = new ImplementorC();
 
    serializer.Populate(jObject.CreateReader(), someInterfaceObject);

    return someInterfaceObject;
  }
```

We still perform the same "property-sniffing" as we did in the list version.

However, this kind of converter must be declared on a property that is of `ISomeInterface`, NOT `List<ISomeInterface>` like we've done before:

```csharp
public class MyObject2 
{
  public string Property1 { get; set; }
  public string Property2 { get; set; }

  [JsonConvert(typeof(SomeInterfaceConverter))]
  public ISomeInterface SomeInterfaceObject { get; set; }
}
```

## Afterthought

One of the issues that may stick out is that we had a finite number of concrete objects to deserialize something to. In our example, what if there was
actually a fourth concrete class that we forgot to "property-sniff"? Our converter wouldn't have recognized it and we would have ended up with an exception.
This is why we need to know and handle ALL possible concrete classes in our interpreter, and should we add a new class that implements `ISomeInterface`,
we will need to update our converter class!

Our example only examined the presence of properties by their name. We did not show an example that examined a value of a common property that all implementors
share. We will leave that to you as an exercise. One hint, though, as what's commonly done, is to provide a `string Type { get; set; }` property at
`ISomeInterface`, and each implementor class will need to set it to an exact value. When we deserialize, we would no longer need to "property-sniff".
Instead, we can examine the *value* of the `Type` property. It seems quite redundant because the value of `Type` most likely will reflect the class name,
if not, be exactly the class name. This seemingly redundant property may also be helpful for the front-end so that it only needs to examine one property
to determine how to render the UI for the particular class.