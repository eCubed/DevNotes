# Angular SSR (Universal) Quick Guide

Upto this point, we wrote our app, tested the heck out of it, proven that it works, ran `ng build --prod`, copied the output (contents
of the `/dist` folder) over to our server, then boom - our SPA is up and running well as intended. We have a fully-functioning app that
meets aesthetics and UX expectations, and the whole app runs on the browser. So far so good? That depends on our app.

For now, let's pretend that our SPA is large, that is, there are perhaps hundreds of routes, components, services, directives, etc. When
a user pulls up our app on their browser, the ENTIRE app is loaded, which could be a number of MB in total size. That will slow down the
load time and eat up bandwidth, and there's a good chance they'd only actually use a small fraction of it. When our huge app is set up with
SSR (server-side-rendering), ONLY the minimal amount will be loaded upon landing on our app - only what's necessary to fully render that
front page. Now, when the user navigates to a new route, the SSR app will actually NOT generate the new page on the browser side, but will
actually call up the back end to go render the necessary HTML/CSS/JS, and then load it for the user.

When our SPA has content to share, we probably have specific routes (that have identifying parameters) that we want Facebook, et. al., to
recognize so we can plop share buttons on them. We know that social media scrapers look for certain meta tags on the html page so they have
enough info to nicely and properly post them on people's timelines. Now, let's first consider the fully-client side implementation as we might
have right now. When FB encounters the full URL for our resource, it will grab the meta tags that it sees *right away*, guaranteed before that
route's `GET /api/resource/:id` call completes. That's a problem - FB won't receive the necessary info to share our resource. With SSR, the
FB requests the same route, but now, that route's HTML/JS/CSS is generated at the back first, with the completed API call, thus meta tag
values, and FB will get the data we intend!

## Angular Universal on Development

We'll need to first run the following at the prompt, and we should be at the root our our Angular app:

`ng add @nguniversal/express-engine`.

This adds files to our project as well as modifies certain files to make our app SSR-ready. We'll need to give npm some time to install all the
necessary packages.

Now, we'll need to run the command

`npm run dev:ssr`

Our app will automatically run on port 4200, so we'll need to open our browser to `http://localhost:4200` to see our app. When we run it, it
looks no different to the user than the fully-client-side implementation. However, what goes on in the background is very different. Instead of
the browser generating the HTML/CSS/JS when the user navigates to a different route, all of that is generated at the server, and then passed on
to the front for display. If we land on a content page of our app (that is, a route whose component will make an API call to obtain data),
and then view the page source, we DO get the full of what we see on the screen. When our app wasn't SSR, the page source will be the same
empty `index.html` page with the `<router-outlet>` tag in it - nothing more, because all the content we see are actually dynamically generated
DOM nodes. Now, for a social media scraper landing on an SSR page, it will immediately have all the meta tag values on first scrape because
the API call to get the data would have already been called and completed before serving the page!

## Angular Universal on Production

To serve up an Angular SSR app on an actual server, we're going to need to perform some setup. I'm using IIS as my web server, but SSR runs on
Node, so I'll need to make those work together. Later on that.

But first, we'll need to make the production build:

`npm run build:ssr`

Once that's done, let's go see what it generated on the `/dist` folder. Right underneath the `/dist` folder, is a folder named the same as our
app when we created it. Inside that folder are two folders - `/browser` and `/server`. The contents of the `/browser` is the same as when we
ran `ng build --prod`. Now, there's the `/server` folder, which contains one file - `main.js`. We need not investigate this file because it's
huge, and uglified. We'll just trust that it will all work. But one important thing to note is that this `main.js` is going to be the ENTRY POINT
now, instead of the good old index.html. This `main.js` has specific code that knows about our app, and contains the magic that will render to
the browser ONLY the HTML/CSS/JS necessary to display that one page.

Before we do anything, we will need to download [IISNode](https://github.com/Azure/iisnode/releases). I chose the latest one, full version,
x64 msi. I'll go ahead and install that by just clicking on the downloaded file and running it.

By this time, I should have already set up the DNS and website for my app, and have already found the root folder of the app according to IIS.
Into that folder, I FTP the entire `/dist` folder INCLUDING the root `/dist` folder itself as a child folder under my app's root per IIS.
Then, I COPY (I'm now at my server) `/dist/my-site/server/main.js` to the root of my app. Then I make the following entry in web.config:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <system.webServer>
      <handlers>
         <add name="iisnode" path="main.js" verb="*" modules="iisnode" />
      </handlers>
      <rewrite>    
        <rules>      
          <rule name="http to https" stopProcessing="true">
             <match url="(.*)" />
             <conditions>
               <add input="{HTTPS}" pattern="^OFF$" />
             </conditions>
             <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" />
          </rule>
          <rule name="Angular Routes">     
            <match url="/*" />
            <action type="Rewrite" url="main.js" />
          </rule>
        </rules>
      </rewrite>
    </system.webServer>
</configuration>
```

Note that I add the handler `iisnode` because we'll need to run that, and this tells IIS to run it when the web app starts. I point to
`main.js`, which I just copied to my app's root as mentioned above.

I included the `http to https` rewrite rule to show that it MUST be stated before the `Angular Routes` rewrite rule. Now, we point
any path to that same `main.js` file because that's the JS file that does all of the magic of SSR for us.

We need to give IUSR and IIS_IUSRS full privileges to our app, AND also to the directory where IISNode was installed. In my case, it got
installed to `C:\Program Files\IISNode`, so I gave IUSR and IIS_IUSRS full access to those.

