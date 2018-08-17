# User Secrets

We have probably stored sensitive API keys, Secrets, and connection strings into that appsettings.json file and checked it
into a repository. Even if we untracked that .json file from our repo, all that sensitive information still made it to
earlier commits. From now on, we shall NEVER store anything sensitive in appsettings.json. Sensitive stuff include, but are
not limited to:

* database connection strings
* API Keys and Secrets
* local hard drive folders
* Encryption/Decryption keys

So what's left, then, that we could save in appsettings.json?

* Form Labels and prompt text
* Short messages that will appear on more than one page

Note: The following is for a machine that has Asp.NET Core 2.O SDK or above installed.

## Creation of the Secret Configuration POCO

We will first create a secret configuration POCO:

```csharp

public class SecretConfiguration
{
  public string ApiKey { get; set; }
  public string ApiSecret { get; set; }
  public string ConnectionString { get; set; }
}
```

This class will make it convenient for us to access the values later.

## Setting up the App to Read Secret Configuration

When the web app was first created off a VS template, it was automatically given a unique secrets GUID. We could observe this
if we right-clicked the project and chose to edit the project's .csproj file. It is under the `PropertyGroup` node:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <UserSecretsId>02f6b775-17b2-4e75-fca9-b07f1ca1e2e3</UserSecretsId>
  </PropertyGroup>
  ...
</Project>
```

Yes, this GUID will be checked into the repo. Yes, this will be the app's unique secrets id on any machine it would be cloned
to. No, this is not considered sensitive information.

### secrets.json
Our application will have a secrets.json file associated with it. No, we did not create this - VS and the operating system did upon
creating our project. No, this file is NOT saved with the project so that it does not get checked into a repo ever. It is a file in a
special location on the developer's machine that both VS and the OS knows about. That user secrets id value is actually the name of
a folder somewhere on the hard drive, which would ensure us that we are properly editing the correct secrets.json file. The name of
this file has to be exactly secrets.json.

If we right-click on our project, and chose "Manage User Secrets", an editor window for secrets.json will show. We can fill this file
with JSON specific to our application:

```json
{
  "SecretConfig": {
    "ApiKey": "LMNOP",
    "ApiSecret": "12345ABC",
    "ConnectionString": "The secret connection string"
  }
}
```

Notice that the .json file is an object with one property that holds our secret configuration data. This is not absolutely required,
but we will need to do it for our purposes. In case we are using a strongly-typed configuration object from appsettings.json, we need
to do this to specifically pick out the secret configuration object. Be aware that the JSON in both secrets.json and appsettings.json
are merged into one object, and we access values via `Configuration["somePropertyName"]` where `somePropertyName` may be in either
.json file.

So now, how do we access the contents of secrets.json? We need to access it from Startup.cs, and then register it as a singleton so that
we can grab it via constructor injection from many places in our app:

```csharp
public void ConfigureServices(IServiceCollection services)
{
  ...
  /* Instantiate blank secret configuration object
  */
  var secretConfig = new SecretConfiguration();

  /* Fill the secret configuration object. Configuration will find that object by the provided key from either appsettings.json
     or secrets.json, and once it finds the property, it will save the JSON data to our object. In production, ASP.net also looks
     for environment variables that have been named in a certain way that reflects the object hierarchy of the SecretConfiguration
     class, and if the system finds those environment variables, they will take effect and override anything from any
     .json file we create.
  */
  Configuration.Bind("SecretConfig", secretConfig);

  /* Add the secret configuration object to the DI container so we can access it later from anywhere via constructor injection
  */
  services.AddSingleton(secretConfig);
  ...
}
```

## User Secrets in Production - Environment Variables

Microsoft intends for us to use user secrets ONLY for development. Now, our project will have scattered around it DI-injected
secret configuration object references. When our app is finally deployed to production, the app will not look for a secrets.json
file. We will need to simulate the contents of the development secrets.json file via system environment variables.

When the app is running in production, Asp.NET Core not only looks in appsettings[.environemntName].json files, but also in
the system environment variables. There is a way in system environment variables to replicate JSON object hierarchy. If we look
at our secrets.json file, we have a top-level "SecretConfig" object, and it has the properties "ApiKey", "ApiSecret", and "ConnectionString".
Those properties and their values translate to "SecretConfig__ApiKey", "SecretConfig__ApiSecret", and "SecretConfig__ConnectionString".
The double underscore is the "syntax" that Asp.NET Core will interpret as the dot in code to access property values. So, we can go ahead and
add those variables to whatever system we're going to deploy the app. We may need to consult OS-specific ways to set environment
variables.

For Windows, though, we will need to run in the Command Prompt `iisreset /noforce` so that IIS will immediately recognize any changes
we made when setting system environment variables. As a caution, resetting IIS may impact other websites already running in the system,
so system-wide planning will be necessary. This `iisreset`, even along with starting and stopping the web app and corresponding app pool
in IIS, will only detect newly-set or edited environment variables, which is what we want. However, removing system variables and then running
these commands does not work - those deleted system variables are still around and won't truly get deleted until we reboot the machine.

Warning: If you set environment variables to test a production deployment ON THE DEVELOPMENT MACHINE, those environment variables will
win (AKA, be the ones that will actually apply), EVEN IN DEVELOPMENT/DEBUG MODE. Yes, we may set secret information in the regular appsettings
files, or even in secrets.json, but values there will ultimately not make it when running the app in debug mode. Asp.NET Core forces the 
environment variables to take effect if found, and will override anything set in any .json settings file!

Heads Up: We have generically named those environment variables just to demonstrate how to set them up and to see that everything works.
The production machine may have multiple websites running on it, who may also need to have similar confidential entries in the system.
We will probably need to name our environment variables to also reflect the app that reads it. This means that we would also need to
construct our secret POCO to reflect the environment variable naming.

In practice, though, we would and should NOT be developing on the same physical machine as production. On the dev machines, we would have
those secret.json files, already sorted out per application. On the production machine, we will have the corresponding environment variables.

## User Secrets in Production - No Control of Environment Variables

What if we need to deploy our app to a server like a shared host, or to one where we do not have access to the OS to set environment variables?
Then we will need to copy the secrets json object from secrets.json and paste it as a top-level object in appsettings.production.json. Now, this
situation is not ideal because it is still highly possible that someone will make the mistake of checking in appsettings.production.json to
source control. For now, let us assume that we've guarded this file not to make it to source control. When we republish and we get a new
appsettings.production.json file, now with the secret json object, the production app will apply those to our secret configuration object
we previously set up in Startup.cs.

