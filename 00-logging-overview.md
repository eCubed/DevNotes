# Logging Overview

Logging is the act of recording significant occurrences throughout an application's lifetime. Logging isn't exactly
the `Console.WriteLine()` that we scatter all over a console app to display textual results on the screen during
development (debugging). It isn't exactly explicitly writing stuff to whatever files we want either (though it's possible
that files will be written, but by the OS in a specific manner).

## When to Log
This is up to the application, but we will typically not log when a new record was created successfully or any standard
CRUD operation, as tutorials would often have us do. We probably would want to log exceptions, such as when, for some 
strange reason, a service that the application depends on crashes.

## How to Set Up Logging

TBD