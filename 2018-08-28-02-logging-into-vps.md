# Logging Into the VPS

I installed PuTTY so I can access the VPS server. I already have my credentials.

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

