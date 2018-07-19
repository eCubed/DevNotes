# Time Cycles

We have defined the concept of a time cycle. We will now explore how to use time cycles to
store schedules.

We need to define an actual structure that needs to be saved, and later, scheduled to, so
we have an `ITimeCycle` interface:

```csharp

public interface ITimeCycle : IIdentifiable<long>
{   
   string Name { get; set; }
   int TotalDuration { get; set; } 
}
```

A TimeCycle is a referenced entity, and it is intended to be created by top-level admins.


# Creating a Time Cycle

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

# Updating a Time Cycle
We will only allow editing of the name. We will not change the total duration because the
time cycle may have entities that depend on it.

```csharp
public async Task<ManagerResult> UpdateAsync(int id, string newName);
```