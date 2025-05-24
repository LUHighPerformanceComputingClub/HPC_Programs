# Paraview 5.10.0/5.11.0
This is a guide that details how to use Paraview on an HPC with OpenHPC/lmod. This guide is not absolute and will not work in EVERY environment, but should work in most standard environments. If you have issues, please contact your HPC administrators


## Getting Started
First, ensure that you have a username and password to ssh into the machine. If you do not have a username and password, contact a member of the HPC administration team for access. 

To login to the specified machine, use the command `ssh user@IP`, where user is your username, and IP is the IP address of the device, to login. There may be other alternative login methods, please contact your HPC administration team for more information. 

## Downloading Paraview/Loading the modules
Second, ensure that the version of Paraview on your local machine is the SAME VERSION of Paraview that is on the HPC. If they are two separate versions, it may cause stability issues. To list the versions of Paraview available, use the command `ml avail` to list the versions available. Use the command `ml load moduleName`, where `moduleName` is the name of the desired version of Paraview, to load it. You may need to load additional environment variables such as gnu12. Next, download the client for the version of Paraview off of their website, and set up the client. 

### Links to download Paraview

Link: https://www.paraview.org/download/

(I would recommend installing the versions that include MPI and the larger versions. There are plenty of installation guides out there)

## Uploading the files
At this point, I would suggest uploading the files to the server, as it may be large files. To upload the files, include all the files in a folder, then either zip the directory (once on the server will have to use the `unzip` command).Uploads can be done via a variety of methods, but the primary method is using the `scp` command.  Here are commands to do so:
1. Zip files (do this on your local device): `tar -czf folderName.tar.gz folderName/` 
2. Upload to server: `scp folderName.tar.gz user@SERVER_IP:~` (This will upload to your home directory, enter the desired path after te colon after the server IP address)
3. Unzip files (after transfer, do this on the server): `tar -xzf folderName.tar.gz`


## Starting the Paraview Server (no SLURM)
To start the Paraview server, run the following command:

``mpiexec -np NumCores pvserver --machinefile nodelist --server-port=12400 --force-offscreen-rendering`` 

Here is a breakdown:
- `mpiexec`: Standard mpi command
- `-np NumCores`: Add a number to run against a specific number of cores. Must be under the number of cores on the current node OR include the next option
- `--machinefile nodelist`: The "nodelist" file is the name of YOUR file that includes the devices that this server will run on.
- `pserver`: Command for paraview server
- `--server-port=12400`: Sets the server port to a specific port (set it to a Non standard port, one that is NOT in this list: https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers )
- `--force-offscreen-rendering`: Forces pserver to render offscreen, is needed to run the command


Link: https://ansyshelp.ansys.com/public/account/secured?returnurl=/Views/Secured/corp/v242/en/flu_tcl/flu_tcl.html

## Starting the Paraview Server (with SLURM)

Work in progress, will update as soon as possible. In the meantime, looks at the ansys guide for a general guide on what a SLURM job looks like.

Link: https://github.com/LUHighPerformanceComputingClub/HPC_Programs/blob/main/ansys_cli/fluent2024R1.md


## Setting up the SSH tunnel
Next, the SSH tunnel from your local machine needs to be setup. Open up another command prompt/PowerShell/terminal window, and run this command:

`ssh -L 8890:SERVER_IP:12400 user@SERVER_IP`

What this will do is forward that Paraview server that is on the server to your local machine. Change the number after SERVER_IP (12400 in this example) to the port that you ran the Paraview server on with the `--server-port=12400` flag earlier. The number before the first colon in that command is what port it will be forwarded to on your local machine (in this case 8890). **DO NOT CLOSE THIS SSH SESSION OR YOUR SERVER WILL DISCONNECT**. You must do the SSH tunnel ***after*** the server is running. Try not to use a standard port for your localhost as well, it could cause issues. If there are issues with timeouts, I would suggest using the command `tmux` when you login, it should keep the server running even if you get timed out of your terminal. 




## Configuring the Client
Next, launch the Paraview Client on your local device. Inside of Paraview, select the button at the top that looks like two servers connecting with a green line titled “Connect”. Inside of the connect panel, there will be options at the bottom of the panel, and you will want to select the “Add Server” button.

Next, what you will want to do is fill in the information for your server. Here is the information that you will want to set:
1. `Name`: Set the Name to whatever you would like to call the forward (for example: HPC_Forward or Server_Forward)
2. `Server Type`: Set the Server Type to `Client/Server`
3. `Host`: Set the host to `localhost` or `127.0.0.1`, as you forwarded the server with the SSH tunnel earlier
4. `Port`: Set the port to the port that you forwarded to your localhost (the first number from the SSH tunnel command from earlier, in my example that was 8890)


## Using the client

Now that the connection is set up, click on the server that you want to connect to, and click the Connect button. It may look like the application is erroring out, which is part of the initialization process, so do not worry. It may take a minute or two to connect. To verify that it is connecting, open the terminal where the server was run, it will say "client connected". Once connected, select the folder button in the upper left corner to navigate to where your project files are stored on the HPC, and load in your desired files. Then, at that point, one should be fine to run Paraview as normal. To disconnect, click the disconnect button, and close your connections to the server. 

## Recap
To recap, here are the general instructions that will have to be followed:
- Get access to the HPC with a user and password, and login to the HPC
- Load environment variables (optional: put the variables in .bashrc so it loads when you login)
- Upload paraview project files to the server
- Start the pserver (with or without a SLURM batch script)
- Set up the SSH tunnel to your localhost
- Configure the client to call your localhost
  