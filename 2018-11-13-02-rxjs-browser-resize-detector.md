# Rxjs - Browser Resize Detector

We have learned quite a few things about `Observable`s and `Subject`s, so now, we're ready to tackle something useful that Rxjs is good for besides trivial examples and
making http calls. We will work on a service that continually detects the browser's pixel width (as we'll continually resize it), but only broadcast to subscribers when the
current size has just crosses a threshhold between small and medium, and medium and large. We'll have 3 distinct sizes for simplicity. So, no, we will not broadcast a size
for every detection of a width change.

## Starting the Service

We've already made the decision that there will be 3 sizes, small, medium, and large. We'll also decide that small is anything upto 400px, medium is 401px to 800px, and
801px and greater will be considered large. The service will return "small", "medium", and "large" when it deems appropriate.

```javascript
import { Injectable } from '@angular/core';

import { Observable, Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ViewportWidthService {

  private currentPixelWidth: number;

  private sizeChanged$: Subject<string> = new Subject<string>();

  constructor() { }

  private determineSize() {
    if (this.currentPixelWidth <= 400)
      this.sizeChanged$.next('small');
    else if (this.currentPixelWidth >= 401 && this.currentPixelWidth <= 800)
      this.sizeChanged$.next('medium');
    else     
      this.sizeChanged$.next('large');
  }

  public size$() : Observable<string> {

    return this.sizeChanged$;
  }
}
```

We set up a `Subject<string>` called $sizeChanged for subscription of when the size changes, and the wrapper function `size$()` that returns an `Observable<string>` 
to hide the fact that `sizeChanged$` really is a `Subject<string>`. We do this so that clients cannot explicitly change the size because that's only this service's job.

We need to keep track of the current pixel width so as the browser width changes, we can update it live. We have also provided the logic of how to determine the size from the 
current pixel width value. Notice that it in this function that we decide which size to broadcast, but at this point, nothing calls `determineSize()`. With this function
alone as we see it, it's going to always broadcast something whenever it's called, even if the actual current pixel width may have changed but not cross a size threshhold.

## Listening for Width Change

Be aware that in this project, we will introduce some new `rxjs` that we haven't seen before.

Regarding width change, we will need to detect two `window` events - `load` and `resize`. We need to detect the `load` so that we'll get a hold of the window's inner width
when the page loads for the first time. `rxjs` provides for us the `fromEvent(context, eventName)` function that returns for us the `Observable` for a `window` event. 
To turn the `windows` `load` event into an `Observable` so we can subscribe to it, we write:

```javascript
fromEvent(window, 'load').subscribe(() => {

});
```

From inside the function that is subscribe's parameter, we have access to `window.innerWidth`, which would be the value we want to continually listen for. We will need to put 
this code in the constructor:

```javascript
import { Injectable, OnInit } from '@angular/core';

import { Observable, Subject, fromEvent } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ViewportWidthService implements OnInit {

  private currentPixelWidth: number;
  private currentSize: string;

  private widthChanged$: Subject<string> = new Subject<string>();

  constructor() {
    fromEvent(window, 'load').subscribe(() => {

    });

    fromEvent(window, 'resize').subscribe(() => {

    });
  }
```

Now, it is inside the `.subscribe()` function's first parameter, which is a callback, that we grab `window.innerWidth` and set it to `this.currentPixelWidth`.
Also, we can go ahead and call `this.determineSize()`. We'll do the same for both `load` and `resize`:

```javascript
constructor() {
    fromEvent(window, 'load').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });

    fromEvent(window, 'resize').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });
  }
```
## Using our Service in a Component

Now, we'll need to get a hold of our service into a component. We can go ahead and subscribe to `size$()`:

```javascript
import { Component, OnInit } from '@angular/core';

import { ViewportWidthService } from '../../services/viewport-width.service';

@Component({
  selector: 'app-viewport-width-tester',
  templateUrl: './viewport-width-tester.component.html',
  styleUrls: ['./viewport-width-tester.component.scss']
})
export class ViewportWidthTesterComponent implements OnInit {

  constructor(
    private viewportWidthService: ViewportWidthService
  ) { }

  ngOnInit() {
    this.viewportWidthService.size$().subscribe(size => {
      console.log(`The size is ${size}`);
    })
  }
}
```

## Broadcast Only When Subject New Value is Different than Previous Value

If we run the app right now, we'll get log messages about the current size of the browser. Indeed, if we cross a threshhold, the size changes. There's a problem, though.
If we look at our log messages at the console, we'll see that we get a new log message for even the tiniest smidgeon of a move when we resize. We wanted our service to 
only broadcast when the value of `size` changes. How do we do that?

We go back to our service and make the following changes:

```javascript
...
import { distinctUntilChanged } from 'rxjs/operators';

export class ViewportWidthService {

    ...

    public size$() : Observable<string> {

    return this.sizeChanged$.pipe(
      distinctUntilChanged()
    );
  }
}
```

We import the operator `distinctUntilChanged` which ensures that a broadcast will only be made if the internal value of the `Subject` is different than the
incoming value when we call `.next()`.

