# RPi-Cluster  
How to create a Raspberry Pi cluster using MPI, NFS, and Python

## Set up the Pi's  
[My rapsbery pi set up](https://github.com/jorgenmiller/Raspberry-Pi-Setup)  

## Install the latest MPICH2  
install from [here](http://www.mcs.anl.gov/research/projects/mpich2/)  

Navigate to the download and uncompress it  
`tar -xzf <mpich-install-folder>`  

Enter mpi folder  
`cd <mpich-folder>`  

Install. Disabling fortran shortens install time, but it still takes a while!    
`./configure --disable-fortran`  

Build and install mpi (This takes a looong time)  
`make; sudo make install`  

For info on your finished install:  
`mpiexec --version`  

More info from [mpi tutorials](http://mpitutorial.com/tutorials/installing-mpich2/)  

## Install MPI for Python  
MPI is orginally meant to be used with C++, but I wanted to use python. This is a handy library for that!  

`sudo pip install mpi4py`  

A hello world to test mpi4py  
`mpiexec -n 5 python -m mpi4py.bench helloworld`  

More mpi4py information [on the docs](https://mpi4py.readthedocs.io/en/stable/install.html)  

## Edit hosts to make aliases  
Aliases will allow you to refer to each node without using its IP address.  

`sudo nano /etc/hosts`  
Add `<IP-address> <alias>` to the file for each node  
Alias all nodes to numbered work nodes. One node should also be aliased to master.  

## Create new user  
New users with the same name on all nodes makes setup easier  
`sudo adduser mpiuser`  

Give `mpiuser` sudo access  
`sudo usermod -aG sudo mpiuser`  

Login to mpiuser  
`su - mpiuser`  

## Set up SSH  
Enable SSH using `sudo raspi-config`  

Generate keys on the master node  
`ssh-keygen`  

Send the key to each node  
`ssh-copy-id <node>`  

## Set up master NFS  
NFS, or Network File System, allows the work nodes to access the file system of the master node as their own, to ease the exchange of data.  

`sudo apt-get install nfs-kernel-server`  

Make the directory to share  
`mkdir cluster`  

`sudo nano /etc/exports`  
Add the line `/home/mpiuser/cluster *(rw,sync,no_root_squash,no_subtree_check)`  

To share the folder every time there is a change:  
`sudo exportfs -a`  

## Set up work node NFS  
`sudo apt-get install nfs-common`  

Create the folder on the work node  
`mkdir cluster`  

Mount the shared folder  
`sudo mount -t nfs master:/home/mpiuser/cluster ~/cluster`  

Use `df -h` to check the shared directory  

To mount access when booting  
`sudo nano /etc/fstab`  
Add the line `master:/home/mpiuser/cluster    /home/mpiuser/cluster   nfs     x-systemd.automount`  

## Using the cluster  
[mpi4py docs](https://mpi4py.readthedocs.io/en/stable/index.html)  

To run your programs, use  
`mpiexec -n 10 -machinefile nodes python -m mpi4py script.py`  
where 10 is the amount of repitiations, `nodes` is a file with a list of the work nodes, and `script.py` is the job.

For comparison,  
`mpiexec -n 10 python -m mpi4py script.py`  
Runs the program only on the master node.  

## Managing the cluster  
In shell, `time <command>` can be used to run a command and return the time taken to complete the job.  
To power off, you can give each user sudo priveleges and shutdown each easily over ssh.  
