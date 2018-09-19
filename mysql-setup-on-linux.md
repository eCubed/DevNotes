# Setting Up MySQL on Linux (Ubuntu)

We are going to set install and set up MySQL on an Ubuntu server.

## Installing Configuring the MySQL Server

Run the following commands in the order specified below:

```
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

When prompted the following, you may choose to answer the same recommendations below. There are more questions, but just take the default values.

- **Validate Password Plugin?** If we want to have strict password rules, like require upper case, special characters, and specific lengths, answer yes. If we want
simple passwords, answer no.
- **Enter a password for root.** Do it.
- **Remove anonymous users and test database.** Yes. If we answered no, anyone who knows the widely public "factory setting" users will be able to get into our
database!

## Preparing MySQL Server for Remote Connection

We will need to create an A-record for a subdomain. The typical format is `mysql.{domainName}.{domainExtension}`. Setting this up may depend on the entire
website system's setup. Once it propagates the Internet, we may be able to log into our installed MySQL server using 'mysql.{domainName}.{domainExtension}` as the
FQDN later from a remote management tool.

### Setting Up Initial Users

Once installed, we can go ahead and logged in. Since this will be our first time logging in, we can simply login anonymously with the command

`sudo mysql`

But after we get in and set passwords and add users, we'll need to type the following later to log in:

`sudo mysql -u {username} -p`

We will get a prompt asking for that user's password.

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
log into our MySQL server from anywhere we need to log in from.

### Installing MySQL Workbench

MySQL Workbench is the Windows GUI software for remotely administering MySQL. We can [download the installer from here](https://dev.mysql.com/downloads/workbench/).
The download button is almost at the bottom of the document, under the tab "Generally Available (GA) Releases". We will be required to register to Oracle if we 
haven't yet. 

Once installed, we can open it, and try to connect. The GUI is pretty clear on how we can accomplish being able to connect. Logging in does have an option, though of
saving passwords in a secured (presumably stored on the local machine) vault. If we choose this option, all we'll need to do is click on a rectangular area containing
our login name and MySQL server name, and we'll be good to go. This may not be recommended especially if we end up installing MySQL Workbench on someone else's
machine just so we could connect to our MySQL server at that moment - it's easy to forget to remove the credential or uninstall MySQL Workbench when we're done.

### Connection String

`server={address};database={databaseName};uid={username};pwd={password}`

`{address}` is the IP address or FQDN. No, we do not need to specify the port number.
`{databaseName}` is the name of the schema or database we're trying to access.
`{username}` is the username.
`{password}` is that username's password.

This will be important for when we start connecting to MySQL from an Asp.NET Core application.

## Next Steps