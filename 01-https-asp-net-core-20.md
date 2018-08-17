# Asp.NET Core Web Application - HTTPS

HTTPS is something that many developers pay little attention to because they are so focused on their data and Asp.NET Core features.
Also, many developers assume that they merely only copy their published code into a folder pointed to by IIS, and IIS will take care
of all things HTTPS. It turns out that yes, in our apps, especially at Startup.cs, we do have to write some HTTPS-related code.

Note that this article is for Asp.NET Core 2.0, not 2.1!!!

# Basic Setup

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

If we merely run the above, our app won't run. We've got to go to Properties > Debug (tab), and check "Enable SSL". When we check the box,
a random port will be assigned as our app's https port. If we uncheck and check that box again, we will get a new random port.

When we run this the first time, we will be prompted to create a self-signed certificate. Choose yes. Note that Firefox doesn't trust the
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

Note that when the app is not in development, it will automatically redirect to the https with the same domain, with port 443 implied.

Let us pretend for a while that our app has been deployed to production. The system administrator would set up our app in IIS, and they
have to specify an https binding, and most a port other than 443. Whatever they choose, we need to know because we need to put this port
number in Startup.cs. Let's say that the system administrator chose port 9561 for the app's https. We'll need to make the following
modification:

```csharp
var options = new RewriteOptions();
            
options = (env.IsDevelopment()) ? options.AddRedirectToHttps(StatusCodes.Status301MovedPermanently, 44368) : 
   options.AddRedirectToHttps(StatusCodes.Status301MovedPermanently, 9561);
app.UseRewriter(options);
```

The port numbers are hard-coded in this sample code, however, we can instead store them to and access them from appsettings.json or other
secure store.

## Simulating Production on a Dev Machine with IIS

We will simulate this simple web application to run on production from our local machine's IIS. Let's publish the app and have IIS point
to the publish output folder. (We assume we know how to publish and basic setup in IIS).

We will need to add another binding (the https) aside from the one automatically created. For this https binding, we can choose whatever
port we want. The default SSL port is 443, though we may still explictly specify 443 if we wanted to. If we specified a different port
for the production forwarding here in IIS, we will need reflect that in Startup.config (see the end of the previous section). If we called
options.AddRedirectToHttps()` without any arguments, we will need to set the https binding's port to 443.

We will also need to select a certificate. Since we're still really in development, just simulating a production scenario,
we will need to choose the IIS Express Development Certificate. In a real production scenario, the appropriate certificate would have already
been installed, can be selected, and would then be selected from IIS.

We can open the browser and browse directly to the https location and our site would load. Note that on FireFox, we would need to browse to
the https url first, and then add it as an exception. Then can we only browse to the unsecure location to see that forwarding will work.
If we browse to the http location first, nothing would ever work. We would need to restart our website again in IIS. Note that this
https-first opening will require another adding to exceptions in Firefox for every time Firefox closes and then restarts.

On Fiddler, we specify the non-secure port, and we can see that it would immediately forward.

## Deploying Asp.NET Core to a Shared Host

If we have access to an actual host or VPS where we can add websites in its IIS, we are all set. Many of us don't have that capability, and some
of us do not want to spend extra on a VPS. There is the cheaper shared hosting, though, where our website literally resides in the same machine
as other customers' websites. There are many questions as to how this would work, many of which depend on what our chosen hosting provider offers.
We may or may not need to alter our so-far established https code in our app's Startup.cs.

- Do we all share a single certificate with other customers? 
- Does the host give us our own certificate that will be automatically applied to our domain? 
- How do we specify an https port for our domain? If they assign one to us, then how do we find out the port number so we can set up port forwarding
in Asp.NET Core code?
- What if we have subdomains and we also want all of them to require https? Can we use the same certificate assigned to us?
- Will we be able to omit https forwarding code in our app's Startup.cs?


## Redirect and URL Rewrite?

The following setup is for the situation when we do not have any https-related code in our app's Startup.cs, and we still want to force-forward
our app to https.

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
9: Apply.

So, it appears that we do not have to do anything in Asp.NET Core to make our app require HTTPS. There are quite a lot of steps involved, and it
seems like writing the few lines of code in Asp.NET Core seems easier. Also, this approach requires that the https port is 443, which we may not
be able to have.

The above steps are real instructions, however, it is not necessary when we deploy Asp.NET Core web application that we set up https in,
in its Startup.cs. This IIS URL rewrite will not take effect when the Asp.NET Core web application itself is https-configured.

We will only keep this section as "reference just in case".

