# Deploying ASP.NET Core App

So, we've already installed Jenkins on the live server and we can access it from anywhere.

As a review, Jenkins is the central mechanism that orchestrate everything it takes to deploy web apps seamlessly.

In a nutshell, Jenkins listens for Github webhooks when we push to a certain branch onto our repo, downloads the source
code as stored in Github, builds the production version, and then calls on IIS to deploy the files.

## Github Setup

We'll need to login to our Github, find the correct repo, and go to its `Settings` page. `I'll go to `Webhooks` on the
left-hand panel.

I'll click `Add Webhook`. I'll enter my Jenkins url in `Payload URL`: `https://jenkins.mydomain.com/github-webhook/`. Yes,
the Jenkins app has a path to `/github-webhook`.

I'll leave Content Type to `application/x-www-form-urlencoded`.

I'll leave Secret blank.

I'll choose `Enable SSL verification`.

I'll choose `Just the Push Event`.

I'll save the changes.

## Creating the Jenkins Job

I'll go ahead and create my first job: `Jenkins` > `New Item`

I'll enter the name of the job - the job is where we specify steps on what to do to deploy the app.

I'll select `Freestyle project, and click `OK` at the bottom.

When I get to the job, I'll choose `Configure` on the left so I can outline the tasks.

Under `Source Code Management`, I'll select `Git`.

It'll let me specify a rep URL, so I enter the correct one. I'll also need to add credentials - the same credentials to log
into Github.

Under Branch Specifier, I can leave it to `*/master`. This means that ANY push to `origin/master` will have Github call our
Jenkins to begin the deployment process. We can designate a different branch for deployment if we need to.

Under Build Triggers, I'll select ONLY `GitHub hook trigger for GITScm polling`.

Now, under the build section, I'll create the first Windows Batch Command - `nuget restore`.

Note than when Github called our particular webhook because of a push to master, Jenkins actually downloads the source code
from Github, which includes the `.csproj` file. The `.csproj` file might list some dependencies that don't come with the ASP.NET
Core runtime (we previously installed). So, that's why we'll need to run `nuget restore` - to get all of the required `.dlls`
so that our app can build.

Now, we'll add another build step of type `Build a Visual Studio project or solution using MSBuild`.

I'll leave the MSBuild Version to Default

For MSBuild Build File, I'll put the path to the `.csproj` file, which is usually at the root (relative to the repo's root).

Under Command Line Arguments, I'll enter':

```
/P:VisualStudioVersion=16.0
/P:TargetFramework=netcoreapp3.1
/P:Configuration=Release
/P:SelfContained=false
/P:WebPublishMethod=MSDeploy
/P:MSDeployPublishMethod=WMSVC
/P:LaunchSiteAfterPublish=true
/P:AllowUntrustedCertificate=true
/P:DeployOnBuild=True
/P:DeployIISAppPath="subdomain-of-app.my-domain.com"
/P:MsDeployServiceUrl=127.0.0.1
/P:Username=username
/P:Password=pw
```

The above configuration is what's proven to work. However, we'll need to change some values according to our app's specifications.

DeployIISAppPath is the name of the website as IIS knows it.
MsDeployServiceUrl IS 127.0.0.1 because Jenkins lives on the same machine as our IIS.
Username and Password are the credentials that IIS recognizes for allowing external access to it for deployment purposes.

## Building

We don't actually have to build explicitly from inside Jenkins. It will actually fail if we do it without first having heard
from Github!

So, all we'll need to do is, from our dev machines, push to the master branch (the remote one), and wait for Jenkins to hear from
Github. After a few moments, a build is created which we could examine the actual DOS output. From there, we can see that the
deployment succeeded or any errors it shows that we'll need to address.



