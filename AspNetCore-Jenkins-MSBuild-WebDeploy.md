# Deploying an ASP.NET Core App using Jenkins, MSBuild, and WebDeploy

The goal is to automatically build and deploy (yes, copy the files to the remote live server) when we merge to a specific branch (yes, our projects should be in
Git repos by now).

I won't go into details into installing each application on the right machines. There are instructions for that. BUT: We need to make sure that:

On the DEV MACHINE, we have installed:

1. Jenkins
2. MSBuild plugin for Jenkins
3. Web Deploy (look online for how to install or check to see if it's installed in the system.)

On the LIVE PROD MACHINE, we have installed:

1. Web Deploy (look online for how to install or check to see if it's installed in the system.)
2. Ensure that Web Management Service (under Services) is running.

## Prepping Jenkins

Before creating the project, we'll need to go to Jenkins > Manage Jenkins > Global Tool Configruation, and go down to the MSBuild section (should've already be there
if the plugin was installed.'. We're going to name it ("msbuild" is simple enough and it'll work), and then provide the path to it. Since I'm using VS 2019 on Windows,
The path I'll use is `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe`.

## Creating the Publish Profile

Back at VS, We'll need to create a publish profile, specifically, an IIS Publish profile. This is where it'll ask us for our server's IP and IIS credentials, which,
in my remote server's case, is the same as the Windows administrator's credentials (to log into Windows). Note that once the file is created, it'll sit in our
project's `\Properties\PublishProfiles\` folder and will have a `.pubxml` extension. We'll need this later for when we set up our Jenkins project.

## The Jenkins Project
Quickly:

I'll create a project, of type `Freestyle Project`.

I won't go through the Git setup since that's a topic that covers a lot, including setting up Jenkins to store Github credentials, setting up Github webhooks, and
using ngrok so that Github webhooks can reach the Jenkins on our machine to tell Jenkins to build and deploy our project.

I'll create the first Windows batch command: `nuget restore`. After our app hears from the latest Git push (to the branch designated for deployment), Jenkins grabs 
the contents of our repo and copies them to our job's workspace, and from there, we'll build our app instead of from our actual project as referenced by VS. We'll
need to run `nuget restore` from inside our job's workspace or else, we'll get an error.

I'll then create another Windows batch command whose job is to copy the IIS publish profile I created earlier from VS. Note that the publish profile by default
doesn't make it to source control, which is a good thing. So, we'll need to copy it to our Jenkins workspace because they contain instructions for deploying to the
live server.

`
if not exist DeployOne\Properties\PublishProfiles\ mkdir DeployOne\Properties\PublishProfiles
copy D:\Labs\LearningDeployment\DeployOne\Properties\PublishProfiles\DeployOneMSDeploy.pubxml DeployOne\Properties\PublishProfiles\DeployOneMSDeploy.pubxml
`

Note that I create the `PublishProfiles` if not already created. It wouldn't if it were the first time we "clone" the repo to our Jenkins workspace. Note that I
also maintain the ame filename when I copy it from my actual development location.

Now, I'll create a build step `Build a Visual Studio project or solution using MSBuild`:

For MSBuild versions, I'll select the one `msbuild` I've previously set up with Jenkins globally. Choosing default will result in an 'application not found' type of
error.

For MSBuild Build File, I'll point to the .csproj file of the project I want to build. I'll need to remember that this path will be relative to the Jenkins workspace.

`DeployOne\DeployOne.csproj` - for example.

Then for command line arguments, I'll enter:

`
/P:VisualStudioVersion=16.0
/P:Password=PasswordToOurRemoveServer
/P:AllowUntrustedCertificate=true
/P:DeployOnBuild=True
/P:PublishProfile=DeployOneMSDeploy
/P:DeployIISAppPath="websiteNameAsDefinedInTheRemoteIIS"
/P:MsDeployServiceUrl=IP_ADDRESS_OF_OUR_REMOTE_SERVER
/P:Configuration=Release
`

That's it. Now, everytime I make a push to the specific branch I designated for Github webhooks and Jenkins, Jenkins will orchestrate everything - grab latest code
from repo, build my project, then LITERALLY push my files to the repo.
