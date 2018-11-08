# Angular - Interface Dependency Injection

In Asp.NET Core, we are able to register an interface with a class that implements it into the IOC. Then, we can access the interface from any controller, middleware,
or another class to be registered to the IOC, via constructor injection. This makes it easy to swap out another concrete implementation (class) of the said interface
without modifying the code where that interface is used, which makes development much cleaner and maintainable.

In Angular, we do not, and we actually cannot inject interfaces into component and service constructors just like how we inject classes. The system wants actual classes
to be injected into constructors. However, there is sort of a way around this, which simulates how DI is done in Asp.NET Core, albeit, has more syntax bloat.

## Creating the Common Interface

Throughout our Angular development, we have gotten accustomed to injecting concrete-class services into constructors. If we wanted to change the service, we'll have to
hunt down every single constructor that injects the service, and then change them all. We will no longer be doing this. Instead, we will "inject" interfaces into 
constructors so that every component or other service will treat that parameter as an interface, not the actual class. We will see this in action shortly.

We first need to create a common interface for the services we would want to swap:

```
export interface ITrivialService {
  getMessage(): string;
}
```

We'll keep it short and trivial so we can more easily see the fundamental routine.

## Creating Services that Implement the Common Interface

For demonstration, we'll create two services, `TrivialAService` and `TrivialBService`. We'll only show the code for `TrivialAService` since the other one is nearly
identical.

```
import { Injectable } from '@angular/core';

import { ITrivialService } from './itrivialservice';

@Injectable({
  providedIn: 'root'
})
export class TrivialAService implements ITrivialService {

  constructor() { }

  getMessage() {
    return "Message from Trivial Service A";
  }
}
```

The service will need to implement the common interface, in our case, `ITrivialInterface`. We then implement the `getMessage()` function.

## Registering a Service with its Interface

At `app.module.ts`, we'll need to register the service with its interface. This way, when anywhere in our app calls for the common interface, the service's methods
would be called. We do this in the `providers` array of the `@NgModule` declaration:

```javascript
@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
  ],
  providers: [
    { provide: 'ITrivialService', useClass: TrivialBService }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

We typically put specific service class names in the `providers` array. It can also take an object that has two properties: `provide` for the name of the common
interface, and `useClass` for the class name of the service that implements the common interface.

## Injecting the Interface

We are now at a component that may want to use an `ITrivialService`. We cannot just inject the common interface as a parameter, or else, we will get an error.
At this point, as coded below, Angular and/or Typescript doesn't know that `ITrivialService` really is `TrivialBService` as registered in `app.module.ts`.

```javascript
export class MyComponent implements OnInit {

  message: string;

  constructor(
    private trivialService: ITrivialService
  ) { }

}
```

We'll have to explicitly use the `@Inject` modifier:

```javascript
import { Component, OnInit, Inject } from '@angular/core';

...

constructor(
    @Inject('ITrivialService') private trivialService: ITrivialService
) { }
```

We'll need to specify the exact string we've registered in `app.module.ts`. This `@Inject` references the registration from `app.module.ts`, and it is the actual
mechanism that will pair together the concrete class declared in `useClass` with the interface declaration (`private trivialService: ITrivialService`) right on
the constructor injection.

Now, here is the full listing of the component that "injects" an interface:

```javascript
import { Component, OnInit, Inject } from '@angular/core';

import { ITrivialService } from '../../services/itrivialservice';

@Component({
...
})
export class InterfaceInjectionComponent implements OnInit {

  message: string;

  constructor(
    @Inject('ITrivialService') private trivialService: ITrivialService
  ) { }

  ngOnInit() {
    this.message = this.trivialService.getMessage();
  }
}
```

If we run the app, the value returned from `TrivialBService.getMessage()` will be saved to `message`. If we stop the application and go back to `app.module.ts` and
change the `useClass: TrivialBService` to `useClass: TrivialAService`, the `getMessage()` of `TrivialAService` would be called.

Now, again, this type of injection is very useful if we have many components that would need an `ITrivialService`. We don't need to modify all of those files to
change the concrete service, since that would be very tedious to do. Instead, we swap out the concrete class name in the interface-concrete-service declaration in
one place, `app.module.ts`, and all dependents will automatically acquire the actual functions and members of that new concrete class.


