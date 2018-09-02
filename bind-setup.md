# BIND Setup

BIND is the DNS software that allows a computer to be a name server. This means that the machine that BIND is installed and running in will store
DNS records for the domains it hosts, that other machines on the Internet will actually query. We, as the machine's administrators manually input
domain and IP address relationships, and the software actually notifies the Internet of our DNS records every time we update our DNS records.

## Installation

`sudo apt update`
`sudo apt install bind9 bind9utils bind9-doc`

Once installed, the BIND service (as bind9) will automatically run.

## Registering our Domain to the System

The first thing we'll do is to edit the `/etc/bind/named.conf.local` file. This file was created with the installation of BIND, and we
"register" our domain by adding a couple of entries in it. The file may have other entries for other domains, so we should not touch those.
We will be appending to this file.

Let "mysite.com" be the domain name, and 143.207.72.25 be the IP address of our machine or VPS. We would just substitute our own values
for those.

So, let's add the following declaration:

```
zone "mysite.com" {
     type master;
     file "/etc/bind/zones/db.mysite.com";
};
```

The above declaration is the beginning of letting our system know that it will host mysite.com and its subdomains. For now, let's assume
`type master` is required and we'll get to the meaning later. The `file` setting is a reference to another file we haven't created yet. 
That file will be where we will write our DNS records (A, CNAME, etc), and it is referred to as a zone record.

It is not required, however, it is customary to put the zone record in the folder `etc/bind/`. It is also not required to name the zone
record with the db prefix followed by the domain name as declared above. However, we should follow conventions because they help us organize
things in the least stressful and mind-blanking manner. The db prefix denotes that our DNS server is practically a database of domain names and
IP address, though the "db" data is saved in files.

Now, we need to also declare a reverse-lookup zone. We'll do this in case we want our server to also resolve an IP address to a domain name. It
could be a requirement, though, at this point, we don't know. Still at the same named.conf.local file, we add:

```
zone "72.207.143.in-addr.arpa" {
     type master;
     file "/etc/bind/zones/db.143";
};
```

The name of the reverse-lookup zone may look strange. The format of the reverse-lookup zone name is "the first three numbers of the IP address, 
separated by a single dot, IN REVERSE ORDER, then append `.in-appr.arpa`. For now, let us assume that this is a requirement. That '25' for the last
digit of the IP which we didn't include in the zone name will come into play later. The file reference is a reference to another file that we'll create.
Note that we also prefix the reverse-lookup file name with "db" for convention. We then follow "db" with a dot, then the first number of the IP address.

Note that the zone name is REQUIRED to be in the specific format, but the name of the reverse-lookup configuration file can be anything we want.

## Creating the Domain-to-IP (Forward) Zone File

If you previously had managed plans in any platform, you most likely entered the DNS info into textboxes in a UI provided. We will still achieve
the same result, but now, we are typing DNS settings by hand.

First, let's go ahead and create the Domain-to-IP Zone file. I like vi, so I'll go ahead and create it:

`sudo nano /etc/bind/zones/db.mysite.com`

Then I'll enter the following:

```
;
; BIND data file for linuxconfig.org
;
$TTL    3h
@       IN      SOA     mysite.com. root.mysite.com. (
                          2018090101  ; Serial
                          3h          ; Refresh after 3 hours
                          1h          ; Retry after 1 hour
                          1w          ; Expire after 1 week
                          1h )        ; Negative caching TTL of 1 day
;
@       IN      NS      ns1.mysite.com.
@       IN      NS      ns2.mysite.com.
@       IN      A       143.207.72.25
ns1     IN      A       143.207.72.25
ns2     IN      A       143.207.72.25
www     IN      CNAME   mysite.com.
```

Note that the space formatting is optional but highly recommended for readability and maintenance.

There are a few things to note:

- Whenever we state our domain name, the TLD will always be followed by a period, hence `mysite.com.`
- For the SOA (Start of authority) section, we add the domain name, then "root", dot, then our domain. For now, let's leave the time parameters for later.
- The first parameter, or the serial, is really any number we want. Every time we make changes to this file, we will need to change the serial's value,
or else, BIND will not detect the change and thus, would not notify the Internet. We typically like to format it with YYYYmmddxx where xx denotes which
revision of this file was made on that day.
- The @ is just a substitute for our domain name.
- ; at the beginning of a line is for commenting the entire line.
- The DNS entries on the bottom half of the document is the very "db" that would be queried when something asks our server for the IP address of
nameservers, websites, ftp, etc.
- We also did NOT include MX and subdomains. We will reserve those for later. At this point, we care about setting up the main website.


## Creating the IP-to-Domain (Reverse) Zone File

`sudo nano /etc/bind/zones/db.143`

```
;
; BIND reverse data file for 72.207.143.in-addr.arpa
;
$TTL    604800
@      IN      SOA     mysite.com. root.mysite.com. (
                         2018090101         ; Serial
                         3h                 ; Refresh after 3 hours
                         1h                 ; Retry after 1 hour
                         1w                 ; Expire after 1 week
                         1h )               ; Negative caching TTL of 1 day
;
@      IN      NS      ns1.mysite.com.
@      IN      NS      ns2.mysite.com.
25     IN      PTR     mysite.com.
```

- The @ will resolve to be 72.207.143.in-addr.arpa, the name of the reverse lookup zone that references this file.
- We include the name servers.
- We create a pointer (PTR) record from the IP address to the domain name. Note that the 25 is specified inside this file instead of in the zone
name.

## Forwarders Setup

In case our VPS gets queried for an IP address and we don't have record of the requested domain, then we'll forward to a stable DNS Server,
typically 8.8.8.8 or 8.8.4.4, which are both Google's.

## Verification of Proper Setup

So we have created two zone files and updated named.conf.local. We also made sure that the file locations and file names are correct and the
references to them are correct. We run the following:

`named-checkconf`

We would like to receive no output for this, because any message means error. We'd then have to fix it.

## Restarting the BIND Nameserver

After we've verified that our DNS setup is correct, we would need to restart the BIND nameserver:

`etc/init.d/bind9 restart`.

## Firewalls

We will need to allow port 53, both TCP and UDP, since that's where our name server will communicate through. We may use `ufw` from inside our VPS 
if our host leaves firewalling upto us. Some hosts have a firewall around our VPS that we can configure from our web-based dashboard 

## Report Nameserver to Registrar

When the setup is correct, and we've tested it locally, we can now go back to our registrar and add our nameservers ns1.mysite.com and ns2.mysite.com.
Since this isn't the web host's name server, and it's our own, we will also need to tell our registrar the IP address of our VPS (that we've set up
as the name server for both of them.)

We will then need to give the Internet some time, upto 48 hours, for our domain to be resolved, that is, may be browsed to at the domain name.


