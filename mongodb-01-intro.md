# MongoDB Intro

## What is it?

As developers, we most likely learned relational databases first, although we learned objects first in our programming careers. During the course of
development, we saw that we still like to work with objects as they seem so natural to work with, as they more closely resemble real life, but we still
like to store data in tabular format because we're used to it. We played with tools like EF Code-First which lets us develop with objects, but saved our
data in tables. Well, there's a different family of database that lets us store our objects as well, objects instead of breaking our objects apart into
multiple records that are stored in many tables. This type of database is called NoSQL. Our objects, no matter how complicated, are stored together
intact. There are no foreign keys, therefore, no confusing joins.

In this article, we're going to explore MongoDB - a NoSQL database. We'll explore where it's a good fit or whether it's just better to stick to a
relational database.

## Installation and Setup

Download from [here](https://www.mongodb.com/try/download/community). I chose Windows MSI current version, and I'll install it as a Windows Service.
This way, it's always running and available when I log into Windows. I'll choose the option "Run service as Network Service User." This will allow us to
just use the service as long as we're logged into Windows. I'll also choose to install MongoDB Compass - the GUI for Mongo.

## First Database Actions with Compass

When I open Compass for the first time, I don't have a connection yet, but I'll want to first connect to the one already running as a Windows Service.
I'll click on `New Connection` on the left-hand menu. I'll leave the connection string blank and click the `Connect` button.

I'll go ahead and create my first database. I'll name it `MyFirstMongoDb` and `People` for my first collection. I'll leave `Capped Collection` and `Use
Custom Collation` unchecked. A **collection** in MongoDB is analogous to a table in a RDB.

I'll head on over to the `People` database and add a record by clicking on the `Add Data` button. I'll choose **document** (analogous to a table row in
a RDB), and type in some simple JSON:

```json
{
  "firstName": "John",
  "lastName": "Smith"  
}
```
I'll click `Insert` to add the record to the database.

I'll add a few more records just to have a good feel of MongoDB.

One immediate thing I notice is unlike an RDB, I do NOT need to define columns before inserting a record. I define the "columns" as object properties
in the JSON I write. These "columns" are called **fields** in MongoDB. I can literally insert data into a collection that have wildly varying fields.
That's the flexibility that comes with a NoSQL database that can be used wrecklessly, so we'll need to be careful. The nice thing is that we don't have
to deal with the pain of defining and redefining a database and making sure that our CLR objects match what's in the database at all times.

So, after inserting a couple of records, I'll go ahead and try to query. In the `Filter` textbox, I'll type `{ "firstName": "John" }`, and then click
`Find`. I should get all records that match the criteria I specified. Notice that the criteria looks like JSON. This example is a simple "match full
value of a field" query. We'll get into more complex queries later like substring, starts with, filtering, aggregating, etc.

If we hover over a record on the Collections UI, there are icons that will pop up, for edit, copy, clone, and delete. Delete is self-explanatory. Clone
will pop up a dialog that pre-fills the exact JSON of the record. I'm not sure what 'Copy' does, except store the record in memory for later use, and
Edit is for updating the record.

If I click edit, the UI will prompt me different tasks, like let me click on an existing field name or value to change it, and even add new fields if I
want to. I went ahead and added `age` as a new field to one of the records, and chose the data type `Int32`. I then click `Update` to save changes.

## Next Steps

I just performed some basic database tasks to a MongoDB database, but I did it all in Compass. In real life, I will most likely need a Mongo client in
my platform of choice that will allow me to perform CRUD to my MongoDB data. I use ASP.NET Core, so I'll need to find a way to bridge my apps to a
MongoDB.

Also, it's probably a good idea to find a MongoDB Host, even for educational purposes. I'll look into some providers.