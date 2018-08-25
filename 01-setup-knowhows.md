# Linux Hosting Know Hows

## Getting a Domain Name
We've already decided that registering a domain name straight to a domain registrar is the best thing to do. This way, the host and domain are
decoupled. We could switch web hosting companies and we would still have our domain name. Once we obtained the name servers from our host, we
need to report them back to the registrar.

## Signing Up for the Linux VPS
Getting a Linux VPS is quite similar to getting its Windows counterpart. In general, we get to choose OS, more specifically, a distribution of Linux.
We also get to choose plans that offer varying amounts of resource allowances as well as payment cycles, most of which have a cheaper monthly cost
the more long-term we choose a deal (though more to pay up-front). There is unmanaged vs. managed. For now, and managed means that we'll most likely 
get a browser-based GUI so we can perform administrative tasks easier (than doing the same in a command prompt), the host will do things for us like
automatically backup, update the OS, offer some protection, and we'll get more technical support. The unmanaged means we'll only get the command
prompt and we'll be on our own. We are just starting, putting up the website isn't urgent, and ultimately, we'll end up in an unmanaged environment
anyway (for work), and managed plans cost more, so we are shooting for unmanaged at this point.

## Remotely Accessing our Linux VPS
No, we will not be logging into a Linux VPS via Remote Desktop because there is no UI. From our preferred Windows dev machine, we will need to download
an SSH client (software) that is specifically designed to communicate securely with a Linux server. We have the host name of our VPS because our host
asked us for it during signup, and we provided it. We should also have the password, too, either because they asked us for it at signup, or they
gave a starting one for us via email confirmation.

## Obtaining and Applying SSL
Not all providers offer SSL, so we need to be careful of this. Even if they offer SSL, we need to be careful because many of them offer only one
certificate, good for only one domain, for only one year. At best, we want a host that offers unlimited free SSLs for all domains and subdomains
as long as we're active with them. Minimally, if they only offer 1 certificate for free, it better be wildcard because we'll start with one domain
with many subdomains. Depending on the host, we will need to know how to obtain, find it in the file system, apply the SSL they offered to all 
of our domains and subdomains. 

Perhaps we may also need to learn how to find SSL providers if our chosen host doesn't offer SSL. This is the less ideal situation. There are some
cheap SSL deals but in general, the cheaper it is, the less trustworthy it is, and the more reputable the SSL provider, the more expensive the
certificate.

## Installing Programs
Even if we have a managed Linux plan, we will still need to get into the prompt to install software. We are likely used to going to our browser,
searching for a download for an installer, and running it. There isn't a nice browser on the command prompt, and we will need to learn how to do
this with typed commands. Some managed Linux plans will allow us to set up various programs from a browser-based control panel. But they're stuff
like WordPress, maybe a PHP bulletin board, or various CMSes, which we're not interested in.

In particular, we will definitely need to install the latest and few previous Asp.NET Core runtimes. We may also need to install and configure
for ourselves MySQL

## Setting up DNS
We have decided also that our Linux VPS should also be our nameserver. It is said that for a business, it is unprofessional to have the host's
public nameservers show up when people look up our website's WhoIs information, therefore, we need to have our own nameservers to make it look like
our business's domain name instead of the host's domain. 

(But a DNS question arises. If a name server's job is supposed to be to return an IP address for the request's domain, then what external entity
(there's got to be) resolves the name server's "url" to an IP address back to the registrar? How would registrar find our VPS if there wasn't the
public name server that would have told the registrar the IP address of our name server?

We have a theory that once our VPS is active, and its DNS server running, our VPS will ultimately be queried for its own IP address, even if the
request comes from the registrar. It is said that DNS servers "know about each other", and would forward to another DNS server if it didn't have
IP information of the request, and so on until a DNS server down the line has the IP of the domain requested. But if we set up our DNS server
ourselves, we figure that no other DNS server out there knows that our VPS even exists, let alone have forwarding info to our DNS server! If ever some
other DNS server somehow knows the whereabouts of our DNS server, then we, too, can have forwarding info to other DNS servers. This way, we participate
in the universal network for search of IP addresses. But for now, let's assume that the moment we commit those DNS settings and our VPS is up
and running, our DNS information will be found.)

(Another curious question: If a name server returns the IP address of the requested website back to the requestor, supposedly, we can relace the
domain name portion of the URL with that IP address. But that IP address is the address of a machine, which may host many websites. Then it wouldn't
know which website to serve if we only typed the IP address!)

Our host might provide DNS setting and manipulation via a browser-based control panel.

## Setting up Websites
There is quite a lot to do to set up a published website, which is everything we've ever done in IIS or a host-provided panel. We figure that Linux
has an IIS equivalent, albeit, a console application running in the background that we write scripts for or execute commands for. We may need to
install a web server ourselves. Apache and Nginx seem like popular choices. Alternatively, though, our host may provide a browser-based control
panel to assist us in setting up web sites.

## Publishing to Website
We prefer to develop in Windows because we get GUI with code completion, because it's more efficient and less taxing on our brains. We also prefer
to publish on our Windows machine, and then copy those files to the correct folder in our VPS. There must be a secured FTP server that we need to
install and set up that sits running on our VPS to be logged into and worked through. On our Windows machine, we need to install a secure FTP client
to be able to access folders on our VPS.

Alternatively, which is the less-favorable approach, we would clone our repos to our VPS via Git, which we should be able to install, check out the
appropriate branch for releases, and then run `dotnet publish`. We would need to set up the website in either Nginx or Apache, then have it point
to published folders.

## MySQL and Asp.NET Core
This is more on the development side, but it is extremely crucial. Ideally, we would like to use the same `DbContext` and `LinqToSQL` we have
for MSSQL, but actually communicate with MySQL in the background. There may be a library that makes this seamless or at least easy for us. It's
2018, and no, we will not resort to writing ADO.NET-like code which is tedious, error-prone, and a ton of code.

Regarding what we may write in Code-First development, some attributes in our POCOs and Fluent API code in our data context may be specific to MSSQL
and might not translate well between C# and MySQL. We're concerned about data type mapping between the two, to start.

We will still develop on a Windows machine, including migrations to the database. We currently use `dotnet ef migrations add` and `dotnet ef
database update` for working with MSSQL. We need to find tooling that will let us communicate with a MySQL so we can migrate.

## Summary

- We will (unfortunately, at least from the start) do most, if not all, administration work via command line, including installing software.
- We will need to learn hundreds of commands and many of their flags, parameters, and allowed parameter values.
- The main difference Asp.NET Core will be code that sets up and uses MySQL instead of MSSQL.