# Asp.NET Core - Proxy Servers - Creating


We will be creating a very simple proxy server utilizing the package `Microsoft.AspNetCore.Proxy` which may be installed from Nuget. Our proxy server is simple, which
means all it does is "sniff" the incoming request's path and then based on the path's pattern, forward to the right servers. `Microsoft.AspNetCore.Proxy` is currently
at version 0.2.0 at the time of writing, and has been for quite a while. Since it's at version 0, it's not really intended for production, so we would be using it at our
own risk if we would deploy our proxy server to production.

## Creating the Proxy Server Web Application

We will need to create an *EMPTY* Asp.NET Core web application project. This will work for either Asp.NET Core 2.0 or 2.1. Then, we go ahead and install to this new project
`Microsoft.AspNetCore.Proxy` from the Nuget Package Manager.

## Configuration

We will be using User Secrets and a strongly-typed configuration for this project. For more information on those, search online about them.

Here, we have the models that make up our configuration. They are constructed based on the needs of `Microsoft.AspNetCore.Proxy`:

```csharp
public class ProxyServerConfig
{
    public EndpointInfo FileServerEndpoint { get; set; }
    public EndpointInfo ResourceServerEndpoint { get; set; }
    public EndpointInfo AuthServerEndpoint { get; set; }
}
```

```csharp
public class EndpointInfo
{
    public string Scheme { get; set; }
    public string Host { get; set; }
    public int Port { get; set; }
}
```

Now, we have the contents of secrets.json, for example:

```json
{
  "ProxyServerConfig": {
    "FileServerEndpoint": {
      "Scheme": "http",
      "Host": "localhost",
      "Port": 50312
    },
    "AuthServerEndpoint": {
      "Scheme": "http",
      "Host": "localhost",
      "Port": 50310
    },
    "ResourceServerEndpoint": {
      "Scheme": "http",
      "Host": "localhost",
      "Port": 50311
    }
  }
}
```

At `Startup.cs`'s Configure services, we read the contents of secrets.json (for dev) or the corresponding environment variables (for production):

```csharp
 public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        ProxyServerConfig proxyServerConfig = new ProxyServerConfig();
        Configuration.Bind("ProxyServerConfig", proxyServerConfig);

        services.AddSingleton(proxyServerConfig);

        services.AddCors();
    }
}
```

Note that we do not need to use User Secrets, but it is a good practice for security of our code base. It is also debatable whether we could have used appsettings.json
instead since we didn't store undoubtedly sensitive information such as connection strings or API credentials. If we choose to use appsettings.json, we can go ahead and
copy and paste *exactly* the same JSON contents that we have in secrets.json.

## Utilizing The Microsoft Proxy Library

In our `Startup.cs`s  `Configure()`, we utilize the builder extensions provided by the Microsoft proxy library:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ProxyServerConfig proxyServerConfig)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseCors(options =>
    {
        options.AllowAnyOrigin();
        options.AllowAnyMethod();
        options.AllowAnyHeader();
    });

    app.MapWhen(context => context.Request.Path.Value.StartsWith("/token", StringComparison.OrdinalIgnoreCase),
        builder => builder.RunProxy(new ProxyOptions {
            Scheme = proxyServerConfig.AuthServerEndpoint.Scheme,
            Host = proxyServerConfig.AuthServerEndpoint.Host,
            Port = proxyServerConfig.AuthServerEndpoint.Port.ToString()                        
        }));

    app.MapWhen(context => context.Request.Path.Value.StartsWith("/api/upload", StringComparison.OrdinalIgnoreCase),
        builder => builder.RunProxy(new ProxyOptions
        {
            Scheme = proxyServerConfig.FileServerEndpoint.Scheme,
            Host = proxyServerConfig.FileServerEndpoint.Host,
            Port = proxyServerConfig.FileServerEndpoint.Port.ToString()
        }));

    app.MapWhen(context => context.Request.Path.Value.StartsWith("/api", StringComparison.OrdinalIgnoreCase),
        builder => builder.RunProxy(new ProxyOptions
        {
            Scheme = proxyServerConfig.ResourceServerEndpoint.Scheme,
            Host = proxyServerConfig.ResourceServerEndpoint.Host,
            Port = proxyServerConfig.ResourceServerEndpoint.Port.ToString()
        }));

    app.MapWhen(context => (!string.IsNullOrEmpty(Path.GetFileName(context.Request.Path.Value))),
        builder => builder.RunProxy(new ProxyOptions
        {
            Scheme = proxyServerConfig.FileServerEndpoint.Scheme,
            Host = proxyServerConfig.FileServerEndpoint.Host,
            Port = proxyServerConfig.FileServerEndpoint.Port.ToString()
        }));
}
```

We detect four patterns of the incoming path with the built-in `.MapWhen` function - token endpoint, uploading, web API calls that return JSON, and paths that end in
a file name. For each pattern, we utilize the `builder.RunProxy` function to specify the appropriate server to call. Note that the library splits the scheme, host, and
port numbers, which is why we also separated them in our configuration file for easy conversion.

That's it. As long as the other servers are running, we may now run this application and it will forward to the right servers according to what we decided in `Startup.cs`.

Note that the above example is *just* an example. Our system may have other servers and other recognized URL patterns, and we would have to write a mapping between pattern
and endpoint as we have done above.

Note that the API calls to be made to this proxy server are *public* and may be called from outside. This is fine because its job is to merely forward, not to authenticate
or authorize.

## Limitations

The `Microsoft.AspNetCore.Proxy` will only forward our requests to the right servers, and return the exact responses from the remote servers. It does not give us the opportunity
to intercept the incoming request, should we need to alter or add to the headers, for example, or intercept the response so that we may perform administrative tasks based on
the response data. To be able to perform those, we will need to explicitly build our own proxy servers.
