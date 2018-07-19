# Scheduling Events

Now that we have the timeline data structure, we can now schedule events to it.

```csharp

public interface IEvent : IIdentifiable<long>
{
   int TimeCycleId { get; set; }
   string Name { get; set; }
   string MetaData { get; set; }
   
   int StartTime { get; set; }
   int Duration { get; set; }
}
```

Of course, we need to specify which `TimeCycle` the event will be scheduled in. This way,
we can detect any scheduling conflicts;

`Name` is a quick identifier for the event scheduled. No, it does NOT have to be unique!

`MetaData` is meant to be a JSON string that stores important information on what is
being scheduled. The data stored here isn't significant to the scheduling system, however
may be important to the implementing application for display purposes.

We have `StartTime` and `Duration`, both in seconds, which comprise the scheduling info
for the event. `StartTime` is a unique-value property.

## Scheduling an Event

This system does all the work of scheduling, including verification and conflict avoidance.
So, this is how we plan to schedule an event:

```csharp
public async Task<ManagerResult> ScheduleAsync(
   long timeCycleId,
   string name,
   string metaData,
   int startTime,
   int duration);

public async Task<ManagerResult> ScheduleAsync<TTime>(
   long timeCycleId,
   string name,
   string metaData,
   TTime startTime,
   TTime duration,
   ITimeConverter<TTime> timeConverter)
```

The first form is the fundamental form. The second form is a wrapper for the first form
that makes it a bit more convenient for the implementer with their custom time.

This function will return errors if scheduling conflicts happen (encroaching on an already-
scheduled event's time, or goes out-of-bounds of the time cycle's time)

## Modifying the Schedule of an Event

We will allow modifying the schedule of an event, only allowing the modification of
metadata, start time, and duration:

```csharp
public async Task<ManagerResult> ModifyScheduleAsync(
   long eventId,
   string name,
   string metaData,
   int startTime,
   int duration);

public async Task<ManagerResult> ModifyScheduleAsync<TTime>(
   long eventId,
   string name,
   string metaData,
   TTime startTime,
   TTime duration,
   ITimeConverter<TTime> timeConverter)
```

Like `ScheduleAsync`, it will also return errors for scheduling conflicts and going out-of-bounds.

## Deleting an Event's Schedule

Deleting is a simple operation, since no scheduling or out-of-bounds conflicts can occur.

```csharp
public async Task<ManagerResult> DeleteAsync(long eventId);
```

## Rolling out a Time Cycle

We have yet to decide what the implications are for rolling out a time cycle. Rolling out a
time cycle means actually creating the real-time schedule. This is akin to a cylindrical
stamp (time cycle) painted, and then rolled on several times onto paper or wall (real-time
schedule).

For now, we will require that the time cycle be specified, and how many times we'll go ahead
and roll out the schedule. Yes, we will create a very large table that stores the info about
what event will happen at what time. We will examine this in a later article addressing
the live schedule.


