# Logging Into the VPS

I installed PuTTY so I can access the VPS server from (putty.org)[http://www.putty.org]. I chose the 64-bit windows installer for my machine.

So I logged in at root, and when they asked for a password, typing it didn't show the asterisks for each character I typed, which is normal behavior
for security right at the monitor. Every time they ask for a password, like to change one once logged in, it won't show anything but it's actually
inputting each key press.

looked around (`ls -la`) and there are no folders. There are a few files that start with a dot. We'll get to those
later in more advanced writeups. But there it is - we were able to log into root.

## Changing the Root Password
The root account is something we probably won't touch much, but it is said that we should change the password of this root account.

So at the prompt, we type `passwd`, then hit Enter. We'll be asked to type a password, and type it again. If all goes well, we'll get
the message "password updated successfully".

Now, we type `exit` to log out. The PuTTY window disappears. We open PuTTY again and log in with the new password. It works.

## Adding new Users

We can only add new users from root when we first log in, so if not already, we need to log in to the root account.

I ran the command:

`useradd admin`,

to add a new user by the user name `admin`, then Enter. Nothing seemed to happen - no confirmations, no error
messages. It just gave me back to the prompt. Actually, that added the new user.

Now, I'll set the password with the command `passwd admin` to set the password to the user `admin`. It asked me to type in a password and type it
again. Finally, it told me that the password was updated successfully.


### Giving the sudo Privilege

Skip this section if the user to be added is not intended to be an administrator.

The user just added is intended to be an administrator, so it needs to be a `sudo` user. I typed the commad `visudo`, and a file showed up to be
edited. This file is for us to register the new user `admin` to have sudo privileges. 

There's this section of the file:

```
# User privilege specification
root     ALL=(ALL:ALL) ALL
```

Right underneath the `root` listing, I added the same entry for `admin`, so now, it looks like this:

```
# User privilege specification
root     ALL=(ALL:ALL) ALL
admin    ALL=(ALL:ALL) ALL
```

I saved the file - Ctrl-X first, then type Y to overwrite. Now, when admin logs in, it can run commands with the `sudo` modifier. 

Later, we will look at the meaning of `ALL=(ALL:ALL) ALL`. For now, we'll just accept that it's needed to add a sudo user.

### Creating the User's Home Directory

When I logged into a user's account that was just created, the prompt told me that the `/home/<username>` folder did not exist, and I didn't know
that Ubuntu (or probably even Linux) expected that. After some research, I found out that there is a command, but I had to be logged in as a sudo
user to do it:

`mkhomedir_helper <username>`

When I log in with that username again, I no longer get the "folder does not exist" message, and the system took me straight to `/home/<username>`.

Note that the `/home` folder is invisible but we can cd into it!





