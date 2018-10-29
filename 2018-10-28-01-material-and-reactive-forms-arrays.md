# Angular Material and Reactive Forms - Arrays of Complex Objects

We have previously managed to nest reactive forms when our end model had a complex typed property, and we created a partial form to layout and manage the
data from that property. We also saw that the partial reactive form's validation was considered at the top level.

Today, we will explore the situation when one of our end model's properties is an array of complex objects. We will be working with the classic ToDo object,
which, in our app, will have a property for `name`, and an array of ToDoItem objects (`items`), each containing `name` and a boolean value `done`. When we are
finished, we will be able to add items to and remove items from the  `items` list.

Let us remind ourselves that in working with reactive forms, we are working with form objects, not the actual model!

## The Partial Form Component for an Array Item

Let us start by first creating the partial reactive form for a `ToDoItem`.

```html
<div fxLayout="row" fxLayoutAlign="space-between center" [formGroup]="toDoItem">
  <mat-checkbox formControlName="done"></mat-checkbox>
  <mat-form-field fxFlex="0 1 calc(50% - 10px)">
    <input matInput type="text" formControlName="name" placeholder="name" />
    <mat-error *ngIf="toDoItem.controls['name'].invalid">
      <mat-error *ngIf="toDoItem.controls['name'].errors.required">
        Name of to do Item is Required
      </mat-error>
    </mat-error>
  </mat-form-field>
  <span><mat-icon>close</mat-icon></span>
</div>
```

We'll treat the top-level div as if it were a top-level form, by setting `[formGroup]` to `toDoItem`, which we'll later see is a `FormGroup`. We have
a `mat-checkbox` that we'll associate with the `done` `FormControl`, and we have the textbox to associate with the `name` `FormControl`. We went ahead
and wrote markup for error messages. We also have an icon (an x) for requesting to delete the item, and we'll address this a bit later.

We have the Typescript code for `ToDoItem`:

```javascript
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';

import { FormGroup, FormControl, Validators } from '@angular/forms';

@Component({
  selector: 'app-to-do-item',
  templateUrl: './to-do-item.component.html',
  styleUrls: ['./to-do-item.component.scss']
})
export class ToDoItemComponent implements OnInit {

  @Input() toDoItem: FormGroup;

  constructor() { }

  ngOnInit() {
  }

  public static createToDoItem(name: string) : FormGroup {
    return new FormGroup({
      name: new FormControl(name, Validators.required),
      done: new FormControl(false)
    });
  }
}
```

`ToDoItemComponent` is a partial reactive form component, and like the partial reactive form component intended to manage a single object, this component
also has an `@Input` of the type `FormGroup`. There are a few differences in this component, though, since it's intended to be an array item, which is
subject to be created and added to an array at runtime by the user. We want to keep the `FormGroup` logic of the `ToDoItem` with the `ToDoItemComponent`
for better organization - we wouldn't want the parent component to have to know about the fields of its complex properties, let alone the validations for
them. This is why we create a static function on the `ToDoItemComponent` that *knows* the form information of the array item, including validation, and then
returns a new instance of a `FormGroup`. This static function `createToDoItem()` takes in a string, which is the name of the new `ToDoItem`, and is called
from the parent component.

## Setting Up a Property of an Array of a Complex Type on the Parent Reactive Form

We have the `ToDoEditorComponent`, which is the top-level reactive form component. We'll first present the Typescript because the markup will only make sense
when we know what's going on in the Typescript.

```javascript
import { Component, OnInit } from '@angular/core';

import { FormGroup, FormControl, FormArray } from '@angular/forms';

import { ToDoItemComponent } from './to-do-item/to-do-item.component';

@Component({
  selector: 'app-to-do-editor',
  templateUrl: './to-do-editor.component.html',
  styleUrls: ['./to-do-editor.component.scss']
})
export class ToDoEditorComponent implements OnInit {

  toDoForm: FormGroup;

  constructor() { }

  ngOnInit() {
    this.toDoForm = new FormGroup({
      name: new FormControl(''),
      items: new FormArray([])
    });
  }

  addNewToDoItem(toDoItemName: string) {
    let itemsArray = this.toDoForm.controls['items'] as FormArray;
    itemsArray.push(ToDoItemComponent.createToDoItem(toDoItemName));
  }
}
```

