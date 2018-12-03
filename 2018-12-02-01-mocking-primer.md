# Angular - Mocking Primer

When we developed Angular apps in the past, chances are we made actual external API calls to obtain and send data. When we say "external", it means that
the API is outside our Angular app, and the API endpoints may be a back-end server running on the same local machine in its own debug mode, or it could
be an actual endpoint with a domain name. Our front-end up was highly dependent on that external API to be up and running, or else, we couldn't even run
the app, let alone develop it. 

We may have never thought of this before, but it *should* be possible to develop our front-end app without having to make http requests. Our app somehow
would need to contain the mock data. One quick and trivial "solution" is to simply create arrays of objects in components that need to display them. There
are obvious problems with that approach. For one, we would crowd our component code with test data. Secondly, it would be tedious to explicitly code detection
of whether the app is running in dev vs. production just to know whether to make the API call or to just access local data.

Ideally, we would want to write our components such that they *don't* know or *don't even care* whether the data obtained came from mock or an actual
API call. This means that our components should *never* access the *httpClient* service directly and explicitly, nor should it even access such a local
mock service directly. Subsequently, we should *never* write code in our components that determines whether the app is running in dev vs. production mode,
and make the decision to make http requests for production and merely access local data for development. We want 100% of our code in all our components
to run *as-is* regardless of development environment. We want to focus on how data and the UI interact, not *how* the data is obtained.

on that deployed Web API for our app to run. However, when it's time to deploy our app to production, we want it to actually make those external API calls
automatically. We wouldn't
