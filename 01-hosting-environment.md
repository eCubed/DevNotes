# Hosting Environment

Throughout an application, we may need to know whether the application is running in development or production. Asp.NET Core provides an easy
way to be able to determine - the `IHostingEnvironment` that we may access via constructor injection, including Startup.cs's own constructor.