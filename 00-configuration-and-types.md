# ASP.net Core Configuration and Types

We basically know how to set up configuration - the one that reads from appsettings.json. So far, our appsettings.json file
contains an object with only one level of properties, and all of those properties are string values. It seems natural that
a settings .json file should be able to contain properties of number and boolean types, arrays, and complex objects (themselves
with their own properties, complex object properties, and arrays). We only know how to access the root properties of a
configuration with `Configuration["propertyName"]`, and it returns a string. How can we read entries of appsettings.json
into an actual CLR object!?

# Creating our CLR Objects
We imagine that what gets read from appsettings.json needs to be deserialized, so, this means we must create our CLR objects!

Let us adhere to the convention of appending `Configuration` to our classes intended to reflect the contents of appsettings.json.

# Startup.cs of a Web Application

```csharp
public class Startup
{
   private IConfiguration Configuration { get; set; }

   public Startup(IConfiguration configuration)
   {
     /* We didn't know that configuration can be injected in Startup.cs's constructor.
        We used to explicitly build the configuration with a ConfigurationBuilder,
        which was quite an amount of code.
     */
     Configuration = configuration;
   }

   public void ConfigureServices(IServiceCollection services)
   {          
     /* Instantiate a blank/default instance of the app's unique configuration object.
     */
     var config = new MyAppConfiguration();

     /* Fill thhe newly-instantiated config object with corresponding values from appsettings.json.
     */
     Configuration.Bind(config);

     /* Register the very config object as a singleton so that we can inject it in controllers, middleware,
        and other services (listed after this services.AddSingleton(config) listing), as the very type itself,
        not as some interface.
     */
     services.AddSingleton(config);

     Configuration.Bind()
   }
}
```

That's it! Easy-peasy!