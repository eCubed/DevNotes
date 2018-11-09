# Angular Practices - Mocking APIs

When I developed full-stack applications even upto currently, I have always created the back end first. The back end ran on debug mode locally on my machine, and I have always created a dev
database that my back end connected to. With our current setup at work, we have set up a separate machine in our network that hosts the database, and it was intended to be our "dev database".
So, my local servers would actually connect to a database that machine within our network.

My front-end app would then make call APIs on my localhost back end servers that connect to an external (to my machine) database. This was not a problem until, for some reason, that computer
that hosts that database goes down. That would suspend my front-end development until IT resolved some issue and put that machine back on. It doesn't happen very often, but when it does,
that's an unknown amount of lost development time. One possible solution is to connect to local databases instead, which we'll pursue. However, the solution we want to pursue will take place
at the front end. The set up that would act like an API-calling service, but the data is actually managed and served in-application.

# Preparing for Swappable Service Instances

To save time, we won't set up our own back-end API to test against. We'll just use one online. We'll check out [JSONPlaceHolder](https://jsonplaceholder.typicode.com/), and we'll choose
the photos resource. We will create a `PhotosService` which will call the `GET /photos` endpoint. We will also create `MockPhotosService` which will actually not call the
external API, but will serve hardcoded data. In order to be able to easily swap these two services, they must first have a common interface, `IPhotosService`:

```javascript
import { Observable } from 'rxjs';

export interface IPhotoService {
  getPhotos$() : Observable<any>;
}
```

We will now create the API-calling service first, `PhotosService`, and it will implement `IPhotosService`:

```javascript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

import { IPhotoService } from './iphotosservice';

@Injectable({
  providedIn: 'root'
})
export class PhotosService implements IPhotosService {

  constructor(
    private httpClient: HttpClient
  ) { }

  getPhotos$() : Observable<any> {
    return this.httpClient.get('https://jsonplaceholder.typicode.com/photos?albumId=1');
  }
}
```

Since `httpClient.get()` returns an `Observable<T>`, we can just return it as-is. We added `?albumId=1` to decrease
the number of results.

If we haven't already, let's remember that we should `import { HttpClientModule } from '@angular/common/http';` in app.module.ts
and then add it to the `imports` array.

We will now create the fake service `MockPhotosService` which will also implement `IPhotosService`;

```javascript
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';

import { IPhotosService } from './iphotosservice';

@Injectable({
  providedIn: 'root'
})
export class MockPhotosService implements IPhotosService {

  photos: Array<any> = [
    {
      "albumId": 6,
      "id": 15,
      "title": "Sunrise Beach",
      "url": "https://via.placeholder.com/600/92c952",
      "thumbnailUrl": "https://via.placeholder.com/150/92c952"
    },
    {
      "albumId": 1,
      "id": 16,
      "title": "Flexing Money",
      "url": "https://via.placeholder.com/600/771796",
      "thumbnailUrl": "https://via.placeholder.com/150/771796"
    },
    {
      "albumId": 1,
      "id": 17,
      "title": "Try Not To Laugh",
      "url": "https://via.placeholder.com/600/24f355",
      "thumbnailUrl": "https://via.placeholder.com/150/24f355"
    },
  ]

  constructor() { }

  getPhotos$() : Observable<any> {
    return of(this.photos);
  }
}
```

Since this is a mock service, we hardcode the array of photos as the real API would return. We just changed the titles so we can see the difference between the mock
and real services.

Note the body of `MockPhotosService`'s `getPhotos$` implementation. We have the `of` operator, which really is a function that would return an `Observable<T>`.
This is how we complete faking a web API call. As we'll see later, if `MockPhotosService` was the chosen photos service at run time, the client will still call
`.subscribe()` just like it were a real http call.

## Registering the Interface and Concrete Service

We discussed registering an interface with a concrete service (that implements the interface) in [this article](2018-11-07-01-interface-di.md). In the
`providers` array, we add the entry:

