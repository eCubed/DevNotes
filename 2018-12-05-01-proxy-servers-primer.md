# Asp.NET Core - Proxy Servers

For our purposes, a proxy server is a Web API application whose job is to examine the incoming request, and based on the request (most often the path of the url, and sometimes headers),
would then forward to the appropriate server. Right now, we have distinct servers for authentication, files, and resource, and front-end applications would need to keep track of 3 distinct
url roots. This isn't a huge concern, but it would be nice if front-end applications would only need to keep track of a single url root, the one that points directly to the proxy server.

The larger problem is that the browser issues pre-flight requests (http OPTION calls) if making API calls to a server (including another subdomain) *outside* the website that the SPA sits on.
Supposedly, the browser *doesn't* always issue OPTION calls, but since we always specify `Content-Type: application/json;charset=utf8` and often have the `Authorization: Bearer 
<token>` header, the browser *will* issue an OPTION call. Many of our requests will have custom headers, and so, we can assume that the browser will issue an OPTION call to *every* http
request our app makes.

The OPTION call isn't very large, however, that's an additional http call made to external servers... for every request. This means two trips, thus abouttwice as long to finally get the 
response. That's a performance issue at the front-end - user needs to wait 2x longer overall, and at the back end - the server would need to handle that OPTION request, which costs some
resources.

So. How would a proxy server reduce or better yet, avoid pre-flight requests? We will need to create the a proxy server that knows how to forward requests to the right servers. The proxy
server *will* have an index.html page that our SPA will be served from, which *is exactly* the very index.html page that the chosen front-end framework deploys to production. From inside
the SPA, the single "url root" will be *internal* or *local*, that is, *not* a URL at all, but a relative path that the proxy server recognizes. The proxy server would take those requests
from the SPA, examines them, and then forwards them to various external web applications. The SPA *never* makes external API calls - only to the local proxy server. Since there would be
no cross-site requests, the browser would *never* make pre-flight requests. The external API calls happen *at the back end* with the proxy server.

So, in this series, we will first examine how to build a proxy server. Then, we will examine how to deploy it along with the front-end application.
