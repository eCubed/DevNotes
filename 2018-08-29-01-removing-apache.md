# Removing Apache

We decided that we want the Nginx web server instead of the Apache that came with the system.

We can view that Apache is installed because it's included in the list when we type the command:

`service --status-all`.

We have Apache2 running and active.

Maybe both Nginx and Apache can coexist, however, it's best that only have one web server to avoid conflicts in port. As Apache is running,
it already has a hold of ports 80 and 443, which Nginx needs to listen to. So, we need to remove Apache from the system. It is not a straight-forward
process, so we'll need to perform the following steps. We must be logged into a sudo account besides root.

We need to first stop the service.

`sudo service apache2 stop`

We then need to purge it. Purge means that it will get rid of the program including any configuration files, which we want:

`sudo apt purge apache2 apache2-utils`

We then need to delete the dependencies of apache2 that are not dependencies of other applications:

`sudo apt autoremove`

But there may still be folders, like preferences folders that contain or refer to apache2. To see those, we will need to run:

`whereis apache2`.

This will list the files and folders that contain the substring `apache2`. For each of those, we will have to "rimraf" or manually delete them:

`sudo rm -rf <path>` for each path listed by the `whereis` command.


