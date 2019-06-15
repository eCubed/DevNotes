# Angular Templating - Fallback Templates

So far, we have worked on letting implementors customize our controllers via templates and the `*ngTemplateOutlet` directive, including giving them the ability
to decide the full markup of the component via layout templates. There is a further step when developing templated controllers, and that is managing the event 
where the user does not supply a template that our component might expect. We will continue to work with our panel template. We will make a few decisions on
what happens when the user does not supply certain templates, and modify our component accordingly.

When we try to run our app and provide absolutely nothing between the tags of our component, as so:

```html
<ec-panel heading="My Custom Heading">  
</ec-panel>
```

Nothing would display. We wouldn't get an error because in our .ts, we null-checked everything before using it, but still, we got nothing to display.

Suppose we supply the `#contentTemplate` only. What would happen? Again, nothing would show. Similarly, if we supplied `#headingTemplate` only, nothing would
show as well. At this point, we know that our component expects a `#layoutTemplate` primarily because it is the contents of the `#layoutTemplate` that prompts
our component to draw our application. So, first, let's deal with the situation where the user does not supply the `#layoutTemplate`.

## Managing the Absence of the Layout Template

If the implementor for some reason does not provide a default layout template, we can either throw an error from our component's `ngOnInit()` function, or be
more gracious by providing a default template. Let us provide the default template: We will only need to make a few modifications to our component's markup:

```html
<ng-container *ngIf="layoutTemplate != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="layoutTemplate">
  </ng-container>
</ng-container>
<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <ng-container *ngTemplateOutlet="headingTemplate; context: { heading: heading, 
      isExpanded: isExpanded, toggleShowOrHide: toggleExpanded } ">
    </ng-container>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </div>
</ng-template>
```

We surround `<ng-container *ngTemplateOutlet="layoutTemplate">` with another `<ng-container>` with an `*ngIf` for when the `#layoutTemplate` was provided.
Remember that we can apply one and only one structural directive per html tag or component! Then, we specify the `#defaultLayoutTemplate` that would show in case
there is no incoming `#layoutTemplate`.

## Managing the Absence of Layout Template plus Absence of Another Template
For now, let us stick with the case where the implementor didn't supply the `#layoutTemplate` since fixing the absence of another template will only require modification
in the markup. Suppose that there was no heading template specified. We would need to supply a `#defaultHeadingTemplate` in a similar fashion we managed the case
where there was no `#layoutTemplate` provided:

```html
<ng-container *ngIf="layoutTemplate != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="layoutTemplate">
  </ng-container>
</ng-container>
<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <ng-container *ngIf="headingTemplate != null; else defaultHeadingTemplate">
      <ng-container *ngTemplateOutlet="headingTemplate; context: { heading: heading, 
        isExpanded: isExpanded, toggleShowOrHide: toggleExpanded } ">
      </ng-container>
    </ng-container>
    <ng-template #defaultHeadingTemplate>
      <h3 (click)="toggleExpanded()">{{ heading }}</h3>
    </ng-template>
    <div *ngIf="isExpanded">
      <ng-container *ngTemplateOutlet="contentTemplate">
      </ng-container>
    </div>
  </div>
</ng-template>
```

We could also do the same in case they did not provide the `#contentTemplate`, which would be a slightly different story than not providing the `#headingTemplate`.
With the `#headingTemplate`, we would display the heading text provided to us via the `@Input() heading: string` property. The `#contentTemplate`, though, is
the major reason why we wanted to create this panel component in the first place, so if it's not provided, then nothing would show up. We have a decision to make
here. Should we through an error from our `ngOnInit()` to crash the app if they didn't provide a `#contentTemplate`? We could, but our decision is to provide
a generic `#defaultContentTemplate` instead that would contain some hard-coded text that would prompt the implementor to supply #contentTemplate:

```html
<ng-container *ngIf="layoutTemplate != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="layoutTemplate">
  </ng-container>
</ng-container>
<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <ng-container *ngIf="headingTemplate != null; else defaultHeadingTemplate">
      <ng-container *ngTemplateOutlet="headingTemplate; context: { heading: heading, 
        isExpanded: isExpanded, toggleShowOrHide: toggleExpanded } ">
      </ng-container>
    </ng-container>
    <ng-template #defaultHeadingTemplate>
      <h3 (click)="toggleExpanded()">{{ heading }}</h3>
    </ng-template>
    <ng-container *ngIf="contentTemplate != null; else defaultContentTemplate">
      <div *ngIf="isExpanded">
        <ng-container *ngTemplateOutlet="contentTemplate">
        </ng-container>
      </div>
    </ng-container>
    <ng-template #defaultContentTemplate>
      You have not supplied a #contentTemplate. Please supply one to replace this message.
    </ng-template>
  </div>
</ng-template>
```

## Managing the Case where Layout Template is Supplied but not the Other Templates Were Omitted

Suppose that the implementor *did* specify the `#layoutTemplate` but *not* either one or both of `#headingTemplate` and `#contentTemplate`. With the way
our code is written, we will get the error `templateRef is not defined`. This is expected because there was no `#headingTemplate`, for example, specified
and we're trying to instantiate using `ViewContainerRef`. Since we coded the instantiation and placement of template content in code, we must also manage
in code. We would need to null-proof the instantiation of a template:

