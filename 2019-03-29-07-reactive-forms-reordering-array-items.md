# Reactive Forms - FormArray Item Reordering

We have been working with `FormArray` items and we have achieved being able to add and remove items at runtime. One not-as-common, but definitely important requirement
is to allow the user to change the order of the items in the array. We'll start with giving the `ToDoItemComponent` capabilities to initiate a move up and a move
down request to the parent component.

## Add Capability to Move Up and Move Down an Array Item

We will give the user the capability to move any `FormArray` item one index up or down per click. We'll start with examining the new code in the Typescript first:

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
  @Input() canMoveUp: boolean;
  @Input() canMoveDown: boolean;
  @Output() onRequestDelete: EventEmitter<number> = new EventEmitter<number>();
  @Output() onRequestMoveUp: EventEmitter<number> = new EventEmitter<number>();
  @Output() onRequestMoveDown: EventEmitter<number> = new EventEmitter<number>();

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

  requestMoveUp() {
    if (this.canMoveUp)
      this.onRequestMoveUp.emit(this.index);
  }

  requestMoveDown() {
    if (this.canMoveDown)
      this.onRequestMoveDown.emit(this.index);
  }

}
```

We have added the two `@Input` properties `canMoveUp` and `canMoveDown`. Since the component shouldn't really know anything about its item's position, albeit
we store the index for the workaround for deletion we implemented, it definitely should not know about the size of the array that its item is in. We will let the
parent component calculate whether an individual component can move up or can move down. We will later see this in action when we modify the parent's html. From
inside this component, we would not want to emit the request move up event if `canMoveUp` is false, and likewise for `canMoveDown`. We will also use `canMoveUp`
and `canMoveDown` to visually alter the look of the move up/move down icons.

We have two `@Output` events for when the user wants to move up or move down an item by one index position.

We now proceed to the item's markup:

```html
<div[formGroup]="toDoItem">
 <input type="checkbox" formControlName="done">
  <div>
    <input type="text" formControlName="name" placeholder="name" />
    <div *ngIf="toDoItem.controls['name'].invalid">
      <div *ngIf="toDoItem.controls['name'].errors.required">
        Name of to do Item is Required
      </div>
    </div>
  </div>
  <button (click)="requestMoveUp()" [class.disabled]="!canMoveUp">Move Up</button>
  <button (click)="requestMoveDown()" [class.disabled]="!canMoveDown">Move Down</button>
  <button (click)="requestDelete()">Delete</buutton>
</div>
```

We have added buttons to allow the user to initiate moving items up or down on the array that contains them. In particular, we perform some Angular class
binding, for example, `[class.disabled]="!canMoveUp` to apply the CSS class name "disabled" when the item isn't supposed to be able to move up (because its position
in the array is 0). The logic is similar for applying the CSS class "disabled" for the move down item when it's not supposed to be able to move down (because it's
at the last position in the array that contains it).

## Perform Move Up and Move Down

The parent component will need to be the one to physically move items up and down the list because it knows about the array that contains the items. So, the parent
component will need to listen for move up and move down requests from its children. 

First, we present the markup that calculates `canMoveUp` and `canMoveDown` for each item, as well as handle the move up and move down requests:

```html
<form>
    <div>
        <input type="text" formControlName="name" placeholder="name" />
        <div>
            <input type="text" #newToDoItem placeholder="Enter new to-do item" />
            <button (click)="addNewToDoItem(newToDoItem.value)">Add</button>
        </div>
    </div>
    <div>
        <h3>Items</h3>
        <app-to-do-item
            [toDoItem]="toDoItem"
            [index]="i"
            [canMoveUp]="i > 0"
            [canMoveDown]="i < toDoForm.controls['items'].controls.length - 1"
            (onRequestDelete)="onRequestDeleteHandler($event)"
            (onRequestMoveUp)="onRequestMoveUpHandler($event)"
            (onRequestMoveDown)="onRequestMoveDownHandler($event)"
            *ngFor="let toDoItem of toDoForm.controls['items'].controls, let i = index">
        </app-to-do-item>
    </div>
    <div>
        <pre>{{ toDoForm.value | json }}</pre>
    </div>
</form>

```
An item can move up only if its position is greater than 0, and can only move down if its position is the second-to-the-last or less in the array it's in.

We then examine what goes on in the move up and move down handlers:

```javascript
...
export class ToDoEditorComponent implements OnInit {

  toDoForm: FormGroup;

  ...

  onRequestMoveUpHandler(oldIndex: number) {
    const itemsArray = this.toDoForm.controls['items'] as FormArray;

    const itemToMove = itemsArray.at(oldIndex);
    itemsArray.removeAt(oldIndex);
    itemsArray.insert(oldIndex - 1, itemToMove);
  }

  onRequestMoveDownHandler(oldIndex: number) {
    const itemsArray = this.toDoForm.controls['items'] as FormArray;

    const itemToMove = itemsArray.at(oldIndex);
    itemsArray.removeAt(oldIndex);
    itemsArray.insert(oldIndex + 1, itemToMove);
  }
}
```

Since we set up `canMoveUp` and `canMoveDown` in the markup, and we know that the child component won't emit a request to move an item based on the calculated
and then passed-in values for `canMoveUp` and `canMoveDown`, we do not perform range checks in the handler functions. For the move up handler, we first get a hold
of the `FormArray` in question. We then obtain the `FormGroup` at the old index from where the item will move. We then remove the item at the old index, and then
insert it at the index before the old index (`oldIndex - 1`). The procedure is similiar for the move down scenario, except we insert the item at the index after
the old index.

If we run the app now, we will see that moving an item up and down will reflect on the UI and more importantly, on the final model.