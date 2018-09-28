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

