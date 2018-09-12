# Updating Angular CLI

This guide is for a current system that has Angular CLI 6.x and above. It may work for older Angular CLIs, though we may proceed with caution. This guide is NOT
about upgrading a current Angular project of older Angular versions than the new target version.

## Checking the Current Version of Angular CLI

`ng --version` Outside of an Angular project directory

We will get a display that tells us what version of the Angular CLI is currently installed in our system. (Unsure) The CLI version equals the current version of
Angular itself that is installed in the system. So, if we install a new Angular CLI, the Angular version will also be the same as that new Angular CLI's version.

## How to Find the Most Current Version of Angular CLI

We may not need to upgrade because we might have the latest Angular CLI. So, to figure out the latest Angular version, we can go to 
[Angular CLI's NPM Page](https://www.npmjs.com/package/@angular/cli). The version will be written right underneath the heading and on the right-hand column.

## How to Update Angular CLI

We've got CLI version 6.0.0, so we may not need to uninstall the current Angular version. We simply need to install with this command:

`npm install -g @angular/cli@latest`

If we run `ng --version` again, we will see that we have the lastes Angular CLI version.

Our old Angular 6.x applications will still run when we run `ng serve`. Some errors may occur if we have a dependency of something older that has been scrapped
in the latest Angular version, but this is a rare occurrence.


