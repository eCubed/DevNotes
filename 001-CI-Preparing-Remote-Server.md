# CI - Preparing the Remote Server

There are a LOT of installations required for the main server. The largest goal is to set up IIS to be able to receive production
files and update the right website. But even before that, we'll need to install a few things.

## Install Dependencies For ASP.NET Core Apps
ALL of these installations are  AT THE LIVE SERVER!

First, we'll need to install the appropriate ASP.NET Core runtime that our ASP.NET Core apps need. The latest at the time of
writing is [here](https://dotnet.microsoft.com/download/dotnet/5.0). I'll choose the Windows x64 Hosting Bundle since my server
is running on Windows.

One important thing to note is that OUR PROJECTS WILL BE BUILT AT OUR LIVE SERVER, NOT ON OUR DEV MACHINES! This means that
we'll need to install something that WILL BUILD ASP.NET Core projects.

So, I'll need to download and install [Visual Studio 2019 Build Tools](https://visualstudio.microsoft.com/downloads). I'll go
down to `Tools for Visual Studio 2019` and select the x64 version of `Build Tools for Visual Studio 2019`. This is NOT an SDK
nor is it the whole VS!

Now, I'll need to open `VS Installer`, Modify `VS Build Tools 2019` and add `.Net Core Build Tools`.

I'll need to download and install [Nuget](https://www.nuget.org/downloads). I'll choose the latest version. Note that it's *the*
`nuget.exe` itself, and I'll need to *manually* install that on my machine to a specific location OUTSIDE `C:\Program Files`. So
conveniently, I'll download it to the folder `C:\Nuget`. Now, I will need to go to my system environment variables and add
`C:\Nuget` to the path. 

## Install Dependencies For Angular Apps
For Angular, We'll need:

[Node](https://nodejs.org/en/). I'll choose the latest LTS version.google

Node already comes with npm, so now, we'll need to install the latest version of Angular:

`npm i -g @angular/cli`.

I'll need to go to Environment Variables and Add the two to `PATH` (The system-wide one, not the one associated with the user only):

`C:\Program Files\Git\cmd`
`C:\Program Files\nodejs\`
`C:\Users\Administrator\AppData\Roaming\npm\node_modules\@angular\cli\bin\`
`C:\Users\Administrator\AppData\Roaming\npm\`

If our Angular App is SSR/Universal, we'll need to download and install [iisnode](https://github.com/azure/iisnode/wiki/iisnode-releases).
I'll pick out `iisnode for iis 7/8 (x64)'. Now, I'll need to go to File Explorer, pick out the folder `C:\Program Files\iisnode`
, and give it full access to `IIS_IUSRS` and `IUSR`.

## Preparation for Jenkins

What is Jenkins? Jenkins is a tool that orchestrates the deployment process. When it's all set up, its job is first to listen for
Github when we push to a particular branch on our repo, pull our source code, call up `dotnet` or `ng` to build our apps, and then
communicate with IIS to push ASP.NET Core published files to the right website.

Jenkins is actually a website application so when it's all set up, we'll actually be able to log into it FROM ANYWHERE, perform 
administrative tasks, and diagnose any problems that may have occurred during deployment. Additionally, Github will need to point
webhooks to this Jenkins application, so we'll actually need to create a publicly available website for it. 

### Subdomain and IIS

So, we'll need to create a subdomain for our Jenkins app and the IIS website for it. I won't go into the details of this since
web hosts vary on how this is set up. However, it is customary and easy to remember `https://jenkins.my-domain.com`, so we'll
need to create a subdomain there. Yes, set up HTTPS as well.

I'll go into IIS and make sure that my Jenkins website's app pool's `.NET CLR version` is set to `No Managed Code`.

Now, for the `web.config` for the `Jenkins site`:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <system.webServer>
        <rewrite>    
            <rules>
                <rule name="http to https" stopProcessing="true">
                    <match url="(.*)" />
                    <conditions>
                    <add input="{HTTPS}" pattern="^OFF$" />
                    </conditions>
                    <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" />
                </rule> 
                <rule name="ReverseProxyToLocalJenkinsRule" stopProcessing="true">
    	            <match url="(.*)" />
                    <conditions>
                        <add input="{HTTPS}" pattern="on" />
                    </conditions>
                    <action type="Rewrite" url="http://localhost:8082/{R:1}" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

Only the relevant portions of `web.config` are shown above. I've included the `http to https` rule because we better be creating
HTTPS websites anyway nowadays. We'll need to place the `http to https` rule BEFORE the reverse proxy rule.

Reverse proxy? Yes. Jenkins is a web application that runs on Java, not IIS, so we'll need to redirect to the Jenkins website
(which is again running on Java). Take note that we rewrite to `http://localhost:8082/{R:1}`, which is where the Jenkins website
will live on our machine. It doesn't have to be https, since `https://jenkins.my-domain.com` is the IIS app that will see the
request first. Later, when we install Jenkins, the installation will ask for the port number, which we'll specify `8082`. Note
that we could have left it at `8080`, but that's already being used by IIS.

Now, it's time to get IIS to receive MSBuild deployment, so we'll need to install Web Deploy. Detailed instructions are
[here](https://docs.microsoft.com/en-us/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later).

### Granting Administrator Full Access to Services

The fact that there will be a local website `http://localhost:8082` that opens to Jenkins, Jenkins actually *actually* runs as
a Windows Service that makes that local website available. Now, we'll need to let a user, specifically, the `Administrator` user
if the host automatically created that account, have full access to this service. I'll search for `Windows Administrative Tools`, then a
window pops up listing a few apps. I'll go to `Local Security Policy` > `Local Policies` > `User Rights Assignment`, then finally,
`Log on as Service`. I'll find the `Administrator` account, and it gets added to the list. I hit `OK`. Now, the `Administrator`
can run this app.

### Java
Download and Install [Java JRE 8](https://www.java.com/en/download/). Since Jenkins runs on Java, we'll need to install this first.
The Jenkins installation doesn't automatically install this dependency.

## Installing Jenkins

I'm now ready to install Jenkins. I can get it [here](https://www.jenkins.io/download/thank-you-downloading-windows-installer-stable/).
That's for Windows since my live server is Windows.

When the installer runs, it'll ask for Jenkins installation folder. The default is `C:\Program Files\Jenkins` but I will change
that to `C:\Jenkins`. I do this because anything inside `Program Files` or `Program Filex (x86)` may have a special distinction
in Windows that would prevent access to Jenkins later if installed there.

When prompted, I'll change the port from '8080' to '8082', as I've set up in a `web.config` above.

When prompted, I'll select `Don't enable firewall exception`.

After successful installation, from my live server's OS, I browse to `http://localhost:8082`. I'll see that Jenkins is continuing
to set up. I'll let it do its job.

Now the Jenkins page will say that it needs to unlock Jenkins. I'll follow the instructions on the page. There's a specific
file that contains some sort of key or hash that must be inputted on the textbox, so find that file, open it, and copy the hash
into the textbox, and then proceed.

Jenkins will suggest us to install suggested plugins. I'll accept it, and let Jenkins install all of them.

Now, the Jenkins website will ask for the first administrative user. I'll enter anything, BUT I'll have to remember that username
and password because those are going to be the credentials needed when I'll access Jenkins from anywhere! I'll also provide a
valid email address.

Jenkins will ask for `instance configuration`. I'll use the publicly available URL for it that I decided earlier:
`https://jenkins.my-domain.com/`, yes, with the trailing forward slash!

Now, I can grab Jenkins online!

Before we move on, I'll log into Jenkins, Go to `Manage Jenkins` > `Manage Plugins` > `Available (tab)`, and then I'll type in
`MSBuild`. I'll select it and then at the bottom, I'll click on `Install without restart`.
