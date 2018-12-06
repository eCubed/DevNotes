# Angular - Mocking - Environment Involvement

So far, we've been able to successfully implement interface dependency injection with service contracts and concrete class registration. Now, we will need to determine which
concrete class to instantiate based on whether the app is running in development or production. We will need to utilize a lesser-known object called `environment`, which
can be accessed from anywhere in our Angular code.

## The `environment` Object

If we look in our application's working directory, we will find a folder at `/src/environments`, and in that folder, we will find two files - `environment.ts` and
`environment.prod.ts`. If we look at the contents of both, we will see that they are declared the same variable name `environment`, and they have a property called
`production`, which is a boolean. In the `environment.prod.ts`, that value is true and it is false in `environment.ts`. These files were created by the CLI when
we first created the new app, and yes, we can modify them, but we must do so very carefully - both *require* the same exact properties declared, or else, we'll get a runtime
error.

## Accessing the `environment` Object

And yes, we can access this `environment` object from any Angular .ts file, and we can probe its `.production` value to tell whether it's running in production or dev.

So, we can write the following in `app.module.ts`:

```javascript
...
import { MockPeopleService }  from '../path/to/mock-people.service`;
import { PeopleService } from '../path/to/people.service`;

import { environment } from '../environments/environment';


@NgModule({
    ...
    providers: [
        ...,
        { provide: 'PeopleService', useClass: (environment.production) ? PeopleService : MockPeopleService }
    ]
})
export class AppModule { }
```

Now, we call up the environment variable and see whether its production property is true, and if it is, register the http-calling `PeopleService`. If it's not true, then
register `MockPeopleService`. When we ran `ng serve` to run our application, Angular knows to apply the `enviroment.ts` file to our running applications. When we
deploy to production with `ng build`, Angular will apply the values in the `environment.prod.ts` instead, so when it's deployed, `environment.production` will be
set to true, thus, in our declaration above, `PeopleService` would be registered.

We can call it done right here if *ever* all we need to do is determine whether we're in production or dev to determine which concrete class to register. However, as our app
grows even larger, it is certainly possible to have more than just the dev and production environments, and it certainly is possible that we have 3 or more concrete services
that we would want to apply depending on the environment we're in. It would be very useful to directly associate an environment with a concrete class so that
we won't have to write that ternary expression in `app.module.ts`. In order to do that, we'll need to abstract the `environment` object with the interface `IEnvironment`.

## Abstracting the `environment` Object

It's a good idea to create an interface for our environment anyway to help prevent missing or extra properties between the two or more environment files. We create the interface
`IEnvironment` to mitigate that problem and to be able to associate a concrete class (that implements an interface) with an actual environment object.

```javascript
import { Type } from '@angular/core';
import { IPeopleService } from '../app/services/ipeopleservice.ts';

export interface IEnvironment {
    production: boolean;
    peopleServiceClass: Type<IPeopleService>;
}
```

It is perfectly fine, and we should for organizational purposes, to place this file in that same environments folder.

Of particular interest, the `peopleServiceClass` has a type of `Type<IPeopleService>`. This is a way to declare a property that should hold a type (eventually, class name)
and not an actual value. We will shortly see how this works when we "implement" `IEnvironment`.

Now, we rewrite both environment files to implement. First, the `environment.ts` for dev:

```javascript
import { IEnvironment } from './ienvironment';
import { MockPeopleService } from '../app/services/mock-people.service';

export const environment: IEnvironment = {
  production: false,
  peopleServiceClass: MockPeopleService
};
```

Now, we've declared `environment` to be of the `IEnvironment` interface, so it now has to have exactly the properties required by `IEnvironment`. Notice that for
`.peopleServiceClass`, we've set the `MockPeopleService` class, which is acceptable because per `IEnvironment` having the type `Type<IPeopleService>`, `MockPeopleService`
*is* of type `IPeopleService`. Now, we make similar modifications to `environment.prod.ts` for production:

```javascript
import { IEnvironment } from './ienvironment';
import { MockPeopleService } from '../app/services/people.service';

export const environment: IEnvironment = {
  production: false,
  peopleServiceClass: PeopleService
};
```

We can now get rid of the ternary expression in `app.module.ts`:

```javascript
...
import { MockPeopleService }  from '../path/to/mock-people.service`;
import { PeopleService } from '../path/to/people.service`;

import { environment } from '../environments/environment';


@NgModule({
    ...
    providers: [
        ...,
        { provide: 'PeopleService', useClass: environment.peopleServiceClass }
    ]
})
export class AppModule { }
```

That *is* much cleaner, and will prepare us in case a new environment will need to be associated with a different concrete class. Depending on which environment is running
the app, the appropriate class will be associated with the identifier "PeopleService", and will apply to every dependency injection across the app.

## Next Steps

How about creating a third and further subsequent environments? It's easy to create a new file called `environment.myenv.ts`, "implement" the common interface the other two
environment files "implement", and then supply different values for each property. However, we will get an error saying that the environment `myenv` is not found. It turns out
that merely creating a new environment file is not enough. There are additional things we'll need to do, and that is a topic beyond the scope of this series.