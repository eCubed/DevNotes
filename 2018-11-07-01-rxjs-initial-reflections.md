# Rxjs - Initial Reflections

I'll be honest - I only know of rxjs because I needed to touch it a little bit when I make http calls from an Angular application. All I know that the http functions
of `@angular/common/http`'s `HttpClient` return an `Observable` that could be *subscribed* to by components, and it is through the subscription that a component
obtains the response of the http call. I also know that when the component's life cycle end, we need to explicitly unsubscribe any `Observable` we subscribed to, in
order to prevent memory leaks.

So far, I use Rxjs just to be able to consume a web API response so I can display information on the screen. I know there's more to it than just being able to perform
CRUD!

Upon reading a few blogs and tutorials regarding Rxjs, I gather that it is a system that lets some working unit "broadcast" the fact that something important happened
from within itself, and that other parts of the app *can* listen for that happening, and obtain information about what happened. Now, these *listeners* will always
hear about another one of the same happening some time in the future, but maybe with updated information.

Now, that was somewhat in vague general terms. Let's come up with a trivial example first. Suppose that there is some sort of worker that increments an integer every 10
seconds. It would want to broadcast to the rest of the app when the number becomes evenly divisible by 10, and also broadcast that very integer when divisibility by 10
happens. Suppose that on the screen, there are 5 components that want to display this number in each of their unique visual ways. Each of those 5 components would need to
listen, or *subscribe* to that "divisible-by-10" worker. When it's all set up, each of those 5 components will update their displays simultaneously, exactly that worker
broadcasted whenever its internal number because evenly divisible by 10.

A more practical example would be a worker that detects the screen width continuously. For every change in width pixel value, it determines if the size is small, medium,
or large, and would only broadcast when the width changes from one of those sizes to another. Subscribing components would then hear small, medium, or large, from this
worker, and can do whatever they need to do depending on which size they get at a particular time. A practical use for knowing the size in real time is to be able to show
and hide elements, or apply certain CSS classes depending on the width size of the browser window.

So, practically, subscribers will continue to hear from the broadcaster multiple times throughout the lifetime of their app, and this concept is referred to as listening
for continuous change over time, which is the central principle of Rxjs.

The divisible-by-10 and width detector examples I mentioned follow that main principle, because the number and size change over time, and we've got components listening for
them continuously. It is quite perplexing and interesting to note that the "continuous" principle of Rxjs seems to conflict with calling an external web API. By nature, an
HTTP request (`httpClient.get()`, for example) happens once, and we hear from it once. It's not like we make the call `httpClient.get()`, and with that one call, and one
subscription, we would hear from the server multiple times in the near future with updated data. If we want to get updated data, we *explicitly* call `httpClient.get()`
again, subscribe to it, and get the new data from that subscription. So, it doesn't seem to make sense that the development team uses a subscription architecture for http
calls because the notion of subscription implies that with that one subscription, we'll hear from the broadcaster multiple times. It turns out that there are a few other
things that we may be able to do with subscriptions and observables besides receiving updated data, that would make sense in one-and-done situations like calling an API.
We'll explore those in this series.