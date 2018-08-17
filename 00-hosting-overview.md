# Hosting Overview

We have our app ready for production deployment. We also now have a place to copy our app's deployed publication. Ultimately, how do we get our
web apps out there so the general public can go to them? What we need is a hosting service that we would need to copy our app to, set up, and they
will serve our app. We will discuss the different types of services that can host our web applications. We will most likely focus on the Microsoft
platform, and possibly later, address other platforms. Many of the principles, concepts, and steps in this unit will most likely apply to setting up
web apps on different platforms.

## Shared Hosting
Shared hosting is one of the less-expensive options to put up a website. It is shared because multiple customers' websites literally reside on the
same machine. This means that all of those websites share the same hardware. Each website is given monthly bandwidth limits, processing power share,
limited memory allocation, and hard disk space. The amounts of any of those may vary, and may even be unlimited, depending on the hosting provider.

This type of arrangement is probably sufficient for most websites, who will not directly communicate with any program on the machine it's hosted in.
Blogs, stores (as in selling products), portfolios, and other data-centric applications are good candidates for shared hosting.

## Virtual Private Server
If we sign up for a virtual private server, we will be given login credentials so we may remote to what appears to be our own machine. Once logged
in as the administrator, we will see an operating system. It is connected to the Internet, so we may actually download and install software on it.
In a VPS, we do have control of website management (IIS in Windows), which means we get down to the nitty-gritty of setting up our websites with
the operating system, as opposed to merely FTPing a bunch of files to a folder provided by shared hosting.

Like shared hosting, our stuff still resides on a single machine. That single machine, though, is set up so that multiple people can have what feels
to them like their own computer. Each account has been allocated processing power, memory, bandwidth, and hard disk space, and only that account's
activities accesses those actual physical allocated resources. This is unlike shared hosting where we're quoted bandwidth, memory, and processing
power, because in a shared hosting, all websites share all the resources, whereas a VPS has been given a chunk of the machine, and only the account
can do anything with the allocated resources.

VPS is generally more expensive than shared hosting because it has a greater level of resource dedication, and allows users to install any program
in it. Disk space and bandwidth may also be more limited than shared hosting, though.

If we have a web application that needs to call up another program in the same machine to perform a particular task, then a VPS is necessary. But
what kind of websites need to communicate with other software in the same machine? For example, services that need to return a generated video,
audio, or image based on user input would necessitate a VPS. It would be extremely difficult, if not, impossible to code and run in a web application
stuff that, for example, Adobe software can provide. Why not have the web app call up Adobe After Effects, for example, to generate a dazzling
video with content provided by the user?

## Dedicated Hosting
Dedicated hosting is the most expensive option. We can do on a dedicated host everything that we can in a VPS, and we have the whole machine to us.
No other customer can access our processor, hard disk, and memory. In general, dedicated hosting provides us with faster websites!