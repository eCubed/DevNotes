# Angular - Mocking Primer

When we developed Angular apps in the past, chances are we made actual external API calls to obtain and send data. Yes, this means we actually made external http
requests *in development*. Those "external" servers were external *to the app*, though for the most part, they were separate processes running on the same local
machine, typically, in my case, an Asp.NET Core Web API application running on debug mode on IIS Express. With this kind of setup, it would be impossible to develop
the front-end app if that external server isn't up and running. If we wanted to share the app with a front-end-only developer in our team because we're asking them
to enhance styling and fix layout issues, they wouldn't be able to even run the app.

We may have never thought of this before, but it *should* be possible to develop our front-end app without having to make http requests. Our app somehow
would need to contain the mock data. One quick and trivial "solution" is to simply create arrays of objects in components that need to display them. There
are obvious problems with that approach. For one, we would crowd our component code with test data. Secondly, it would be tedious to explicitly determine
dev vs. production just to know whether to make the API call or to just access local data.

Ideally, we would want to write our components such that they *don't* know or *don't even care* whether the data obtained came from mock or an actual
API call. This means that our components should *never* access the *httpClient* service directly and explicitly, nor should it even access such a local
mock service directly. Subsequently, we should *never* write code in our components that determines whether the app is running in dev vs. production mode,
and make the decision to make http requests for production and merely access local data for development. We want 100% of our code in all our components
to run *as-is* regardless of runtime environment. We want to focus on how data and the UI interact, not *how* the data is obtained.

We will need to make major changes to our application in order to set up and utilize mock data for development and when we deploy to production, the app would actually
make the API calls. There are quite a few principles involved, which may seem daunting at first, but it pays off quite well knowing that our component code is
neat and to-the-point without the fluff of hard-coded mock data and environment detection.
