# Linux Deployment Plans

Suppose that I've already learned enough at this point to be able to deploy a website live. What would I do in order?

1. Deploy a Hello World website, no https. This will be a beginning exercise just to experience what it is and what it's like to host a website on
the linux platform. It will be found in `hworld.mywebsite.com`. This will involve creating a CNAME DNS record, and setting up the website (the
equivalent of what we do in IIS). We may also get into permissions since I anticipate that we can't just expose contents on the hard drive to the
outside world.

2. Deploy a service that clients may upload files to. I already have this working from a Windows environment. I may have to tweak the project
a little bit because we would be using MySQL instead of MS SQL. It just occurred to me that as developers in General, we probably should consider,
for many projects (not all), depending on platform-specific services, as that was the intention of Asp.NET Core. Though EF and MSSQL may run on
Linux or Mac (assuming that the database resides on a remote Windows server), we should try a cross-platform database like MySQL.