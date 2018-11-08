# Angular Practices - Mocking APIs

When I developed full-stack applications even upto currently, I have always created the back end first. The back end ran on debug mode locally on my machine, and I have always created a dev
database that my back end connected to. With our current setup at work, we have set up a separate machine in our network that hosts the database, and it was intendedto be our "dev database".
So, my local servers would actually connect to a database that machine within our network.

My front-end app would then make call APIs on my localhost back end servers that connect to an external (to my machine) database. This was not a problem until, for some reason, that computer
that hosts that database goes down. That would suspend my front-end development until IT resolves some issue and puts that machine back on. It doesn't happen very often, but when it does,
that's an unknown amount of lost development time. One possible solution is to connect to local databases instead, which we'll pursue. However, the solution we want to pursue at the front
end is to create a set up that would act like an API-calling service, but the data is actually managed and served in-application. How would I pull such a thing?