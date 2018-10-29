# Angular Material and Reactive Forms - Deleting Array Items

In the previous article, we have achieved adding array items of complex types, and we saw that the addition of an item is reflected in the eventual model. Now, we
need to work on allowing the user to delete an item. We will need to perform some workarounds in making item deletion work. Let us be reminded that we are not deleting
a POJO from an array - we're deleting a form group.

## Provide a Means for Deletion from the Item Partial Reactive Form Component

We will need to provide the `ToDoItem` component with the capability to let the rest of our app know that it wishes to be deleted. We will need to put a means for
the user to initiate the deleting of an item, and this is usually with an icon. We will then need to hook up a function to that icon's click event, and then we can
proceed in Typescript.

We have the markup with the delete icon:

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
  <span (click)="requestDelete()"><mat-icon>close</mat-icon></span>
</div>
```

We then proceed to the Typescript and set up an `EventEmitter`:

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
  @Input() index: number;
  @Output() onRequestDelete: EventEmitter<number> = new EventEmitter<number>();

  constructor() { }

  ngOnInit() {
  }

  public static createToDoItem(name: string) : FormGroup {
    return new FormGroup({
      name: new FormControl(name, Validators.required),
      done: new FormControl(false)
    });
  }

  requestDelete() {
    this.onRequestDelete.emit(this.index);
  }
}
```
The workaround starts here with the `@Input() index: number.` We call it a *workaround* because typically, any item component in any framework should not know or
should care where its item's position is in the array that contains its data. With Reactive Forms at this moment, it will be necessary for the array item partial
reactive form component to store the array item's index, or position in the array it's in, since the index is *the* only way that we can properly identify an array
item to a `FormArray`. Thus, we set the parameter of the `EventEmitter` to a number, which will be the index of the array item in the `FormArray` it's in.
We will more clearly see this when we return to the parent component to handle the request delete event.

## Deleting an Item from a FormArray from the Parent Reactive Component

We have to make some modifications to the listing of each `ToDoItem` in the parent reactive component's markup - input the index, and hook up `(onRequestDelete)`
to a function that will actually perform the deletion:

```html
<div>
      <h3>Items</h3>
      <app-to-do-item
        [toDoItem]="toDoItem"
        [index]="i"
        (onRequestDelete)="onRequestDeleteHandler($event)"
        *ngFor="let toDoItem of toDoForm.controls['items'].controls, let i = index">
      </app-to-do-item>
    </div>
```

We added the `let i = index` to the `*ngFor` directive, which is a way for us to access the index of the item in its containing array, then we pass it to the
child component's `[index]` `@Input` field. We then need to write the function `onRequestDeleteHandler($event)` where `$event` is actually the index of
the item the user clicks.

```javascript
onRequestDeleteHandler(index: number) {
    let itemsArray = this.toDoForm.controls['items'] as FormArray;    
    itemsArray.removeAt(index);
  }
```

We'll need to first get a hold of the `FormArray` from the root `FormGroup`. `FormArray` has the `removeAt` function to which we'l pass in the index of the
item that the user intends to delete.

When we run the application, we will see that we can delete items by clicking on an item's delete icon, and it will be reflected in our eventual model.

At this time, `FormArray` doesn't expose a indexOf(obj)` function that a normal array does. If it did, there would not have been a need to store the index to
the child component. We would typically pass the entire object, in our case, the `FormGroup` itself to the `EventEmitter`'s `.emit` function. Then in
the request delete event handler, we would have used `indexOf` to obtain the index, and then call `.removeAt`. But for now, `FormArray` doesn't have
an `indexOf` function, which is why we have to perform that `@Input index: number` workaround.

## Next Steps

Another possible requirement that we want the user to be able to do is to move items up and down the list, one index at a time. This may be a good exercise to
pursue, but we will need to disable move up or move down depending on their position in the array and how many items are in that array.

Another thing we haven't done is validation against the array itself in terms of required item count. Another validation task might need to peer inside each items'
properties to make sure their values meet certain criteria. For example, for our to-do list, we could establish a rule that says no two items have the same name, or
for some reason, we'll require that there are at least 3 items marked done.