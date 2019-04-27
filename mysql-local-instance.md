# MySQL Local Instance

It is highly recommended (virtually critical) to create local dev databases on the dev machine. This way, we do not need to rely on an live internet connection
for our database in development. Also, we can easily wipe out a database (schema in MySQL), automatically recreate it, and then seed the new database.

# Installing a Local MySQL Instance

If we have already installed MySQL Workbench, we may get the impression that a local MySQL instance was automatically installed for us. This is not true. We still
need to install MySQL on our machines. MySQL Workbench is only a client that can connect to remote and local MySQL databases!

We will need to go to [Download MySQL Installer](https://dev.mysql.com/downloads/installer/) to download MySQL. There are two options. It is sufficient to choose
the smaller version, the "web-community" .msi file. Download and install.

Warning: Only the installer will be installed when we run that downloaded program. Once that's installed, we will need to run it so that we
can choose to install MySQL Server! When prompted, select the option `Standalone MySQL Server / Classing MySQL Replication`. Choose
the defaults in the subsequent dialogs.

Root Password: The installer will ask for a root password, and we can specify it at that point. We will need to remember the user name root
and the password we just entered because we will need to use them to later connect to MySQL from Workbench.

# Connecting to the Local MySQL Instance from Workbench

We will nedd to add a MySQL connection in the Workbench UI. There are only a few options we'll need to set:

Hostname will be 'localhost', and the port will be 3306.

For the first time logging into MySQL, we will need to specify root as the user name and the password will be the one
we specified during installation. When we get into MySQL, we can add a user account whose credentials we would use in a
connection string later. We will need to give that account all of the roles so that we can connect to the database with
a connection string from a native app later.

# Connecting to a Local MySQL Instance Database/Schema from an App

We only need to supply the following connection string, very similar to a remote connection string:

`server=localhost;port=3306;database=mydb;uid=username;pwd=password`

We explicitly specify port 3306 with the `port` option, not `server=localhost:3306` (which will not work).

# Automatically Creating the Database If Not Already Created (Development Only)

This is an external topic, but it is good to briefly address it here. This is for developing in a Windows machine.

Suppose that we have an application and we created the EF db context for it and the seeder to initially populate our db with required data. Our EF context would
have Fluent code that would declare things like delete cascade and indexes. Our seeder populating code would know how to detect uniqueness of records so that
it won't create duplicate records when the seeder is run again (because it will be ran multiple times in development).

In our seeder, it is important to first ensure that our database is created already, before writing a single line of code that inserts records to the database.

`await db.Database.EnsureCreatedAsync();`

Somewhere in our `Startup.cs` file, we would instantiate our seeder, and then call its principal population function, whatever we've named it.

At this point, our local db instance is set up, but we have NOT created our database. That is fine because it would automatically be created for us when we run
the app. Note that no migration task is performed during this development phase. AKA, we did NOT need to run `dotnet ef migrations add` and `dotnet ef database
update` and then examine migration code!

