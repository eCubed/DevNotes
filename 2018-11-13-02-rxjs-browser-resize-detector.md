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

To be continued...
