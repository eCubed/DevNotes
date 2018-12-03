# Debug Mode Databases

Throughout my backend development for the last few years, I have always relied on an actual MSSQL database for development work. I referred to it as the
dev database, and for the most part, it resided on a different machine in our company's network. For projects at home, though, I ran an actual MSSQL server
instance on my machine, and created the dev database there. 

Supposedly, that setup was wrong, despite being the approach taught by ASP.net Core and EntityFramework tutorials and even official documentation. Dev databases
are NEVER supposed to reside on a live SQL server. So, in practice, we should NEVER set up a live SQL database and regard it a "development database". And
by "live", I mean the SQL server is actually set up on a physical machine and there exists a subdomain that points to it. Supposedly, I shouldn't even have
set up an SQL server *even on my local machine* for the purpose of storing and retrieving dev or debug data. So, what is the right approach and setup?

## Problem with Remote Database Setup

There were times that our "dev database server" went down, and when it did, we could NOT develop. This meant that we couldn't test our logic because our setup
required the external database connection. Many times, it would be hours until IT resolved it and put it back on.

Even when the remote database server is up, there were still problems. All developers had to write code based on the schema that is currently on that "dev
database" at that given time, and subsequently, all developers had access to the same exact test data. Ideally, each developer wants to see and manipulate
*only* the data they saved, but with this current setup, all developers will see each others' test data, which isn't really that harmful, but makes it more
difficult to focus on their own development. 

Suppose that it was time to change the schema, and one developer went ahead and performed the code-first migration. Depending on the changes, the schema 
change could all of a sudden break another developer's development, because now, tables and/or columns may be gone. Other developers would be getting
unexpected errors, and that would be a serious issue that causes frustration and lost productivity.

So what is the cure for the problems with a remote database setup? In short, each developer would need to have their own local databases on their own machines.
We will address proper local database setup shortly.

## Problem with Local Machine Database Instance Setup

If each developer had local copies of the database in their machines, that would be far better than if everyone shared the same remote database. However, if the
local database was created on the local SQL server's instance, there are still problems. Each developer can now make their own changes to the schema without
intruding on another developer's development. But each developer still needs to perform code-first migrations to their test databases, and those migration
files would severely conflict when it's time to merge everyone's development.

## The Ideal (and Happens to be the Correct) Solution

Yes, each developer will have a local database for their development, but NOT on the local SQL server instance. There would be 0 code-first migrations to
perform in development. The database would be *automatically* created if not already created, and seeded with the *absolute minimum required data*. At any
time, each developer should be able to totally obliterate the current database an be able to "start from scratch" seamlessly, that is, the database would
be automatically recreated and reseeded. How would we pull this?

## Requirements

We must already have an ASP.net Core web application running on a local machine, yes, in debug mode off VS. The app can be a faceless Web API or an MVC app.
There must be a `Seeder` class whose job is to populate the database with the minimum required data, and it must be called from `Startup.cs` in development
mode. The `Seeder` should be written such that it will only create records if not already existing, and each app will determine uniqueness for seed records.


**IMPORTANT**. On the `Seeder`, we need to make to include the line `db.Database.EnsureCreated()` or `await db.Database.EnsureCreatedAsync()` before
the code that creates the first record.

## LocalDB

Many of us might not know this, but just because we have VS, we automatically have the tool called SQL Server Express LocalDB. No, it is *not* the same as the
local SQL server instance. It is a very basic version of SQL server that is *intended* for development, per official documentation. Therefore, the local
SQL server instance is *not* intended for development. It must be intended for applications we want to deploy that would only run on our local machines.

So, what do we do? We simply set the connection string to this format:

`Server=(localdb)\\mssqllocaldb;Database=MyDb;Trusted_Connection=True;MultipleActiveResultSets=true`

We won't get into how we would store and retrieve that string since that's a different topic. However, regardless of where that connection string is
stored, whether `appsettings.json` or `secret.json`, or even hardcoded, it needs to be set to the above format. The `(localdb)` part is required
as-is, including the parentheses. The `mssqllocaldb` is also required *to be there*, but we can rename it to whatever we want. The `mssqllocaldb` is the
name of the LocalDB instance where we can host our dev databases. Then, for our specific database, we can replace `MyDb` with our database name. We'll also keep the settings `Trusted_Connection`
and `MultipleActiveResultSets` as they are.

So, now, what do we do? We build the project to make sure there are no errors. We run it from inside VS in debug mode. The system actually creates an mdf
file and its accompanying log file, named after the database name we've specified in the connection string. The app can now run and will actually now save
to and read from that mdf file. That mdf file is stored under `C:/Users/<Username>/` on Windows.

Back in VS, we can go View > Sql Server Object Explorer. It may be empty at first, so we'll need to right-click on `SQL Server` > Add SQL Server. A 
dialog pops up. If we twirl open the item called Local, we should see a list, and one of the items should be `mssqllocaldb`. We would select that and
click `Connect`. From inside VS, we can find our database, view the tables and records in a similar but limited fashion as we would have in SQL
Management Studio.

## Starting Over

We're still in development, so it is perfectly acceptable and a normal part of our workflow to scrap our LocalDB database and start over again. We will
want to start over again if our schema changes significantly. We can also start over again if we've decided that we're done with the test data and can move
on without them. By the way, in practice, once we have proven that a particular mechanism works, the data that we stored during developing and testing that
mechanism can be scrapped.

To start over again, we need to do a couple of things. One, is to delete the mdf file and its log right from File Explorer. The second is to actually
delete the entire LocalDb instance. For this, we need to go to VS's Package Manager Console, then execute the following commands, one after the other:

`SqlLocalDb.exe stop MSSQLLocalDB`

`SqlLocalDb.exe delete MSSQLLocalDB`.

So now, our dev database is totally gone. All we have to do is to run the web app again, and our database would be recreated and reseeded. It would be
recreated based on any changes we made to our entities and our `DbContext`, though, which is what we want.

## Sharing the Application to Team Members

Other developers in our team may go ahead and clone the repo into their machines. They will need to set up the *same connection string* on their machines,
though, and the way to do that depends on whether they use `appsettings.json` or `secrets.json`, or hardcoding. However, `secrets.json` is highly
recommended for both security and not needing to alter any file on the repo. Once cloned, that other developer would then run the application in debug
mode, and the database would be created and seeded for them.

Now, each team member can work on their own branch, and in their branch, they may make some database-invasive changes for their own development, and
these changes will not affect anybody else's development.

## Unexplored Territory

One major task we didn't explore is EF migrations on LocalDB. My guess is we can use code-first migrations as long as we have the connection string.
However, we probably should refrain from doing code-first migrations in development because that would create files that would be inadvertently
checked into the repo, and would conflict with another developer's code-first migrations.






