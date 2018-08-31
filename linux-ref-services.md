# Linux Services

Services are special programs that run in the background, waiting for some input (not necessarily from the user), and then doing something based on the input.

## Viewing All Services

`service --status-all`

The sudo modifier is not required.

## Manipulating Services

`sudo service {serviceName} {command}`

where `{command}` can be `start`, `stop`, or `restart`.