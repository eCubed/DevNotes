# Logging Overview

Logging is the act of recording significant occurrences throughout an application's lifetime. 

Logging isn't exactly the `Console.WriteLine()` that we scatter all over a console app to display textual results on the screen during
development. With logging, there is a more formal way to display such messages, and we don't actually display messages on our very app.
We'll look into this later.

Logging isn't exactly explicitly detecting an occurrence (i.e. when a variable's value becomes a certain amount) and then writing it to
to whatever files we want either. However, logging *can* write to certain files *but* the logging framework will perform the task for us,
deciding for us what the file name scheme is, where those files will be stored, and the format of the contents of those files.

## When and What to Log
The decision of when to log something depends on the application and the developer's needs. There is no exact science of whether to log
an occurrence. When we learn how to log properly, we will most likely log the most trivial circumstances such as when the app starts, or
a "hello world" type of logging just about anywhere in our app's runtime. There are times that we might want to log when a record gets
created, edited, or deleted, though most of the time, we won't log CRUD events. In practice, and we say this as a mere speculation but
we'll actually do it, we will most likely log when exceptions occurr for when our app is running in production. Since we cannot debug
in production, we will want to see what happened, and the only way to see the errors is via a log file.

## Next Steps

We will explore how to set up and perform various types of logging first in a Web API application. We will perform logging to see events
*in* VS, not our running app. We will also distinguish the environment so we know whether to log certain occurrences.
