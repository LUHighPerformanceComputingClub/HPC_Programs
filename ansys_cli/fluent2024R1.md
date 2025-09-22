# Ansys 2024 R1 Guide
This is a guide that details how to use Ansys 2024 R1 on a HPC with lmod enabled. Note that this guide is subject to change in the future, and may not work for every server. Please look throughout the guide for the information that you need, and also **PLEASE look at the section which describes how to run fluent with a SLURM job. MOST OF THE TIME YOU WILL BE USING A SLURM JOB**


## Getting Started
First, ensure that you have a username and password to ssh into the machine. If you do not have a username and password, contact a member of the HPC administration team for access. 

To login to the specified machine, use the command `ssh user@IP`, where user is your username, and IP is the IP address of the device, to login. There may be other alternative login methods, please contact your HPC adminsitration team for more information. 

## Fluent CLI basics
There are a variety ways to start fluent. I will leave some references below for quick ways to run it, but I would higly suggest checking out the fluent guides on how to run it from the cli.

Link: https://ansyshelp.ansys.com/public/account/secured?returnurl=/Views/Secured/corp/v242/en/flu_tcl/flu_tcl.html

## Fluent CLI options
Here is a breakdown of what is needed to start the application from the CLI

` fluent <type> -g -tNUM example.cas `

- `fluent`: the application that is being called
- `<type>`: The type of fluent that is being run. Options are `2d`,`3d`, `2dpp` (2d double precision), and `3dpp` (3d double precision)
- `-g`: Run the fluent cli without a GUI (will not run unless this option is specified)
- `-tNUM`: Run the fluent with a specific number of cores. So for example, if you want to run fluent with 10 cores, it would be `-t10`, or 15 cores would be `-t20`
- `example.cas`: Your .cas file that you uploaded to the server.

So, as example, to run fluent 3d with 4 cores, use the following command to start the application.

` fluent 3d -g -t4 example.cas`


## Fluent with SLURM built in
Fluent should typically be run with SLURM, and in most cases jobs can only be run across nodes with SLURM. SLURM allows users to queue compute jobs in an orderly fashion, utilizing resources effectivley and preventing one user from hogging all the compute resources of a system. Fluent has SLURM functionality built-in, one just needs to know how to call it.

In many systems, aliases are set up in order to run fluent with SLURM to make it easier on the users. Here is how it works on our systems:


`fluentSlurm 3d -g -t10 --jobtime=20`
- `fluentSlurm` is a bash script that does the options for you. The bash script is included at the bottom of this document. 
- `--jobtime`: Specifies the time that the job needs to run for



<br><br>
In other systems, alias are NOT set up, and the user may need to run these options manually. Here is a base example of fluent with SLURM:

`fluent 3d -g -t4 -scheduler=slurm -scheduler_opt="--time=20" -scheduler_opt="--ntasks=4"`
- `fluent`: the application that is being called
- `<type>`: The type of fluent that is being run. Options are `2d`,`3d`, `2dpp` (2d double precision), and `3dpp` (3d double precision)
- `-g`: Run the fluent cli without a GUI (will not run unless this option is specified)
- `-tNUM`: Run the fluent with a specific number of cores. So for example, if you want to run fluent with 10 cores, it would be `-t10`, or 15 cores would be `-t20`
- `-scheduler_opt`: Run a slurm sbatch option. For example, `--time` specifies the time in minutes for the job, `--numtasks` specify the number of cores for the SLURM job, and should match the number after t. 

## Fluent on a SLURM job
Fluent also can, and should, be run with a SLURM job for simulations on an HPC. To run it =with SLURm, there are a few extra requirements that are needed.
- SLURM job script
- Journal file for headless start

First, here is a sample SLURM script, as well as a breakdown of what it is. Here is my `example.sbatch` file:

```bash
#!/bin/bash
#SBATCH --job-name=fluent_mpi
#SBATCH --partition=normal
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8
#SBATCH --time=01:00:00
#SBATCH --output=fluent_log.log

# Loading modules
ml load gnu12/12.4.0
ml load mpich/3.4.3-ofi
ml load libfabric/1.19.0
ml load apps/ansys/fluent/24.1 


# Total tasks = nodes * tasks-per-node
# Fluent detects cores from -t option
# -g = no GUI, -t16 = 16 cores
fluent 3d -g -t16 -i solve.jou
```
### Slurm Script breakdown

