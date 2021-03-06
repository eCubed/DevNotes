# Rxjs - Broadcasting Streams

Most of us have experienced `rxjs` first with `HttpClient` CRUD functions. We knew that we needed to subscribe to observables in order for the actual
API call to happen and then we can obtain the data returned by the server. We may wonder how and when `HttpClient` issues those `Observable`s. In this
article, we will explore how to issue our own `Observable`s that clients subscribe to, as well as explicitly broadcast when the data we want our subscribers
to receive changes.

First of all, what is a stream when we're talking about `Observable`s? We can think of a stream as a channel that data travels through from broadcaster
to subscriber. A stream is kept intact so that more data (updated data, that is) can be sent to the subscriber over time.

## Observable and the of Operator

We're going to issue an `Observable<string>` that will broadcast a single message.

```javascript
import { Component, OnInit } from '@angular/core';
import { Observable, of } from 'rxjs';

@Component({
  selector: 'app-observables-study',
  templateUrl: './observables-study.component.html',
  styleUrls: ['./observables-study.component.scss']
})
export class ObservablesStudyComponent implements OnInit {

  message$: Observable<string>;

  constructor() { }

  ngOnInit() {
    this.message$ = of("Hello world!");

    this.message$.subscribe(message => {
      console.log(`Message: ${message}`);
    });
  }
}
```

We import `Observable` and the `of` operator. The `of` operator is the way to instantiate an `Observable` with a single value, as of Rxjs 6.0.

In this very trivial example, we create an object `message$` typed `Observable<string>`. Note that we append `$` as a convention to easily see and know
that the object we're working with *is* an `Observable` and thus, can be subscribed to.

The `Observable`'s `.subscribe` function takes in a function as a parameter that inputs a string, since `message$` is an `Observable<string>`. In that
callback function, we will have access to the value passed into the `of` operator when we constructed the `Observable<string>`.

When we run the application, we will see the message `'Message: Hello World!'`.

Now, this seems very trivial and we could have just merely console-logged the exact string "Message: Hello World!" without using `Observable`s. At this time,
that is true, however, we are learning about a new mechanism and a new way of thinking. As soon as we called `.subscribe()` of the `Observable`, a stream
was actually created, and the message traveled from the broadcaster to the client. This is different than merely displaying a message on the console. We
will later see why we would want to set up streams to broadcast messages, which will come in handy when different subscribers to the same `Observable`
are in different components.

There's a problem with this observable, though - it will issue only one broadcast ever, which seems to defeat the purpose of being able to broadcast
updated data multiple times. So then, what good is the `of` operator? It is good for mocking `HttpClient`'s CRUD operations that return `Observables`.
We would discuss mocking in another article.

So, then, is it possible to even change the instantiated value of an `Observable`? If so, how would we do that, and most importantly, when we change the
message, how would we ensure that the subscribers will be notified of each update? We'll address this in the next section.

## Manually Instantiating an Observable

We can actually `new`-up an `Observable` besides instantiating one with the `of` operator.

```javascript
export class ObservablesStudyComponent implements OnInit {

  changingMessage$: Observable<string>;

  constructor() { }

  ngOnInit() {
    this.changingMessage$ = new Observable<string>(subscriber => {
      subscriber.next("First Message");
      subscriber.next("Second Message");
      subscriber.next("Third Message");
    });

    this.changingMessage$.subscribe(message => {
      console.log(`Message: ${message}`);
    });
  }
}
```

The constructor of an `Observable<T>` takes in a parameter that is a function that takes in a `Subscriber<T>`. A `Subscriber` represents an instance
of having subscribed to the `Observable` with `Observable<T>.subscribe(...)`. In order to broadcast, we make the call `subscriber.next(value)`,
which really means "OK, subscriber, here's your next value. I'm notifying you now, so you should receive it shortly." In our example, albeit trivial, we
send off three messages in succession. We only subscribed to this observable once, but the subscriber *did* hear all three broadcasts, which results
in the 3 messages being displayed in the console.

## Multiple Subscribers to an Observable

The whole point of a subscription mechanism is to have multiple subscribers. Indeed, we can create another subscription with another call to `.subscribe`.
Before we do that, let's add a `console.log` inside the `Observable` instantiation:

```javascript
this.changingMessage$ = new Observable<string>(subscriber => {

    console.log("----- Instantiating an Observable ----- ");

    subscriber.next("First Message");
    subscriber.next("Second Message");
    subscriber.next("Third Message");
});
```

Now, let's modify the current subscription so that it's distinct ("Message to subscriber 1") from another subscription that we'll also create:

```javascript
this.changingMessage$.subscribe(message => {
    console.log(`Message to subscriber 1: ${message}`);
});

this.changingMessage$.subscribe(message => {
    console.log(`Message to subscriber 2: ${message}`);
});
```

If we run the application, we'll get the following output:

```
----- Instantiating an Observable -----
Message to subscriber 1: First Message
Message to subscriber 1: Second Message
Message to subscriber 1: Third Message
----- Instantiating an Observable -----
Message to subscriber 2: First Message
Message to subscriber 2: Second Message
Message to subscriber 2: Third Message

```

It's great that both subscriptions heard all 3 broadcasts sent by the `Observable`. There is a problem, though. There ended up being two setups for
broadcasting just those 3 messages, one for each subscriber. The whole idea is to broadcast only 1 set of messages for all subscribers, and all
subscribers should hear from the same set of messages. This becomes a problem when we make http calls:

