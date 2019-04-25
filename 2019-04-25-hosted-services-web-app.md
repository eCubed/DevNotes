# Hosted Services on Web Applications

A background service is an execution we will want our application to run throughout its lifetime in the background. This article is about creating and setting
up background services for web applications, and we surely can set them up in console apps as well. While the service runs in the background, our web 
application may still take requests to serve data or to display web pages as usual. The background service will automatically run on its own thread, thus will
not block execution of normal requests.

There are a few considerations with background services unique to web applications that are not applicable to console apps, and we will discuss them here.

## Creating a Hosted Service

Asp.NET Core has the interface `IHostedService` which we may implement. However, it is more likely and more practical to instead derive the abstract class
`BackgroundService` as it already has the base code needed to actually execute our hosted service.

The following is a skeleton hosted service, deriving `BackgroundService`

```csharp
public class BasicHostedService : BackgroundService
{
	public BasicHostedService()
	{
	}    

	protected override async Task ExecuteAsync(CancellationToken stoppingToken)
	{    
		while (!stoppingToken.IsCancellationRequested)
		{
			/* Code to execute repeatedly goes here */
			Thread.Sleep(5000);
		}

		await Task.FromResult(0);        
    }
}
```

We shall assume that a hosted service will indefinitely execute the same code throughout our app's lifetime. We put that code inside the while loop as above,
with the condition `!stoppingToken.IsCancellationRequested`, which we will not get into detail in this article. We could simply put `true`, and it could
work as we intend, but it is a good practice to execute that while loop while cancellation isn't requested! It is also good practice to end the loop with a
pause of a few seconds, 5, in our example so that our system can "breath" a little instead of executing the loop as many times as it can within a given time
interval.

## Registering a Hosted Service

We'll need to tell our app at `Startup.cs` that we want our hosted service to run, so we add the following line:

```csharp
	
public void ConfigureServices(IServiceCollection services)
{
	...
    services.AddHostedService<BasicHostedService>();
	...
}
```

When we run our app, a single instance of our hosted service will be created, and will execute on a thread separate from our main thread.

## Injecting Services into a Hosted Service

Yes, hosted services *do* participate in dependency injection mechanism as Controllers, Middleware, and any services we may write. However, care must be taken
in selecting and managing what services we can inject into our hosted service.

A hosted service *is* a singleton, thus, we can only inject services into our hosted service that we know for sure are singletons, and do *not* themselves inject
non-singleton services. Transient services like an EF database may be accessed from our hosted service, but we will have to pull that in a different way.


```csharp
public class DependencyHostedService : BackgroundService
{
	private ISingletonService SingletonService { get; set; }
	private IAnotherSingletonService AnotherSingletonService { get; set; }
	private IServiceProvider ServiceProvider {get; set;}

	public DepencyHostedService(ISingletonService singletonService, IAnotherSingletonService anotherSingletonService, 
		IServiceProvider serviceProvider)
	{
		SingletonService = singletonService;
		AnotherSingletonService = anotherSingletonService;
		ServiceProvider = serviceProvider;
	}    

	protected override async Task ExecuteAsync(CancellationToken stoppingToken)
	{
		using (var scope = ServiceProvider.CreateScope())
		{
			ITransientService transientService = scope.ServiceProvider.GetService<ITransientService>();
			MyDbContext db = scope.ServiceProvider.GetService<MyDbContext>();

			while (!stoppingToken.IsCancellationRequested)
			{
				/* Code to execute repeatedly goes here */
				...
				SingletonService.DoSomething();
				AnotherSingletonService.DoSomethingElse();
				...
				transientService.DoYourThing();
				var entity = await db.SomeEntities.SingleOrDefaultAsync(id => id == 1);
				...
				Thread.Sleep(5000);
			}
		}

		await Task.FromResult(0);        
    }
}
```

In the code above, `ISingletonService` and `IAnotherSingletonService` are guaranteed to be actual singletons that don't depend on any services that are not
singletons themselves, all the way up to the root-most ascendant service. We can just inject them via constructor injection as we do with Controllers and Middleware.
We can readily use them in the `ExecuteAsync` function.

To get access to transient and other non-singleton services, we will need to inject `IServiceProvider`, which is something that the Asp.NET Core provides that
we can just inject anywhere we need, thus we didn't need to register at `Startup.cs`. We will need to get a `scope` from the injected ServiceProvider, which is
the means to safely obtain any transient service as registered in `Startup.cs`. (Verification needed) The scope from which we'll obtain transient services
is different from the scope from which our normal Controllers, Middleware, etc., thus separate instances we can safely use from inside the singleton hosted service.
Note that we need to wrap the scope in a `using` block so that all of the services we called upon will automatically be disposed when the hosted service's
lifetime is over.

## A Few Curiosities

**Can a Hosted Service be Injected to a Controller, Middleware, etc.?** Since we add it to the service collection, *yes*, and our code will compile if we attempt
to, **but we shoudn't**. By design, a hosted service is supposed to be created and left alone to execute in the background *on a different thread*. So, getting
a hold of it from the main thread in a controller can be potentially unpredictable and dangerous.

**How do we programmatically stop a hosted service, and manually start it again during the same lifetime of our app?** We can't. By design, it's automatically
created by the framework, and the `ExecuteAsync` function would be called again with the proper cancellation token when the hosted service is taken down at the
end of our app. One workaround we can do is to inject a singleton service that keeps a state of whether to keep on running or not. We would set this service
to run or stop from a controller, middleware, or another service, while the hosted service would continually read the run/stop state, and then decide when
to stop/restart.

## Next Steps

We will need to address how we can create a background service in a console or forms application. It will be similar to a web application.

