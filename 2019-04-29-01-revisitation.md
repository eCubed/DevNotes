# Revisitation

It's been a while since I've toyed with deploying an Asp.NET Core app in Linux. I have done it before, and as I am writing this, I vaguely remember everything
that it takes to set one up. This article is to walk through the process without really explaining things in great detail.

## Getting Around
I know that there isn't a GUI, and I'll have to find my way around the system, and relearn some basic commands.

I logged into my VPS with PUTTY, using a full-privileged account previously created by the admin account. Upon login, I'm not sure which folder I'm in. It turns
out that upon login, I am NOT in the root foler, but in the home folder set up when the non-admin account was created. I `cd ..` to take me one folder up, and
when I ran `ls -la`, I see the `admin` and folders - each named after the username of each of the accounts. When I `cd ..` again, I am taken to the
very root directory of the entire machine.

Upon looking around, the *home folder* is actually located on `/home/{username}`. This tells us that upon logging in, we'll probably want to `cd ..` twice
to get to the machine's root since we'll need to access some folders in it.

In particular, there are two folders off root that we'll access most often, whose names don't always seem to reflect what their actual purpose is:

`/etc` - seems like this is where programs resides, specifically the ones we pick out and install.

`/var` - In particular, it also has the folder `/www`, which will contain folders - one folder for each of our deployed web apps (of our domains and subdomains).

`/sys` - The system folder. We'll need to examine, modify, and create files in this system folder. More on this later.

## Bash
To make navigating the system easier and more pleasant to look at, I have installed `bash` in my VPS. It's nicer because it color-codes things (though I don't
always know what they mean), but it makes it easier for me to know at a glance, for example, whether something is an actual file or directory, or a link to a file or
directory. It also let's me access my previously-run commands by pressing the up-arrow key, which results in garbled text without `bash`.

## Domains and Subdomains
The purpose of getting this VPS in the first place is to be able to serve websites, and we'll need to declare somewhere and somehow on our machine what domans and
subdomains it serves.

The first file is at `/etc/bind/named.conf.local`. The contents of this file declare *only* the top-level domains and their reverse-lookup entries. They point
to other files we previously created that describe in greater detail subdomains and other services we'd like to host.

To see subdomains and services, we go to the file `/etc/bind/db.mysite.com`. We'll most likely add entries to this file when we add/remove subdomains.

I'm using the `nano` text editor. As a warning, using the numeric keypad doesn't work. We'll have to type in numbers using the top-row of the keyboard.

Also, I need to type in the command `sudo nano db.mysite.com` to actually be able to save it. The prompt will ask for a sudo password, so I'll need to
supply it.

Another detail is that I'll need to manually change the value of the first parameter (the serial) for every time I make changes I want to to save in this
file. If I forget to change the serial, then bind will not recognize the change!

To verify that my changes were legit, I run `named-checkconf`.

Now, I'll need to restart the bind server with the command `sudo /etc/init.d/bind9 restart`. Now, my machine will broadcast my changes to the Internet!

## Deploying the Asp.NET Core Web App

Let's assume that we our web app is ready to be deployed, we have already published it, and we've already transferred the production files to our
VPS's `/var/www/{subdomain}` directory.

At this point, the app isn't yet running. Yes, we can go to `var/www/subdomain` and run `dotnet mysubdomain.dll`, but that's unmanaged.
There is another file that we'll need to create or modify, called the `service file`, at `/etc/systemd/system/{subdomain}.service`.
This service file contains the command `dotnet mysubdomain.dll` (however, with full paths because this runs from root), as well as the
production environment variables that our deployed app will expect.


Now that we have set up our domains and subdomains in BIND, we'll need actual websites to serve when we get a request to our VPS to serve up a website with
a certain FQDN. Our primary focus is Asp.Net Core Websites,












