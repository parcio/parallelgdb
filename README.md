# Parallel GDB
This program is for debugging parallel programs either on the local machine or on a remote cluster using SSH. It provides support to launch the processes via mpirun and srun.

# Structure
This debugger consists of two parts: 
1. The GUI server running on the host machine
2. The clients running on the host machine or the remote cluster

# Building
To build these two separate executables, you can use the following command from the top level directory.

	mkdir bin
	make

This will generate a `server` and a `client` executable in the `bin` directory.

If you only want a specific executable you can run 

	make server

or 

	make client

to get the corresponding executable.

Use

	make clean

to delete all generated files.

# Dependencies
## Compile-Time
The following dependencies must be available for building:
- libgtkmm-3.0
- libgtksourceviewmm-3.0
- libssh

On Debian based systems, you can use the following command to install them:

	apt install libgtkmm-3.0-dev libgtksourceviewmm-3.0-dev libssh-dev

## Run-Time
For the debugger to work, the following programs need to be installed:
- mpi / slurm: To start the client instances.
- gdb: To debug the target program.
- socat: To handle the I/O of the gdb instance and the target instance.

Furthermore, when debugging on a remote cluster, the `client` executable needs to be copied to or build on this machine.

# Using the debugger
The debugging server is started by using the command:

	./bin/server

In the start dialog you need to set all the corresponding paths and parameters. This configuration can be exported and imported at the next start.

The server will start the specified number of clients, each of which will start the gdb instance, running the target program, and two socat instances, handling the I/O of gdb and the target.

If SSH is enabled, the server logs on to the remote cluster and starts the clients there.

## Custom launcher command
If you need to make specific changes to the start command, or need to use a completely different launcher, you can check the Custom Command option in the startup dialog. This command will invalidate the set paths, IP address and partition. SSH options will not be influenced. 

For the server to know how many connections to open the number of processes and nodes still needs to be set in the startup dialog. Make SURE they match up with what you set in your custom command.

The client supports two possibilities to set its rank. Under normal conditions it will try to read the environment variables set by OpenMPI/MPICH/slurm:

	OMPI_COMM_WORLD_RANK
	PMI_RANK

If neither of those is set (not using OpenMPI/MPICH/slurm) the Parallel GDB specific environment variable:

	PGDB_RANK

is read. If desired, this one can be set by the user.

Furthermore there are two command line arguments which also can be used to set the rank. First is 

	-r <rank>

which directly sets this processes rank.

Second is

	-e <name>

which adds an additional environment variable name which is tried to read before the others.