Let's focus on creating the top-level `FormGroup` first. We see that we specify a `FormControl` for the `name`, which we've seen before. Now, we declare
`items`, which is an instance of a `FormArray`. This is how the form will know that we intend to have an array of items. Notice that `FormArray`'s
constructor's first parameter is an array object - specifically, an array of `FormGroups`. Since we have no starting data, we pass in an empty array.
`FormArray`'s constructor does also take in validators the same way as `FormGroup` and `FormControl` do. For now, we won't validate the `items` array.

We will want our editor to be able to add a new item from the UI, so we included `addNewToDoItem()` to perform just that. In this function, we're getting a
hold of the items array by accessing `this.toDoForm.controls['items'] as FormArray`. We specify `as FormArray` because a `FormGroup'`s controls
collection is of an abstract type that `FormArray`, `FormControl`, and `FormGroup` derive from - this polymorphism is what allows us to nest forms. We
specify `as FormArray` because we know that `items` *is* a `FormArray`, and we'd like to treat it that way. Then, we push an item into `itemsArray`.
Notice that we use the `ToDoItemComponent`'s static `createToDoItem()` function to create an instance of a `toDoItem` `FormGroup`. Again, we didn't want
this top-level parent component to know the details of the partial form, which is why we're using a static function on `ToDoItemComponent`. This
`addNewToDoItem` will be called from a button click in the markup.

Now, we have the markup:

```html
<form [formGroup]="toDoForm" fxLayout="row" fxLayout.lt-md="column" fxLayoutAlign="space-between">

  <div fxLayout="column" fxLayoutAlign="space-between" fxFlex="0 1 calc(70%-10px)">
    <mat-form-field>
      <input matInput type="text" formControlName="name" placeholder="name" />
    </mat-form-field>
    <div fxLayout="row" fxLayout.lt-md="column" fxLayoutAlign="space-between">
      <mat-form-field>
        <input matInput type="text" #newToDoItem placeholder="Enter new to-do item" />
      </mat-form-field>
      <button mat-button-raised (click)="addNewToDoItem(newToDoItem.value)">Add</button>
    </div>
    <div>
      <h3>Items</h3>
      <app-to-do-item
        [toDoItem]="toDoItem"
        *ngFor="let toDoItem of toDoForm.controls['items'].controls">
      </app-to-do-item>
    </div>
  </div>
  <div fxFlex="0 1 calc(30%-10px)">
    <pre>{{ toDoForm.value | json }}</pre>
  </div>
</form>
```

Let's focus first on the second `mat-form-field` which contains the UI that allows the user to add a to-do item. The input element here is *not* part
of the `toDoForm` `FormGroup`, so we give it an id `#newToDoItem` so we can access this text box's value from the button click. The button below
this textbox has a `(click)` event that is hooked up to the `addNewToDoItem(string)` function, and passes the value of the textbox marked with
`#newToDoItem`. We use the # type of element identification so that we won't have to deal with models and binding in the Typescript. After all, this is
reactive forms, not template-driven forms.

Let's examine now the Items markup. We've created the `ToDoItemComponent` whose tag is `app-to-do-item`. Since we know that `app-to-do-item` is an
array item, we will need to iterate using the `*ngFor` directive as we have before. However, we are iterating over the controls of `toDoForm.controls['items'] `.
Since this partial component must take in an `@Input` of type `FormGroup`, we pass in a `toDoItem`, which itself is a `FormGroup`.

If we run the app now, we'll notice that we get to add new items to the `items` list, and we can see that it is reflected in our model. When we also check
and uncheck the `done` checkbox, we also see that reflected on the correct item in the `items` list. Editing the name also reflects on the final model.
Validation of each object is also heard from the very top.

## Next Steps

We will most likely want to be able to remove items from the `items` list. This is no easy task to pull with reactive forms, and there are necessary workarounds
that need to be implemented, so we'll dedicate the next article to be able to perform item deletion of an item in a `FormArray`.


