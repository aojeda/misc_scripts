# Network File System Configuration

The following is a tutorial on how to install and configure a Network File System ([NFS](https://help.ubuntu.com/lts/serverguide/network-file-system.html)) that serves several workstations connected to a LAN.

The tutorial will focus on Debian-based GNU/Linux shared network drives, but much of what we will explain can be easily adapted to other architectures. All the commands must be entered on a [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) terminal. Most commands used in this tutorial  need super-user (su) rights to be executed, you can switch to *su* as follows
```bash
sudo su 	# be extra careful, remember that with extra power comes extra responsibility!
```

Let's start by defining a few things:

* server: this is the computer that will run the nfs server process, which is in charge of hosting and exposing the shared directories to the LAN and while keeping the consistency among the files accessed by multiple clients,

* client: this is the computer(s) that access the files hosted by the server.

### Server-side configuration
Install the NFS server:
```bash
apt-get update
apt-get install rpcbind nfs-kernel-server
```
Make sure that the server is running with the following command:
```bash
service nfs-kernel-server restart
```
All the folders that will be shared must be listed on the */etc/exports* file. In principle, it is possible to export any directory in the host computer just by listing it in */etc/exports*, however, it is considered a good practice to isolate all NFS exports in single directory, where the real directories will be mounted with the --bind option. We illustrate this process with the following example.

Let's say we want to export the directories */media/datadrive1* and */media/datadrive2* hosted on the server. First we create the export file system:
``` bash
mkdir -p /export/datadrive1 
mkdir -p /export/datadrive2 
```
Then we update the */etc/fstab* file so that we don't have to manually  mount the drives every time the server is restarted, use any text editor to add the following lines at the bottom of the */etc/fstab* file
```bash
/media/datadrive1    /export/datadrive1   none    bind  0  0
/media/datadrive2    /export/datadrive2   none    bind  0  0
```
Save & close the editor and issue the following command to mount the drives without restarting the server.
```bash
mount -a
```
Next, use any text editor to add the drives that will be shared to the */etc/exports* file. For this example it should look as follows:
```bash
/export            		192.168.0.0/24(rw,fsid=0,no_subtree_check,async)
/export/datadrive1		192.168.0.0/24(rw,nohide,no_subtree_check,async)
/export/datadrive2		192.168.0.0/24(rw,nohide,no_subtree_check,async)
```
where we have used the following syntax,
```bash
directory_to_share       client(share_option1,...,share_optionN)
```
The 192.168.0 are the hypothetical first three numbers of server's ip address and 0/24 means that the folders will be shared with all the clients in the LAN without restrictions. It is possible to set per-client restrictions specifying the client's ip address and modifying its options (see [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-14-04) for details).

#### Security
Suppose that we want to make *datadrive2* accessible to one user only, we can accomplish this easily by modifying the export configuration as follows:
```bash
/export/datadrive2		192.168.0.0/24(rw,nohide,no_subtree_check,async, anonuid=1000,anongid=1000)
```
where the options *anonuid* and *anongid* correspond to the id (uid) and group id (gid) of the user that will be allowed to access the shared folder. The configuration above allows user 1000 to mount *datadrive2* from any client in the network, as mentioned earlier, we can also restrict which clients are allowed to mount the folder by specifying their ip addresses. In this example we used 1000 as a placeholder, you can find these numbers for username *john* with the following command:
```bach
id john
```
Client-side authentication has the inconvenience that user john has to have the same *id* in all the clients in the network. If instead user jane has *id* 1000 in some clients, security will be compromised. Although it is possible to set correct user *id*s in all the client machines, it requires a meticulous work from the network administrator. To sidestep these issues we can use [Kerberos](https://wiki.debian.org/NFS/Kerberos).

Every time that we modify the */etc/exports* file we need to update the NFS export table using the following command
```bash
exportfs -ra
```
At this point we have the NSF server running and properly configured, next we explain the client-side.


### Client-side configuration
Install nfs-common package on the client:
```bash
apt-get install nfs-common
```
Use any text editor to add the shared folders to client's */etc/fstab* file, for this example it should look as follows:
```bash
192.168.0.6:/export/datadrive1	/media/datadrive1   nfs     auto	0	0
192.168.0.6:/export/datadrive2	/media/datadrive2	nfs     auto	0	0
```
where we have assumed that 192.168.0.6 is the ip address of the server.

To mount the drives without restarting the client run the following command:
```bash
mount -a
```
This is all we need on the client side related to the NFS.

### Set Static IP Addresses
The last thing we need to do is to set the ip addresses of the server and client machines to static, so that every time they are restarted they get assigned the same ip and all our configuration files stay consistent. You can find a detailed tutorial [here](https://www.swiftstack.com/docs/install/configure_networking.html). For our example, we first run
```bash
ifconfig -a
```
to find the network configuration of each machine, then use this information to add the following lines to */etc/network/interfaces*
```bash
auto eth1
iface eth1 inet static
address		192.168.0.4
netmask		255.255.255.0
network		192.168.0.1
broadcast	192.168.0.255
gateway		192.168.0.1
dns-nameservers	209.18.47.61 209.18.47.62
``` 
where we have assumed that the network adapter is called eth1. Finally, restart the network server with the following command:
```bash
service networking restart
```
and that's it, we have installed and configured a network file system (NFS) on a LAN of GNU/Linux machines.
