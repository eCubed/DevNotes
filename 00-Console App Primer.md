## ASP.net Core Console App Primer

Setting up a base console app goes beyond just adding a console app project via the VS UI. The following are
settings for `netcoreapp2.0`.

## Installing Microsoft.AspNetCore.All

We will need to Nuget-install `Microsoft.AspNetCore.All` to our application. For this article, we chose version
2.0.5, since our app is on `netcoreapp2.0`. (Verify later) This will make our app ready for EF, Linq, and other 
basic functionality. Also, when we deploy, only .dlls not found in the machine's ASP.net Core SDK will be created,
thus reducing the deployed size of our application.

## Configuration and appsettings.json

When our app needs to read settings from a file, we will need to set up configuration:

```csharp
using Microsoft.Extensions.Configuration;
using System;
using System.IO;

namespace ConsoleAppStudy
{
  class Program
  {   
    public static IConfigurationRoot Configuration { get; set; }
    
    private static void Configure()
    {
      var builder = new ConfigurationBuilder();

      builder.SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddEnvironmentVariables();
      
	  Configuration = builder.Build();
    }

    static void Main(string[] args)
    {
      Configure();
      Console.ReadLine();
    }
  }
}

```

We created a `Configure()` function for more organized code.

We instansiate a `ConfigurationBuilder` to get started.

We then read the contents of `appsettings.json` so we may extract values from it later.

We need to create the file `appsettings.json`. Once copied, though, we need to right-click on it in the
Solution Explorer, Properties (last item of the popup context menu), and then set "Copy to Output Directory" to
"Copy if newer". This setting will copy `appsettings.json` when we build an eploy the app.