```javascript
  loadTemplates() {
    if (this.headingPlaceholder != null) {
      this.headingPlaceholder.clear();

      if (this.headingTemplate != null) {
        this.headingPlaceholder.append(this.headingTemplate,
          { heading: this.heading,
            toggleShowOrHide: this.toggleExpanded,
            isExpanded: this.isExpanded
          });
      } else {
        // What would we put here if they didn't provide the headingTemplate?
      }
    }
    ...
  }
```

Now, what do we do when the `#headingTemplate` wasn't provided? Recall that we *do* have a template on our markup labeled `#defaultHeadingTemplate`.
We would need to reference that in our code. Instead of declaring it with `@ContentChild`, we will declare it as `@ViewChild` because that 
`<ng-template #defaultHeadingTemplate>` was declared *in-component*, not pulled from an external `<ng-template>` to make its way into our component via
`<ng-container *ngTemplateOutlet="headingTemplate">`. So, let's go ahead and add the `@ViewChild` declaration for `#defaultHeadingTemplate`, and
when the `#headingTemplate` isn't available, we'll instead instantiate `#defaultHeadingTemplate`:

```javascript

  @ViewChild('defaultHeadingTemplate', { read: TemplateRef }) defaultHeadingTemplate: TemplateRef<any>;

  loadTemplates() {
    if (this.headingPlaceholder != null) {
      this.headingPlaceholder.clear();

      if (this.headingTemplate != null) {
        this.headingPlaceholder.append(this.headingTemplate,
          {
            heading: this.heading,
            toggleShowOrHide: this.toggleExpanded,
            isExpanded: this.isExpanded
          });
      } else {
        this.headingPlaceholder.append(this.defaultHeadingTemplate,
          {
            heading: this.heading,
            toggleShowOrHide: this.toggleExpanded,
            isExpanded: this.isExpanded
          });
      }
      ...
    }
```

Alas, if we ran the app, we would get the error `templateRef is not defined` because the `#defaultHeadingTemplate` could not be found. But it's *in* the
markup, so it should find the `#defaultHeadingTemplate`, right? If we look carefully at our markup, `<ng-template #defaultHeadingTemplate>` is placed
*inside* `<ng-template #defaultLayoutTemplate>`, which would only show up if there wasn't a `#layoutTemplate` specified. But we're in the case where
`#layoutTemplate` *was*, specified, and so `<ng-template #defaultLayoutTemplate>` wouldn't be activated, thus Angular will not find
`<ng-template #defaultHeadingTemplate`>, which is why we're getting that error. The "cure" for this, is to go ahead and pull `<ng-template #defaultHeadingTemplate>`
out of `<ng-template #defaultHeadingTemplate>`, and put it at the top-level of our component's markup code. Now, our app would show the default heading.

Remember that the fallback default templates' <ng-template #someDefaultTemplate>` does not have to be placed right next to the `<ng-container>` that
references it from its `*ngIf`'s `else` branch. After all, it's just an `<ng-template>` that would be called upon when needed. Also, the top-level
placing of default ng templates will work for when the `#defaultLayoutTemplate` was specified!

So far, here's the modified markup, and we went ahead and extracted out `<ng-template #defaultContent>` as well:

```html
<ng-container *ngIf="layoutTemplate != null; else defaultLayoutTemplate">
  <ng-container *ngTemplateOutlet="layoutTemplate">
  </ng-container>
</ng-container>
<ng-template #defaultLayoutTemplate>
  <div class="panel">
    <ng-container *ngIf="headingTemplate != null; else defaultHeadingTemplate">
      <ng-container *ngTemplateOutlet="headingTemplate; context: { heading: heading, 
        isExpanded: isExpanded, toggleShowOrHide: toggleExpanded } ">
      </ng-container>
    </ng-container>
    <ng-container *ngIf="contentTemplate != null; else defaultContentTemplate">
      <div *ngIf="isExpanded">
        <ng-container *ngTemplateOutlet="contentTemplate">
        </ng-container>
      </div>
    </ng-container>
  </div>
</ng-template>

<ng-template #defaultHeadingTemplate>
  <h3 (click)="toggleExpanded()">{{ heading }}</h3>
</ng-template>
<ng-template #defaultContentTemplate>
  You have not supplied a #contentTemplate. Please supply one to replace this message.
</ng-template>
```

Here is the updated and refactored `loadTemplates()` function:

```javascript
  loadTemplates() {
    if (this.headingPlaceholder != null) {
      this.headingPlaceholder.clear();
      this.headingPlaceholder.append(this.headingTemplate || this.defaultHeadingTemplate,
        {
          heading: this.heading,
          toggleShowOrHide: this.toggleExpanded,
          isExpanded: this.isExpanded
        });
    }

    if (this.contentPlaceholder != null) {
      this.contentPlaceholder.clear();
      if (this.isExpanded) {
        this.contentPlaceholder.append(this.contentTemplate || this.defaultContentTemplate);
      }
    }
  }
```

Now, we have a component that handles all of the cases of combinations of ommitted and supplied templates!




