# What Happens When a Website is Requested

Many articles and tutorials about putting up a website instruct from the perspective of the developer. We will approach it from the perspective
of the request, and then provide what a developer would need to do during each step.

## 1 - Someone Types the Url on their Browser, Domain Registrars get Called
When we type a website's url on our browsers to go to it, that's a request to pull up the html so our browser can display it. What happens
out there, though?

(Guess) Our internet service provider hears our request for that website, and then queries various **domain registrars**. A domain registrar
can answer the question "does such a website exist?". The author of the website would have had to sign up to a domain registrar to *reserve* the
domain name, which means that nobody else can claim that domain name as their own. Companies or web authors pay money, usually per year to
state that such a domain name is theirs and nobody else's. If it expires, then the website becomes inaccessible from the world.

## 2 - Domain Registrar Queries Name Servers

The domain registrar already answered the question "does the website exist?". Now that the website exists, the next question is "is the website
actually set up, and if so, where is the html?" That question is the responsibility of **name servers**.

When a user registers their domain name to a domain registrar, the domain registrar asks for a few name servers. These are actual computers typically
outside of the registrar's control, so the web author needs to provide name servers, which to a domain registrar, look like urls of another domain.
They typically look like `ns1.somewebhost.com`, `ns2.somewebhost.com`, etc. So, how does a web author get a hold of name servers?

The web author would need to sign up to a **web host**, which is a company apart from the domain registrar. The web host provides the web author
with at least some disk space to store web files (html, js, css, and if it applies, server-side code) and monthly traffic. The web host also provides
the name servers to the web author, so then [s]he needs to report those name servers back to their domain registrar. The web host also asks the
web author what the domain name is so that it can create records and file folder names based on the domain name.

## 3 - Name Servers Resolves Domain to IP Address
In the simplest terms and cases, the name server stores the information of which computer within the web host's network contains the website files
of the url requested. This means that a name server keeps record of domain names and IP addresses. This way, when the name server is hit with a
request to view a particular website (by the url), it knows an IP address associated with it, and if the url is found with an IP address, the request
is passed on to the computer that "lives" in that IP address.

DNS records may be a complicated topic for this overview type of an article, since it contains intricacies. We will examine DNS records in greater
detail in a later article. For now, it's good to know that DNS records answers questions like "what to do if the user didn't type www in front of
the domain name?" or "how does an author have subdomains with a web host?".

## 4 - The Actual Web Server at IP Address Gets Called
So, at this point, the name server found the computer that stores the web author's code. This computer is the actual **web server**, which contains
the actual website and/or application. It is the computer that actually *serves* the content requested by the user.

There are many types of web servers, and their types are called **web hosting options**. There is a lot of material to cover when considering web
hosting options, and we will discuss those on other articles. However, regardless of web hosting options, the web server still stores the author's
code and runs it. When the web server gets a request passed on by the name server, it includes the full url including any query parameters as well as
the body. The web server would know the exact drive letter and subfolder of where that author's code is stored, and when it finds that folder, it
will then pass on the very http request to the application that lives there.

So, what did the web author need to do to store their website or web application to the web server? The host typically provides a control panel to
the web author. This control panel may be downloadable software or a web application they may log into to manage their websites with the web host.
Depending on the web host, hosting options, or level of plan purchased, the control panel may allow varying degrees of control as to what a user
may or may not be able to do from the control panel alone. The web author will always obtain the following, though:

- a means to upload their web files to the web server. Some may allow direct upload straight from inside the control panel. Some may have the web 
author set up FTP so that they may upload files via FTP using a separate piece of software.
- a way to add domains and subdomains, if allowed by plan and/or web hosting option. Depending on the web host, the user may be given varying
degrees of control on adding a domain or subdomain. Sometimes, adding domains or subdomains will require a support ticket, which is really an
explicit message to technical suppport requesting for a new domain or subdomain. Tech support would then allocate and create disk space for the
requested domain and subdomain, set up DNS information for that new domain and subdomain, and set up the created folders for them as running
web applications. Then will tech support correspond back completion of the request, and reflect that completion back at the control panel. Some
web hosts or a particular plan in a web host will allow total control of DNS information setup, website application setup with the operating system
(if VPS or dedicated server hosting options were chosen), and addition and removal of domains and subdomains.
- a way to enable/disable domains and subdomains.

