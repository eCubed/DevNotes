# Reactive Forms - Async Validators

Sometimes, the legitimacy of the data inputted by the user cannot be fully determined by just running some checks on the content of the data at the front. Input values
may be considered properly formed and does not violate any logic according to just what was inputted. However, there are external determinations that would still make
such a validly-formatted input value invalid. A classic example is an email or a username that may follow the rules of what they can look like (number of characters,
required characters, etc.), but it may already be taken by a different user. We would want to show an error to the user in that case, and disallow submitting the form.

We will explore async validators, or validators that will require an API call to further determine the legitimacy of an inputted value.

## Preparing a Service

We do not really need to create a service, and we could just use the built-in http service in an async validator function, but that is bad practice as it encourages
tightly-coupling. Right now, we will create a service that would seem like it were actually making an API call. We will pretend that the service is able to check
against a list of user names already in the database.

```javascript
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';
import { delay, debounceTime } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class MockUsersService {

  constructor() { }

  isUsernameTaken$(username: string) : Observable<boolean> {
    let usernames = ['admin','user1','bill','jane','alex','heather'];

    return of(usernames.includes(username)).pipe(
      delay(1000),
      debounceTime(500)
    );
  }
}
```

There are quite a few things going on here, most of which are `rxjs`-related. As usual, a function of a service should return an `Observable<T>` so that the client
can subscribe to it and get the data from it. Once subscribed, the client will receive a `boolean` value - true if the username is taken, false if not taken.

Since we are mocking an API call, we use the `of` operator which returns an `Observable` of the type of the object we pass into its parameter. We simply use the
`includes()` function of an array to see whether the string (username) is in the list, and we pass that result to `of`.

Now, we would need to modify the observable a bit before letting the client subscribe to it - one is a delay of 1000 ms to simulate calling a remote API, and
debounceTime of 500 ms so that the request won't happen again if the duration between keystrokes is less than 500 ms.

## Writing an Async Validator

We can write an async validator function right onto the component that hosts our reactive form. The function we'll write can to return an
`Observable<{[key: string]: any | null}`, since that's an allowable return type for an async validator function. Inside our function, we call upon the
service:

```javascript
import { Component, OnInit } from '@angular/core';
import { Observable, } from 'rxjs';
import { map } from 'rxjs/operators';

import { FormControl, FormGroup, Validators, AbstractControl } from '@angular/forms';

import { MockUsersService } from './mock-users-service.service';

@Component({
  selector: 'app-custom-validators',
  templateUrl: './custom-validators.component.html',
  styleUrls: ['./custom-validators.component.scss'],
  providers: [ MockUsersService ]
})
export class CustomValidatorsComponent implements OnInit {

  cvForm: FormGroup;

  constructor(private mockUsersService: MockUsersService) { }

  usernameTaken(control: AbstractControl) : Observable<{[key: string]: any} | null> {
    return this.mockUsersService.isUsernameTaken$(control.value)
      .pipe(
        map(isTaken => {
          return (isTaken) ? { 'usernameTaken' : true } : null;
        })
      );
  }

  ...
}
```

At this stage, we are NOT subscribing to the observable returned by our `MockUsersService`'s `isUsernameTaken$` function. We still want to return that very
observable because the reactive form framework will be the one to subscribe to it. However, we want to translate `Observable<bool>` which is being returned by
`MockUsersService`, to `Observable<{[key: string]: any} | null>`, which the reactive forms framework expects for an async validator function. This is where
the `pipe` function of an observable comes in, which is sort of like a series of transformations we'd like to put in a row that would execute. We need to translate
a boolean value to `{[key: string]: any} | null`, which is what we're accomplishing with `map()`.

## Using the Async Validator

Let's first register our new validator to the `FormControl` we need:

```javascript
ngOnInit() {
    this.cvForm = new FormGroup({
      name: new FormControl('',
        Validators.compose([
          Validators.required,
          this.cannotContainIdiot,
          this.cannotContainSubstring('moron')
        ]),
        Validators.composeAsync([
          this.usernameTaken.bind(this)
        ])
      )
    });
  }
```

The `FormControl`'s contructor actually takes in 3 parameters. We've already used the first two, which are initial value, followed by either a regular (synchronous)
validator or an array of validators. The third parameter is for either a single async validator, or an array of async validators. In case we have more than one
async validator in the near future, we went ahead and used `Validators.composeAsync([])`, to which we can put all of our async validators to.

In particular, which looks rather annoying, we will need to `.bind(this)` our validator when registering it to a `FormControl`. Since our in-component async
validation function calls up `MockUsersService` from inside, for some reason, the scope of `this` gets lost when our validation function gets invoked from
somewhere else, which it does as automatically done by the reactive forms framework when needed. Alternatively, we could have written our function to wrap the
function we already have, which has a `MockUsersService` parameter instead. We have done a similar type of construction with regular (synchronous) validators,
and we'll leave that as an exercise. When done this other way, we won't have to write `.bind(this)`.

## Reflecting an Async Validator on the Markup

We detect async validation errors exactly the same way as we do with synchronous validations.

```html
<div>
    <input type="text" formControlName="name" placeholder="User name" />
    <div *ngIf="cvForm.controls['name'].invalid">
        <div *ngIf="cvForm.controls['name'].errors.required">
            First name is required.
        </div>
        ...
        <div *ngIf="cvForm.controls['name'].errors.usernameTaken">
            The username provided has already been taken.
        </div>
    </div>
    <div *ngIf="cvForm.controls['name'].status === 'PENDING'">
        Checking...
    </div>
</div>
```

We also add a container that would show up while the API call is being made by the async validator. It would show up when the `FormControl` in question has a
status of `PENDING`.