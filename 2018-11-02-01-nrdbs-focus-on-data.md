# Non-Relational Databases - Focusing on Data

At this point, I have not actually surveyed (played around with) several NRDBs to explore how they are better than RDBs in general. As a back-end developer, one
of my concerns is relationships between data.

An NRDBs may be good for:

* Entities where all of its properties are primitive types.
* If ever an entity requires nested objects, either as immediate properties or an array of them, those nested objects MUST ONLY belong to that entity. No other
entity should own any of the nested objects already owned by one other entity.

In my opinion, that's an extremely limited scope for practical use of a NRDB. Only trivial applications like ToDo seem to be a great fit for a NRDB.

One non-trivial application where NRDB may be a good fit is to manage artists, albums, and tracks. We've got many artists. Each artist has an album. Only the
artist owns that one album. Each album has tracks (songs), that no other album has. 

The problem arises if we need to implement featured artists on someone else's track. Sure, we can add a featured artist property on the track, but that's where 
the problems begin. Suppose we edit the main record for that featured artist and change her last name because she may have just gotten married. We would then have
to hunt down every supposed reference to that artist in everyone's tracks and update those tracks. That's cumbersome. What if the users want to see, right there, all of the
albums of that featured user? What if the user wants to see all the artists that featured them in any of their tracks? What if the user wants to see all other
tracks that the artist is featured in? We can anticipate more queries, and our object would blow up pretty quickly. As soon as we see a requirement for a sub-property
to have a property of the same entity as one of its ascendants, we would better be off using a RDB.