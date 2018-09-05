# Environment Variables in Linux

We are interested in Environment Variables because our production web applications will rely on them for sensitive data we wouldn't want to expose on appsettings.json
files. Asp.NET Core will read them for us on whatever platform we are hosting our apps in.

Our goal in this article is to set system-wide, persistent variables and their values.

## Viewing All Environment Variables

`printenv`

No sudo is necessary.

## Viewing the Value of One Environment Variable

`echo ${VAR_NAME}`

where `{VAR_NAME}` is our environment variable name. That leading `$` must always be there.

If the variable name we specified doesn't exist, we'll get a blank response.

## Setting Environment Variables

For this particular distro, we will store environment variables in `/etc/profile.d/`. We declare environment variables by first creating a file in this folder
with the extension `.sh`. So, let's create `/etc/profile.d/myvars.sh`, and here are its contents:

```
HELLO_MESSAGE='Hello. This is the hello message'
FAVORITE_NUMBER='142857'
```
Once saved, it will not yet take to effect. If we try to `echo ${VAR_NAME}`, we won't see the values we set. What we'll need to do is log out. When we log in
again, any user, and we echo those variable names, we'll get their values in their output.



