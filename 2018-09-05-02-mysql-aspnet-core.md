# MySQL in Asp.NET Core

I am thrilled to report that I have successfully performed code-first migration to a MySQL database. There are a very few things that I needed to do:

## Database Migration
I needed to install the Nuget package Pomelo.EntityFrameworkCore.MySql, the latest version. At the time of writing this, the latest version is 2.1.2. 
Though it depends on framework 2.0, it runs on my framework 2.1 application!

Then, on startup:

```csharp
public void ConfigureServices(IServiceCollection services)
{
   ...
   
   services.AddDbContext<MySqlDbContext>(options =>
   {
      /*
      options.UseSqlServer(connectionString, opts => {
         opts.UseRowNumberForPaging(); // if using SQL Server 2008 or earlier
      });
      */

      options.UseMySql(Configuration["ConnectionString"]);
   });    
}
```
I just left the MSSQL setup commented out for comparison.

I also do this in the Context Factory file:

```csharp
public class MySqlDbContextFactory : IDesignTimeDbContextFactory<MySqlDbContext>
{
   public MySqlDbContext CreateDbContext(string[] args)
   {
      IConfigurationRoot configuration = new ConfigurationBuilder()
         .SetBasePath(Directory.GetCurrentDirectory())
         .AddJsonFile("appsettings.json")
         .Build();

      var builder = new DbContextOptionsBuilder<MySqlDbContext>();

      var connectionString = configuration["ConnectionString"];
      
      //builder.UseSqlServer(connectionString, b => b.MigrationsAssembly("MyProject.DevDeploy"));
      builder.UseMySql(connectionString, b => b.MigrationsAssembly("MyProject.DevDeploy"));
      
      return new MySqlDbContext(builder.Options);
   }
}
```

I'll fix the formatting later.

That is pretty much it. I can run the same migration commands all the way upto `dotnet ef database update`, and it will migrate to the database!

I found that I am unable to set a default value SQL for a date field, though. The workaround for that is to create a BEFORE INSERT trigger at MySQL. Later on that.

## Communicating with MySQL in an Asp.NET Web App

Much of the work is already done, per the `ConfigureServices()` function shown above. All we need to do is to get a hold of the
data context from wherever we need from our app, as usual with the MSSQL we're used to. We can dependency-inject the MySQL
data context via constructor (our homemade Provider classes, Web API controllers) and to functions that the framework calls for
us (like the `Invoke()` function of a piece of middleware).

