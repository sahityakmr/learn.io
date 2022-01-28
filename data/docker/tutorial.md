History of Containers: Chroot

CurrentOS:
cat /etc/os-release 

sudo apt install debootstrap
sudo mkdir /mnt/chroot-ubuntu-trusty
sudo debootstrap trusty /mnt/chroot-ubuntu-trusty http://archive.ubuntu.com/ubuntu
sudo chroot /mnt/chroot-ubuntu-trusty /bin/bash
cat /etc/os-release 
same as
cat /mnt/chroot-ubuntu-trusty/etc/os-release 


Containers:
namespace: for resource isolation
cgroups: for resource allocation

cd /proc/ : dir contains info about all the process on the system
cd pid/ns: to jump into namespace of that process id
sudo ls -l


cd /sys/fs/cgroup
cd memory/ (for memory configs)
cat memory.swappiness
All pids defined in /sys/fs/cgroup/memory/tasks file will have swappiness value as defined in /sys/fs/cgroup/memory/memory.swappiness file

create new dir in memory folder, it will automatically create other config files.
Define the new swappiness value here

# DOcker manipulates above system properties of host machine to run a container.



Open Container Initiative (OCI - Spec)
runC is a program that is first implementation of this spec.


sudo apt install runc

cd /mnt/
sudo runc spec (creates a config file)
update the root dir in this file to "chroot-ubuntu-trusty"
sudo runc run <container_name>
e.g. sudo runc run my_first_container

can experiment updating the process.args to `date` command
or removing network namespace from config (container will use namespace of the host system)


::47:21