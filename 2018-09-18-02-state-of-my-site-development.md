# Current State of My Site Development

I have recently rebooted my own personal site, and this time around I am using a different operating system and database. Perhaps the biggest advances I've made in
this iteration is I have implemented my own custom name server and set up SSL for one domain. Under this one domain, I will have multiple applications that will
showcase what I've learned, from the front-end all the way to the data layer. I am still using the Asp.NET platform, more specifically, Asp.NET Core.

## Personal Site
At this point, I have only created the data and EF implementation (with MySQL) for the personal website - resource and auth servers only. I have not yet designed the
front-end website or the administration application that would populate its blogs. The personal site will contain only a journal or blog - very simple. It will allow
multiple user accounts, though I will disallow registration so that only I, the administrator can add content to it.

## Files Service
I have also created and successfully tested a secured service where clients may upload and retrieve their files. This way, any of my applications can subscribe to
the service and will not have to worry about creating uploading and file management mechanisms on their own.

## Content Areas
There are quite a few things that I would like to show on this domain, like photos, audio, and video, and also a cookbook and a dictionary (my made-up words that I believe
would be at least handy to make it to the actual English language). Each of those (media, cookbook, dictionary) will be their own application, meaning, they will get
their own database, Web API (resource server), administration app, and subdomain.