## VPS or Dedicated Server Hosting Option
We will look deeper into step #4 in the case where we chose the VPS or Dedicated Server web hosting option. This is where we're given remote login
credentials since we will be explicitly logging to an experience that looks like we are working on a desktop machine, complete with operating system
and some software. Between a VPS and a dedicated server, the authoring/admin experience is the same, however, in a dedicated server, we are literally
logging into a single, physical machine whose computing power is ours and our only. A VPS *only feels* like we're logging into a single machine
that's ours only - it's actually logging into a single machine whose resources are partitioned into virtual computers. Although it looks like our
own computer, other VPS customers are actually logging into the same physical machine. This is slower than a dedicated server because that single
machine has to manage many different virtual operating systems at one time.

When we have a VPS or dedicated server option, we have full control of where exactly on the hard disk to save our many websites, serving them up 
from the machine (virtual or actual), and setting DNS records. This type of control may be quite advanced for someone accustomed to a shared hosting 
arrangement, since with shared hosting, websites and DNS information are automatically set up when adding domains or subdomains. This option
may be heavily involved, so the web developer needs to be proficient in DNS and IIS (Windows), or at least willing to learn (which takes heavy
research and potentially lots of time not being able to show the website due to learning and figuring out proper setup).

From our remote host, we may be given software to directly create and maintain DNS records. If not, the hosting company may provide a means to 
set DNS records from a browser-based control panel application. This is where we declare things such as subdomains and IP addresses of our remote
host.

For the remainder of this section, we will discuss setting up websites and domains with the Windows platform. This means we'll be working with IIS.

So, the at this point, the name server found our remote host at a particular IP address, and handed off the original request to our remote host
(which is always listening for web requests). We know that we would have multiple websites set up on that remote host, so how would our remote host
know which website to serve? We already know that we will include localhost bindings to each website so that we can run it from the web browser 
installed on the remote host. (We may have experienced this when we were trying to simulate production on a local machine.) For each website set up
in IIS on our remote host, we also need to add the DNS-recognized bindings, not just the localhost. This is how IIS would know which website to
serve when it gets hit with a request.

### What about HTTPS?
Our remote host also sees the protocol of the request, which is whether it was plain http or https. When it sees https, IIS looks for a binding
that matches the domain of the request AND https specified. 

So now, we will need to add a separate binding for https for our website (with the same host name as the http binding), and from this binding, we can
specify a port other than 443 if we're going to have more than one secure website. When creating an https binding, it will ask for a certificate. 
There are many ways to obtain certificates, and each certificate may be good for only the main website, may cover subdomains, or may cover all
websites (and their subdomains) managed by the web author. (We will discuss certificates in a different article.) When properly installed on the 
remote host, the certificate becomes available for selection from inside IIS.

## Shared Hosting Option
We will look deeper into step #4 in the case where we chose the shared hosting option. The shared hosting option is when our websites and other
customers' websites are stored and served from a single physical machine. Everyone's websites on that one machine are on the same hard drive, and
share the all of the memory and computing power of that machine. This is way cheaper and also slower than a VPS or dedicated server. We also do
not have the same level of control as we have in a VPS or dedicated server. We cannot just create any folder we want in the shared host
machine to store our website files. We create a website via the control panel application they give us, and our website's folder gets created (with
the domain or subdomain name as the root), the DNS records set, and our website setup and running in IIS, for us automatically.

### HTTPS in Shared Hosting Option
Unlike a VPS or dedicated server, since we don't have OS-level control, we cannot purchase and install our own certificate on a shared host. The
web hosting company may offer a certificate, albeit, a shared certificate for all customers who want to opt for it. If that is the case, then it
may be a matter of choosing an option in the control panel for the website. If there isn't such a feature, then there may be a way to upload
a certificate via the control panel, which would still save that certificate on the shared host machine, but we didn't need to install it manually.
Each hosting company may handle HTTPS differently for shared hosting, so we may want to seek their support for HTTPS/SSL.