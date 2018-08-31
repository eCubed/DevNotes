# Linux Programs

This reference deals with installing, updating, possibly configuring, and removing programs from the system.

## Updating the System Before Installing a Program

Unfortunately, we cannot just download and install anything and all the dependencies of what we are about to install will be resolved. Typically, we first
update the system before we install anyting.

`sudo apt update`

## Installing a New Program

`sudo apt install {program1} [{program2} [{program3} [..]]]`

There can be more than 1 program to install in one command. The name of the program is most likely from a guide on the Internet that instructs how to install
a specific program.