# Angular - Mocking - Injecting Interface Types

As we've said in the previous article, we can only dependency-inject concrete services into constructors. In the end, the runtime *will* need to know *exactly* which concrete
service to resolve for the component (or other service, directive, pipe, etc.) that injects it. Now, out of curiosity, we literally *can* inject an interface type into a
constructor and it would not be considered a development error, since it's still valid Typescript. However, when it's time to create that component that asks for an interface
type, the runtime would not know which service (that implements that interface) to instantiate to ultimately give to that component once it's created.

We will need to go back to `app.module.ts` and make a declaration. This declaration will associate a string, which is an identifier, and a concrete class. In the `providers`
array, we add the following item:

```javascript
...
import { MockPeopleService }  from '../path/to/mock-people.service`;


@NgModule({
    ...
    providers: [
        ...,
        { provide: 'PeopleService', useClass: MockPeopleService }
    ]
})
export class AppModule { }
```

We `provide` the identifier `PeopleService` to the system, and associate it via `useClass` with the concrete class `MockService`, one of the concrete services 
we've created that implements `IPeopleService`. This will come into play shortly. For now, our entire app knows that the string `PeopleService` means *exactly* the
concrete class `MockPeopleService`.

## Injecting Service Interface

The following injection attempt, though acceptable Typescript, will fail:

```javascript
...
import { IPeopleService } from '../path/to/ipeopleservice.ts'

...
export class PeopleComponent implements OnInit {
    
    people: any[];

    constructor(
        private peopleService: IPeopleService
    )

    ngOnInit() {
        this.peopleService.getPeople$().subscribe(people => {
            this.people = people;
        });
    }
}
```

Yes, we've associated the string "PeopleService" with the class `MockPeopleService`, and our app knows it, but when `PeopleComponent` is instantiated, it still *doesn't*
know *that* `IPeopleService` should resolve to `MockPeopleService` as we've declared in `app.module.ts`. What about the identifier we've provided in `app.module.ts`?
Shouldn't it come into play when we constructor-inject an interface type? Yes. We will make the following modifications:


```javascript
...
import { ..., Inject } from '@angular/core';
import { IPeopleService } from '../path/to/ipeopleservice.ts'

...
export class PeopleComponent implements OnInit {
    
    people: any[];

    constructor(
        @Inject('PeopleService') private peopleService: IPeopleService
    )

    ngOnInit() {
        this.peopleService.getPeople$().subscribe(people => {
            this.people = people;
        });
    }
}
```

Ah. With the `@Inject` decorator, we can specify the identifier we've set in `app.module.ts` to tell the system *which* class to *instantiate* during dependency
resolution when the component is created. The catch is, though, is that the class registered in `app.module.ts` *must* implement the interface declared in the constructor.
If not, the above code is still valid Typescript, and will only cause an error at runtime.

This becomes very useful when our app grows large enough that we would want to grab a list of people from multiple components, even other services, etc. Should we decide to
switch the concrete class, we can do it in only one place - in `app.module.ts`, and we're guaranteed that all `IPeopleService` dependencies of all those components,
services, etc., will be an instance of that new class.

## Next Steps

It's still quite annoying, and more importantly, truly erroneous, though, to remember to keep on switching the concrete class name in the `provide-useClass` declaration
in `app.module.ts` when switching between production and dev. How would we automate which class to specify depending on whether the app is running on dev vs. production?
We will need to leverage the little-known `environment` object available from anywhere in our Angular code.