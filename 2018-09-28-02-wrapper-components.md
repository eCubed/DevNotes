# Wrapper Components

I have always created components where their templates are "as-is". This means that the html of the components are set, and any changes seen on screen are a result
of changing values of that component's very properties. All I need to do is plop the component's selector name onto a hosting component's template. I've never
created a component whose template may have a space reserved for any content. For example, I would like to be able to do this:

```html
<my-wrapper-component [someProperty]="someValue">
  <div>
     Anything I want
  </div>
</my-wrapper-component>
```

I don't think the above is possible without doing something special.

## Motivation for Wrapper Components

In web applications, I typically like to group content in a panel with a header. That content can be anything from plain text, only html elements, or a combination
including even custom-made Angular components.

It is certainly possible to identify all the groupings, and for each of them across an entire application, I would create components with header and panel markup
to explicitly contain them. But that is tedious. All of them have a common html markup and they only differ by what's in the content. Also, creating a component for
the sole purpose of being able to group content feels like overkill.

It is also possible to explicitly wrap content and components with a div with a header sibling, and both of those would have a parent div that makes up the entire
panel. That, too, is very tedious. There has got to be a better way than two too unmaintainable (html-crowded and files-crowded) plans.

## Transclusion

That title seems like a very tricky term. In the most practical sense, all it means is to replace some placeholder with some content, which sounds a lot like
cut-paste-replace. There is a way to create such a reserved space for any content in an Angular 2+ component, and that transclusion mechanism is exactly the
`<ng-content>`.

I'll create a new component called `EmpanelComponent`, whose job is to provide the html markup for the panel and to host any content we want!

The .ts file:

```javascript
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'lib-empanel',
  templateUrl: './empanel.component.html',
  styleUrls: ['./empanel.component.scss']
})
export class EmpanelComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

The template .html:

```html
<div class="panel">
  <h3>My Panel</h3>
  <div>
    <ng-content></ng-content>
  </div>
</div>
```

The scss so we can better see the effect on the screen:

```css
.panel {
  & > h3 {
    box-sizing: border-box;
    margin-bottom: 0;
    color: white;
    background: #8ca800;
    padding: 10px;
  }

  & > div {
    box-sizing: border-box;
    padding: 10px;
    background-color: #eeeeee;
  }
}
```

Now, the usage on the hosting component

```html
<lib-empanel>
  <p>
    The quick brown fox jumps over the head of the lazy dog.
  </p>
  <p>
    This is another paragraph.
  </p>
</lib-empanel>
```

The most important player here is the `<ng-content>` tag in the component's markup. It serves as a placeholder for the contents of the component once that component
is plopped onto a host component. In our case, that `<ng-content>` is replaced by the two paragraphs inside the `<lib-empanel>` tag.

## Adding a Customizable Title

The following isn't really about wrapper components, however it's a good idea to include it here because it's a common corequisite of creating wrapper components.

We need a way for the component's title to be changed upon plopping the markup onto a host component. That is done with the `@Input()`. The new .ts file is now this:

```javascript
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'lib-empanel',
  templateUrl: './empanel.component.html',
  styleUrls: ['./empanel.component.scss']
})
export class EmpanelComponent implements OnInit {

  @Input() title: string;

  constructor() { }

  ngOnInit() {
    if (!this.title)
      this.title = "Default Heading"
  }
}
```

Modified template markup:

```html
<div class="panel">
  <h3>{{ title }}</h3>
  <div>
    <ng-content></ng-content>
  </div>
</div>
```

Usage:

```html
<lib-empanel [title]="'Two Paragraphs'">
  <p>
    The quick brown fox jumps over the head of the lazy dog.
  </p>
  <p>
    This is another paragraph.
  </p>
</lib-empanel>
```

I'm not going into detail on what `@Input` is and how it works, since this article is really about wrapper components.

## Next Steps

We would like to use this component in a structural directive. We would like the structural directive to pass its template ref to this component, and then plop
that component onto the screen. This will result in less coding on a hosting component!