The `pipe` function is our opportunity to modify our `Observable` before we expose it. There are various manipulations that we would at times want to do, like
"clean up" the response of the API call into the exact object we want our subscribers to receive (the `.map` operator), or even delay execution (the `.delay` operator).
In our case, we want to use `distinctUntilChanged`, and we list it as a parameter of the `pipe` function.

Now, if we run the application again, we will no longer see all those log messages for even the tiniest move of a resize. We will now only get a new log message when the
we cross a size threshhold!

## Cleaning Up Subscriptions

Upto this point, we have always called `.subscribe()` for the sole purpose of obtaining data, either from an API call, or from a home-made service like this one we're
working on. The `.subscribe()` function actually returns a `Subscription` object, from which we can stop listening to broadcasts if we choose. In practice, we will always
want to get a hold of `Subscriptions` because we will need to *manually* and *explicity* unsubscribe them when our services, components, and directives are destroyed.
If we are not careful with our `Subscriptions`, or more specifically, if we "leave them hanging", they will run in the background unnecessarily.

Let's take for example our width detector service. If we're on a page that wants to use it, and we are currently on that page as we resize the browser, that's fine.
However, if we move to a different page *that doesn't even inject* our service, that subscription from that other page we're no longer on *will still execute*. That's a lot
of unnecessary work for our app to perform, and if we've got enough subscriptions from many components and not clean up or unsubscribe, we could really slow down our
app. This phenomenon is called memory leak.

Let us start with our service and clean up any dangling subscriptions. We have in the constructor two `fromEvent(..).subscribe(..)` calls. We will need to get a hold
of the `Subscription` returned by `.subscribe(..)`, so we can later be able to unsubscribe it. Since unsubscribing is done in a different function, we will want to
add `Subscription` properties to the service. We mark them with the `private` modifier because they're supposed to be internal only to our service.

```javascript
...
import { Observable, Subject, Subscription, fromEvent } from 'rxjs';
...

export class ViewportWidthService {

  ...

  private sizeChanged$: Subject<string> = new Subject<string>();
  private onWindowLoadSubscription: Subscription;
  private onWindowResizeSubscription: Subscription;

  constructor() {
    this.onWindowLoadSubscription = fromEvent(window, 'load').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });

    this.onWindowResizeSubscription = fromEvent(window, 'resize').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });
  }
```

The next thing we'll need to do is to implement `OnDestroy`, which will require us to implement the function `ngOnDestroy()`. It is in this function that we will need
to unsubscribe our subscription. The Angular runtime will automatically call `ngOnDestroy()` if we implement `OnDestroy`, when our services, directives, and components
are automatically taken down. We won't cover lifecycle in this article, so if we desire, we can research when exactly `ngOnDestroy()` gets called.

```javascript
ngOnDestroy() {
    if (this.onWindowLoadSubscription)
        this.onWindowLoadSubscription.unsubscribe();

    if (this.onWindowResizeSubscription)
        this.onWindowResizeSubscription.unsubscribe();
}
```

As a good and often necessary practice, we'll need to first check the existence of a `Subscription` before we can call unsubscribe. When our components get complex enough,
it will be highly possible that we may *not* have subscribed to an `Observable` before `ngOnDestroy()` is called.

Now, let's go to our component that injects our service. On the `ngOnInit()` function, there is a `Subscription` to `size$()` that isn't caught. We'll need to manage
that.

```javascript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';

import { ViewportWidthService } from '../../services/viewport-width.service';

@Component({
  selector: 'app-viewport-width-tester',
  templateUrl: './viewport-width-tester.component.html',
  styleUrls: ['./viewport-width-tester.component.scss']
})
export class ViewportWidthTesterComponent implements OnInit, OnDestroy {

  private sizeSubscription: Subscription;

  constructor(
    private viewportWidthService: ViewportWidthService
  ) { }

  ngOnInit() {
    this.sizeSubscription = this.viewportWidthService.size$().subscribe(size => {
      console.log(`The size is ${size}`);
    });
  }

  ngOnDestroy() {
    if (this.sizeSubscription)
      this.sizeSubscription.unsubscribe();
  }
}
```

The procedure for setting up `Subscription`s so we can later unsubscribe them is very similar to how we managed `Subscription`s in our service. In fact, this is
how we'll manage subscriptions in any component, service, and directives we'll write. We will need to:

1. Always create a private `Subscription` property in the component, service, or directive for every `.subscribe()` call to an `Observable`.
2. Catch the `Subscription` into the `Subscription` property we set up.
3. Implement `OnDestroy` on the component, service, or directive. We will now need to implement the function `ngOnDestroy()`.
4. On the `ngOnDestroy()`, for every `Subscription` property our class has, we'll need to check for its existence, and if it exists, we call unsubscribe.

Henceforth, in anything we write, we'll need to unsubscribe all our `Subscription`s!

## Problem When we Navigate to a Page that Injects Service

