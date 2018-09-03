# Reflection and Next Steps, 2018-09-03

It is early in the day, so I do not have much time for a more thorough research, following tutorials, and development.

Yesterday, I left off with having ironed out the FTP pursuit - I'm going to be using WinSCP to transfer files from my Windows dev
machine to my VPS. It's practically the same feel and operation as the old FTP (unsecure) programs I used to use, except WinSCP is
secure. Now that I have that straightened out, I am now ready to deploy an ASP.net Core application for my Linux VPS to serve.


## Asp.NET Core Linux Deployment Preview and Thoughts
It turns out that there is a lot of work involved just to deploy an Asp.NET Core website, so I'm not really sure I should pursue it
just yet. The easy part is installing the Asp.NET Core runtimes, and the tricky part is setting things up with Nginx. I'll have to
create an Nginx server for my Asp.NET web application, and it will act as a reverse proxy. I just quickly browsed the official
Microsoft guide of how to deploy something to production. There is a lot that I have to set up that I didn't used to with IIS.
One is that I have to create a few headers at the Nginx server declaration (that I believe IIS creates automatically). Another is
I literally have to manually run my web application separately first so that Nginx can call it up.

I will need at least a 1-hour block of time, preferrably 2 hours, to try this out. It may take me a minimum of 1 week (few hours a day)
to get even the basics of this correctly. It's pretty involved that there are a lot of "opportunities" to make mistakes. Well, I need
to remember that this undertaking isn't urgent, personally at least, but I want to keep pushing in case it becomes a requirement for
work all of a sudden.

## SQL Server for Linux?

I happened to come across a video on YouTube, and on the first slide, they mentioned SQL Server for Linux. The possibility of SQL
Server in Linux seems appealing because I wouldn't have to learn and set up MySQL, furthermore, code my web appswith MySQL specifically
in mind. Upon quick research, I found out that it requires 2G of RAM. I only have 1GB in my VPS, so that settles it:

I'm sticking with MySQL.

## Next Steps

I'll go ahead and install Asp.NET Core 2.1 runtime on my VPS. I'll follow the Microsoft documentation closely and try it out. This
undertaking may take several days, but that's fine.

