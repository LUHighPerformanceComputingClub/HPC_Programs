## Blender Version X.X Guide
This is a guide that details how to use Blender on an HPC with OpenHPC/lmod. This guide is not absolute and will not work in EVERY environment, but should work in most standard environments. If you have issues, please contact your HPC administrators.

## Getting Started
First, ensure that you have a username and password to ssh into the machine. If you do not have a username and password, contact a member of the HPC administration team for access. 

To login to the specified machine, use the command `ssh user@IP`, where user is your username, and IP is the IP address of the device, to login. There may be other alternative login methods, please contact your HPC administration team for more information. 

## Loading the blender modules
Second, ensure that the version of Blender on your local machine is a similar, or the same version of Blender that is on the HPC. If they are two separate versions, it may cause stability issues and may not render. To list the versions of Blender available, use the command `ml avail` to list the versions available. Use the command `ml load moduleName`, where `moduleName` is the name of the desired version of Blender, to load it. You may need to load additional environment variables such as gnu12.

## Uploading the files
At this point, I would suggest uploading the files to the server, as it may be large files. For blender, I would recommend uploading your .blend files. To upload the files, include all the files in a folder, then either zip the directory (once on the server will have to use the `unzip` command). Uploads can be done via a variety of methods, but the primary method is using the `scp` command.  Here are commands to do so:
1. Zip files (do this on your local device): `tar -czf folderName.tar.gz folderName/`  (where folderName/ is a directory with all your .blend files)
2. Upload to server: `scp folderName.tar.gz user@SERVER_IP:~` (This will upload to your home directory, enter the desired path after te colon after the server IP address)
3. Unzip files (after transfer, do this on the server): `tar -xzf folderName.tar.gz`





## Running a blender render (no SLURM)



## Running a blender render (with SLURM)


## Copying the files back over

After the render has been completed, you will want to copy all of the files in the `/output` folder back to your local system. Here is what you will want to do:

1. Zip the files: `tar -czf output.tar.gz output/`
2. Copy the files back to your machine (run from your local machine): ``scp user@SERVER_IP:/path/to/zip .`
3. Unzip the files on your local machine: `tar -xzf folderName.tar.gz`

## Recap
To recap, here are the general instructions that will have to be followed:
- Get access to the HPC with a user and password, and login to the HPC
- Load environment variables (optional: put the variables in .bashrc so it loads when you login)
- Upload blender files in a zip folder, then unzip it. 
- Start the blender render (with or without a SLURM batch script)
- Zip the image files, and copy it back to your local computer to view.