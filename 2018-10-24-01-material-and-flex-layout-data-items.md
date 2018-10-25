# Angular Material and Flex Layout - Data Items

In practically any app, we will need to display items of an array, and we won't always want to visually present them one on top of the next in one column. We sometimes
want to present them from left to right, and then wrapping to the next line for subsequent items. We also want each item to be the same width and height, and we want
consistent spacing between each item in all 4 directions. The challenge is, we would want fewer columns the narrower the width is. Sure, writing media queries in CSS
is certainly possible, but it's extremely tedious. We'd have to write a ton of media queries for each page we want to display an array of content. How can Material and
Flex Layout help us with this?

## Preparation

We have created an array of objects on the hosting component. The object has two properties - name (string) and age (number). As appropriate, we will name this array
`people`. We prepopulate (hard-code) this `people` array in `ngOnInit()`. We don't show the code as we want to focus on Flex Layout.

We will display each item using Material's `MatCardModule`, so we will need to register that in our `MaterialModule` module.

We then head over to the template (html markup). We'll start with the following html:

```html
<div>
  <mat-card *ngFor="let person of people">
    <mat-card-title>{{ person.name }}</mat-card-title>
    <mat-card-content>
      Age: {{ person.age }}
    </mat-card-content>
  </mat-card>
</div>
```

There is nothing too fancy about the markup - just the good old `*ngFor` directive and then the `mat-card` tag and its children. We won't go through `mat-card` in
much detail, so if you're curious, check out the documentation. For this demo, we're interested in `mat-card-title` and `mat-card-content`. As a small review,
like any Material component, we are accepting any CSS that comes with `mat-card` and related tags. So we'll get a rounded rectangle with faint border with a little bit
of a shadow. The title will also display in a large font.

If we run the example, we will get a listing of one column that takes up the entire width of the screen, where each item is listed vertically. If we shrink or expand the
browser's width, it will always remain 1 column - too wide for large width, but just right for smaller width. How do we "cure" this problem?

## Angular Flex Layout Directives

For now, let's focus on the wide screen scenario. Let's decide right now that we want to display the data in 4 columns, and give some horizontal space between the cards.
When it's time to display the 5th item, it should wrap to the "next line". 

Let's start with horizontal layout where the last item that cannot fit in the remaining horizontal space goes to the next line, which `fxLayout="row wrap"` does:

```html
<div fxLayout="row wrap">
  <mat-card *ngFor="let person of people">
    <mat-card-title>{{ person.name }}</mat-card-title>
    <mat-card-content>
      Age: {{ person.age }}
    </mat-card-content>
  </mat-card>
</div>
```

If we ran the app now  we won't see the 4 columns. Each box will take up only as much horizontal space as it needs, similar to how letters show up in a book whose text 
isn't justified.

In order to achieve the 4 columns, we'll need to specify the width of each of those `mat-card`s. To do that, we declare `fxFlex="0 1 calc(25% - 10px)"` on the
`mat-card`. In this scenario, we'll take the 0 and the 1 as requirements for our purpose, but we need to focus on the `calc` function. We want the width of the
element plus the 10px gutter to actually equal 25% of the parent's width, which means the width of the element must be 10px less than 25% of the parent's width,
hence calc(25% - 10px).

If we ran the app now, we will indeed get 4 columns, but it seems like we have a huge space to the right-hand side, which is the 40px gutter for each child's 10px gutter.
We'll need to fix this so that the right-most element of every row will fill all the way upto the parent's right. This is tough to pull with CSS alone, but to it's easy
in Flex Layout with the directive `fxLayoutAlign="space-between"`. This means spread out the elements evenly, with the same horizontal space between them so that the
left-most child will butt up against the left edge of the parent, and the right-most child will butt up gainst the right edge of the parent. Since we have already 
calculated the exact width of each element to be 25% of the parent's width minus the 10px gutter, there will be 10px between the children elements.

So far, our html is as follows:

```html
<div fxLayout="row wrap" fxLayoutAlign="space-between">
  <mat-card fxFlex="0 1 calc(25% - 10px)" *ngFor="let person of people">
    <mat-card-title>{{ person.name }}</mat-card-title>
    <mat-card-content>
      Age: {{ person.age }}
    </mat-card-content>
  </mat-card>
</div>
```

If we ran it again, we will get the 4 columns and horizontal spacing we want... if the viewport was wide. If the width is small, it would crunch up 4 columns in such a tight space.
We don't want that. So, when the viewport is anything less than medium, we want to display all the data in 1 column. We do this by applying the directive `fxLayout.lt-md="column"`
on the container element. When we specified `fxLayout="row wrap"`, that means always display the children in rows, then wrap, no matter what the width of the screen is.
We now specify a more specific `fxLayout` directive that would kick in if we wanted to see only 1 column when the viewport's width is anything less than medium (which
would be small and extra-small), specified by `.lt-md`. Now, if we expand and contract our browser's width, we will see that the layout changes around 600px width.

The full listing now should look like this:

```html
<div fxLayout="row wrap" fxLayout.lt-md="column" fxLayoutAlign="space-between">
  <mat-card fxFlex="0 1 calc(25% - 10px)" *ngFor="let person of people">
    <mat-card-title>{{ person.name }}</mat-card-title>
    <mat-card-content>
      Age: {{ person.age }}
    </mat-card-content>
  </mat-card>
</div>
```

We can also specify 3 columns for a medium width, but we'll leave that for an exercise. Hint: add another `fxFlex` on `mat-card` followed by a dot, then a media
query specifier (for example, lt-md which is anything less than medium width - Google those).

## Discussion

As the goal has been, we didn't write a single line of CSS or Typescript to pull this feat. The API has us specify what may look like physical feature attributes like
`fxLayoutGap="10px"` or worse yet, `fxFlex="0 1 calc(25% - 10px)"`, which look like inline CSS or HTML 1 attributes(like BGCOLOR and CELLPADDING). Though glaring
to look at for purists, they don't flood our markup too much, and it definitely is still much less to type than the required equivalent in a stylesheet.

Perhaps one glaring visual from this layout so far is that there is no vertical spacing between the rows. There are two reasons for that. One important thing about
Flex Layout, and thus, CSS FlexBox, is that parent containers are NOT grids. We can only specify that children will be laid out in a row or a column. We only get the
illusion of columns when we specify wrapping and we require all elements to have the same width. The `fxLayoutAlign="space-between"` will only apply to a row layout
or a column layout if we know exactly how tall in pixels the parent container is. This is one of those scenarios where the powers of Material and Flex Layout do not
address. This is where we can apply CSS ourselves to each child item to have a margin-bottom value. We will leave this as an exercise.

Another common thing to do with the parent container is to make it obvious visually that there actually is a physical container that contains the children. This means
that we should give the parent a background color, and some padding. Be careful of the bottom-padding of the parent, though, if we specify margin-bottom to each child!
If that's the case, then margin-bottom should be 0! We'll leave this as an exercise as well.

At least in this article, we have been able to declare right in markup, the widths of elements depending on the viewport's width without any CSS, TypeScript, or any
Angular binding!

## Next Steps

We are now ready to create forms using Angular Material and Flex Layout. Be forewared that Material equivalents of our basic input types are not exactly straight-forward
to put on the markup! Let's move on!