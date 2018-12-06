# Angular - Mocking - the Service Contract

At first, we utilized the HttpClient service straight-up in our components because that's what tutorials told us to do because the main point was to get data from outside
the app and display it on the screen *as quickly as possible*. Later, we learned about services, so we moved all of our http-calling code from our component to the services.
This cleaned up our components a great deal. Now, the component is only responsible for receiving and displaying data while the service is responsible for obtaining it.
Moving the data access code to the service was also beneficial especially when multiple components would need to make the same API calls. Without services, we would have
had to write the same http-calling code in many places. With services, all we need to do is to constructor-inject them to components that need them. Then components can
subscribe to receive the data.

There is a step above services, or actually *concrete* services as we've created so far. We will need to create an abstraction of a given service, and we'll call this
abstraction the **service contract**. In code, a service contract is literally an *interface* containing methods that our service would need to implement. Why would we need
to create an interface for a single service to implement, and why not just stick with the concrete service in the first place? Because we will be creating *more than one*
concrete service that implements the same service contract - one is the http-calling service, and the other one is a mock service. 

## Abstracting a Current Concrete Service

For simplicity, we have the following concrete service with only one function - to get a list of people:

```javascript
...
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

...
export class PeopleService {
    
    constructor(private http:HttpClient) {
    }

    getPeople$(): Observable<any> {
        return this.http.get<any>('http://mystuff.com/api/people');
    }
}
```

Let's abstract this and create an *interface* called `IPeopleService`. We are choosing the C# convention of prefixing the interface name with a capital I so that when we
code later, we'll know that we're working with an interface, not an exact specific type:

```javascript
import { Observable } from 'rxjs';

export interface IPeopleService {
    
    getPeople$(): Observable<any>;
}
```

Now, we modifiy `PeopleService` to implement `IPeopleService`:

```javascript
...
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { IPeopleService } from './ipeopleservice';

...
export class PeopleService implements IPeopleService {
    
    constructor(private http:HttpClient) {
    }

    getPeople$(): Observable<any> {
        return this.http.get<any>('http://mystuff.com/api/people').pipe(
           map(response => response.data);
        );
    }
}
```

If we run this code now, and kept our components who inject PeopleService untouched, the app would run as expected. So, why abstract again? Because we'll implement `IPeopleService`
with a mock service.

## Creating the Mock Service

We'll create a mock service that implements `IPeopleService`. The biggest difference between the mock service and the "regular" service is that the mock service does NOT make
http calls.

```javascript
...
import { Observable, of } from 'rxjs';
import { delay } from 'rxjs/operators';
import { IPeopleService } from './ipeopleservice';

...
export class MockPeopleService implements IPeopleService {
    
    people = [
        {
            'firstName': 'John',
            'lastName': 'Smith'
        },
        ...
    ];
    
    constructor() {
    }

    getPeople$(): Observable<any> {
        return of(people).pipe(
            delay(1000)
        );
    }
}
```

We use the `rxjs` `of` operator to wrap an observable around our hard-coded mock data so we can return an observable for our people array. We use the `delay` operator on the
observable's `pipe` function to simulate some delay that occurs naturally on a live http call. Notice that no http calls are made in a mock service.

Let's pause for a moment and focus on what the external server returns from an http call vs. what we actually return in both the mock service and http-calling service. If we
look at `PeopleService.getPeople$()`, we see that we use the `map()` function to "convert" what the external server returns to exactly *just* the data we want for our
components to receive. For all we know, the server could return a bunch of other unneeded data for our purposes, so we pick out only the part that we want, in our case, the
response JSON's `data` which is the array of people.

When creating the mock data for the mock service, we *need to know* beforehand the objects of interest that the server would actually return. In a team environment, this should
have been discussed by the project manager and all developers involved, both front-end and back-end developers. This way, the front-end developer can start developing the app
with mock data that reflect the objects of interest that the real server would eventually return. At the same time, the back-end developer can work on the web application that
returns exactly the data structures of the objects of interest. Once the back-end is completed, the front-end developer needs to simply production-deploy the app and when it
runs live, it will make the http calls and expect the same objects as hard-coded in the mock data.

## Next Steps

So we now have a service interface and two concrete services that implement it. With the two concrete services we have, how do we inject them into our components? It is a fact
that anything that we constructor-inject *must* be of a concrete type, so we could pick the mock or http-calling service explicitly. But we're stuck with one for both dev and
production deployment, though, and we can specify *ever* only one. We could try to inject the service interface, but we would get an error which will tell us that a concrete
type is required for injection. What would we need to do? We'll discuss how to resolve the interface vs. concrete type requirement for DI in the next article.