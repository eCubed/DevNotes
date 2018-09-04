# MySQL Pursuit

I want to host my own MySQL server on my VPS, just as I've hosted MSSQL on a Windows VPS in the past. At this point, with MySQL, I'm only concerned with being able to
set it up so I can access it online via a desktop application, much like SQL Server Management Studio is to MSSQL. At this point, I'm not too concerned about connecting
to it from my Asp.NET Core apps.

## K in KWL for MySQL

- MySQL communicates through port 3306.
- I'll need to install it on my VPS myself.
- The IDE for MySQL is MySQL Workbench, and I will install it on Windows dev machines and connect to my VPS's MySQL with it.

## W in KWL for MySQL

1. I often see what looks to me like database subdomains, for example, mssql.example.com or mysql.anothersite.com. How did they set this up? I want to connect to
MySQL from both MySQL Workbench and via web app connection string with that friendly url, NOT the IP address. Do I need to set this up in DNS? If so, what kind of
records would they be?
2. Are MySQL users automatically the Linux users when it's set up?

## L in KWL for MySQL

- **Are MySQL users automatically the Linux users when it's set up?** From reading some configuration information for MySQL, its users are separate from the hosting OS's system. We are able to add users to MySQL that are not users
of the system. We addressed Question #2.
- **How to allow remote connection to MySQL Server with a FQDN?** This is a multi-step process, but yes, we will need to create an A record for mysql.mysite.com,
propagate it, and the rest of the steps involves configuring MySQL Server once installed. Once completed, we can now connect to our MySQL Server from MySQL
Workbench and use the FQDN in a connection string.

## Installing Configuring the MySQL Server

I'm on Ubuntu, so I ran

```
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

in that order.

When propmted the following (paraphrase), I answered:

- **Validate Password Plugin?** No because I didn't want strict password rules for this system.
- **Enter a password for root.** I did, and I also retyped it when they asked.
- **Remove anonymous users and test database.** Yes. I want a clean install, and I didn't want anyone to monkey around with my database with that anonymous user.

## Preparing MySQL Server for Remote Connection

Since I wanted to remotely access this database with an FQDN, I added an A record for it, and had it propagate.

### Setting Up Initial Users

The very first time we log into MySQL from our server, we can simply type

`sudo mysql`

But after we get in and set passwords and add users, we'll need to type

`sudo mysql -u {username} -p`

Which will then ask for that username's password to get in.

When login is successful, we're given the `mysql>` prompt.

For security, we will need the root user to require native password validation (because for some reason, default is auth_socket, which is less secure):

`mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '{password}';`
`mysql> FLUSH PRIVILEGES;`

where `{password}` is a secure password of choice. Yes, mysql commands and queries must end in semi-colons. Now, the root user will need to supply a password
every time it needs to log in.

We would not want to log in with the root user, though, so we'll need to create another user:

`mysql> CREATE USER '{username}'@'%' IDENTIFIED BY '{password}';`
`mysql> GRANT ALL PRIVILEGES ON *.* TO '{username}'@'%' WITH GRANT OPTION;`

Where `{username}` is the desired username and `{password}` is a secure password. The `'%'` means allow this user to connect remotely from any IP address.
According to MySQL, it is insecure to allow access to the server from any IP address. We're relaxing this one a bit, since MSSQL didn't have this requirement.
Besides, the IP address of where we're accessing MySQL might change, then we can't get in. Also, this gives us the flexibility (albeit allegedly dangerous) to
log into our MySQL server from anywhere we need to be.

### Installing MySQL Workbench

Google for the download link. When we attempt to download, it will ask us to register if not already, or log into our online MySQL account (not to be confused with
the MySQL server we just set up). Once installed, we can open it, and try to connect. If all goes well, we'll be allowed in.

### MySQL First Exploration

I'm coming into MySQL Workbench with knowledge of MSSQL. Much should be similar in database management and query syntax. From the UI, we can create databases,
which are called schemas in MySQL. Then we can do the rest of our usual SQL tasks we used to do in MSSQL Management Studio.

### Connection String

`server={address};database={databaseName};uid={username};pwd={password}`

`{address}` is the IP address or FQDN. No, we do not need to specify the port number.
`{databaseName}` is the name of the schema or database we're trying to access.
`{username}` is the username.
`{password}` is that username's password.

This will be important for when we start connecting to MySQL from an Asp.NET Core application.

## Next Steps

The next logical step is to attempt to connect to MySQL from an Asp.NET Core app.