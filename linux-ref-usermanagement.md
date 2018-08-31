# Linux User Management

## Changing the Root Password
The root account is something we probably won't touch much, but it is said that we should change the password of this root account.

So at the prompt, we type

`passwd`

then hit Enter. We'll be asked to type a password, and type it again. If all goes well, we'll get
the message "password updated successfully".

## Adding new Users

We can only add new users from the root account, or from an account with sudo privileges. The command is

`sudo useradd {username}`

where `{username}` will be replaced by the desired username. Note that from now on, we will always surround parameter values with `{}` to denote what it is
that the user should replace with the desired value, and the value should not include the opening and closing braces.

We will be returned to the prompt with no confirmation or error messages. The new user was actually added.

We will then need to set the password to the new user:

`sudo passwd {username}`

We will be prompted to enter a password, and again, then we should get the confirmation.

### Giving the sudo Privilege

Skip this section if the user to be added is not intended to be a sudo (superuser) account.

The user just added is intended to be an administrator, so it needs to be a `sudo` user. At the prompt, type

`visudo`

A file's contents will show up and we would need to edit it. This file is for us to register the new user `admin` to have sudo privileges. 

There's this section of the file:

```
# User privilege specification
root     ALL=(ALL:ALL) ALL
```

Right underneath the `root` listing, we need to add a similar entry for the user, so:

```
# User privilege specification
root          ALL=(ALL:ALL) ALL
{username}    ALL=(ALL:ALL) ALL
```

To save, we need to Ctrl-X first.

Later, we will look at the meaning of `ALL=(ALL:ALL) ALL`. For now, we'll just accept that it's needed to add a sudo user.

### Creating the User's Home Directory

When a user's account was just created, they did not get a home directory, which would be the directory that the user will be in once they've successfully logged in.
So, we will need to give the user a home directory with the command

`sudo mkhomedir_helper {username}`

Note that the `/home` folder is invisible but we can cd into it!