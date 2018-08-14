# Asp.NET Core Hosting Environment

It will be helpful to know in different places of our project whether the app is running in development vs. production mode.
Whether running on development or production determines which appsettings.config to apply, for example. We will examine a
trivial app that is both running in development and production mode, as well as how to actually run in development vs.
production modes. We shall also assume that we know how to set up strongly-typed configurations.

## Determining Development vs. Production

The framework provides us the `IHostingEnvironment` interface that helps us determine whether the app is running under
development or production. At `Startup.cs`, we can readily access `IHostingEnvironment` by simply adding it as a parameter
in the `Configure()` function. If you did not already know, we can inject services to this `Configure()` function as
additional parameters, as if it were a constructor of a `Controller`, another dependency-injected service, or the `Startup`
class itself.

At the `Configure()` function that takes in `IHostingEnvironment` as a parameter, we typically would want to determine
if it's running under development, and if it is, show developer exception pages (ones containing stack trace that we would
not want to show in production). The VS template for creating a blank web application provides us with the following:\

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
  if (env.IsDevelopment())
  {
    app.UseDeveloperExceptionPage();
  }
  ...
}
```

Like many services, we can also inject `IHostingEnvironment` in a controller. We will only demonstrate this in order to
examine `IHostingEnvironment`'s values. We've created a default `ValuesController` that we've injected `IHostingEnvironment`
to:

```csharp
[Route("api/[controller]")]
public class ValuesController : Controller
{
  private IHostingEnvironment HostingEnvironment { get; set; }

  public ValuesController(MyTopConfiguration configuration, SecretConfiguration secretConfiguration, IHostingEnvironment hostingEnvironment)
  {
    HostingEnvironment = hostingEnvironment;
  }
  
  ...
  
  [HttpGet("{id}")]
  public IActionResult Get(int id)
  {
    return Ok(new { environment = HostingEnvironment });
  }
```

We simply return the property values of the `IHostingEnvironment` instance. The returned JSON is the following, run
under debug mode:

```json
{
  "environment": {
    "environmentName": "Development",
    "applicationName": "MyApp",
    "webRootPath": null,
    "webRootFileProvider": {},
    "contentRootPath": "C:\\path\\to\\MyApp",
    "contentRootFileProvider": {
      "root": "C:\\path\\to\\MyApp"
    }
  }
}
```
CAUTION: When we run the app from inside Visual Studio in release mode, the value of `environmentName` is still "Development"!
It is easy to assume that the release mode is exactly equivalent to as if we're running it on a live server, but that is not
the case! What do we do to make that `environmentName` value `Production`? Unfortunately, we would have to deploy it and see. This is
why, at least in this test application, we directly output the values of `IHostingEnvironment` as the body of an API call.

So, let's go ahead and publish this app, and point IIS to the publish output, and then run. (We did not show the steps on how 
to set that up.) Now, the value of `environmentName` is "Production". 

## appsettings.environmentName.json

We often see companions to the appsettings.json file - one for each environment: appsettings.development.json,
and appsettings.production.json on many people's project repos, and we often see in their Startup.cs file the explicit
instructions to recognize those. We know that the framework will first read appsettings.json, and then look for the
corresponding production or development appsettings file, and merge them if the environment-specific appsettings file is found,
and take the value in the environment-specific file to override anything stated in the appsettings.json file.

However, upon some research, apparently, we do not need to explicitly tell our app to go look for appsettings.production.json 
or appsettings.development.json because the framework already does that for us.

IMPORTANT: When we are about to deploy to production, we must set the "Copy to Output Directory" of those 
appsettings[.environmentName].json files to "Copy if newer". This is done via right-clicking each file and selecting Properties
at the bottom of the popped-up context menu. This will copy those files to the publish output folder.
