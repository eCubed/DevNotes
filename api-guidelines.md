# FCore.Interactions Api Guidelines
An Interaction is what is done to an existing entity, without adding columns to that entity to store
that additional piece of data, and also without adding any additional tables per existing entity type
just to store that additional piece of data. This article focuses on API calls for these interactions.

## General POST
```json
POST api/{interact}/{systemObjectName}/{recordId}/{dataForInteraction}
```
This POST has no payload, and all of the data necessary should be in the URL

`{interact}` - verb in the imperative mood that states exactly what we desire to do. For example,
tag, like, categorize, flag, etc.

`{systemObjectName}` - which entity we are interacting with. This is a singular noun, and it should
be all lower case in the URL.

`{recordId}` - which record of the system object we are wanting to interact with.

`{dataForInteraction}` - a single string value that we need to store for the interaction. For example,
for tagging, this value would be the tag we want to associate with the record.

## PUT

There is no PUT in interactions. They're only created, read, and deleted.

## General DELETE
This is for deleting a particular interaction.
```json
DELETE api/{interact}/{systemObjectName}/{interactionId}
```

`{interact}` - verb in the imperative mood that states exactly what we desire to do. For example,
tag, like, categorize, flag, etc. Note that the verb is NOT the opposite of the interaction verb
in the POST counterpart. The DELETE HTTP verb should be enough to indicate we are removing an interaction.

`{systemObjectName}` - which entity we are interacting with. This is a singular noun, and it should
be all lower case in the URL.

`{interactionId}` - the very id of the interaction. This should have been supplied to the front end


