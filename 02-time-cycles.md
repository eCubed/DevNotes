# Time Cycles and Time Cycle Types

We have defined the concept of a time cycle. We will now explore how to use time cycles to
store schedules.

## TimeCycleType

We need to define an actual structure that stores basic info about a time cycle. We have 
`ITimeCycleType` that defines `TotalDuration` in seconds. When people create their own
lineups (their actual time cycles), that entity will refer to the TimeCycleType.

```csharp
public interface ITimeCycleType : IIdentifiable<long>
{   
   string Name { get; set; }
   int TotalDuration { get; set; } 
}
```

A TimeCycle is a referenced entity, and it is intended to be created by top-level admins.


## Creating a TimeCycleType

We will have a `TimeCycleManager<TTimeCycle>` class that will manage creation of
time cycles, as well as other operations.

```csharp
public async Task<ManagerResult> CreateAsync(
   string name,
   int totalDuration
   );

public async Task<ManagerResult> CreateAsync(
   string name,
   TimeCycleDuration timeCycleDuration,
   );
```

There are two variants - one that takes in an integer, and the other that takes in an `enum`.
The variant that takes in an integer is for allowing a duration length other than the ones
provided in the `TimeCycleDuration` `enum`. The variant that takes in `TimeCycleDuration`
is for convenience.

Once a time cycle is created, we can now schedule against it.

## Updating a TimeCycleType
We will only allow editing of the name. We will not change the total duration because the
time cycle may have entities that depend on it.

```csharp
public async Task<ManagerResult> UpdateAsync(int id, string newName);
```

## TimeCycles

A `TimeCycle` is the very lineup of events that are actually scheduled against. It references
a `TimeCycleType` so that events can be scheduled and conflict-checked. It is also owned
by a particular entity.

```csharp
public interface ITimeCycle : IIdentifiable<long>, IInteraction
{
    string Name { get; set; }
    int TimeCycleTypeId { get; set; }
}
```

The `IInteraction` interface means that it will have `SystemObjectId` and `RecordId`
properties so we can associate a time cycle with an actual entity at implementation.