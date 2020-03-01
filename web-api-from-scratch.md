# Web API From Scratch

We are going to create the rudimentary Web API web application (.net Framework) from scratch. No, we won't use the template that scaffolds
all the files and references for us.

I'm using VS 2019 for this writeup.

I create an ASP.NET Web Application (.NET Framework) in C#. Note that in VS 2019, we'll have to search for it as it isn't readily availabe. Also,
the dropdowns MUST all have selected "All languages", "All platforms", and "All project types". For some reason, if we filter by C#, the template
we want won't show. Strangely, we need to be careful because there also is a VB option which shows up on search more often, and we don't want
VB. For this app, I'll target ASP.NET 4.5.1.

I'll select "Empty" because I don't want any other features I won't need. I'll also make sure that the checkboxes "Web Forms", "MVC", and even
"Web API" are UNCHECKED!

Now, granted, I've already created a .NET Framework library with all of my classes and data context (for overall organization), and I'll go ahead
and add a reference to it.

Now, I'll need to install `Microsoft.AspNet.WebApi` via Nuget Package Manager, whatever the latest version is. It happens to be 5.2.7 at the
time of this writing. I'll go ahead and accept installation.

I'll also need to install `EntityFramework` via Nuget Package Manager if I need to expose database entries from this app. This installation
will modify web.config, FYI.

In order to be able to connect to my database, I'll need to make the following entry in web.config:

```xml
   <configuration>
    ...
        <connectionStrings>
            <add name="MyStuffConnectionString"
              connectionString="Data Source=My-Machine\SQLEXPRESS;Initial Catalog=MyStuffDotNet;Integrated Security=True"
              providerName="System.Data.SqlClient"/>
        </connectionStrings>
    </configuration>
```

The name `MyStuffConnectionString` should have been already hard-coded in `MyStuffDbContext` in my base library. So, whenever I instantiate an instance of my db context,
it already points to the database associated with the connection string `MyStuff`.

## Creating Configuration Files for Web API

First, we'll need to create `WebApiConfig.cs` under the folder `App_Start`. I'll need to create that folder.

```csharp
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services

            // Web API routes
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
```

I'll then need to create a `global.asax` file at the root of the project. To properly create this file, I will need to right-click the project,
choose Add, then new item, then select `Global Application Class`

```csharp
using MyStuff.DotNet.Server.App_Start;
using System;
using System.Web.Http;

namespace MyStuff.DotNet.Server
{
    public class Global : System.Web.HttpApplication
    {
        protected void Application_Start(object sender, EventArgs e)
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
}
```

I deleted the other event functions but kept `Application_Start`, and put `GlobalConfiguration.Configure(WebApiConfig.Register);` inside it as the single line.

The setup of both of these files tells the whole app that it is a Web API application.

Now, we can create the `Controllers` folder, and create our Web API 2.0 controllers there. If we set up our routes correctly (which we don't cover here), then we should
be able to access them via the expected URLs.

