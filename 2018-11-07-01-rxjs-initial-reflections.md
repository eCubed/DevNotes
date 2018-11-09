# Rxjs - Initial Reflections

I'll be honest - I only know of rxjs because I needed to touch it a little bit when I make http calls from an Angular application. All I know that the http functions
of `@angular/common/http`'s `HttpClient` return an `Observable` that could be *subscribed* to by components, and it is through the subscription that a component
obtains the response of the http call. I also know that when the component's life cycle ends, we need to explicitly unsubscribe any `Observable` we subscribed to, in
order to prevent memory leaks.

Upon reading a few blogs and tutorials regarding Rxjs, I gather that it is a system that lets some working unit "broadcast" the fact that something important happened
from within itself, and that other parts of the app *can* listen for that happening, and obtain information about what happened. These listeners *subscribe* to the happening
once and only once. A part of a subscription is a function that would execute everytime the broadcaster issues a happening, and subsequent happenings may have updated data.
In that function, the listener may do anything it sees fit depending of what it hears from the broadcast.

One good example of `rxjs` use beyond `HttpClient` would be a service that detects the screen width continuously. For every change in width pixel value, it determines if 
the size is small, medium, or large, and would only broadcast when the width changes from one of those sizes to another. Subscribing components would then hear small, medium, 
or large, from the service and can do whatever they need to do depending on which size they get at a particular time. A practical use for knowing the size in real time is to 
be able to show and hide elements, or apply certain CSS classes depending on the width size of the browser window. This would be very useful for reactive apps.

The main principle of `rxjs` is for subscribers to receive a continuous stream of ever-changing data. It's not "continuous" in the sense that a subscriber *must* hear something
from the broadcaster every nanosecond. It's "continuous" in the sense that while subscribed, it will bee *continuously alert* to receive broadcasts with updated data multiple times
at some times in the future. We see this with our width-detection service, where subscribers would be alerted whenever the browser's width crosses a size threshhold, and will
continue to listen and react accordingly as we resize the width back and forth.

It is quite interesting to note, though, that the continuous intention of rxjs is used for API calls, because when we make an API call, we're only going to hear a response once
and once only. When we call `HttpClient.get()`, we make a remote call, and only obtain the data once via subscription. Yes, subscribers of `HttpClient` CRUD functions will
still sit there, waiting for the next server response, which won't happen. The continuous philosophy of `rxjs` would make more sense for connecting to a an actual continuous service
like sockets. We connect to a socket once, and as long as we're connected, we can hear for updates multiple times over time. 

There is probably a way, or maybe not, to make a single `HttpClient` CRUD subscription to be actually continuous, we don't know. If not, then there probably are other reasons
why we would want to implement a continuous subscription mechanism to something that is by nature, one-and-done like web API calls.