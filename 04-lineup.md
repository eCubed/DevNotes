# The Lineup

We have discussed and finalized the time cycle and scheduled events to it, including time conflict management. Events on time cycles
do NOT have absolute date and times assigned to them. Event schedulings are only relative to each other inside the same time cycle.
We now need to take a time cycle and translate it to a definite schedule, which is exactly when each of the events will take place.
Since they're time cycles, they will most likely be repeated a set number of times. There are several approaches to saving a definite
schedule. Initially, we have thought of creating a record for each event in a separate list, and there would be a host that would
execute that list from start to finish. While this is straightforward, this will result in a very large list. Instead, we will
create a lineup, as `ILineup` which will represent exactly what is to be executed in sequence, without creating a huge list.

We can think of a lineup as a TV channel schedule for a particular season. Let's take for example, the lineup for season Summer 2016 of
a TV channel. We would create a single time cycle which lasts for a week. We then schedule shows and movies on that time cycle (disregarding
specific episodes, though - that's the responsibility of the implementer). Now, enter the lineup, which will specify how many times
that cycle will play to complete the season. In our case, it's 13 weeks.

However, a season's schedule isn't always one time cycle playing over and over again until the season is done. We may insert a different
time cycle in the middle of the season for let's say 4 weeks, to accomodate, for example, the Olympics. We would then resume with the
original time cycle to complete the rest of the season.

An `ILineup` is also an interaction to associate it with a given record. We may think of a SO-recordId combination as a single channel
to schedule lineups.

```csharp
public interface ILineup : IIdentifiable<long>, IInteraction
{
    string Name { get; set; }
    DateTime StartDateTime { get; set;}
}
```

```csharp
public interface ITimeCycleLineup : IIdentifiable<long>
{
   long LineupId { get; set; }
   long TimeCycleId { get; set; }
   int StartMark { get; set; }
   int CycleCount { get; set; }
}
```

We can see that we still need to perform some conflict checks to ensure that a time cycle will NOT intrude on another time cycle's
schedule in the lineup.

We need to note that the time cycle lineup needs the `StartMark`, which is the seconds mark it needs to start relative to the
lineup. `CycleCount` is how many times the time cycle will execute in a row.

The `ILineup` has a `StartDateTime`, which is when the first event of the first time cycle iteration will start. `StartDateTime`
is named with the -`DateTime` suffix because the time of day, upto the seconds mark will be significant.