# Starting Over

I believe that I may have messed up some configurations in the system, which may be the cause of why the Internet cannot see my name server at my IP address.
There isn't very much data in there. Also, I mistakenly set the firewall to remove the allow port 22 rule, which made me unable to access the server via PuTTY.

My host lets its users reinstall the VPS from the account control panel, which I did. I chose the Ubuntu 18.04 distribution again. However, this time, it gave
me a choice of minimal vs. standard installation. The standard installation comes with MySQL, PHP, and Apache - pretty much the LAMP stack without the
Angular. That's not the development stack I wanted, so I chose minimal.

I was able to create users as before. However, I will need to manually install BIND now, which was included in the standard installation. I'll need to add Nginx
as before for my web server.

## Firewall

I previously set up the firewall straight from `ufw` in Ubuntu. I will no longer tweak settings from inside the server. It turns out that there is a firewall
beyond my VPS that I can still tweak from the account control panel. So, I edited those and allowed TCP AND UDP for ports 22 (SSH), 80 (http), 443 (https), 
and 3306 (MySQL).

## Installing BIND

`sudo apt update`
`sudo apt install bind9 bind9utils bind9-doc`

And yes, the BIND service is running automatically.

## Configuring BIND

I initially said that I'd configure Nginx first before BIND, but I'll install and configure BIND first this time around.

I have done the configuration

## Configuring Nginx

All I did was

`sudo apt update`
`sudo apt install nginx`

When I go to the browser and type http, then my IP address, and I get the default nginx page.

## Name Server Fiasco [Solved]

The first problem was that I didn't exactly know how to point my domain to my VPS. All I knew was I merely changed the default name servers to ns1.mysite.com and
ns2.mysite.com. This solution is correct only when I have a shared hosting plan and they readily gave me those name servers, and those name servers have already
had their IPs resolved and propagated. My situation was a bit different - I wanted to host my own name servers, and of course, the Internet wouldn't know where
they were if I merely pointed my domain to them. It turns out that I registered ns1.mysite.com and ns2.mysite.com *right at the registrar*, and I needed to also
provide the IP address of my VPS. Now, it was making sense.

I had previously set up BIND, and `named-checkconf` returned no errors, which meant that all is well and my DNS was ready to serve. At one point, my name servers
and IP addresses have propagated, however, mysite.com wouldn't resolve to the IP address! It turns out that the surrounding firewall that I could set up from the
account control panel blocked everything except those listed. It occurred to me that maybe name servers would listen on a particular port, and that turned out to
be port 53. I added an ALLOW rule for port 53, both TCP and UDP. Soon after, the Internet was able to resolve mysite.com to my VPS's IP.

