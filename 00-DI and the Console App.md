# Dependency Inection in the Console App

In a web application project, we set up dependency injection in `Startup.cs`, and simply inject services
as interfaces in `Controller`, fellow service, and middleware constructors as well as additional parameters
of a middleware's `Invoke()` function and `Startup.cs`'s `Configure()` function.

It is possible to perform DI in a Console App, however, we have to do it a bit differently because we don't have
a `Startup.cs` and we don't readily have separate components (like controllers and middleware) that the framework
can instantiate and inject services to.

We will need to address how to do the following:

* Read the contents of `appsettings[*.dev|*.prod].json`
* Find out whether we are in dev vs. production mode
* Pair up concrete services with their interfaces at initialization code so that we may later access the services
by their interfaces.
* Access services by their interfaces from anywhere in our console app without automatic constructor injection.

## Getting Started

Let's go ahead and create an Asp.NET Core Console app. We will start with the following "skeleton" code:

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
/* other using statements not shown for brevity */

namespace MyConsoleApp
{
  class Program
  {
    public static IConfigurationRoot Configuration { get; set; }
    public static ServiceProvider ServiceProvider { get; set; }
    
    private static void Configure()
    {
      /* We will perform the DI setup here. */
    }

    static void Main(string[] args)
    {
      Configure();

      /* The rest of the console app code. */
    }
  }
}
```

Since we need to access settings from `appsettings.json` such as a database connection string, we will
need to keep an `IConfigurationRoot` object. We will later see how the instance of a configuration root
is created.

This is a console app, so the framework will not instantiate any class (besides the main program itself)
in the background where it would've had the opportunity to inject services into constructors. Instead,
we will need to keep an instance of a `ServiceProvider`. It is through this object where we will fake
the act of DI, AKA, from where we'll obtain the actual instances of the services we'll set up.

We also create the `Configure()` function. It is totally optional, but it's a good organizational practice.
In the `Main()` function, the first thing we'll do is to call `Configure()`, then we can go on with the
rest of the app.

## Dev vs. Prod

We will sometimes need to know whether the environment is "Development" or "Production". This is optional
for our purposes of demonstration, however, your app may need to perform certain code depending on what
the environment value is.

```csharp
private static void Configure()
{
  string environmentName = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
}
```

We access the environemt value via the static class `Environment`, part of `System`, as in `using System;`.
The key for the environment name is `ASPNETCORE_ENVIRONMENT`.

In this example, we stored the environment name in a local variable inside `Configure()`. Your app may need
to access the variable after `Configure()` executes, so you may put `environmentName` as a member of the
`Program` class.

We need to go to our project properties and manually add a key-value pair for `ASPNETCORE_ENVIRONMENT`.
This is automatically added when we create a fresh web app application, but not for a console app. Right-click
on the project in the Solution Explorer, go to the Debug Tab, and add the `ASPNETCORE_ENVIRONMENT` variable
and set the value to `Development`. Once set and we run the application in debug mode, the value of `environmentName`
becomes `Development`. When running in Release mode (production), we should get the value of `Production`. The
framework will provide that for us.

Note that in a web app, the environment is passed to us via `IHostingEnvironment`, which the framework provides
in `Startup.cs` if we ask for it in the `Startup.cs`'s constructor. This is a console app, so we must access
the environment via `System.Environment` instead.

## Setting up the Configuration

Before we are able to read from `appsettings.json`, we will need to create an instance of a builder so we can
build our configuration. This is exactly how we would do it in a web app's `Startup.cs`.

```csharp
private static void Configure()
{
  ...
  var builder = new ConfigurationBuilder();

  builder.SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
    .AddEnvironmentVariables();

  Configuration = builder.Build();

  ...
}
```

## Setting up Services

In a web application, an instance of an `IServiceCollection` is provided to us as a parameter of the
`ConfigureServices()` function. The runtime of the console app doesn't call a `ConfigureServices()` function,
so we won't get an `IServiceCollection` instance. We have to instantiate it ourselves:

```csharp
private static void Configure()
{
  ...
  var services = new ServiceCollection();
  
  services.AddSingleton<ISomeService, SpecificService>(Configuration);
  /* More service interface-to-concrete-class declarations as needed */

  ServiceProvider = services.BuildServiceProvider();
  ...
}
```

We create our own instance of a `ServiceCollection`, which is an implementation of `IServiceCollection`. We
add services to it via the `Add` functions. Since we are in a console application, there needs to be only one instance
of each service, so we're fine with `AddSingleton`, unlike a web application that gets multiple requests that may necessitate service instances per request
instead of one instance as long as the website is up and running.

After we've added all of our necessary services, we must now build the `ServiceProvider` instance which we'll later
use to access our services.

## Setting up an EF DataContext

We need to place the EF DataContext setup code still in the `Configure()` function, but we need to ensure that we place
it *after* both `Configuration = builder.Build();` and `var services = new ServiceCollection();`.

```csharp
{
  ...
  Configuration = builder.Build();
  ...
  var services = new ServiceCollection();
  
  services.AddDbContext<MyDbContext>(options =>
  {
    options.UseSqlServer(connectionString, opts => {
      /* opts.UseRowNumberForPaging(); */ // Set this if using MSSQL 2008 and earlier
    });
  });

  /* Include the code below if you will be needing Microsoft Identify functionality in your app.
  */
  services.AddIdentity<MyUser, MyRole>()
                .AddEntityFrameworkStores<MyDbContext>()
                .AddDefaultTokenProviders();

  ServiceProvider = services.BuildServiceProvider();
  ...
}
```

This is very similar to how we would set it up in a web application.

## Accessing a Service

We will typically access services in the `Main()` function:

```csharp
static void Main(string[] args)
{
  ...  
  MyDbContext db = ServiceProvider.GetService<MyDbContext>();
  ISomeService specificService = ServiceProvider.GetService<ISomeService>();
  ...
}
```
If we ever need to access any service declared in `Program.cs`'s `Configure()` function, from anywhere in our app,
we will need to manually pass `ServiceProvider` everywhere. In both a web and a console application, we write classes 
whose constructors contain interface-typed parameters representing our services. Unlike a web app, though, the runtime
won't automatically instantiate and dependency-inject for us - we now have to explicitly instantiate and pass parameters
ourselves!

## Full Listing of a Base Console App with DI Setup

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
/* other using statements not shown for brevity */

namespace MyConsoleApp
{
  class Program
  {
    public static IConfigurationRoot Configuration { get; set; }
    public static ServiceProvider ServiceProvider { get; set; }
    
    private static void Configure()
    {
      string environmentName = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");

      var builder = new ConfigurationBuilder();

      builder.SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddEnvironmentVariables();

      Configuration = builder.Build();

      var services = new ServiceCollection();
  
      /* We set up the database first because the services we need might depend on it.
      */
      services.AddDbContext<MyDbContext>(options =>
      {
        options.UseSqlServer(connectionString, opts => {
          /* opts.UseRowNumberForPaging(); */ // Set this if using MSSQL 2008 and earlier, and app has pagination queries
        });
      });

      /* Include the code below if you will be needing Microsoft Identify functionality in your app.
      */
      services.AddIdentity<MyUser, MyRole>()
        .AddEntityFrameworkStores<MyDbContext>()
        .AddDefaultTokenProviders();
        
      services.AddSingleton<ISomeService, SpecificService>();
      /* More service interface-to-concrete-class declarations as needed */

      ServiceProvider = services.BuildServiceProvider();
    }

    static void Main(string[] args)
    {
      Configure();

      ISomeService someService = ServiceProvider.GetService<ISomeService>();
      MyDbContext db = ServiceProvider.GetService<MyDbContext>();

      /* The rest of the console app code. */

    }
  }
}

```
