# Ansys Mechanical 2025 R2 CLI Guide
This is a guide that details how to use Ansys Mechanical 2025 R2 on a HPC with lmod enabled. Note that this guide is subject to change in the future, and may not work for every server. Please look throughout the guide for the information that you need, and also **PLEASE look at the section which describes how to run fluent with a SLURM job. MOST OF THE TIME YOU WILL BE USING A SLURM JOB**



Please note that at the moment of writing, there have been issues with running mechanical CLI on multiple nodes, so the guide below is a way to achieve a single node run with SLURM. 

## SLURM guide (1 node)
1. Enter a tmux window tmux. 

2. Reserve a node to run on `srun -N 1 --time=23:00:00 --pty bash`. This will give you a shell/reserve a singular node and last for 23 hours. Please end this early if you believe you wont use all the time.

3. Load the module. `ml load apps/ansys/mechanical/25.2 `

4. Check which node you are on using `hostname`. What that returns is what you will use for the next step. 

5. Run ansys mechanical from the cli. `ansys252 -b -machines cXX:48 < input.dat > output.out` . This command will run this on node cXX (put the hostname there), using 48 cores (the number after the colon), using your input data, and output it to your output.out file. 

6. If you want to leave this simulation running and exit ssh, press Ctrl B , then press D, then type exit in your terminal. If you want to see the results of when you previously did that, type tmux attach and it should reattach that session. (To list all your sessions, type tmux ls, and to reatach a specific one do tmux attach -t sessionid)