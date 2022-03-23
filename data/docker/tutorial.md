# Starting with Dockers & Containers
## History of Containers: 

### Chroot
`An Operation on Unix-like OS that changes the apparent root directory for the current running process and it's children`


### To get CurrentOS:
```
cat /etc/os-release
```

## Setup chroot with ubuntu trusty image:
```
# Debootstrap : A software that allows installation of a debian based system into a subdirectory of another, already installed OS
sudo apt install debootstrap

# Installing chroot-ubuntu-trusty:
sudo mkdir /mnt/chroot-ubuntu-trusty
sudo debootstrap trusty /mnt/chroot-ubuntu-trusty http://archive.ubuntu.com/ubuntu

# Changing root and starting a bash:
sudo chroot /mnt/chroot-ubuntu-trusty /bin/bash

# Verifying OS name, should be same as the output of `cat /mnt/chroot-ubuntu-trusty/etc/os-release`
cat /etc/os-release 
```

## Containers:
`namespace: for resource isolation`

`cgroups: for resource allocation`

```
# dir contains info about all the process on the system
cd /proc/ 

# to jump into namespace of that process id
cd pid/ns

sudo ls -l

cd /sys/fs/cgroup

# for memory configs
cd memory/ 

cat memory.swappiness
```

- All pids defined in /sys/fs/cgroup/memory/tasks file will have swappiness value as defined in /sys/fs/cgroup/memory/memory.swappiness file

- create new dir in memory folder, it will automatically create other config files.
  
- Define the new swappiness value here

## Docker manipulates above system properties of host machine to run a container.

- Open Container Initiative (OCI - Spec)
- runC is a program that is first implementation of this spec.

```
sudo apt install runc
cd /mnt/

# creates a config file
sudo runc spec
```
update the root dir in [the newly created config file](./runc-config/config.json#L54) to "chroot-ubuntu-trusty"
```
# sudo runc run <container_name>
sudo runc run my_first_container
```

can experiment updating the process.args to [date command](./runc-config/config.json#L10)
or [removing network namespace](./runc-config/config-without-network-namespace.jsonL143) from config (container will use namespace of the host system)

## Docker Installation & Experimentation:
```
sudo apt install docker.io -y
docker version
```

- containerd is a program that can spin up multiple runC or other OCI compliant runtimes.
- Docker Engine communicates with containerd
- docker client and server communicates over unix socker using REST calls, or remote server over TCP calls.
- hub.docker.com is registry of image repository

```
sudo docker info
docker container --help

# to run a docker container in interactive mode using alpine image and default sh command.
docker container run -i -t alpine sh

# to see os-release of the image used in container
cat /etc/os-release

# list the process
ps aux

# list networking configs
ip a

# to exit and kill the container as well
exit 

# use flag "-d" as following to run container in background
docker container run -d --name web nginx

# list running containers
docker container ls
docker ps

# inspect container
docker container inspect container_name

# pid from inspect: 5624 -> gives nginx
ps aux | grep 5624

curl 172.17.0,2

# list running and stopped containers
docker container ls -a

# ctrl + p, ctrl + 1 will detach from container
docker container run -i -t alpine sh

# make two files
date > f1
date > f2

# will show the diff made to the container
docker container diff container_name

# to export the container image in current state
docker container commit old_name new_name

# list all local images
docker image ls

# here we can see f1 and f2, since post file creation container was commited
docker container run -it new_name sh

# pulling an image to local
docker image pull alpine 

# create container without starting
docker container create --name image1 -it alpine sh

# start the created container
docker container start image1

# docker containr run is the combination of above two commands.

# attach to a running container
docker container attach container_name

# attach to web container: >> ctrl + c >> this will kill nginx
docker container attach web 

# gives stdout stderr
docker container logs container_name 

# stop container
docker container stop container_name

# restart container
docker container restart container_name

# pause container : this command suspends all processes in the specified containers. On Linux, this uses the freezer cgroup
docker container pause container_name

# to unpause
docker container unpause container_name

# rename container
docker container rename container_name_1 container_name_2

# remove container
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
```

`End TimeStamp: Video1 : 1:44:40`
