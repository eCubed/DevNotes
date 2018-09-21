# Publishing and Consuming a Nuget Package

Sometimes, we will want to create Nuget packages that we don't want to publicly share at Nuget.org. It is possible to share Nuget packages within an organization's
network, and install it similar to installing it from Nuget.org.

## Preparation

**Shared Folder on Network Drive** that will contain the organization's Nuget package, and that everyone should have access to it.

**How to Create Nuget Package.** See the article on how to create a Nuget Package [here](publishing-csharp-project.md). We will need to perform the steps upto
creating the .nupkg file. We will not proceed to publish it to nuget.org.

**Let VS Know about Nuget Shared Folder on Network Drive.** On dev machines with VS 2017 installed, we will need to add that shared network folder to Nuget sources.
That is found in Tools (Top Menu) > Options (last/bottom menu item, pops up dialog) > Nuget Package Manager (top-level item on navigation tree) > Package Sources.
From there, we can add the path to the shared folder on the network drive and name it something meaningful.

## Local Publishing

We would first need to publish the package within the project. When the .nupkg file is created as a result of it, we will need to "deploy" the package to the shared
network drive folder we've set up above. Note that we do not simply copy files from our project's output folders. The local Nuget engine requires that whatever is
saved to that shared folder should have a specific file structure. Fortunately, we don't have to manually set that up.

We will need to go to the command prompt, and cd to our project's `/bin/Release` folder where that .nupkg file is. Then we will need to run:

`nuget add PackageName.0.0.1.nupkg -source \\LOCAL-NETWORK-DRIVE/Nuget`

Of course, we would replace `PackageName.0.0.1.nupkg` with our package's actual name and current version, and `\\LOCAL-NETWORK-DRIVE/Nuget` with the actual
drive and path to the Nuget shared folder. That shared folder is considered the local Nuget Feed.

## Consumption

We can now install our locally published package almost identical to how we would find and install a package from nuget.org. We will need to open a different project,
or better yet, another solution. From any project, we right-click on the Dependencies node and select Manage Nuget Packages.. as usual. On the upper-right-hand
corner of the Nuget pane, there is a Package source drop-down, and nuget.org is most likely going to be selected first. If we twirl that open, the local Nuget feed 
we've created is available. If we select it, then all of our local Nuget packages will be listed. The rest is exactly the same as installing a package from Nuget.org.
