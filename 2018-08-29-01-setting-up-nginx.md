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

We need to add some security before serving web pages, so we need to activate the firewall:

`sudo ufw enable`

It wasn't enabled by default when the VPS was created.

But now, we'll need to allow Nginx through. By default, the firewall blocks everything.

`sudo ufw allow 'Nginx Full'`

Since we activated the firewall, let's allow OpenSSH as well to make sure we can log in next time.

`sudo ufw allow OpenSSH`

Note that single quotes were necessary when what we want to allow has two or more words.

Now, let's look at the firewall status:

`sudo ufw status`

This lists only what we've allowed through.

