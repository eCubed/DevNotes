# Abstract Level Coding for Interactions

We have adopted the Manager/Store system and FCore.Interactions does conform to it. However, we will
need to modify it a little bit.

## The IInteraction Interface

When we want to save a piece of interaction data, we need to know for which record it's for and for
which system object it is.

```csharp

public interface IInteraction
{
    string RecordId { get; set; }
    int SystemObjectId { get; set; }
}
```

`RecordId` is a string because this system should not care what data type the actual entity's PK is.
`SystemObjectId` depicts which entity type we're interacting with.

## Specific Interaction Interfaces

We will name these as nouns, either in the gerund form, or one that signifies the interaction saved. 
Suppose we want to store who likes a particular post on a social media site. Then the interface would be 
the following:

```csharp
public interface ILiking : IIdentifiable<long>, IInteraction 
{
    string LikerId { get; set; }   
}
```

The `IIdentifiable<long>` will make `ILiking` have an `Id` field of type `long`. For simplicity's sake,
we will have large integer PKs in this system. That it implements `IInteraction` will mean that `ILiking`
will also have the `RecordId` and `SystemObjectId` properties.

Notice that the name is `ILiking`, where "Liking" is in the gerund noun form. This is important for consistency.
It must be in a noun that is exactly a depiction of the act of interacting.

### Interaction with Accompanying Data

Often times, an interaction will require an additional piece of data to be stored. Suppose that we also
want  users to be able to mark something with a "love", "laugh", or "dislike". We wouldn't want to have
a separate interface for those quick responses. Instead, we can create the following interaction:

```csharp
public interface IQuickResponding : IIdentifiable<long>, IInteraction
{
    int QuickResponseId { get; set; }
    string ResponderId { get; set; }
}
```

`QuickResponseId` would indicate which type of response it is, i.e., Like, Dislike, Laugh, Love, etc., hence,
we also have `IResponse`

```csharp
public interface IQuickResponse : IIdentifiable<int> 
{
    string Name { get; set; }
}
```

Since `IQuickResponse` has a relatively small number of responses, and unlikely to grow quickly, its
`int` PK is sufficient as indicated by `IIdentifiable<int>`.

## The Interaction Manager
This is the class where all logic is to be placed regarding the interaction. 

### Interaction Manager with Accompanying Data

Typically, a manager is for performing CRUD and other logic checks on one entity only. However, when we
have an interaction that also stores an additional piece of data, that interaction's manager should also
take in an extra parameter type. We'll take the QuickResponse as an example:

```csharp
public class QuickRespondingManager<TQuickResponding, TQuickResponse> : ManagerBase<TQuickResponding, long>
    where TQuickResponse : class, IQuickResponse, new()
    where TQuickResponding : class, IQuickResponding, new()
{
    private IQuickResponseStore<TQuickResponse> QuickResponseStore { get; set; }

    public QuickRespondingManager(
	    IQuickRespondingStore<TQuickResponding> quickRespondingStore,
	    IQuickResponseStore<TQuickResponse> quickResponseStore) : base(quickRespondingStore)
    {
	    QuickResponseStore = quickResponseStore;
    }

    ...
}
```

From this, we can go ahead and CRUD quick responses as well as `TQuickResponding`s.