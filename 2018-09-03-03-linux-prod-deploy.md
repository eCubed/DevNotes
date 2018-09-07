# Deploying to Linux

My first project will be a basic Asp.NET 2.1 Web Api, to reside as hwres1.mysite.com. I won't go through the how-tos of BIND and Nginx.

I've got the app published, and I WinSCPed it to the folder I want in the VPS.

I've also added an A record for hwres.mysite.com.


## Server file for an Asp.NET Web Application
I've created the server file for hwres1.mysite.com and linked it to Nginx's sites-enabled folder:

```
server {
   listen 80;
   listen [::]:80;

   root /var/www/hwres1.mysite.com;
   index index.html;

   server_name hwres1.mysite.com;

   location / {
      proxy_pass http://localhost:5000;
   }
}
```

Note that there isn't a redirect 404 line in the `location` block since our app will have forward slashes in the URL that isn't
meant to be navigating the file system.

We need to set what's called a reverse proxy. Nginx listens on port 80 for requests, but since we're serving more than just static
files, we will need to delegate the request to a web application running on a localhost port. The `proxy_pass` declaration
tells us which local web application to refer to and pass the request to it.

No, I don't restart Nginx yet because at this point, nothing is running on localhost port 5000. I could go to the application's
folder and run `dotnet hwres1.dll`, and then restart Nginx to recognize this new website, and sure, it would all work, but that's
quite an unmanaged way to do it.

## Service file for an Asp.NET Web Application

It is a great idea to "wrap" our application in a service. This way, it will have an identifiable handle so that we can start, stop,
and restart our web application at will, as well as check on its status while running, just like any service.

I'll now create the service file:

`sudo nano /etc/systemd/system/hwres1.service`

Here is the file's contents:

```
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/hwres1.mysite.com
ExecStart=/usr/bin/dotnet /var/www/hres1.mysite.com/hwres1.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
SyslogIdentifier=dotnet-hwres1.dll
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

There may be more settings but the ones that we need to note are `WorkingDirectory` (what folder where our application resides),
`ExecStart` (the actual command to run our application), and the `Environment` declarations (especially setting ASPNETCORE_ENVIRONMENT
to Production).

If the application needs a production counterpart to development's secrets.json, We will need to add those variables and their values here in this file as well.
The Asp.NET Core app running here will NOT pick up system-wide environment variables, but will pick up variables set here. This is good because we know that the
environment variables we set here will only apply to this application!

Now, I need to enable the service:

`sudo systemctl enable hwres1.service`

Then I run it:

`sudo service hwres1 start`

And look at its status:

`sudo service hwres1 status`

At this point, we can now restart Nginx. Assuming that the A record has been set in BIND and already propagated worldwide, we can now
visit `http://hwres1.example.com` on a browser.

## Next Steps

Maybe I'll publish a few more web applications and put them on subdomains for exercise and familiarity.

The next step is to obtain my SSL from my host, and then I'll learn to apply it to my domain and subdomains!