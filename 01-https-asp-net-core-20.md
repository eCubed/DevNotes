# Asp.NET Core Web Application - HTTPS

HTTPS is something that many developers pay little attention to because they are so focused on the data and Asp.NET Core features.
Also, many developers assume that they merely only copy their published code into a folder pointed to by IIS, and IIS will take care
of all things HTTPS. It turns out that yes, in our apps, especially at Startup.cs, we do have to write some HTTPS-related code. This
matter isn't very simple, so we'll go through various setups, including development.

Note that this article is for Asp.NET Core 2.0, not 2.1!!!

# Basic Setup in Development

Startup.cs:

```csharp
public class Startup
{        
   public void ConfigureServices(IServiceCollection services)
   {
      /* We need to let MVC know that all calls to this application will need to require
         https.
      */
      services.AddMvc(options =>
      {
          options.Filters.Add(new RequireHttpsAttribute());
      });
   }
       
   public void Configure(IApplicationBuilder app, IHostingEnvironment env)
   {
      /* If ever a call comes in with only http, redirect it to https.
      */
      var options = new RewriteOptions()
          .AddRedirectToHttps();

      app.UseRewriter(options);

      if (env.IsDevelopment())
      {
        app.UseDeveloperExceptionPage();
      }

      app.UseMvc();
      
      app.Run(async (context) =>
      {
          await context.Response.WriteAsync("Hello World!");
      });
   }
}
```

If we merely run the above, we our app won't run. We've got to go to Properties > Debug (tab), and check "Enable SSL". When we check the box,
a random port will be assigned as our app's https port. If we uncheck and check that box again, we will get a new random port.

When we run this the first time, we will be prompted to create a self-signed security. Choose yes. Note that Firefox doesn't trust the
self-certificate, so we will need to add it to its exceptions. Chrome automatically trusts it.

Once we run the app on the browser, it will forward to `https://localhost` without the port number. Actually, that default port is 443,
but that's not our VS IIS Express https port for our app in development. So, let's make some changes in `Configure()` :

```csharp

var options = new RewriteOptions();
            
options = (env.IsDevelopment()) ? options.AddRedirectToHttps(StatusCodes.Status308PermanentRedirect, 44368) : 
   options.AddRedirectToHttps();
app.UseRewriter(options);
```

Note that the port number is the same one as assigned in the Debug tab when we enabled SSL, and yes, we hard-code that!

Note that when the app is not in development, it will automatically redirect to the https with the same domain, so port 443.
If we want to specify a different https port for production, we can, but we need to make sure to reflect the same port number
in IIS later.

## Simulating Production on a Dev Machine with IIS

We will simulate this simple web application to run on production from our local machine's IIS. Let's publish the app and have IIS point
to the publish output folder. (We assume we know how to publish and basic setup in IIS).

We will need to add another binding (the https) aside from the one automatically created. For this https binding, we can choose whatever
port we want. The default SSL port is 443, though we may still explictly specify 443 if we wanted to. If we specified a different port
for the production forwarding, we need to reflect it here in IIS. If we called `options.AddRedirectToHttps()` without any arguments,
we will need to set the https binding's port to 443.

We will also need to select a certificate. Since we're still really in development, just simulating a production scenario,
we will need to choose the IIS Express Development Certificate.

We can open the browser and browse directly to the https location and our site would load. Note that on FireFox, we would need to browse to
the https url first, and then add it as an exception. Then can we only browse to the unsecure location to see that forwarding will work.
If we browse to the http location first, nothing would ever work. We would need to restart our website again in IIS. Note that this
https-first opening will require another adding to exceptions in Firefox for every time Firefox opens.

On Fiddler, we specify the non-secure port, and we can see that it would immediately forward. 

## Redirect and URL Rewrite?

So, it turns out that we need to install the [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite). Now that we've installed
it, we can go back to IIS, to our website, and double-click the URL Rewrite Tab.

1. Actions (right-hand-side pane) > (click) Add Rule(s).
2. Inbound rule (heading) > Blank rule (brings us to Edit Inboud Rule)
3. Choose Requested URL: Matches the Patter, Using: Regular Expression.
4. Set Pattern to `(.*)`, Ensure checked Ignore case.
5. Twirl Open Conditions, ensure Logical groupoing: Match All.
6. In Conditions pane, click Add. There's a popup.
7. Set Condition input: `{HTTPS}`, Check if input string: Matches the Pattern, Pattern: `^OFF$`, Ensure ignore case checked, OK.
8. In Actions pane: Action type: Redirect, Redirect URL: `https://{HTTP_POST}/{R:1}`, Ensure Append query string checked,
Redirect type: Permanent (301).
9: Apply

The above steps are real instructions, however, it is not necessary when we deploy Asp.NET Core web application that we set up https in,
in its Startup.cs. This IIS URL rewrite will not take effect when the Asp.NET Core web application itself is https-configured.

We will only keep this section as "reference just in case".

