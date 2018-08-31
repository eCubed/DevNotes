# Starting Over

I believe that I may have messed up some configurations in the system, which may be the cause of why the Internet cannot see my name server at my IP address.
There isn't very much data in there. Also, I mistakenly set the firewall to remove the allow port 22 rule, which made me unable to access the server via PuTTY.

My host lets its users reinstall the VPS from the account control panel, which I did. I chose the Ubuntu 18.04 distribution again. However, this time, it gave
me a choice of minimal vs. standard installation. The standard installation comes with MySQL, PHP, and Apache - pretty much the LAMP stack without the
Angular. That's not the development stack I wanted, so I chose minimal.

I was able to create users as before. However, I will need to manually install BIND now, which was included in the standard installation. I'll need to add Nginx
as before for my web server.

## Firewall

I previously set up the firewall straight from `ufw` in Ubuntu. I will no longer tweak settings. It turns out that there is a firewall beyond my VPS that I can
still tweak from the account control panel. So, I edited those and allowed TCP AND UDP for ports 22 (SSH), 80 (http), 443 (https), and 3306 (MySQL).