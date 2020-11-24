# CI - Github and Jenkins Setup

## Github Setup

We'll need to login to our Github, find the correct repo, and go to its `Settings` page. `I'll go to `Webhooks` on the
left-hand panel.

I'll click `Add Webhook`. I'll enter my Jenkins url in `Payload URL`: `https://jenkins.mydomain.com/github-webhook/`. Yes,
the Jenkins app has a path to `/github-webhook`.

I'll leave Content Type to `application/x-www-form-urlencoded`.

I'll leave Secret blank.

I'll choose `Enable SSL verification`.

I'll choose `Just the Push Event`.

I'll save the changes.

## Creating the Jenkins Job

I'll go ahead and create my first job: `Jenkins` > `New Item`

I'll enter the name of the job - the job is where we specify steps on what to do to deploy the app.

I'll select `Freestyle project, and click `OK` at the bottom.

When I get to the job, I'll choose `Configure` on the left so I can outline the tasks.

Under `Source Code Management`, I'll select `Git`.

It'll let me specify a repo URL, so I enter the correct one. I'll also need to add credentials - the same credentials to log
into Github.

Under Branch Specifier, I can leave it to `*/master`. This means that ANY push to `origin/master` will have Github call our
Jenkins to begin the deployment process. We can designate a different branch for deployment if we need to.

Under Build Triggers, I'll select ONLY `GitHub hook trigger for GITScm polling`.