# Non-Relational Databases - An Initial Reflection

I first learned what even just a database was back in college, and the first one I learned was MySQL, a relational database. I've used it throughout my education,
and professionally, I've worked with MSSQL so far. So, for persistent data storage, I am very conditioned to think of data in terms of tables, columns, and rows,
and relationships between tables are determined by foreign keys.

A few years ago, I have heard of persistent data storage that sounded to me like we store objects as they are, including all of their complex properties and arrays,
intact and in one whole piece. This would be in contrast to the relational database model where each complex property and each array item would be saved to
different tables. Well, this all sounded like how I used to save and retrieve data from the file system before I learned about databases at all. I stored the
data in some proprietary format I concocted, and I wrote code that would parse the string contents of that file to explicitly rebuild the entire object. I didn't
use JSON or XML at the time because they were unheard of. It turns out that there are actual services regarded as databases that store objects, and they are 
referred to as *non-relational databases*, or NoSQL.

Later, I dabbled in a bit of non-relational database development with Firebase. In short, I do store objects as they are, in JSON, and retrieve them back as JSON.
It was just an introduction to me, and I only saved either simple objects (whose properties are all primitive types), or objects that may have complex properties
that belong only to their parent object (such as an individual person's address) and not refered to by another record. So I said to myself, "oh - it's just another
way to store and retrieve persistent data; that's nice." I didn't at the time study its implications, its advantages, its drawbacks, and how to decide if NoSQL is
appropriate for particular projects with unique data needs. I ended up putting NoSQL in the back burner for a while, as I needed to complete projects whose persistent
data store was a relational database which I was more familiar with.

Now, in late 2018, I was asked to check out NoSQL for possible integration and implementation for near-future projects. I did a bit of reading and there are a
few things that I picked up. NoSQL is good for quick, small, and frequent transactions, such as adding financial transactions to someone's bank account or adding
comments onto a post on social media. My guess is it's quick because the system wouldn't need to query against multiple tables with joins, which are quite computationally
expensive with relational database - we're focusing on one record and one record only, and drilling down to its own objects. There's the talk about horizontal
scalability, which I as a programmer have no direct coding involvement in, that is supposed to be easier to understand, setup, administer, and implement with
NoSQL than with a relational database.

So, I'm going to focus on considerations that I as a programmer have control with when deciding whether NoSQL is a better solution than an RDB (relational database)
for a particular project. So far, all of the projects I've worked on professionally require an RDB because we have data that is administratively created, but
referred to, and drives the data stored in users' content. It wouldn't make sense to store the same huge object multiple times in many user records. That's
obviously a redundancy issue, but it's more of a synching issue. What if we needed to update administrative data? We'd need to find all user records that have it
and perform updating on every single one. That's not a very feasible solution. Another large unknown for me is how I would store and manage user information with
NoSQL. For example, one of my concerns is how would I make sure that, in a non-RDB, the list of roles for a particular user are all allowed values for my app?
How would I implement things like password reset or manage multiple logins for the same person?

So far, NoSQL feels like a no-go for me if the data I need to work with seem like a better fit with RDBs. However, I don't want to completely trash it as something
I feel like I'll never use. As a developer, I feel that I need experience in technology I don't think I'll use for my own marketability.
