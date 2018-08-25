# Linux - Overview and Motivation

This series is for my own education, experience, and exploration, so read these Linux articles as if a developer new to Linux is speaking about
their experience in adding it to their skills.

## My Development Background

I currently develop on a Windows machine with the Microsoft platform. My main server-side and desktop application programming language is C# using
the .NET Core framework. I also create front-end administration applications using Angular 6. My main data store is MSSQL with Entity Framework.

I studied Computer Science in college, and C++ was the main programming language, developed in the Unix platform. I also programmed in Java,
JavaScript, and PHP, as well as stored data in MySQL.

On the side, I have also used the same tools to develop my own personal website. I have some experience with both shared and VPS hosting for the
personal website, using Microsoft products.

## Motivation

I have taken down my personal website because I didn't have time to maintain it. At work recently, there became quite an amount of downtime, so the
thought of learning more beyond programming has crossed my mind. I didn't really have a good grasp on DNS and web servers, and I relied on my hosts
in the past to set up all of that for me, and SSL was something I blatantly ignored (because the IT department set that up for projects I've worked
on). I think it's a good time to add new skills, and web hosting seems to be an excellent logical next step (or levelling up as I like to call it).

I also know that I can run Asp.NET Core web applications on a Linux server, which means that I can develop applications on a Windows machine as
I always have (and always will), and yes, I've always written .NET Core ensuring that apps would to run on Mac and Linux machines. Then I'll
have to copy published files to the Linux server, which shouldn't be too difficult to do.

I have heard my supervisor discuss the possibility of someday hosting on the Linux platform for performance, for future projects. I've also noticed
how much cheaper Linux is in any kind of hosting, compared to Windows. Since I want to learn and spend less money, I think that I would shoot for
a huge challenge - to pursue Linux VPS hosting even if it means that most, if not all, of what I need to do has to be done on a command prompt. So,
if I learned it now, I'll be ready if ever the time comes that we need to host on a Linux machine. If not, at least it's another skill to add to
my development toolkit (or arsenal for more graphic imagery).

It's not urgent that I have to put up my personal website hosted on Linux, so I am willing to spend months learning before I can make that possible.
I've had some UNIX experience in college, along with writing and running programs on command prompts, so I may be able to pull this off. Plus, I
heard that the Linux community is quite sympathetic to newcomers, and love to share their experiences and secrets.

## Challenges

I'm so accustomed to developing and hosting in the Windows platform already, and I acknowledge that it will be a very steep learning curve for me to
learn Linux hosting. From what I've read, as I have mentioned, practically everything that I used to do in IIS and Windows, I now have to type
corresponding commands (which have tons of flags, parameters, and cryptic syntax), and write scripts in languages I don't know, are about to take
place. Right now, I have a very basic knowledge of navigating the file system on command prompts. I have some experience running
console applications, though - like `dotnet run <appName>`, or `npm install <somePackage>`. I don't know, for example, how to view even
system stats and states from a command prompt, let alone tweak OS settings and preferences from there. It will be very arduous, tedious, and 
error-prone learning experience. I reckon that I will need to learn hundreds of commands.

If I commit to the Linux platform, then this means that I will no longer work with MSSQL, which I've gotten very accustomed to and comfortable.
I figure that setting up MySQL won't be too difficult, but I'm not sure how I would communicate with it from .NET Core code. No, I would NOT want
to communicate with MySQL ala ADO.net-like API, which is very tedious and error prone. Hopefully, there is a package that I can use that allows
me to communicate with MySQL via LINQ like I do with EntityFramework and MSSQL.

As much as possible, I don't want to pay anything. I realize that I can download a Linux server distribution and play with it on my local machine,
but that cannot simulate the live VPS scenario, though. Yes, I may be able to set up DNS or the web server, but how would I call up that machine
from another machine to test that it works? So, I'm on my quest to find the cheapest Linux VPS host I can just to learn all of this.

## Approach

I will first collect all of the tasks I used to perform in the Windows platform on the next article. Then, I'll go back and write the answers
as I learn or discover them. After I finish with this learning venture, I will write an actual series on how to set up everything in order to
serve websites on the Linux platform.

