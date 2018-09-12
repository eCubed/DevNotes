# Angular Production Deployment

When our app is ready, we need to compile it and put it on a server.

Note that the compilation that is the exact web files generated to display the app on `localhost:4200` is not the same as what would be deployed.

If we examine the scripts section of our project's `package.json` file, there is `build` command from which we may run `npm run build` or `ng build` from the
command prompt. Running the build command as-is will create a compilation of our app into the `/dist/myapp` folder.

We currently have a very minimal app with 4 routes, minimal styling, and no dependencies outside whatever came with Angular. With `ng build` alone, the total file
size of what is to be deployed is 6.69 Megs - way too much for such a simple application, and in fact, even too much for a fairly complex and large application.

We will need to figure out how to shrink it to the bare minimum! so, we would run `ng build --prod` instead and see what happens. The total size is 318 kb, which
is far smaller than 6.69 Megs.

**Important:" Do not run `npm run build --prod`. The `--prod` flag we specified will be ignored! Instead, use the CLI and run `ng build --prod` instead.