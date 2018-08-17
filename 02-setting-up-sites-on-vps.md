# Setting Up Websites on VPS

Suppose that we signed up for a VPS because our websites actually need to communicate with other programs running on a machine. Let's say that
we've installed all necessary programs on the VPS. We've already published our websites locally and all we need to do is copy files to the VPS.
We have many questions.

## VPS and Unique IP Address
Does our VPS get a unique IP address?

## Resolution of Domain Name to a Website in our VPS's IIS
Suppose that our VPS gets its unique IP address. Suppose that we placed our deployed websites on our VPS's hard drive, and pointed IIS to them.
We would have assigned unique ports to our websites. If someone were to type the URL of one of our websites, how does our VPS know which website
to serve for that request? We'll just outline our wild guess process:

1. First up, we will need to register each domain (website) to a DNS registrar. There is a single universal repository that stores IP address and
domain name associations. Ultimately, one of our domain name gets associated with the IP address of our VPS, so when someone types our address 
on their browser and hits enter, our VPS will be called (for now, let's assume by magic because we don't know how networks work). But even then, 
if our VPS is targeted, how will our VPS know which website out of many that we manage, to serve back to the requestor? (This implies that there
can be one IP address that hosts multiple domains.)
2. When our VPS gets hit with a request, we guess that the full URL, any query parameters included, will be given to our VPS, and it's up to IIS to
figure out which website to serve up just by looking at the domain portion of the URL. It turns out, then, that in IIS, we would have also added
a binding, besides localhost with a port assigned to it, whose host name is that of our domain name. Ah, so this must be how the VPS knows which
website to serve. Should someone from the outside world figure out our VPS's IP address, they will also need to specify the port of the website they
want to view, or else, they will be shown the default IIS website. So no, the domain registration does not store port numbers. Once the IP address
is figured out, the machine or VPS assigned to that IP address will be responsible for resolving ports.

But we've heard of such things as A-records and other DNS terminology that we need to set up. There's also the name servers, which we don't
exactly know what they are and what for, if, supposedly, all that the WWW needs to know is one's domain name and the associated IP address? Then
there's also dynamic vs. static IP.

A curious question arises, as an aside: Somewhere, the domain name of the request gets resolved to a single IP address, right, so then how does
that IP address find our VPS?

### Update

We are quite off in our guess above, but have some of the correct ideas.

1. We first sign up our domain name (ourcompany.com) to a **domain registrar**. This could either be a service that manages domain names only, or
it could be our hosting provider (who, really, stands between the customer and an actual domain registrar). When someone navigates to our website,
the domain registrar will be queried first (but then again, what entity knows which domain registrar to call up in the first place upon seeing
the domain name?).
2. Then there are things called **name servers**, which the domain registrar asks for. OK, so far, the domain registrar is only responsible for
telling the world that our domain name exists. It's not actually the mechanism that resolves a domain name to an IP address. So, back to name
servers. These are actual machines that our host (that we signed up for to store our deployed website files) manages. These name servers are
supposed to contain **DNS records** about the websites of their customers. We speculate that DNS records are automatically created for us by
our host when we go through some UI and add domains and subdomains. Consequently, we speculate that these name servers resolve the domain name
with an IP address, which could vary over time even for the same domain name - also known as dynamic IP. If the actual IP address of a website
can change over time, then there must be a mechanism within a hosting provider that issues and removes IP addresses, and associates them to
the very machine that hosts our website files. But there are multiple name servers per host, though - why?
3. A name server contains DNS records, so what exactly are DNS records and who dials them in when we have a VPS set up? There are also such things
as **DNS Zones** and **DNS Record Types**. What are they and why are they necessary? We'll need to consult
[this article](https://pressable.com/blog/2014/12/11/understanding-dns-record-types/) for more detailed info on these concepts. So a DNS record
(the A Record specifically) really is an entry for each subdomain we might have, which specifies a single IP address (we guess that we cannot 
change to whatever we want - we must choose one that our host provides, or we can't choose at all), and zone is a collection of 
DNS records of a single domain.
4. So a DNS record in a name server resolves the IP address (however it was assigned), and so within our host, some mechanism knows how to find 
the very machine by that IP address. When the machine is targeted, that machine still gets the full URL so that its IIS knows which website to
pull up (provided that added the correct universal (not IP address) binding).
5. The appropriate website in our VPS processes the request, and ultimately, the response is returned to the requestor.

## Https

We've heard that in order for us to serve up https, we will need to apply for another IP address, which would, we figure, still be associated
with our VPS. This IP address is supposed to be static, and should be the one associated to the https binding of our website.

OK. Suppose we were granted that static IP for our https. We would then need to add an A Record entry of our domain that points to that static
IP. We also need to specify this same IP in IIS for our https binding (another wild guess). This way, the name server would know to resolve
the https static ip.

Another curious question: If we'll have 2 A Records, one for http, the other for https, the name server (we figure) doesn't know to make a
distinction between http and https, since there's no distinction for those in the any DNS record. So, by only looking at the domain name, how does
a name server know which IP to resolve to? We presume that it gets passed to both, and when the machine gets either request, it will then know
because its IIS looks for the protocol!
