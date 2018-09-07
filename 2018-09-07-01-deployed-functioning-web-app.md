# Deployed Functioning Web App

I have deployed a web application that I used to develop for IIS and MSSQL. There are some differences and some newly-discovered useful code that I need to
highlight here.

## Deployment "Life Cycle" of a Web App
The "lifecycle" of a web app hosted on Linux is as follows, in summary:

1. Develop the web application, probably in a different machine with possibly a different OS/platform.
2. Publish it, then copy the published output to the appropriate folder on the VPS that's supposed to host the website.
3. Create a service file for the web app - it will contain the command to run the web application, set environment variables that the app may expect, and other
settings.
4. Enable the service.
5. Run (start) the service. Now, the web app is running, and can be locally accessible by programs in the machine by `http://localhost:{portNumber}`. However, it can't
yet be accessed from outside with a URL with a FQDN.
6. Assuming that we've already set up the DNS A record for the website, we create a server file that would listen specifically on port 80 for the FQDN. Its job is to
call up (reverse proxy) the web app by `http://localhost:{portNumber}`, and pass the request to the web app.
7. The web app takes in anything passed from the reverse proxying, including any headers, query parameters, and payload, and performs its expected tasks.

## Forwarded Headers

When we ran the service for our app, our app still thinks that its host is still `localhost` and the protocol still always `http`. This makes sense because it's
running as a process on the local machine, much like when we run a web app on debug mode from a development machine. We'll need to fix this.

First, I need to let my app expect to listen for forwarding headers, AKA, headers that the reverse proxy will explicitly pass. The following piece of code must be
placed after any diagnostic middleware and before anything else. This is the `Configure()` function of `Startup.cs`:

```csharp
public void Configure(IApplicationBuilder app)
{
   /* Diagnostic Section */
   if (env.IsDevelopment())
   {
      app.UseDeveloperExceptionPage();
   }

   app.UseForwardedHeaders(new ForwardedHeadersOptions
   {
      ForwardedHeaders = ForwardedHeaders.All
   });

   /* The rest of the app's middleware */
}
```

I chose the `ForwardedHeaders.All` option to ensure that the app will get the FQDN and scheme (whether it was http or https) that we need. This way, if I ever
need to know the FQDN and scheme in my logic or what I need to return to the user a URL of a resource, my app now thinks it is of the right scheme from the right
FQDN, instead of `localhost` at some port.

## Nginx Setup with Forwarded Headers in Mind

I told the app to expect forwarded headers so that it thinks it's the actual website itself, not just any `localhost` on a particular port. But now, I need to
explicitly pass it to my running web app in the reverse proxy process. This is the app's Nginx server file:

```
server {
  listen 80;
  listen [::]:80;

  server_name fs.mysite.com;
  return 307 https://$server_name$request_uri;
}

server {
  listen 443 ssl;
  listen [::]:443 ssl;

  ssl on;
  ssl_certificate /etc/ssl/mysite_com_ssl.crt;
  ssl_certificate_key /etc/ssl/mysite_com.key;

  root /var/www/fs.mysite.com;
  index index.html;

  server_name fs.mysite.com;

  location / {
    proxy_pass http://localhost:56002;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection keep-alive;
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

The `location` block has additional settings besides passing on (`proxy-pass`) the request to the app via `localhost` at a particular port. In particular,
we set `proxy_set_header Host $host`, which is the FQDN, and then `proxy_set_header X-Forwarded-Proto $scheme` which is whether it is http or https.
At this point, I don't know what the other settings are, since Microsoft official documentation states them without really explaining what those are.

## Backward Slash for File Paths Don't Work in Linux

This depends on the application. My application happens to explicitly formulate a relative path to a file in a system with a backward slash, which I have always
hard-coded because I've always worked and deployed apps to a Windows system. When Linux sees this, it thinks that the whole path is a single string, thus, it will
end up being the name of one directory or filename. This will result in a file not found error. So, instead of hard-coding paths, I'll now need to use

`Path.Combine("path","to","my","file", "filename.txt");`

This way, it will become `/path/to/my/file/filename.txt` if the platform is Linux, and `\path\to\my\file\filename.txt` in Windows, and we haven't explicitly
sniffed the OS in our code.

## Reading Configuration from Program.cs

I've searched far and wide for how to be able to read configuration data from Program.cs in addition to from Startup.cs. In the past, I've always hardcoded the
string I passed to `IWebHostBuilder.UseWebRoot()` to be a specific location other than `/wwwroot` which is both the default and inside the application. I
wanted to serve files well outside the application's directory, and I would like to read that string from either `appsettings.json`, user secrets, or
environment variables. Now, I can. Here is code in the `CreateWebHostBuilder` function of `Program.cs` that allows me to read configuration data:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args)
{
   IConfiguration configuration = new ConfigurationBuilder()
     .SetBasePath(Directory.GetCurrentDirectory())
     .AddUserSecrets<Startup>()
     .AddEnvironmentVariables()
     .Build();

   MyConfig myConfig = new MyConfig();
   configuration.Bind("MyConfig", myConfig);

   return WebHost.CreateDefaultBuilder(args)
     .UseStartup<Startup>()
     .UseWebRoot(myConfig.CustomWebRootPath);
}
```

I would need to build the configuration builder myself. `AddUserSecrets<Startup>` will get me the values I put in secrets.json, and `AddEnvironmentVariables()`
will get me the values stored there for production. For my case, I didn't need to read stuff off appsettings.json. If you want to read from appsettings.json, you
could include the function for it in the build chain: `.AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)`.

## Next Steps

I think that I've addressed a majority of my concerns regarding serving an Asp.NET Core web application on Linux. If anything, the majority of my upcoming concerns
will probably be with Asp.NET Core itself and MySQL.