So far, all is well - we hear from the `Observable` when the size of the browser changes. On full page load, we also hear about the size because from our service,
we have also taken into account the `window` `load` event.

Let us first catch the size from our subscription, store it to a variable in our component, and then display the value on the screen.

```javascript
export class ViewportWidthTesterComponent implements OnInit, OnDestroy {

  private sizeSubscription: Subscription;
  private size: string;

  constructor(
    private viewportWidthService: ViewportWidthService
  ) { }

  ngOnInit() {
    this.sizeSubscription = this.viewportWidthService.size$().subscribe(size => {
      this.size = size;
    });
  }

  ngOnDestroy() {
    if (this.sizeSubscription)
      this.sizeSubscription.unsubscribe();
  }

```html
<p>
  viewport-width-tester works. The size is {{ size }}
</p>
```

If we fully reload the page, and then resize the browser, we'll see that the size will be displayed and updated as expected.

Now, let's go to another route *in our application*. Let's then fully reload the application in our browser. Now, let's navigate back to our page that injects our service.
If we look at the screen display, we will see that `{{ size }}` is undefined, resulting in an empty string display. What happened here? Since we fully loaded the page,
a `window` `load` event actually happened, but *our service wasn't instantiated yet*, because that other route we were in *did not* inject our service in its constructor.
So, if our service wasn't yet instantiated, then it could not have intercepted that `window` `load`. Now, when we navigated to the route that *does* inject our service,
our service was instantiated, but by that time, there was *no `window` `load`* that happened. 

There is a hack for this case. Upon instantiation of our service, its constructor is called, and at the constructor, before we listen for `load` and `resize`, we *already*
have access to `window.innerWidth`. We'll now take the opportunity, first thing, in the constructor, to set `this.currentPixelWidth` to `window.innerWidth`, and
then call `this.determineSize()`, which would fire off broadcasting. However, we need to wrap those two lines with `setTimeout`, and give a small delay value.

```javascript
export class ViewportWidthService implements OnDestroy {

  private currentPixelWidth: number;

  private sizeChanged$: Subject<string> = new Subject<string>();
  private onWindowLoadSubscription: Subscription;
  private onWindowResizeSubscription: Subscription;

  constructor() {

    setTimeout(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    }, 10);

    this.onWindowLoadSubscription = fromEvent(window, 'load').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });

    this.onWindowResizeSubscription = fromEvent(window, 'resize').subscribe(() => {
      this.currentPixelWidth = window.innerWidth;
      this.determineSize();
    });
  }
```

Now, let's save the code. Let's go to a route in our app that doesn't inject the service. Then, let's reload the page. Now, let's navigate back to the route that
does inject the service. Upon navigating to this page, the service gets instantiated, and at the constructor, we would grab the `window.innerWidth`, and then
fire off the broadcast with `this.determineSize`. For some odd reason, not delaying (or no wrapping with `setTimeout`) will not fire off the broadcast we want.

There is another problem. Right where we are, on the page that injects our service, let's navigate away to another route still in our app. Let's navigate back. Now,
our `{{ size }}` is empty again. What happened? Why didn't the code in our constructor run that accesses `window.innerWidth` without listening for `load` and
`resize`? By this time, our services was *already instantiated*, so its constructor won't run again, and also, no `load` or `resize` happened, so there is no way
for our service to broadcast a size change message. What is the solution for this?

We will need to keep track of a `currentSize` property on our service, that is set at the same time as when we broadcast with `next`. We will make `currentSize`
public because a component that injects our service will also need to *access* the `currentSize` *before* it subscribes to `size$()`! Let's start with our service:

```javascript
export class ViewportWidthService implements OnDestroy {

  private currentPixelWidth: number;

  private sizeChanged$: Subject<string> = new Subject<string>();
  private onWindowLoadSubscription: Subscription;
  private onWindowResizeSubscription: Subscription;

  public currentSize: string;

  ...

  private determineSize() {

    if (this.currentPixelWidth <= 400) {
      this.currentSize = 'small';
      this.sizeChanged$.next('small');
    }
    else if (this.currentPixelWidth >= 401 && this.currentPixelWidth <= 800) {
      this.currentSize = 'medium';
      this.sizeChanged$.next('medium');
    }
    else {
      this.currentSize = 'large';
      this.sizeChanged$.next('large');
    }
  }
  ...
}
```

Now, let's head onto the injecting component:

```javascript
export class ViewportWidthTesterComponent implements OnInit, OnDestroy {

  private sizeSubscription: Subscription;
  private size: string;

  constructor(
    private viewportWidthService: ViewportWidthService
  ) { }

  ngOnInit() {

    this.size = this.viewportWidthService.currentSize;
        
    this.sizeSubscription = this.viewportWidthService.size$().subscribe(size => {
      console.log(`The size is ${size}`);
      this.size = size;
    });
  }
```

Now, if we go through our app, and refresh the page in different places, leave, return, etc., we will now always have access to the viewport width's current size.
Now, at any moment, components that inject and subscribe to our service's `Observable`s, will always get a size, and can use that size for its own logic!