```javascript
...
import { PhotosService } from './services/photos.service';
import { MockPhotosService } from './services/mock-photos.service';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
  ],
  providers: [
    ...
    { provide: 'IPhotosService', useClass: PhotosService }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

`provide` means register a string key for the class name of the concrete service, as specified with `useClass`. Note that the string key is the name
of the interface by convention so that it visually looks like we're registering the interface with the actual class name. We will reference this string
key later when we constructor-inject the interface in a component.

## Creating the Component that Injects the Interface as a Service

We will now create a component that injects the interface as a service in its constructor. This component needs to only know that the service *is* an `IPhotosService`
that gets photos, not whether it's precisely the `MockPhotosService` or `PhotosService`.

```javascript
import { Component, OnInit, OnDestroy, Inject } from '@angular/core';
import { Subscription } from 'rxjs';

import { IPhotosService } from '../../services/iphotosservice';

@Component({
  selector: 'app-photos',
  templateUrl: './photos.component.html',
  styleUrls: ['./photos.component.scss']
})
export class PhotosComponent implements OnInit, OnDestroy {

  photos: Array<any> = new Array<any>();

  getPhotosSubscription: Subscription;

  constructor(
    @Inject('IPhotosService') private aPhotosService: IPhotosService
  ) { }

  ngOnInit() {

    this.aPhotosService.getPhotos$().subscribe(photos => {
      this.photos = photos;
    });

  }

  ngOnDestroy() {
    if (this.getPhotosSubscription)
      this.getPhotosSubscription.unsubscribe();
  }
}
```

Note how we inject the interface as a service in the constructor. We want to treat the service as an interface, so we typed it as `IPhotosService`.
Since Angular and/or Typescript want an actual concrete instance for constructor injection, we have the `@Inject('provide key')`, which points to
the actual class instance that is associated with that provide key we have registered in `app.module.ts`.

Here is the template markup:

```html
<h2>Photos</h2>

<div fxLayout="row wrap" fxLayoutAlign="space-between" style="{width: 100%}">
  <div fxFlex="0 1 calc(33% - 10px)" fxLayout="column" *ngFor="let photo of photos">
    <img src="{{ photo.thumbnailUrl }}" />
    <span>{{ photo.title }}</span>
  </div>
</div>

```

## Automatically Choosing Which Service Class to Use Based on Development vs. Production

We have registered the `PhotosService` to the key `IPhotosService` from 'app.module.ts`. When we run the app, the content showed on the page actually 
come from the external web API call. If we swap out `PhotosService` for `MockPhotosService`, we will see the 3 images as we've hardcoded in 
`MockPhotosService`.

In real life, we would probably want to automate the selection of `PhotosService` vs `MockPhotosService` depending on *how* the app is run. Since
we're running the app locally with `ng serve`, we probably would want to choose mock services. However, we would want to select the service that calls
the external API when our app is running when deployed as production.

Angular apps automatically come with the folder `src/environments`. It contains two files, `environment.ts` for development and `environment.prod.ts`
for production. If we look inside each of these files, there's a boolean property called `production`, which is set to false in `environment.ts` and
set to true in `environment.prod.ts`. From anywhere in our app, we can ` import { environment } from '../relative/path/to/../environments/environment';`,
and then access `environment.production`, and write logic according to whatever value it gives us. Angular will automatically give us the values set in 
`environment.ts` when we run our app with `ng serve`, and will automatically give us the values set in `environment.prod.ts` when our app has been built
and deployed to a live server. It turns out that we can add properties that have different values depending on run mode that we may want to access from different
parts of our app. We must be careful, though, that both files MUST declare EXACTLY the same properties!

We use `enviroment` in `app.module.ts` to determine which of `MockPhotosService` and `PhotosService` to use:

```javascript
import { environment } from '../environments/environment';

import { PhotosService } from './services/photos.service';
import { MockPhotosService } from './services/mock-photos.service';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
  ],
  providers: [
    ...
    { provide: 'IPhotosService', useClass: (environment.production) ? PhotosService : MockPhotosService }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

We import `environment`, and we use a ternary operator using `event.production` as a test. If the value of `event.production` is true, we'll
use `PhotosService`, else we'll use `MockPhotosService`.

Registering interfaces and services will be helpful throughout our app, especially if it's growing larger quite rapidly. There are times that a service may
be used by multiple components and other services, so instead of manually updating the constructors of all of those components and services to pick out which
concrete service we want, it's much easier to determine it centrally from `app.module.ts`.