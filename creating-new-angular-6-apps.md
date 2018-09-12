# Creating a New Angular 6.x App

Creating a new app is pretty straight-forward, but this guide addresses what to do once the starter Angular app has been created, since the default app generated
doesn't come with features that enough developers require.

## Creating the Initial App
Right off the bat, we would like to create an app that will have routing and to use SCSS for styling, so we'll run the command

`ng new {appName} --routing --style=scss`

We will need to wait a while for NPM to install all the needed dependencies.

We can run it really quick to see what it looks like. First, we need to `cd` into the new app's folder, then we can run it (in development).

`cd {pathToNewApp}`
`ng serve`

At this point, both the browser's console and the bash prompt should output any errors.

## Creating the First Routes

We will need to remove the image and links from index.html.

For better organization, we put all our components that are meant to display a page in our application in a `/routes` folder underneath `/src/app`, so we'll need
to create the folder first, cd into it, and then create our first components: Home, About, Contact, and NotFound.

`app.module.ts` gets updated to import the new components, and added to the declarations array. Our routes don't get set up automatically, unfortunately, so we'll
need to register them ourselves in `app-routing.module.ts`:

```javascript
const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  },
  {
    path: 'about',
    component: AboutComponent
  },
  {
    path: 'contact',
    component: ContactComponent
  },
  {
    path: '**',
    component: NotFoundComponent
  },  
];
```

We want a blank route to take the user to the home component. Also note the last route `**`. This means "match any route not registered in the Routes array", and
take them to a particular component/page, which is `NotFoundComponent`.

## Creating the Main Navigation Bar

We will need to create our first "widget", which is the application's main navigation bar. For components meant to be widgets like this one, we will store them in
the folder `/src/app/components` for better organization. So, we'll need to create this folder if not already created. Once we're in this new folder, we can go
ahead and create our `NavBar` component. Here is its template markup:

```html
<nav>
  <ul>
    <li>
      <a routerLink="/">Home</a>
    </li>
    <li>
      <a routerLink="/about">About</a>
    </li>
    <li>      
      <a routerLink="/contact">Contact</a>
    </li>
  </ul>
</nav>
```

We'll then add this new component to `app.component.html` right above `<router-outlet></router-outlet>`:

```html
<div style="text-align:center">
  <h1>
    Welcome to {{ title }}!
  </h1>
</div>
<app-nav-bar></app-nav-bar>
<router-outlet></router-outlet>
```

We now have a clickable and navigable navigation bar, albeit, not styled. We will get into that later.

## Next Steps

We have created and setup the most barebones functioning Angular 6.x app with routing. We haven't discussed CSS yet since that will require its own article, which
is the next logical task we'll perform - styling.

After setting up styling, the components, routes, and others we'll add will depend on the very applications we're developing.


