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
a TV station airing the same show over and over for every TV tuned to that station at that time - not viable. So far, a plain `Observable` is good if
we know for sure that there will be only one subscriber. So, is there a way to broadcast once for all subscribers? Yes.

