# Reflection and Next Steps

We have achieved the very basics of setting up a name server and serving the websites of the main website and one of its
subdomains, only with CLI commands and writing configuration information to files, with no GUI or assistance from the host.
There is much to learn in this arduous but rewarding journey. We will need to tackle the following, though most likely not
in the order presented:

- SSL and certificates
- Email (the email server and the website client for it not to interfere with our Nginx setup. We may learn to do this in
C# just for experience - we would do the front-end in Angular.)
- FTP (allow users to log in and restrict them to only one folder. Will the ftp users be the same as the VPS users?)
- Installing Asp.NET Core 2.x runtimes
- MySQL Setup including the website client that would not interfere with our Nginx setup.