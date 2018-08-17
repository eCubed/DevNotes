# Asp.NET Core Security Overview

This section addresses several security considerations in Asp.NET Core development, including encryption and decryption,
token generation and consumption, etc.

## Problems with our Functioning Jwt Middleware
The primary motivation for brewing our own Jwt middleware is ease and total control. We have so far created our own auth and 
jwt middleware because we felt we had full control of the entire process, especially working with claims. We have also stubbed out via interface the mechanism 
that encrypts and decrypts strings. This way, our middleware isn't tightly coupled with an encryption and decryption algorithms.

The secondary motivation is the encrypted value was issued by the auth serer was invalid per the consuming resource server.
The reason for this was that the encryption/decryption mechanism which we had no control, from the libraries we utilized,
relied on some unique key from each of their prospective machines, which was different. Our current implementation relies on
a string token saved in appsettings.config, whose value was copied from auth server to all consuming servers (resource and
file). This way, we are guaranteed that the token issued by the auth server will be recognized (cryptographically) by the
resource server.

But we may have a problem. Our implementation might not adhere to standards and may be insecure. So at this point, we will
revisit, research, and study middleware that ASP.net Core has for us out-of-the-box to utilize.

## Sensitive Information We Currently Store in appsettings.json
From various tutorials, we have adapted the practice of putting the connection string in plain text into appsettings.json.
Although we told our repository not to track appsettings.json, the connection string must have made it to the repo in an earlier
commit before we added the entry to .gitignore. To this day, we are making sure we first fix .gitignore to untrack appsettings
files, but it still isn't secure.

There is supposed to be a way to surely manage these connection strings, which does not store the plain text in appsettings.json.
We are still not so sure about whether we will be able to obtain back the plain text connection string should we need to
clone the repo onto another team member's machine. We will need to investigate this.

See the [User Secrets](01-user-secrets.md) article. We have researched and tried it out. It works well. However, we would not
get back the secretly-stored settings prepared in one machine when we clone the repo to another machine. On that other machine,
the secrets need to be set again manually.

## Certificates?
These are supposedly a security feature that must be implemented for the deployed project. We are not sure about them, especially
how to obtain them, where to store them on the very machines that will host the servers we write, and how/when/why we need to
access them in our code. Also, we don't know how we'll handle these certificates (if we would even, at all) during development.

## Https
We are working with Asp.NET Core 2.0, and we do not know exactly how to set this up. At this time, we only like to think that
we do absolutely nothing different in our base code, not even some setup code in Startup.cs. We would like to assume that
the host (IIS) simply need to state that the website is https. It turns out that we may have to do more than just rely on
the host to take care of HTTPS.

We have examined [https with Asp.NET Core 2.0](01-https-asp-net-core-20.md). It turns out that there are scenarios that we would
give our app or IIS more control, which results in the same thing - forcing https, and the amount of control to allocate for
either our app or IIS is based on preference.
