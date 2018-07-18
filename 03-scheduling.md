# Scheduling

Now that we have the timeline data structure, we can now schedule events to it.

```csharp

public interface IEvent : IIdentifiable<long>, IInteractible
{
   /* Shown as courtesy for the fact that an ITimeCycle is an IInteraction
   int SystemObjectId { get; set; }
   string RecordId { get; set; }
   */

   long TimeCycleId { get; set; }
   string MetaData { get; set; }
   
   int StartTime { get; set; }
   int Duration { get; set; }
}
```

Of course, we need to specify which `TimeCycle` the event will be scheduled in. This way,
we can detect any conflicts.

`MetaData` is meant to be a JSON string that stores important information on what is
being scheduled. The data stored here isn't significant to the scheduling system, however
may be important to the implementing application for display purposes.

The `IEvent` is an `IInteractible` to associate it with whatever entity is necessary.
Note that different events of different entities MAY show up in the same time cycle.

We have `StartTime` and `Duration`, both in seconds, which comprise the scheduling info
for the event.

## Scheduling an Event

This system does all the work of scheduling, including verification and conflict avoidance.
So, this is how we plan to schedule an event:

```csharp
public async Task<ManagerResult> ScheduleAsync(
   int systemObjectId,
   string RecordId,
   long timeCycleId,
   string metaData,
   int startTime,
   int duration);

public async Task<ManagerResult> ScheduleAsync<TTime>(
   int systemObjectId,
   string RecordId,
   long timeCycleId,
   string metaData,
   TTime startTime,
   TTime duration,
   ITimeConverter<TTime> timeConverter)
   where TTime : class
```

The first form is the fundamental form. The second form is a wrapper for the first form
that makes it a bit more convenient for the implementer with their custom time.

This function will return errors if scheduling conflicts happen (encroaching on an already-
scheduled event's time, or goes out-of-bounds of the time cycle's time)

