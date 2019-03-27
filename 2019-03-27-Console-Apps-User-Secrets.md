# Console Apps and User Secrets

When we develop a console app, it is possible to use user secrets for development. However, Visual Studio's UI doesn't allow us to conveniently set up user secrets for
a console app as it does with a web app. That is, the console app's context menu doesn't have the option "Manage User Secrets", which, for a web app, does a few things -
(1) generate a GUID and automatically add that to the app's .csproj file, and (2), create the file `secrets.json` for our app outside the project. Unfortunately, we will
need to do all of this by hand for a console app.

# Setting Up User Secrets For a Console App

I am on a Windows machine. The basic procedures may be similar for other platforms, only differing where each puts each app's `secrets.json` file.

First, we will need to obtain a GUID. We can go to [this website](https://www.guidgenerator.com/online-guid-generator.aspx) to generate one. We need not to worry about
possibly duplicating another app's GUID on our machine since that is practically unlikely to happen. Once a GUID is generated, we will need to copy it (Ctrl-C) because
we will use it shortly.

We will need to open our console app's `.csproj` file and add the `<UserSecretsId>` tag under `<PropertyGroup>`.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.1</TargetFramework>
    <UserSecretsId>aac02bd1-0dc0-4b98-84ff-65fce1690f1a</UserSecretsId>
  </PropertyGroup>

  ...
</Project>
```

Save.

Now, we'll need to create a folder in a special location on our hard drive, whose name is *exactly* the GUID we generated. I am on Windows 7, so the location for the this
main secrets folder is `C:\Users\MY_USERNAME\AppData\Roaming\Microsoft\UserSecrets` where `MY_USERNAME` would be your Windows username of the account you're logged
into for developing ASP.net Core apps.

We will need to create `secrets.json` at the following location: 

`C:\Users\MY_USERNAME\AppData\Roaming\Microsoft\UserSecrets\aac02bd1-0dc0-4b98-84ff-65fce1690f1a\secrets.json`.

Once created, we will want to keep this file open in a text editor, and we can open it from inside VS, though we'll have to hunt down this new file via the Open File dialog.

Right now, our app will read configuration from this `secrets.json` file.

# Telling our App to Read `secrets.json` when Debugging

First, we'll need to add an environment variable. Right-click on the app's root in the Solution explorer > Properties > Debug Tab. We will need to add key
`ASPNETCORE_ENVIRONMENT` with the value `Development`. Note that this step is automatically done for us when we create a web application. We need to manually create
this entry for a console app!

Then in our builder code in `Program.cs`, we call `AddUserSecrets():`

```csharp
private static void Configure()
{
    var builder = new ConfigurationBuilder();

    builder.SetBasePath(Directory.GetCurrentDirectory())
        .AddUserSecrets<Program>()
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true);
           
    Configuration = builder.Build();

	...
}
```

The `.AddUserSecrets<Program>()` function will tell our builder to go look for the appropriate `secrets.json` file if it exists. If it doesn't exist, it will then attempt
to find `appsettings.json`.

We have somewhat of an atypical setup here - apply user secrets when in development, and apply `appsettings.json` for production. We've decided not to create an `appsettings.json`
file in-project because then it would win over `appsettings.json`.

We don't address the confidentiality of `appsettings.json` here as we are only demonstrating. In actuality, we would set up environment variables in our system, use the function
`.AddEnvironmentVariables()`, but that's a topic for another article since it's quite involved.

# Deploying the App for Production

We will need to change "Debug" to "Release" on the toolbar, then build our application to ensure there are no errors. 

We publish (right-click on the app's root on the Solution Explorer > Publish). On the dialog, we can choose the output folder, but we'll accept the auto-generated output path.
We then click publish on this dialog, but the actual publishing didn't yet occur.

The publish settings on the Publish tab lists `Target Location`, which is the output path we just set, `Configuration` set to `Release`, `Target Framework` set to 
`netcoreapp2.1` or `netcoreapp2.0`, whichever one we use, and `Target Runtime` set to `Portable`. For now, we'll want to leave it at `Portable` which will generate a
minimal number of files for the publication. The `Portable` version means we can copy the output folder's contents even to a different machine of a different platform, as long
as the destination machine has a suitable Asp.NET Core framework installed.

To test the production deployment, we'll go to the command prompt and CD to our app's publish output folder, in our case `bin/release/netcoreapp2.1/publish` folder, and
run the app with `dotnet MyApp.dll`.

Interesting - at this point, the published app still applies the values from `secrets.json`. That's because we're still running it on the same dev machine. To fix this, we'll need
to manually create `appsettings.json` on that publish folder.





