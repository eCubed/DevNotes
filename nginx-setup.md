# Setting up Nginx on Linux Server

Nginx to the Linux VPS is what IIS is to the Windows platform. This is where we'll declare what websites we'll host from the VPS, which contains quite a
few settings that determine which folders our websites are in, what the index files are, SSL, bindings, and more. Everything that we may have done in IIS
with a GUI, we will run with commands and write configurations in files.

## Installation

We need to log into the system with a sudo user.

Before we install anything, we must do a system update:

`sudo apt update`

Then we can install nginx:

`sudo apt install nginx`

We will see logs that packages are being downloaded and setup, much like what happens when we see run `npm install`.

## First Viewing

By default, pointing our browser to our IP address will automatically show the default Nginx page. This is a sign that
Nginx is up and running.

## Server (Website in IIS)

A server really is a website associated with a domain or subdomain. We will create a file for each server. We create server files in the folder
`/etc/nginx/sites-available/` and we are will name them by the exact domain or subdomain. The name of the file is not strict, though it will help us
keep organized. Since we'll start with the main site, let's go ahead and create its server file:


### The Server File
`sudo nano /etc/nginx/sites-available/www.mysite.com`

Then we write in that file:

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  root /var/www/www.mysite.com;
  index index.html;

  server_name mysite.com www.mysite.com;

  location / {
     try_files $uri $uri/ =404;
  }
}

- The first two lines state that we're listening on port 80 for a request to view the website. It will always be port 80 for all domains and
subdomains.
- The root is the very folder that contains web files (.html, .js, .css, etc.). Notice that we store websites in `/var/www` by convention, and
the website foler name is exactly the domain or subdomain name for better organization. Again, file names and locations aren't strict, but let's
stick to convention and consistency.
- The index "flag" lets us specify what the index file is/are, that would automatically load when only the domain was specified.
- The server_name is like the bindings in IIS where we specify which domains will be served by the web files specified at root.
- At this point, we don't know what the location means, but for now, it's required, so we inclue it. We will adress this later.
- Every declarative single line ends with a semi-colon.
- Note that we inicated the `default_server` modifier in the listen key. This means that this server's website will be serve in case someone
requests a subdomain whose server file hasn't yet been created and enabled.
```

### Enabling the Server (Website)
In IIS, we get to click play, pause, and stop to activate/deactivate website. The mechanism is different with Nginx. When we finished writing the server file
for our website, it's disabled. So, we need to enable it. To do that, we must add a symbolic link to the `/etc/nginx/sites-enabled` folder:

`sudo ln -s /etc/nginx/sites-available/www.mysite.com /etc/nginx/sites-enabled/www.mysite.com`

A symbolic link to a website on `/etc/nginx/sites-enabled` specifically tells Nginx to serve the website.

### Resetting Nginx
For our websites to take effect, Nginx must be restarted. We didn't have to do this in IIS because it lets us activate/deactivate websites with immediate
effect.

`sudo service nginx restart`

If the DNS has been configured correctly and fully propagated, we should see our website when we browse to it.

## Setting up Subdomains

It's basically the same as setting up the main domain, except:

- We do not include the `default_server` modifier on the listen lines.
- The root location will be different.
- The server name will also be different, reflecting the full domain name including subdomain.
- The server file name will also reflect the full domain name including the subdomain.

Then, we create a symbolic link to `/etc/nginx-sites-enabled`, and then restart Nginx.

HOWEVER, we are not yet done!!! None of this will work until we create an A record for our subdomain in our domain's forward zone file:

Suppose our subdomain was `foobar`, then we add the A record:

`foobar  IN  A  143.207.72.25`

We then need to remember to increment the serial number of the forward zone file before saving, and then restart the bind service. We would then need to
wait for our subdomain to propagate before we can view it in our browser.




