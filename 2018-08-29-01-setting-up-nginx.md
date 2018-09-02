# Setting up Nginx on Linux Server

Nginx to the Linux VPS is what IIS is to the Windows platform. Typically, we would first install and configure the DNS server. However, it may be better to
set up the web server first to make sure it works. We will browse to a default website using our IP address first. That default website is one that the Nginx
installation creates and points to initially when a request comes in through port 80.

## Installation

We need to log into the system with a sudo user.

Before we install anything, we must do a system update:

`sudo apt update`

Then we can install nginx:

`sudo apt install nginx`

We will see logs that packages are being downloaded and setup, much like what happens when we see run `npm install`.

## Firewall Setup

The Linux firewall is known as `ufw`. Since my host sets up a firewall right outside my VPS, I will not bother with
enabling the firewall from inside my VPS.

## First Viewing

By default, pointing our browser to our IP address will automatically show the default Nginx page. This is a sign that
Nginx is up and running.

## Setting up Websites

From having some IIS experience, I have the following questions:

- Do I have to create a CNAME record for each subdomain?
- What is the equivalent of setting up bindings for each website?
- How about SSL and using certificates?
- Do I have to set permissions for each folder that contains web files?
- How do I set default pages at the root?
