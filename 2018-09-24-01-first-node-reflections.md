# First Node Reflections

I have been recently been told the possibility of developing and deploying a Node.js application. I have known that Node existed for quite a few years now. I have
only installed it in my systems because it was a prerequisite for much of front-end development. I've only ever installed Node packages because I needed them in my
client-side applications. Supposedly we can actually write entire websites entirely in Node.js, and even host that website without web server software like IIS,
Apache, or Nginx - all of those capabilities with just pure JavaScript. 

To many developers, this seems outrageous in so many aspects. So, pretty much, we just install Node in our systems, and write websites that already know how to 
listen to actual web requests from out there. We would then need to write our apps to recognize routes and respond accordingly. But such an app will also need to
connect to a persistent data store. Some apps will also need to be secured with tokens and SSL. There is no Entity Framework. There is no LINQ. We will handle
loosely-typed data. There isn't typescript that will make development easier. As front-end developers, we are used to having tooling that would minify and uglify
for production. It seems like what we code in .js files are deployed as-is. How would we run a Node application?

What was described so far seems like back-end development in pure JavaScript. Would we actually use Node to actually serve front-end websites? If so, what would be
the purpose of still developing with a front-end MVV* framework?

One strong point of Node is that it is an event-driven, non-blocking runtime environment. This means that we will need to hook our code to events that will just
happen at any time, and that other events and their event handlers happen concurrently. The non-blocking part means that if one request takes long to complete, it
won't freeze the entire application until it's done. While that one long-running request is executing, the app can take and complete requests. This means that we
need to familiarize ourselves with events and their parameters so we can handle them.