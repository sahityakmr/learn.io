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

# Docker manipulates above system properties of host machine to run a container.



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



sudo apt install docker.io -y
docker version

-> containerd is a program that can spin up multiple runC or other OCI compliant runtimes.
-> Docker Engine communicates with containerd
-> docker client and server communicates over unix socker using REST calls, or remote server over TCP calls.
-> hub.docker.com is registry of image repository

sudo docker info
docker container --help
docker container run -i -t alpine sh
    cat /etc/os-release
    ps aux
    ip a

exit -> this will kill the container as well, so use flag "-d" as following

docker container run -d --name web nginx

docker container ls or docker ps

docker container inspect container_name

ps aux | grep 5624(pid from inspect) -> gives nginx

curl 172.17.0,2

docker container ls -a ->Gives running and stopped containers

docker container run -i -t alpine sh
ctrl + p, ctrl + 1 will detach from container

# to export the container image in current state
docker container diff container_name
docker container commit old_name new_name

docker image ls

docker container run -it new_name sh >> here we can see f1 and f2

docker image pull alpine 

docker container create --name image1 -it alpine sh
docker container start image1

=> docker containr run is the combination of above two commands.

docker container attach container_name

docker container attach web >> ctrl + c >> this will kill nginx

docker container logs container_name >> gives stdout stderr

docker container stop container_name

docker container start container_name
docker container restart container_name

docker container pause container_name
docker container unpause container_name

docker container rename container_name_1 container_name_2

1:38:30

docker container rm container_name
 
 # removes container once work is done
docker container run --rm --name container_name alpine ping -c 3 google.com

# change hostname
docker container run -h new_hostname -it --rm alpine sh

# change working directory
docker container run -w /var -it --rm alpine sh

# setting env variable
docker container run --env "WEB_HOST=172.168.1.1" -it alpine sh

## ulimit -a
tells max u limit of a system

1:44:40