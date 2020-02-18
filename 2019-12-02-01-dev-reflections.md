# Dev Reflections 12-02-2019

I haven't reflected on my overall development for quite some time. I think it's time as I have quite a few important things to note.

The last entry I made was almost four months ago, in August 2019. I did mention about exploring other frameworks and platforms, both front and back-end.
I pretty much settled with Angular 7+ for the front-end, Asp.NET with C# in the back end, and MSSQL for my persistent data storage.

Since then, I have worked on a few projects, and to my knowledge, they are complete based on all of the information given to me explicitly. Interested
companies even came to the office to discuss what we have done so far and what they would want done next, but it seems like they were just ideas thrown
at the table without specifications like what needs to be done, by who, who will analyze which data, and what would be done with the analysis. The last
two months have been relatively quiet as far as further instructions on current projects or anything about new projects. What did I do to fill up
all of that time?

## Personal Website

I have built personal websites before, but with older technology which I found quite cumbersome to develop in. Now, with Asp.Net Core 3 and Angular, I
have redone my entire personal website - with a blog, a dictionary, and a cataloguing system for my musical works at the first release. I have a file
server, a resource server, a public-facing front-end app, and an administrative (also front-end) app for managing the data I want to share. I also got
a VPS hosting package because my website requires specific software that I need to install and configure myself, since they were not offered in a
shared hosting package. This is my website, so I know exactly what I want. From getting down my ideas and sketching UI on my paper, to finally launching
the entire suite, it took approximately 4 weeks to publish the initial release. After the initial release, I have done a few minor tweaks, but the time
and effort spent on those were relatively negligible - about 2-3 hours total.

## Now What?

Even after my website is complete and I have no plans for new types of content at the moment, there is still a ton of downtime, and I'm not sure what to
do. I have still not received further instructions beyond those that I've been explicitly told, and there are no new projects to start working on. There
has been a discussion in the past that I should be proactively working on projects "that would help the company" instead of learning technologies and 
frameworks - new, or existing ones that I personally, and the company hasn't implemented. I've always lamented on figuring out what exactly is meant
by "that would help the company", since I know that in this field, keeping up with new technology is extremely important, and in the past, I have learned
tools, software, and framework that I currently use in recent projects. 

I only know that the projects we've been working on always have have a visual end product (usually a video or some sort of presentation) whose content is
a mix of user-inputted data or decided by some specific logic/scheme. For the life of me, I can't think of any more projects that would do such a thing, 
and if I did, I couldn't complete it anyway because I don't have training and access to the actual software that would generate videos based on data the
app may collect. No, I will not spend thousands of dollars out of my own pocket for industrial software and training for a system that would generate
advanced graphics.

So, I'm stuck right now. I'm not sure what to learn and what specific projects to tackle independently, even if the goal of such projects aren't immediately
the nature of user inputting data and they'll get a video out of it.

## Reinventing Wheels?

I've worked on a lot of projects over the years, but I'll admit I have not implemented many that people use on a regular basis. I know people use social
media and systems that let them browse and purchase products (a store). I have implemented neither of those. Is it worth my time and the company's time
to pursue implementing my own social media or a store app, even with the most basic features? 

I don't know. Out of the two, though, I think it's more worth it to create a store, even a generic one that I could somewhat more easily apply and/or 
incorporate into any project that needs it. I have an idea of the relationships between products, orders, brands, and models, as well as the UI and API 
calls required for browsing/searching, filling a cart, and creating the order. I'm a bit leary, though, with the payment system part of the whole system.
I think most payment providers will require a page redirect, though, which would be tough or impossible to perform with an SPA. Then there's the issue of
not supposed to be using my own UI to collect sensitive information from the user (even if I'm on SSL). Then there's me needing to pay the payment provider
and I don't actually understand how that works.

Another idea that comes to mind that I've roughly implemented is albums and media. I know I've created a files service that lets the requestor upload
a single or multiple files in one API call, and it would return the absolute URL of all of the uploaded files. This is only a files service. It does
NOT generate various images or sizes, and return a family of URLs for each one. Perhaps I can create an intermediary service that DOES allow uploads just
like the file service, HOWEVER, its job is to look at the file extension, and create versions of the file (different sizes for image/video, for example)
LOCALLY, and then upload each of those generated files to the file server.