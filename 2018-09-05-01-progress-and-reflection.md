# Progress and Reflection

## Environment Variables in Linux
Yesterday, I discussed the need to know about environment variables in Linux. So, I researched how to do it, and it seemed like one of those activities that I
only learn to do it for the sake of learning. At this point, I don't know why it's significant to store a shell variable or even an environment variable that
any user can access. It turns out that an Asp.NET Core web application doesn't pick those up, and neither does it pick up environment variables stored in
the file `/etc/systemd/myvars.sh`. So at this point, I take it as one of those Linux things that are "nice to know how to do", though I don't really know
what for.

However, I found that I can set environment variables in a web application's service file, and yes, the Asp.NET Core application does pick those up. The only
thing, though, is when I edit and save the service file, I'll need to restart my web app's service for the variables to take effect. This is probably better
than storing it system-wide if only a single web application depends on it.

## File System Object Ownership and Permissions

I'm still a bit confused about how Linux handles security of files and folders. So far, I think I [sort of] understand the following:

- **R,W, and X**. These stand for read, write, and execute, which are actions that someone is able to do to a file. Read means someone can open the folder to
see its contents, or if a file, they may open it up in a text editor to see it. Write means they're able to rename or change the contents. Execute, I so though,
means they can run the file, if it's a script or executable. It turns out that it also (?) means they can at least view subfolders and their contents (which
seems like an odd definition for 'execute').
- **Owner, Group, Other**. For each of these three entities, a user classified as one of them or being a part of one of them, for a particular file system
object, can be granted or denied read, write, or execute access. At this point, I'm a bit confused as to why a file or folder needs to be owned, and by the way,
a file system object can only have one owner at any one time. Further more, if someone were the owner of a file system object, it *should* mean that they get
full access (read, write, AND execute) to that system object, but it's still possible to restrict some access, for example, not let the owner of that file
be able to edit and save it. It seems like it wouldn't make sense, though, that we could literally grant others higher privileges to a file system object than
its owner.

I guess I understand what the little pieces are about ownership and permission, but I'm not certain about some of the reasons and reprecussions of setting
owners and privileges to a file system object.

## Next Steps

MySQL and Asp.NET Core seems to be the final big thing I need to iron out for Linux hosting. My Asp.NET Core apps target 2.1, and the runtime installed in the
system is also 2.1. However, the current available .NET Core packages only support upto 2.0, so we might have a problem. There is a slight possibility that we
can use those 2.0 packages on a 2.1 project. We shall see.

The first thing we'll do is try to perform code-first migrations. We're not sure if that's possible.
