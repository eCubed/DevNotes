# Publishing a Library to NuGet

We will assume that we have already registered with NuGet and obtained an API key so that we can publish.

We are using Visual Studio 2017 to develop our libraries, and we will use it as well to manage our libraries'
NuGet Packages.

# Preparing a Project
Once we have tested our code and prove that everything works, we may now prepare for deployment to NuGet.

1. Set build to Release
2. Build the library to ensure that there are no errors.
3. Ensure that the library does NOT have any local, "home-brewn" project dependencies. If the project's only
dependencies are from Microsoft, and/or third-party NuGet packages, our library is ready to be deployed. If
the project has other local project dependencies, that local dependency must first be published to NuGet,
and then referenced in our project as a NuGet package.
4. Go to the project's Properties Page (right click on project, Properties). Under Package (left nav), we can
fill out the package form. Pay close attention to the Package version and Assembly and File versions. If it's
the project's first time to be published, we typically start with a package version of 0.0.1 or 1.0.0. The
starting version can be anything. However, the package and assembly versions MUST be manually incremented
on subsequent publishings to NuGet.
5. When ready to create the NuGet package, check Generate NuGet package on build. This will create 
LibraryName.0.0.1.nupkg into the project's bin/Release folder. We are now ready to publish.

# Publishing a Project (Library)
1. Open up command prompt and CD to the project's bin/Release folder.
2. Type `dotnet nuget push LibraryName.0.0.1.nupkg -k <your API key> -s https://api.nuget.org/v3/index.json`, 
placing the API key issued after the -k flag. The value after the -s flag is always going to be `https://api.nuget.org/v3/index.json`.
3. Hit Enter. If all goes well (you are connected to the Internet, the API key is correct, and the .nupkg file exists), there will
be a message saying that it is pushed to NuGet.

# Referencing the Published NuGet Package
Unfortunately, we cannot access a NuGet package right away once it's been published. It must first go through validation, and then
indexing at NuGet before it may be searched and referenced. The process takes somewhere between 12 minutes to 1 hour.

If logged into NuGet, under "My Packages", we can see the status of our newly-published project. It will indicate whether it's still
validating, has validated but is still being indexed, and finally, ready for use.

Once indexing is complete, we can now search for our NuGet package from Visual Studio (Manage NugGet Package), and install
it as a dependency to any project.

If a project previously referred to an older version of our NuGet package, the Manage NuGet Package screen will prompt us of
any new versions of a currently referenced package. The Update tab will have a small number in a blue square, which indicates
how many NuGet packages referenced has a newer version out. From here, we can update our packages.

## Creating the .nuspec File.