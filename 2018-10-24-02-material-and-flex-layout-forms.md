# Angular Material and Flex Layout - Laying out Forms

We have some experience laying out a website's navigation and presenting data items using Material and Flex Layout. We will now apply our knowledge and learn
more in order to properly lay out a form.

We will use only text boxes and text areas for simplicity since the other widgets are quite complex and may require Typescript code. We will focus on how to properly 
layout a basic form. We will not focus on data-binding or form verification. We will accept the default look of each element.

## Laying out the Basic Form Before Applying Flex Layout

Our form will contain 5 inputs on two groups. The first group will contain 3 text boxes for full name, email address, and "how did you find us"? The second group
contains two text areas, for "say hi" and suggestions. If not already done, we will need to import `MatFormFieldModule` and `MatInputModule` to our `MaterialModule`.

```html
<form>
  <div>
    <div>
      <mat-form-field>
        <input matInput placeholder="Full Name" />
      </mat-form-field>
      <mat-form-field>
        <input matInput placeholder="Email Address" />
      </mat-form-field>
      <mat-form-field>
        <input matInput placeholder="How did you find us?" />
      </mat-form-field>
    </div>
    <div>
      <mat-form-field>
        <textarea matInput placeholder="Say Hi"></textarea>
      </mat-form-field>
      <mat-form-field>
        <textarea matInput placeholder="Suggestions"></textarea>
      </mat-form-field>
    </div>
  </div>
</form>
```

We have a wrapper div to wrap both groups. Group 1 has the three text fields and Group 2 has the text areas. Note that we surround the normal input elements and text
areas with `<mat-form-field>`. `<mat-form-field>` is a special tag that may also contain other Material-specific tags for error messages, labels, etc., which we'll 
not address at this time. We include it now for good practice and preparation for the future when we address form validation. Since we surround our elements with
`<mat-form-field>` and apply the `matInput` directive on the basic elements, we will be accepting the basic Material look and behavior.

When we run the application, the two sections are stacked vertically, and within each section, the elements have widths that are automatically calculated or set, and 
they are laid out horizontally. We will need to fix some layout issues using Flex Layout.

## Applying Flex Layout

For now, we will apply some Flex Layout directives only considering the wide screen scenario.

The first fix we'll make is that we would want to place both sections side-by-side, each taking up 50% of the parent width. In order to do that, we will need to set
`fxLayout="row"` on the parent div, and `fxFlex="50%" on the two sections.

```html
<form>
  <div fxLayout="row">
    <div fxFlex="50%">
      <mat-form-field>
        <input matInput placeholder="Full Name" />
      </mat-form-field>
      <mat-form-field>
        <input matInput placeholder="Email Address" />
      </mat-form-field>
      <mat-form-field>
        <input matInput placeholder="How did you find us?" />
      </mat-form-field>
    </div>
    <div fxFlex="50%">
      <mat-form-field>
        <textarea matInput placeholder="Say Hi"></textarea>
      </mat-form-field>
      <mat-form-field>
        <textarea matInput placeholder="Suggestions"></textarea>
      </mat-form-field>
    </div>
  </div>
</form>
```

Let's focus on the first section. Right now, the 3 text boxes don't fill their parent's width all the way. To do that, we will need to regard that parent div as a
parent to those `<mat-form-field>`s. We'll need to apply `fxLayout="row"` to it, and then set `fxFlex="33.3%" to each of the `<mat-form-field>`s.

For the second section, we want to display those text areas vertically, and fill the width of their parent. We'll need to apply `fxLayout="column"` to the parent, and
by doing this, those text areas will automatically fill the width of the parent.

So far, we have the following html:

```html
<form>
  <div fxLayout="row">
    <div fxLayout="row" fxFlex="50%">
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="Full Name" />
      </mat-form-field>
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="Email Address" />
      </mat-form-field>
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="How did you find us?" />
      </mat-form-field>
    </div>
    <div fxLayout="column" fxFlex="50%">
      <mat-form-field>
        <textarea matInput placeholder="Say Hi"></textarea>
      </mat-form-field>
      <mat-form-field>
        <textarea matInput placeholder="Suggestions"></textarea>
      </mat-form-field>
    </div>
  </div>
</form>
```

If we try to view right now, we will see that everything we wanted so far works for wide screens, but if we decreased the width of the browser, it would be very cramped.
The 50% width for each screen looks too crowded. It would be appropriate that on screens smaller than medium, each of those elements should appear vertically. First,
the two sections need to be displayed vertically. Next, each of those input fields in the first group should be displayed vertically as well. So, `fxLayout.lt-md="column"
would need to be set on the parent-most div, and the first group's parent div.

Our html should now consider narrower widths:

```html
<form>
  <div fxLayout="row" fxLayout.lt-md="column">
    <div fxLayout="row" fxLayout.lt-md="column" fxFlex="50%">
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="Full Name" />
      </mat-form-field>
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="Email Address" />
      </mat-form-field>
      <mat-form-field fxFlex="33.3%">
        <input matInput placeholder="How did you find us?" />
      </mat-form-field>
    </div>
    <div fxLayout="column" fxFlex="50%">
      <mat-form-field>
        <textarea matInput placeholder="Say Hi"></textarea>
      </mat-form-field>
      <mat-form-field>
        <textarea matInput placeholder="Suggestions"></textarea>
      </mat-form-field>
    </div>
  </div>
</form>
```
If we decrease the width of our browser, we should now see a 1-column vertical layout with all 5 input controls one on top of each other.

## Discussion

Just those few directives on the markup gave us the responsiveness we needed. Again, to highlight how useful Flex Layout is, we did NOT need to write a single line of
CSS to accomplish responsiveness!

There are a few things that we could improve upon this form. Notice that the three text boxes touch with no space between them. This could be fixed with applying
`fxLayoutAlign="space-between"` to their parent div, and then modifying the `fxFlex` of the `<mat-form-field>`s to `fxFlex="0 1 calc(33.3% - 10px)`.

We could also add background colors and padding to the divs, but those are outside the responsibility of Angular Material and Flex Layout. We would need to write our
own CSS for those kinds of modifications, and if we need to, we need to make sure NEVER to include any FlexBox or positioning/sizing rules, or any media querying!
The custom CSS for our Flex Layout containers and children must solely be for decoration, not for laying out!

## Next Steps
We have seen how powerful and easy it is to use Flex Layout. The Flex Layout declarations we've used so far cover a large portion of layout needs, so we will not delve
into more advanced Flex Layout for a while. However, Angular Material has some complicated widgets that require more than just plopping down on the markup. There's also
form validation, or per-element validation that we will need to pair up with Angular Material. We'll cover those in the next articles. Those next articles will include
Flex Layout for laying out nicely, but will not discuss them.