# Rxjs - Observables on Services

So far, we have explored `Observable`s and `Subject`s, however, we coded everything in the component. That is fine for introduction and demonstration. However, when we
actually develop an app that implements `Observable`s, we should actually transfer them to services. For one, those `Observable`s become accessible from many components
and services themselves, allowing those multiple subscriptions. Also, even if the component is the only one that subscribes to a certain `Observable`, we would still
clean up component code so that all it needs to do is to subscribe, not actually code things like actually making the http call, performing any transformations, etc.

## Building a Service that Hosts Observables

We will take the `GET /posts/{id}` API call that we worked on previously and transfer it to the service:

```javascript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class PostsService {

  private post$: Subject<any> = new Subject<any>();

  constructor(
    private httpClient: HttpClient
  ) { }

  getPost$(id: number): Subject<any> {
    this.httpClient.get(`https://jsonplaceholder.typicode.com/posts/${id.toString()}`)
      .subscribe(data => this.post$.next(data));

    return this.post$;
  }
}
```

Let's pause for a moment since there are quite a few things going on here.

First, instead of the component storing the instance of a `Subject`, the service will store it instead, and keep it private.

When exposing `Subject`s or `Observable`s from a service, we typically do so with a function. This way, we are able to write any necessary logic that would govern
what data will be received upon subscription, and when subscribers will hear from us. In the `getPost$` function, we decide that a single post is to be served, and that
subscribers will receive that post whenever the `httpClient` receives it. When the `httpClient` receives it, that is when we notify subscribers (of `post$`) that we
have the data they're asking for. We typically append `$` to the function name that exposes `Subject`s and `Observable`s. As an aside note, we made `getPost$` take in
an id for some flexibility.

Now, let us inject this newly-created service to our component. If not already, let's make sure that we have added this service to the `providers` array in `app.module.ts`.

```javascript
import { Component, OnInit } from '@angular/core';
import { Observable, Subject } from 'rxjs';

import { PostsService } from '../../services/posts.service';

@Component({
  selector: 'app-observables-study',
  templateUrl: './observables-study.component.html',
  styleUrls: ['./observables-study.component.scss']
})
export class ObservablesStudyComponent implements OnInit {

  post$: Subject<any> = new Subject<any>();

  constructor(
    private postsService: PostsService
  ) { }

  ngOnInit() {

    this.post$ = this.postsService.getPost$(1);
    
    this.post$.subscribe(post => {
      console.log(`Data received: ${JSON.stringify(post)}`);
    });
  }
}
```

Notice that we first get a hold of the `Subject` return from `postsService.getPost$(id)`. We don't always have to do this. We could have gone straight to
`postsService.getPost$(id).subscribe(post => { ... })`, which is subscribe to the `Subject` right away. However, we're doing this to illustrate a point later on.

When we run the app right now, we'll see in the console the post we're expecting.

## No, Do Not Expose Subjects from Services

Recall that one difference between a `Subject` and an `Observable` is that a `Subject` gives us the capability to change the data by calling its function `.next(data)`.
We're exposing a `Subject` from our service, so right from component code, we *can* change the data:

```javascript
this.post$ = this.postsService.getPost$(1);

this.post$.subscribe(post => {
    console.log(`Data received 1: ${JSON.stringify(post)}`);
});

this.post$.next({ data: `Data that I probably shouldn't explicity set!`});
```

When we run the app, we will get two console logs, one that the API call actually returned, and the other one, what we explicitly set.

This is a problem. While subscribing to a `Subject` won't result in multiple identical http requests, which is our desired behavior, we're also exposing to the client the
ability to change the data. Our original intent is to make the http request, not let the client monkey around with deciding what subscribers receive and when they'll receive
new messages. In order to hide the ability `.next(data)`, we will rewrite the `getPost$()` function to return an `Observable` instead of a `Subject`:

```javascript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class PostsService {

  private post$: Subject<any> = new Subject<any>();

  constructor(
    private httpClient: HttpClient
  ) { }

  getPost$(id: number): Observable<any> {
    this.httpClient.get(`https://jsonplaceholder.typicode.com/posts/${id.toString()}`)
      .subscribe(data => this.post$.next(data));

    return this.post$;
  }
}
```

Notice that we don't get typing errors for returning a `Subject` when the function expects a return type of `Observable`. This is because a `Subject` *is* an `Observable`.
Now, if we go back to our component, we will need to change `Subject` to `Observable` to match the return type of `getPost$`:

```javascript 
import { Component, OnInit } from '@angular/core';
import { Observable, Subject } from 'rxjs';

import { PostsService } from '../../services/posts.service';

@Component({
  selector: 'app-observables-study',
  templateUrl: './observables-study.component.html',
  styleUrls: ['./observables-study.component.scss']
})
export class ObservablesStudyComponent implements OnInit {

  post$: Observable<any> = new Observable<any>();

  constructor(
    private postsService: PostsService
  ) { }

  ngOnInit() {

    this.post$ = this.postsService.getPost$(1);
    
    this.post$.subscribe(post => {
      console.log(`Data received: ${JSON.stringify(post)}`);
    });
  }
}
```

Now, `post$` is typed as `Observable`, not `Subject`, though it really is a subject. Now, we can no longer call `.next()` that would inappropriately tinker with the
stream. Let's say that we'll subscribe to `post$` again. Since `post$` is now typed as `Observable`, which would execute per subscription, would we get multiple http
requests as we don't want to happen?

```javascript
ngOnInit() {

    this.post$ = this.postsService.getPost$(1);
    
    this.post$.subscribe(post => {
        console.log(`Data received 1: ${JSON.stringify(post)}`);
    });

    this.post$.subscribe(post => {
        console.log(`Data received 2: ${JSON.stringify(post)}`);
    });
}
```

If we run the page again and we look at the console, we will see that there was only one http request made, and both our clients heard about the message!

## Recap

We will typically want to hide the details of `Observable` and especially `Subject` setup from subscribers for both safety and code maintainability. There are a few things
to remember when using a service to manage `Observable`s and `Subject`s:

* The service privately holds an instance of a `Subject`.
* We will provide a function wrapper that sets up an `Observable` and/or `Subject`. In the case of the `Subject`, we will *never* expose it as a `Subject`. Yes, we
return the very `Subject` itself, however, the return type of this function should be `Observable` to hide the `Subject`'s `.next()` capability.
* At the client, we may get a hold of the returned `Observable` first before subscribing, or we may subscribe immediately. This depends on the app.



