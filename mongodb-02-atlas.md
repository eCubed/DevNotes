# MongoDB Atlas

When I deploy, I will need to sign up for a MongoDB Provider, so that my app's data will be stored there. I signed up for the free tier of MongoDB Atlas.

The first thing I'm asked to do is to set up a project. A project is one of the upper levels of the Atlas Hierarchy. It can denote a different company
for which I manage its data. For the sake of initial immersion into MongoDB hosting, I'll accept the name of the first project `Project 0`.

Then there's the cluster, which, per MongoDB Atlas, is `a set of nodes comprising a MongoDB Deployment`. For now, we can think of clusters as allocated
resources (CPU, memory, and hard disk) for our databases.

Now, each cluster can contain more than one database. So, I'll set one up. I'll navigate to `Data Storage > Clusters`, and I'll see the first cluster
created for me, which happens to be `Cluster0`. I click on the `Collections` tab, and I'll create a database there. I'll choose `Add My Own Data`
since `Load a Sample Dataset` is huge. I'll specify `MyFirstMongoDb` as the database name and `People` as the collection name, just like how I specified
in my local MongoDb with Compass.

Now, the interface looks very similar to Compass when it comes to performing CRUD on my database. I won't touch this now.

I'll go to the left-hand menu and click on `Database Access` under `Security`. I'm going to add a user by going through the dialog prompts. I'll choose
`Password` for the Authentication Method, set the user name and password, and choose `Read and write to any database` for `Database User Privileges`.
I want the user to be permanent, so I'll need to make sure that `Temporary User` is set to `off`. I will need to wait for a few moments while Atlas
is configuring my database after I add the new user.

Now, I'll need to figure out a way to connect to this database on the cloud. I will need a connection string. I'll need to head back to the left-hand
menu and select `Clusters`, and click `Connect`. I will need to list the IP addresses of machines that will host apps and any other software that will
access this MongoDB on the cloud. Under the step `Choose a connection method`, I'll choose `Connect your application`. I'll choose `C#/.NET` version 
`2.11 or later`, Then, I'll get my connection string formulated for me with placeholders where I'll specify password and the database name.