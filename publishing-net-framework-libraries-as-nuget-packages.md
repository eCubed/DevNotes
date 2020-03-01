# Publishing .net Framework Libraries as Nuget Packages

Unfortunately, publishing .net Framework libraries isn't a easy as publishing .net Core ones. There isn't really a UI way to do it. For .net Core, we can just turn on a
checkbox that would create the .nupkg when we build the project. There isn't this option in VS 2019 for .net Framework libraries!

But First, let's do what we can with the UI available to us. At this point, we would have already created our .net Framework library and we know that it functions as
expected. We'll right-click on the project root, Properties, choose the Application tab, and click the `Assembly Information...` button. A dialog pops up. On here, we can see
that a few fields are pre-populated. We'll leave those as they are. We may want to provide a description and company name because those would show up on other people's
Nuget package manager screens. Now, we'll need to note the assembly version. When it's the first time we're about to publish to Nuget, both assembly and file versions are
1.0.0.0. We can leave it this way, but the important thing to remember is we will need to increment the numbers here accordingly for each time we publish a new version to
Nuget. We won't delve into the semantics of versioning right now. It's just important to remember that we'll need to increase the values here for every time we want to
release a new version of our new library.

Ok, so we'll now need to set to `Release` and build our library. This produces a .dll file in our project's `/bin/Release` folder whose name matches the name of our project.

## Creating the .nuspec File

We'll need to cd back to the root of our project where the .csproj file is and then run the command:

`nuget spec MyAssembly.csproj` where `MyAssembly.csproj` is the full file name of the .csproj file.

This creates a .nuspec file, which contains the following:

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>$version$</version>
    <title>$title$</title>
    <authors>$author$</authors>
    <owners>$author$</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2020</copyright>
    <tags>Tag1 Tag2</tags>
  </metadata>
</package>
```

That's the default-generated .nuspec. We cannot just build the package with these values. We have to change them. Now, licenseUrl, projectUrl, and iconUrl are actually
optional, and if we don't have values for those, we'll have to remove the tags. We cannot keep these tags with empty text values.

We will need to change `$version$` to the actual version. Contrary to what actually happens, `$version$` actually does NOT pull the updated values from the `AssemblyInfo.cs`
for file and assembly versions. It is unfortunate that we will MANUALLY need to match the assembly/file version and the version we'll put here.

Also, for some odd reason, `$author$` isn't recognized by Nuget. People have suggested in various places to change this to `$authors$` but that also doesn't work. We are
going to get the error `Authors is required.` So, what we'll need to do is fill the `<authors>` and `<owners>` tags with values. This could be our name or our company's
name, whichever suits the package or whatever we want.

We'll also get warnings for keeping tag value `Tag1 Tag2` and releaseNotes having `Summary of changes made...`. We'll need to change these as well.

Here's the xml file edited with valid values:

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>$id$</id>
    <version>1.0.0</version>
    <title>$title$</title>
    <authors>Me</authors>
    <owners>Me</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>$description$</description>
    <releaseNotes>First release</releaseNotes>
    <copyright>Copyright 2020</copyright>
    <tags>first library</tags>
  </metadata>
</package>
```

## Building the Package

Before we build the package, we'll need to edit something in the .csproj file. In the first `<PropertyGroup>` Tag is the `<Configuration>` tag. We'll need to change
its value from `Debug` to `Release`. This is an unfortunate edit because it is easily forgettable, tedious, and NOT documented in the official Microsoft docs as a necessary
step. If we don't perform this step, we'll end up shipping the Debug library once we're done. Of course, we want the `Release` version.

```xml
 <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Release</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
    ...
 </PropertyGroup>
```

Now, we can go back to the prompt (ensuring we're at our project's root) and run

`nuget pack`

Just like that without specifying a file name. It will generate one for the latest package.

## Publishing the Package

1. We need to ensure we're still at the project root where the .nupkg file is.
2. We type `dotnet nuget push LibraryName.0.0.1.nupkg -k <your API key> -s https://api.nuget.org/v3/index.json`, 
placing the API key issued after the -k flag. The value after the -s flag is always going to be `https://api.nuget.org/v3/index.json`.
3. We hit Enter. If all goes well (you are connected to the Internet, the API key is correct, and the .nupkg file exists), there will
be a message saying that it is pushed to NuGet.
