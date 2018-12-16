# Logging - Levels

So far, we have logged to the console and debug output with `.LogInformation("message")`. We have logged at the information level, or
if we want, a heads-up notification. It is also possible to log at different levels, like at the warning and critical levels. For each
of the logging levels, there is an associated function named `Log{Level}(...)`. Choosing which level to log doesn't affect our app.
They are just for display in the event where we need to read logs to help us fix bugs.

The following are the Asp.NET Core logging levels, with their severity (0 is safe, 5 is the most dangerous), and comments on how we determine
which one to pick.

**Trace, 0**. Messages indicating where we are in the execution of particular tasks in the app.

**Debug, 1**. Messages that show values of variables to help us pinpoint any logical errors.

**Information, 2**. What just happened, like when something was sent, received, successfully.

**Warning, 3**. When some values may be wrong or unexpected, but shouldn't crash the app. Typically for logging exceptions
that we know what to do when we catch them.

**Error, 4**.  This is a good candidate for dysfunctions such inability to make http calls, or for when an IO operation failed.

**Critical, 5**. The most severe scenario, like when we're out of memory, hard disk space, etc.

The above descriptions are just guidelines, and we may use them for our own development and troubleshooting. You and if applicable, your
team needs to decide how to determine which kind of occurrences fit under the appropriate level so that everyone is on the same page when
fixing a bug.

## Log Event Ids

It is possible to assign unique ids to each event, and these unique ids need to be integers. This is so that those ids will display along
with the lab messages:

`Logger.LogInformation(1500, "Video was just uploaded");`

Which will result in an example display

`LoggingFocus.Controllers.ValuesController:Information: Video was just uploaded`.

As a good practice, which we won't go into detail here, we will want to create a static class that contains all the log event ids associated
with a friendly names so that the logging reads much clearer like:

`Logger.LogInformation(MyLoggingEvents.VideoUploaded, "Video was just uploaded");`

where we may have associated the property `VideoUloaded` to 1500.

We don't need to absolutely need to use this override, but it may be useful in some cases.

## Logging Exceptions

There is also a logging override function that takes in an exception:

```csharp
try
{
    ...
}
catch(Exception e) {
    Logger.LogError(e, "Something wrong happened.");
}
```

This is very useful for us to see the stack trace for when an exception occurs. Instead of hacking away at the `Exception` object caught
and then putting the values in the log message, we can simply pass in the caught exception (first parameter), and then our logging message.
This way, we can focus on other development tasks.





