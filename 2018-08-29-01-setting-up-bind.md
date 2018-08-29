# Setting Up BIND

Now that we have the web service Nginx working (we got to display the default website by pointing the browser to our IP address), we will now
want to set up the DNS Server. We will want to:

- make our VPS the name server
- make the DNS be queryable only for the domains hosted in our VPS.

Well, it looks like BIND (as the bind9 service) is already installed in our system and actively running. We will now need to configure it.

## Registering our Domain to the System

The first thing we'll do is to edit the `/etc/bind/named.conf.local` file. This file was created with the installation of BIND, and we
"register" our domain by adding a couple of entries in it. The file may have other entries for other domains, so we should not touch those.
We will be appending to this file.

So, let's add the following declaration:

```
zone "mysite.com" {
     type master;
     file "/etc/bind/zones/db.mysite.com";
};
```

Where of course, we would swap `mysite.com`, with our actual domain name. The above declaration is the beginning of letting our system know
that it will host mysite.com and its subdomains. For now, let's assume `type master` is required and we'll get to the meaning later. The
`file` setting is a reference to another file we haven't created yet. That file will be where we will write our DNS records (A, CNAME, etc),
and it is referred to as a zone record.

It is not required, however, it is customary to put the zone record in the folder `etc/bind/zones`. It is also not required to name the zone
record with the db prefix followed by the domain name as declared above. However, we should follow conventions because they help us organize
things in the least stressful and mind-blanking manner. The db prefix denotes that our DNS server is practically a database of domain names and
IP address, though the "db" data is saved in files.

Now, we need to also declare a reverse-lookup zone. We'll do this in case we want our server to also resolve an IP address to a domain name. It
could be a requirement, though, at this point, we don't know. Still at the same named.conf.local file, we add:

```
zone "72.207.143.in-appr.arpa" {
     type master;
     file "/etc/bind/zones/db.72.207.143";
};
```

The name of the reverse-lookup zone may look strange, but suppose that our VPS's IP is 143.207.72.25. The format of the reverse-lookup zone
reference is "the first three numbers of the IP address, separated by a single dot, IN REVERSE ORDER, then append `.in-appr.arpa`. For now,
let us assume that this is a requirement. That '25' for the last digit of the IP will come into play later. The file reference is a reference to
another file that we'll create. Note that we also prefix the reverse-lookup file with "db" for convention. We then follow "db" with a dot, then
the first 3 numbers of the IP address reversed, separated by a dot.

## Creating the Domain-to-IP (Forward) Zone File

If you previously had managed plans in any platform, you most likely entered the DNS info into textboxes in a UI provided. We will still achieve
the same result, but now, we are typing DNS settings by hand.

First, let's go ahead and create the Domain-to-IP Zone file. I like vi, so I'll go ahead and create it:

`vi /etc/bind/zones/db.mysite.com`

Then I'll enter the following:

```
;
; BIND data file for linuxconfig.org
;
$TTL    3h
@       IN      SOA     ns1.mysite.com. admin.mysite.com. (
                          1        ; Serial
                          3h       ; Refresh after 3 hours
                          1h       ; Retry after 1 hour
                          1w       ; Expire after 1 week
                          1h )     ; Negative caching TTL of 1 day
;
@       IN      NS      ns1.mysite.com.
@       IN      NS      ns2.mysite.com.


mysite.com.     IN      MX      10      mail.mysite.com.
mysite.com.     IN      A       143.207.72.25
ns1             IN      A       143.207.72.25
ns2             IN      A       143.207.72.25
www             IN      CNAME   mysite.com.
mail            IN      A       143.207.72.25
ftp             IN      CNAME   mysite.com.
```

Note that the space formatting is optional but highly recommended for readability.

There are a few things to note:

- Whenever we state our domain name, the TLD will always be followed by a period, hence `mysite.com.`
- For the SOA (Start of authority) section, we add the primary name server and then "admin", dot, then our domain. For now, let's leave the time 
parameters for later.
- ; at the beginning of a line is for commenting the entire line.
- (Not certain) The lines that we declare `@ IN NS ns1.mysite.com.` mean that we're setting up our VPS as a nameserver, and BIND will take care
of letting the world expect that our system is also the nameserver.
- The DNS entries on the bottom half of the document is the very "db" that would be queried when something asks our server for the IP address of
nameservers, websites, ftp, etc.

## Creating the IP-to-Domain (Reverse) Zone File

`vi /etc/bind/zones/db.72.207.143`

Then I'll enter the following:

```
;
; BIND reverse data file for 72.207.143.in-addr.arpa
;
$TTL    604800
72.207.143.in-addr.arpa.      IN      SOA     ns1.mysite.com. admin.mysite.com. (
                          1         ; Serial
                          3h       ; Refresh after 3 hours
                          1h       ; Retry after 1 hour
                          1w       ; Expire after 1 week
                          1h )     ; Negative caching TTL of 1 day
;
72.207.143.in-addr.arpa.       IN      NS      ns1.mysite.com.
72.207.143.in-addr.arpa.       IN      NS      ns2.mysite.com.

25.72.207.143.in-addr.arpa.    IN      PTR     mysite.com.
```

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

## Testing the BIND Configuration

`dig @143.207.72.25 www.mysite.com`

We better not get errors.

## Report Nameserver to Registrar

When the setup is correct, and we've tested it locally, we can now go back to our registrar and add our nameservers ns1.mysite.com and ns2.mysite.com.
