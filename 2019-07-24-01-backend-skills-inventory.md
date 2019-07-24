# Backend Skills Inventory

In the last few years, I have embraced Asp.NET Core as my staple back end development framework. Since web APIs are now crucial, it is much easier to develop
them in Asp.NET Core than in Asp.NET 4.x and below, including the capability to be able to issue tokens and protect APIs by requiring the token. Since SPAs
are becoming more popular and as I'm gaining experience in developing them in Angular, I found that their very nature to make REST API calls is the reason to
also write web API applications myself.

## Missing Skills in the Back End?

Since I develop the front-end in Angular due to giving the user a better experience, and make API calls to a web API applications, I did NOT bother to learn
the latest MVC framework from the latest Asp.NET Core suite. That's right - I just admitted that I currently don't know how to develop a modern server-side
application in Asp.NET Core. Perhaps now is the time to learn, even if I can write the whole front end with a client-side JS Framework.

Another thing that comes to me every now and then is the realization that there are other back-end platforms that I could be using. What happened to JSP and
PHP that I've previously touched? What about Ruby on Rails? I am tempted to pursue all of those, but I think that the reasons of making myself feel better
that I know more and that all this knowledge will make me more marketable are NOT strong enough compared to the vast amounts of time learning all of them and
ensuring that I don't get confused. I've got projects I need to work on, and I have to use Asp.NET Core and other frameworks to get the jobs done in a timely
manner! Besides, I came across a vlog that advised to only learn new things on an as-needed basis.

There are also databases besides MSSQL, and no, I have not had enough time to explore all of them. I learned MySQL first, but didn't stick to it, though. I've
heard of NoSQL stuff, and I found them quite confusing so far. I definitely am not comfortable using it for production, so I'll default to MSSQL for now for
more urgent projects.

## Getting into MVC?

Honestly, I still find no reason to pursue Asp.NET Core MVC besides curiosity and to load my skills set. For one, MVC is a server-side framework, so once I
get into it, I am stripped of the capabilities that Angular afforded me. All of a sudden, I can no longer easily make API calls, for example, to verify
that a username is already taken before allowing the new user to register. I can no longer easily pop out modal dialogs and alerts. Managing responsiveness
has to now be all done in plain CSS with messy media queries, and I can no longer use the browser's width to determine whether to make certain API calls or
not.

Much of the data that I have the user manipulate are very complex - levels deep and with quite a few arrays of complex objects. Maybe it's possible, though
tedious to render all the required HTML upon page load. However, removing items in an array naturally plays nicely with client-side manipulation. Sure, I
could use plain JS or jQuery to dynamically add elements on the page, but the server-side wouldn't know about the stuff I thought I added to the model, since
it knows only the data-element pairings of only the server-side-rendered elements on page load. No, I don't want to refresh the entire page just because I
added, deleted, or re-ordered items in an array!

I took a peek at Razor, the "view" aspect looks downright scary! I actually played around with MVC since Asp.NET 4.5.x upto Asp.NET Core 1.0. The lack
of code completion help made developing in Razor, combined with so much "syntax" to have to learn that felt unnatural for me were frightening enough.

Right now, I'm 