```javascript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of } from 'rxjs';

@Component({
  ...
})
export class ObservablesStudyComponent implements OnInit {

  constructor(
    private httpClient: HttpClient
  ) { }

  ngOnInit() {  

    let observableForGet$ = this.httpClient.get('https://jsonplaceholder.typicode.com/posts/1');

    observableForGet$.subscribe(data => {
      console.log(`Data to subscriber 1: ${JSON.stringify(data)} `);
    });

    observableForGet$.subscribe(data => {
      console.log(`Data to subscriber 2: ${JSON.stringify(data)} `);
    });
  }
}
```
By looking at the above code, great - we have two subscribers that will receive the http response for that one http call. That is just our impression
because we see *only one* call to `.get(url)`. If we observe our network traffic in our browser's dev tools, we will see that there were actually *two*
http requests made. We must remember that simply calling `.get(url)` doesn't actually make the http call. An http call will be made only when we subscribe,
and for an `Observable`, there will be an http request made *for each subscription*. This might be only a bandwidth and performance problem for a GET request,
but if the GET request generates a log at the server, we would get duplicate logs. It's much worse for POST, including upload - we'll end up with duplicate
records or duplicate files.

`Observable`s seem pointless then if it will process for each subscriber, and we only want one processing that all subscribers will hear. That's like
a TV station airing the same show over and over for every TV tuned to that channel at that time - not viable. So far, a plain `Observable` is good if
we know for sure that there will be only one subscriber. So, is there a way to broadcast once for all subscribers? Yes.

## Subjects

Subjects are a special kind of `Observable` that would put out one broadcast and multiple subscribers would hear *that very same one* broadcast. This is the
more expected behavior we would want.

```javascript
import { Component, OnInit } from '@angular/core';
import { Subject } from 'rxjs';

@Component({
  ...
})
export class ObservablesStudyComponent implements OnInit {

  constructor(
    private httpClient: HttpClient
  ) { }

  ngOnInit() {  

    let subject$ = new Subject<string>();

    subject$.subscribe(message => {
      console.log(`Subscriber 1 receives: ${message}`);
    });

    subject$.subscribe(message => {
      console.log(`Subscriber 2 receives: ${message}`);
    });

    subject$.next((new Date()).getTime().toString());
  }
}
```

First, we instantiate a `Subject<string>`, since we want the data to be broadcasted to be a string. At this point, we have not yet set a value for what we
would want the `Subject` to broadcast.

We then go ahead and create two subscriptions. Finally, we broadcast a message with `subject$.next(message)`. In this case, we want to broadcast the current
timestamp, which `(new Date()).getTime().toString()` would get us.

When we run the page right now, we will receive the following output, or one similar to this:

```
Subscriber 1 receives: 1541905542958
Subscriber 2 receives: 1541905542958
```

We know that there was only one broadcast made that the two subscribers heard. Had this been an actual `Observable`, we would get two different time stamps,
which means that there were actually two broadcasts, one for each subscriber.

Another thing of interest to note with `Subject`s that we could not do with regular `Observable`s is to directly update the broadcast message with `.next()`.
A `Subject` *has* the function `.next()`, but an `Observable` *does not*. With `Observables`, we could only call `next()` from a function we would need
to pass to its constructor as we've done above.

Since `Subject`s possess the behavior we want, again, broadcast once, hear that same broadcast in multiple subscriptions, we may wonder why the dev team at
Angular didn't implement a `Subject` for `HttpClient` CRUD functions. Those functions return an observable, and if subscribed to *n* times, there would
also be *n* http requests made, one for each of those subscriptions. So, what would we need to do in order to be able to end up calling an `HttpClient`
CRUD function, with only one resulting http request, but be able to subscribe to it multiple times?

## Ensuring Http Calls are Executed Once When There are Multiple Subscribers

There is no sure way to subscribe multiple times to a plain `Observable` that results in only one http call. We will need to make the following workaround:

We will *not* have clients subscribe to the `Observable` directly. Instead, we will create a `Subject` that clients will subscribe to multiple times. The single,
direct subscription to the `Observable` isn't considered a client subscription; it is where we'll call `Subject.next` and feed to it the data obtained from the
`Observable`'s network call:

```javascript
export class ObservablesStudyComponent implements OnInit {

  posts$: Subject<any> = new Subject<any>();

  constructor(
    private httpClient: HttpClient
  ) { }

  ngOnInit() {

    this.httpClient.get('https://jsonplaceholder.typicode.com/posts/1')
      .subscribe(data => this.posts$.next(data));

    this.posts$.subscribe(data => {
      console.log(`Data received from subscriber 1: ${JSON.stringify(data)}`);
    });

    this.posts$.subscribe(data => {
      console.log(`Data received from subscriber 2: ${JSON.stringify(data)}`);
    });
  }
}
```

If we look in our browser's console output, we will see the following result:

```
XHR GET https://jsonplaceholder.typicode.com/posts/1

Data received from subscriber 1: {"userId":1,"id":1,...}
Data received from subscriber 2: {"userId":1,"id":1,...}
```

We can see that both subscriptions heard the single broadcast. Note that clients subscribed to `posts$`, which is a `Subject`, not directly to the `Observable`.

## Next Steps

We've coded everything in-component. In reality, we will want to put http calls in services so that components will only need to be responsible for subscription and obtaining
data. When we transfer http calls into services, there are a few things we'll need to consider. Of course, we'll want multiple subscriptions to the same `Observable` but ensure
only 1 http call will be made for all those subscriptions. We've discovered how to work around that by exposing a `Subject` instead. However, exposing a `Subject` would give
the client the ability to manipulate it by explicitly calling `.next()`, which would yield wrong results. We'll discuss this in a near-future writeup.




