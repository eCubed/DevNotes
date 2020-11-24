# Continuous Deployment - Primer?

## Old School Way

When we have proven that our app works for a release, we built the prod version. For a VS Project, we published. For an Angular
project, we ran `ng build --prod` or `npm run build:ssr`. After the CLI has done their jobs of building, we manually copied those
exact built files to the server, via FTP usually.

## The New Way

When everything has been set up, all we need to do is to push the repo of a project to a particular branch, usually `master`,
and after a few moments, we can go to the live website and see the changes reflected.

So, that's no more manually publishing from VS or running `ng build` from the dev machine. There's no more logging in to an
FTP client and nit-picking which files should be uploaded.

## What To Do?

There is a LOT of preparation - at the hosting server and at Github. We'll cover that in the next article.