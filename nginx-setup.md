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

## Setting up Websites

From having some IIS experience, I have the following questions:

- Do I have to create a CNAME record for each subdomain?
- What is the equivalent of setting up bindings for each website?
- How about SSL and using certificates?
- Do I have to set permissions for each folder that contains web files?
- How do I set default pages at the root?
