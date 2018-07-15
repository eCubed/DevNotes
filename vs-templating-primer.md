# Visual Studio Templating Primer

Since we have developed quite a few projects, we may have noticed that the start of our development
is very similar across projects, if not, nearly identical except for the name of our entire system.
We have literally created files from scratch, and cut-copy-pasted a lot of code snippets from a previous
project to a subsequent project, and performed,  significant amount of string replacement. This is
tedious, time-consuming, and error-prone. At the end of this series, we will be able to write a template
for an **entire solution**. Once our template is complete, we will be able to, with a few
clicks and minimal typing, create a starting solution to an entire system, complete with starting
business logic, EF implementation, auth server, resource server, file server, and dev and prod deploy
projects. We will no longer need to manually create many files, cut-copy-paste from older projects, and
mass string-replace. We can dive right in and start concentrating on our app's data needs. We can
go ahead and start adding our interfaces, logic, and EF implementation, and soon after, begin writing tests
and migrating to a dev database.

We will develop the template both in Visual Studio and Notepad, as some of the files cannot be directly
edited from inside Visual Studio. Be forewarned that the code we will write in Visual Studio is going to
be syntactically wrong because it will contain placeholders inserted in many places that would cause
Intellisense to mark our code wrong. While it becomes more difficult to read and develop, it will pay off
in the long run when starting a new project becomes worry-free of basic ASP.net Core setup.