# EF Migrations

This writeup is for EF6 in with Asp.NET 4.x.y. The .net Core migration process is quite different!

Let's say we created all of our classes, with the appropriate decorations per property as needed. For me, I like to create a library that
contains all of my entity classes, data context, and seeder. This library will NOT contain any migration code because that's just extra
baggage when the dll makes it to production.

Here's the db context:

```csharp
    using System.Data.Entity;
    ...
    public class MyStuffDbContext : DbContext
    {
        public DbSet<MyThing> MyThings { get; set; }

        public MyStuffDbContext() : base("name=MyStuffConnectionString")
        {
        }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
        }
    }
```

The constructor utilizes `base("name=MyStuffConnectionString")`, which tells the system to go find the connection string in web.config with
the name `MyStuffConnectionString`. In our library, we don't have a web.config, so we'll need to create an app that references our library.

## Creation of the Migration App

I'm using VS 2019, so I create an Asp.NET Framework Web App (C#, not VB), .net 4.5.1 (because that's the target of my library), and reference
the library mentioned above.

Now, I need to install via Nuget the package `EntityFramework` even if my class library may already reference that.

Now, I'll need to add the connection string entry in web.config:

```xml
    <configuration>
    ...
        <connectionStrings>
            <add name="MyStuffConnectionString"
              connectionString="Data Source=My-Machine\SQLEXPRESS;Initial Catalog=MyStuffDotNet;Integrated Security=True"
              providerName="System.Data.SqlClient"/>
        </connectionStrings>
    </configuration>
```

Note that this web.config DOES make it to source control, and I'm exposing the connection string. This is dev, so it's alright. We'll just have to be
careful when we actually need to deploy this. Unfortunately, Manage User Secrets is not available in .net Framework applications.

I'm at the Package Manager Console now inside VS 2019. First, I'll need to select Default project to be my migration app. Also, I'll need to
ensure that I'm at the root of my migration app at the PM prompt. I may have to `cd` to it if not already there. This is NOT obvious, but extremely important -
the migration app NEEDS TO BE SET as the startup app of the solution when we run following commands!

Now, I run `Enable-Migrations` because I'm about to perform migration for the first time. This creates a `Migrations` folder in the migration
web app with a single fine `Configuration.cs` on it. I'm not going to touch this file at this point.

I then run `Add-Migration InitMigration` where `InitMigration` is the name of my (what's going to be my first) migration attempt. This creates
a time-stamped file name appened by the migration name (`InitMigration`) that I just specified.

Finally, I'll run `Update-Database`. This should create the database if not already, and create all of the tables and their relationships as needed.

