# Angular 6+ Module Development and Publication

Creating and publishing modules in older versions of Angular were some fairly dreadful and mind-numbing experiences that required a lot
of setup, manual building for testing updated module code, and second-guessing how to properly reference the library from a consuming
(testing) project. When it was time to publish it to npm, we would have had to go all our references to our modules and update them. It
was an extremely tedious process. The difficult workflow from Angular 5 and older is now gone. Angular 6 now has a built-in library
development suite to make this task far easier and straight-forward.

## Creating the Workspace

We go to the prompt and create a new workspace:

`ng new {workspaceName}`

In Angular 6, what we used to call an app is now called a workspace, which can contain multiple of those apps!

## Creating a Library

We need to cd to the workspace's root and stay there. From the prompt, we create a library:

`ng g library {libraryName}`

And for `{libraryName}`, let's go ahead and name it like an npm package, using `@mystuff/first-library`, with the @ and the forward
slash. After we run the command above, we will have a folder `/projects/mystuff/first-library` automatically created and pre-filled
with starting module files, ready for publication.

We would create classes, components, services, etc. in the folder `/project/mystuff/first-library/src/lib`.

Once we have created our unique module's files, we edit `/projects/mystuff/first-library/src/public_api.ts` and export the classes,
components, services, etc., that we need to so that a consuming application may access them.

Note that we included the @ in our package's name, but it didn't make it in the folder name. However, if we look at the file
`/projects/mystuff/first-library/package.json`, the name of our package does begin with the @ like we expected.

## Building the Library

We no longer have to keep on manually building the library so that our consuming apps can reflect the new code. As of Angular 6.2,
we can keep on editing our code, and as we make changes and save files, our library gets rebuilt:

`ng build @mystuff/first-library --watch`

The `--watch` flag is the key to automatic rebuilding, and as soon as its rebuilt, the consuming app will immediately recognize the
changes.

Note that we included the @ character when we built the app. We build by the library's package name, not by its location on hard disk.

This library's build output will be written to `/dist/{libraryPath}`, in our case `/dist/mystuff/first-library`. That `--watch`
flag will continue to write to and overwrite files in this library's output folder.

## Consuming the Library

The current workspace still has an `/src/` folder, which is the root application of the workspace. From any .ts file in that root `/src`
folder, we can import this way:

`@import { SomeClass } from '@mystuff/first-library';`

The import statement looks like as if we actually npm-installed the package, but the workspace knows to look for it in the the library's
output path. When we ran `ng build`, the file `tsconfig.json` was also edited. It added the association between our package name
`@mystuff/first-library` to the build output folder `dist/mystuff/first-library`. This association is how the app knows where to
look when it encounters import statements that refer to a package.

## Publishing the Library to npm

If we haven't yet registered an account with npm, we would need to do so first, since publishing requires credentials. Once we've
registered, we run the following commands in sequence:

```
ng build @mystuff/first-library --prod
cd dist/mystuff/first-library
npm publish
```

It looks like the build command we issued during development, however, we have the `--prod` flag, which creates a more compact and
uglified package suitable for publishing to npm. 

Note that all existing compilations on `/dist` from development will be wiped out when we build to production. We should then not
run build with the `--prod` flag while we are developing. We would need first to stop the development watching for changes.

Once publishing to npm succeeds, it is immediately available to can npm-install.