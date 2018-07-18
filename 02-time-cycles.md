# Time Cycles

We have defined the concept of a time cycle. We will now explore how to use time cycles to
store schedules.

We need to define an actual structure that needs to be saved, and later, scheduled to, so
we have an `ITimeCycle` interface:

```csharp

public interface ITimeCycle : IIdentifiable<long>, IInteraction
{
   /* Shown as courtesy for the fact that an ITimeCycle is an IInteraction
   int SystemObjectId { get; set; }
   string RecordId { get; set; }
   */

   int TotalDuration { get; set; } 
}

```

Since this is very abstract, we do not know what holds a sequence of events. It could
be a TV channel. It could be a thread in a forum. So, we will let records of various
entities be associated with a given time cycle to be scheduled against.

A TimeCycle is a write-once-read-only entity. The `TotalDuration` property is the
time cycle duration.

We now have a structure to schedule to.

# Creating a Time Cycle

We will have a `TimeCycleManager<TTimeCycle>` class that will manage creation of
time cycles, as well as other operations.

```csharp
public async Task<ManagerResult<TTimeCycle>> CreateAsync(
   int systemObjectId,
   string recordId,
   int totalDuration
   );

public async Task<ManagerResult<TTimeCycle>> CreateAsync(
   int systemObjectId,
   string recordId,
   TimeCycleDuration timeCycleDuration,
   );
```

There are two variants - one that takes in an integer, and the other that takes in an `enum`.
The variant that takes in an integer is for allowing a duration length other than the ones
provided in the `TimeCycleDuration` `enum`. The variant that takes in `TimeCycleDuration`
is for convenience.

Once a time cycle is created, we can now schedule against it.