1. Sbatch arguments
- `#SBATCH --job-name=fluent_mpi`: The name that you would like to call the job (in this case it is fluent_mpi)
- `#SBATCH --partition=normal`: The partition of clusters that the job is run against (use the command `sinfo` to list the partition)
- `#SBATCH --nodes=2`: The number of nodes that this will be running against (use the command  `scontrol show node` to list the information for the nodes). This number CANNOT exceed the number of nodes in the partition. 
- `#SBATCH --ntasks-per-node=8`:The number of cores that this job will run against. This is PER node, so the total cores will be the number of nodes * the number of tasks per node. This number CANNOT exceed the number of threads (logical processors) per node. 
- `#SBATCH --time=01:00:00`: The amount of time for the job to run. This will run for as long as the job runs, or until 1 hour is hit, after which the job will END, even if it is mid computation. There is also a max time limit that a job can have. (use the command `sinfo` to show the time limit)
- `#SBATCH --output=fluent_log.log`: The output log for the job. This isnt the output file for the job, but rather a log if something ends up erroring out or cancelling early. 

2. Loading modules
Loading modules will vary massivley per system. At minimum, the module for the version of fluent that you need **must** be loaded. To search for the module that you will need, use the command `ml avail` or `module avail`.

3. Fluent command 
At the bottom of the sbatch file, enter the command that is needed to run your fluent simulation. Please remembver that the number of cores that you use in the fluent command MUST match the SBATCH nodes * ntasks-per-node. So if you have 10 nodes, and 12 tasks per node 



### Journal file for use with batch script
Secondly, here is what a journal file would look like. Essentially, the commands that you would run from the application TUI (the thing that launches when you run the terminal command), one would enter into this journal file. Here is an example of a journal file:

```bash
/file/read-case example.cas
/solve/set/flux-type yes
/solve/initialize/initialize-flow
/solve/iterate 500
/file/write-case-data result_after_500_iters
/exit
yes
```
In short, this journal reads in my example.cas file, sets the flux type, initializes flow, solves, writes the data to the proper file, then exits. If you do not know how to write a journal file like this, I would suggest using the fluent commands from prior in this guide to run it without a batch script, and expiriment with the commands. Use the **`help`** command after loading fluent, it will be extremley useful. Utilizing resources such as fluent's documentation or AI resources may also be helpful if additional help is required. 


## Recap
To recap, here are the general instructions that will have to be followed:
- Get access to the HPC with a user and password, and login to the HPC
- Load environment variables (optional: put the variables in .bashrc so it loads when you login)
- Upload .cas files to the server so that it can be solved
- Run the ansys fluent application on the HPC through the CLI or a SLURM script (reccomended to do the SLURM script)


## Bash Scripts


### slurmFluent.sh
```bash
#!/bin/bash

# Defaults
MPI=openmmpi
NUM_CORES=2   # default if -t not given
JobTime=3
MinTime=1

# New array to hold args (minus custom flags we parse ourselves)
PASSTHRU_ARGS=()

# Parse command line args
for arg in "$@"; do
    if [[ $arg =~ ^-t([0-9]+)$ ]]; then
        NUM_CORES="${BASH_REMATCH[1]}"
        PASSTHRU_ARGS+=("$arg")
    elif [[ $arg =~ ^--jobtime=([0-9]+)$ ]]; then
        JobTime="${BASH_REMATCH[1]}"
        # don’t add this to PASSTHRU_ARGS (Fluent doesn’t know it)
    else
        PASSTHRU_ARGS+=("$arg")
    fi
done

# Debug (optional)
# echo "NUM_CORES=$NUM_CORES"
# echo "JobTime=$JobTime"

# Launch Fluent with Slurm defaults + cleaned user args
fluent "${PASSTHRU_ARGS[@]}" \
    -pib \
    -scheduler=slurm \
    -scheduler_nodeonly \
    -scheduler_opt="--time=$JobTime" \
    -scheduler_opt="--ntasks=$NUM_CORES"

```