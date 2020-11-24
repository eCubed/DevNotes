# CI - Deploying Angular SSR Apps

Set up Github and Jenkins according to the [Github and Jenkins Setup](002-CI-Github and Jenkins Setup.md) article.

Now, we'll add some build steps.

Build Step 1 - Execute Windows Batch Command: `npm i`

After the source code gets downloaded from the repo, we'll need to run `npm i` to ensure that we have all the needed
dependencies as outlined in `package.json`. So, yes, quite unfortunately - Each Angular project in Jenkins will have its
own `node_modules` folder, which is very space-consuming. At this time, this is inevitable and we'll wait for better
technology. Be forewarned that the first Jenkins build of an Angular project will take quite long because of this
necessary command.

Build Step 2 - Execute Windows Batch Command: `npm run build:ssr`

This will compile a production version of the app to the Jenkins job's workspace. This build step takes quite long.

Build Step 3 - Execute Windows Batch Command:

```
if exist C:\path\to\my-angular-app\dist (rmdir C:\path\to\my-angular-app\dist /s /q)
xcopy .\dist C:\path\to\my-angular-app\dist /E/H/C/I/y
xcopy C:\path\to\my-angular-app\dist\my-angular-app\server\main.js C:\path\to\my-angular-app\main.js /y
```

This step happens after the build. We'll want to clear out the `\dist` folder on the IIS app first to ensure we'll get a
fresh, clean build every time. So, we delete it if it's there.

Now we copy the `.\dist` folder from the Jenkins workspace to the IIS app's path.

We then copy `\server\main.js` to the IIS app's root. This is a required step.

## web.config

Angular build doesn't automatically create a web.config file, and it has to be manually created only on the first deployment.
Subsequent deployments won't touch it. We're hosting an Angular app on IIS, so we need a web.config:

```xml
<system.webServer>
  <handlers>
     <add name="iisnode" path="main.js" verb="*" modules="iisnode" />
  </handlers>
  <rewrite>    
    <rules>     
      <rule name="http to https" stopProcessing="true">
         <match url="(.*)" />
         <conditions>
           <add input="{HTTPS}" pattern="^OFF$" />
         </conditions>
         <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" />
      </rule>
      <rule name="Angular Routes">     
        <match url="/*" />
        <action type="Rewrite" url="main.js" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

Note that we reference the `iisnode` handler and specify the path to `main.js`, which we copied to the IIS app's root.
Since the Angular app runs on node, this handler does the job of running the node app and reverse-proxying to that node
running instance. 

I'll put the `http to https` rewrite rules first, then the rewrite rules for `Angular Routes`.
