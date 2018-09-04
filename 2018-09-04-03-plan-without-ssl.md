# Plans Without SSL

At this point, SSL is the only top-level missing piece of this Linux hosting undertaking. I've just set up MySQL and remotely accessed it from MySQL Workbench.
Right now, I'm still waiting to call tech support, so in the meantime, I'm thinking about what to focus on while SSL is uncertain.

- **Linux Files and Directories Permissions.** I've gotten a taste of protecting files and folders when working with IIS, as well as seeing some permission settings
when I worked with Unix in college. I know that there are the read, write, and execute privileges even per file, but I don't exactly know how to set and unset it.
In Linux, there is also the concept of ownership of a file or folder, presumably ownership of a user. I'm not sure, though, whether only one user or many users
may own a file or folder at a particular time. I'll definitely need to read up on this topic.
- **Familiarize with MySQL.** Since it's a lot in common with MSSQL, I think it's best for me to come across the more obscure features as my projects need. I'm almost
positive that most of the difference is syntax. I can already see that MySQL has data types that are not explicitly in MSSQL. When I set up MySQL on the server, the
user and privilege SQL "queries" looked a bit strange with odd keywords like `IDENTIFIED WITH .. BY ..`, which sounds to me like a `WHERE` clause in regular
SQL.
- **Linux Admin Tasks and Apps.** I may need to research what an administrator does to secure, fine-tune, and monitor a Linux server. So far, I know services -
how to view them, how to bring up their status, stop, start, and restart them. I don't know how to check system memory, hard disk space, etc.. - pretty much, tasks
that I perform in Windows with a GUI.
- **Play around with a Linux OS with a GUI.** I am quite hesitant to install a Linux OS on a Windows machine since the setup for that seems so rigorous and I
might break something. I do have a spare machine where I think I can wipe its contents (Windows Vista). But I'll try to be able to dual boot it, though.