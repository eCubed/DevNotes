# Non-Relational Databases - An Initial Reflection

I first learned what even just a database was back in college, and the first one I learned was MySQL, a relational database. I've used it throughout my education,
and professionally, I've worked with MSSQL so far. So, for persistent data storage, I am very conditioned to think of data in terms of tables, columns, and rows,
and relationships between tables are determined by foreign keys.

A few years ago, I have heard of persistent data storage that sounded to me like we store objects as they are, including all of their complex properties and arrays,
intact and in one whole piece, all the way upto where it's stored on the hard drive. This would be in contrast to the relational database model where each complex
property and each array item would be saved to different tables. Well, this all sounded like how I used to save and retrieve data from the file system before I 
learned about databases at all. I stored the data in some proprietary format I concocted, and I wrote code that would parse the string contents of that file to 
explicitly rebuild the entire object. I knew about JSON back then, but the idea of using it outside Javascript was unheard of, let alone serializing, deserializing,
and transporting it over the wire. It turns out that there are actual services regarded as databases that store objects, and they are referred to as 
*non-relational databases*, or NoSQL.

Later, I dabbled in a bit of non-relational database development with Firebase. In short, I do store objects as they are, in JSON, and retrieve them back as JSON.
It was just an introduction to me, and I only saved either simple objects (whose properties are all primitive types), or objects that may have complex properties
that belong only to their parent object (such as an individual person's address) and not refered to by another record. So I said to myself, "oh - it's just another
way to store and retrieve persistent data; that's nice." I didn't at the time study its implications, its advantages, its drawbacks, and how to decide if NoSQL is
appropriate for particular projects with unique data needs. I ended up putting NoSQL in the back burner for a while, as I needed to complete projects whose persistent
data store was a relational database which I was more familiar with.

Now, in late 2018, I was asked to check out NoSQL for possible integration and implementation for near-future projects. I did a bit of reading and there are a
few things that I picked up. NoSQL is good for quick, small, and frequent transactions, such as adding financial transactions to someone's bank account or adding
comments onto a post on social media. My guess is it's quick because the system wouldn't need to query against multiple tables with joins, which are quite computationally
expensive with relational database. With NoSQL, we'd be focusing on one record and one record only, and drilling down to its own objects. There's the talk about horizontal
scalability, which I as a programmer have no direct coding involvement in, that is supposed to be easier to understand, setup, administer, and implement with
NoSQL than with a relational database.

So, I'm going to focus on considerations that I as a programmer have control with when deciding whether NoSQL is a better solution than an RDB (relational database)
for a particular project. It seems like we could just store any object in a NRDB (non-relational database), which means that we do NOT set up schema like we do
in an RDB. OK, that's interesting that we wouldn't have to go through fairly involved processes like migration or manually constructing tables. If we can just save
any object, then two objects that are supposed to be conceptually of the same kind (of the same data type) can have wildly different content in a NRDB. So then it
seems like a NRDB really is a "loosely-typed" persistent data store. This seems to be true when I've read somewhere that the client (either front-end or back-end)
will need to be responsible for ensuring that the objects saved to a NRDB have the appropriate and/or required properties and values. That does add quite a major
responsibility in code, and working with loosely-typed dynamic objects, at least to me, feels very fragile, uncertain, and unpredictable to work with. In NoSQL
terms, these loosely-typed objects are referred to as *unstructured data*, which scares me because they're difficult to work with in development.

I don't want to trash NoSQL just because what I understand so far feels very uncomfortable and risky. I'll read up on some more to get a better appreciation
on what it can do and where it's more appropriate to use it than an RDB.

