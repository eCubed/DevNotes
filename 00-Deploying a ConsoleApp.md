# Deploying an Asp.NET Core Console Application

Once everything is our console app is working, we can no go ahead and deploy.

Make sure that our build mode is "Release", not "Debug".

Publish the app. Accept that it will save to the relative folder (to the project) `bin\Release\netcoreapp2.0\publish\`.

The default settings choose a Target Runtime of `Portable`. Accept this. Click Publish.

If we navigate to that folder in File Explorer, we will see a minimal number of libraries an other files. Now, let's go to the command
prompt, cd to that `publish` folder, and then run `dotnet MyConsoleApp.dll`, where `MyConsoleApp` can be replaced with whatever
you named the console application project. Yes, we need to include the `.dll` extension. And we can see that our app will run.

But there is NO .exe file that we expect, though. That's because we chose the `Portable` which means that we can straight-up copy
the contents of this folder, and paste it even to a Mac or Linux system (where the DotNet Core SDK is installed), and it would run
with its `dotnet` "command" as we ran above on Windows. A `Portable` publication relies on the host machine's installation of a
DotNet Core SDK.

## Deployment of a Self-Contained Application
There are some times that we will need to deploy a DotNet Core console application as a stand-alone, self-contained application,
meaning that it won't rely on a host machine's installation of a DotNet Core SDK. Once we publish the app as self-contained, all of
the files in the output folder are all that's needed to run the application, so we can go ahead and literally copy it to a different
folder, and it will run on the machine, even if it doesn't have DotNet Core installed. Yes, the deployment (files and folders) will be
larger in terms of both number of files and total size, because it must now include the libraries that our app depends on. Additionally,
there will be that sought-after .exe file that could just be run. This type of deployment is referred to as **self-contained**. We will
need to tweak some settings in order to achieve **self-contained deployment**.

When we are about to publish, AKA, we're already at the Publish window/tab, we need to click on the `Configure` link. A dialog pops out
with a few options. We will make the following selections:

* Deployment Mode: Self-contained
* Target Runtime: win-x64

I chose win-x64 because I know that I want to run the .exe from a 64-bit Windows Machine. Your installation of VS may give you other
runtime options. For Windows self-contained deployment, though, we need to be careful that we intend to run the deployed package on the same
machine, or if a different Windows machine, the Windows versions should match. For example, if I deployed in a Windows 10 machine and I
copy the files to a Windows 7 machine, it won't run on the Windows 7 machine even if both are 64-bit systems.

Click Save (exits dialog), then Publish button. Give it some time even for a small application. The contents of a self-contained publication
will end up in `bin\Release\netcoreapp2.0\win-x64\` instead of the `..\publish\` folder. We can go ahead and navigate to this `..\win-x64\`,
and simply run `MyConsoleApp`.