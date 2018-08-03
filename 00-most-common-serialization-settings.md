# Most Common Serialization Setting

The typical serialization/deserialization setting we want follow the following rules:

* Camel-cased property names
* Do not include a property in serialization if its value is null or default

We will set up the `JsonSerializer` object, which describes how we would serialize or
deserialize an object or string, respectively:

```csharp
JsonSerializerSettings jsonSerializerSettings = new JsonSerializerSettings
{
  ContractResolver = new CamelCasePropertyNamesContractResolver(),
  DefaultValueHandling = DefaultValueHandling.Ignore  
};
```

## Serialization in Code (on the fly)

```csharp

SomeObject data = new SomeObject(...);

string serialized = JsonConvert.SerializeObject(data, new JsonSerializerSettings
{
  ContractResolver = new CamelCasePropertyNamesContractResolver(),
  DefaultValueHandling = DefaultValueHandling.Ignore,
  NullValueHandling = NullValueHandling.Ignore
});
```

## Deserialization in Code (on the fly)

We use the same exact `JsonSerializerSetting` to be on the safe side. We need to let the deserialization
process know that the incoming serialized JSON string has camel-cased property names and to expect that
it won't include properties whose values, when deserialized, are default per type.

```csharp
string serialized = "{...}";
SomeObject deserializedObject = JsonConvert.DeserializeObject<SomeObject>(serialized, new JsonSerializerSettings
{
  ContractResolver = new CamelCasePropertyNamesContractResolver(),
  DefaultValueHandling = DefaultValueHandling.Ignore,  
  NullValueHandling = NullValueHandling.Ignore
});

```

## Setting in an MVC or Web API application

When we want to return (serialization) or take in as payload (deserialization) JSON data, and we want to
apply the above rules, we set up serialization/deserialization in the web application's `Startup.cs` file's
`ConfigureServices` function:

```csharp
public void ConfigureServices(IServiceCollection services)
{
  ...  
  services.AddMvcCore().AddJsonFormatters(jsonSerializerSettings => {
    jsonSerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver(),
    jsonSerializerSettings.DefaultValueHandling = DefaultValueHandling.Ignore,  
    jsonSerializerSettings.NullValueHandling = NullValueHandling.Ignore
  });
  ...
}
```

The only difference is that we don't instantiate for ourselves a `JsonSerializerSettings` object. It
is handed to us in an `Action` for us to customize how JSON data is returned and received.
