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
     ...
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
     ...
   }
}
```

That's it! Easy-peasy!

# Usage

The unique configuration object is now ready for injection in controllers, middleware, and other services that may need it.
We simply include it as its actual type, not interface as we do with services, as a parameter. Here is an controller
example that requests it:

```csharp

public class ValuesController : Controller
{
   private MyAppConfiguration MyAppConfiguration { get; set; }

   public ValuesController(MyAppConfiguration myAppConfiguration) 
   {
     MyAppConfiguration = myAppConfiguration;
   }
}
```

We can now access that configuration object as an object easily - we'll get Intellisense! We used to have to type 
`Configuration["Key"]` where we usually forgot what the actual key was so we'd have to look up what we put down
in the appsettings.json file. This was tedious and error prone. We would've had to convert to boolean or number
when we did it the `Configuration["Key"]` way, which was additional typing. Now, we get non-strings as their
actual type!

