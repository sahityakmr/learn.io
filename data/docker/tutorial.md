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

# tells max u limit of a system
ulimit -a 

# set ulimit at container level
sudo docker container run -it --ulimit nproc=10 --rm alpine sh

# give set of CPUs of host
sudo docker container run -d --name cpuLimitedAlpine --cpuset-cpus="0" alpine top


# give set of CPUs of host
sudo docker container run -d --name memoryLimitedAlpine --memory="200m" alpine top
```

### Some Advance Operations
```
# get into container with different program
sudo docker container exec -it web sh

# policy to restart container if it dies due to some issue
sudo docker container run -d --restart=always --name web nginx
sudo docker container run -d --restart=on-failure:3 --name web nginx

# copy a file from host to container
sudo docker container cp index.html web:/usr/share/nginx/html

# give a label to a container
sudo docker container run -d --label env=dev nginx

# will help filtering container by label
sudo docker container ls --filter label=env=dev

# get all the ids of the running containers
sudo docker container ls -q

# get all the ids of the containers
sudo docker container ls -q -a

# remove all containers
sudo docker container rm -f `sudo docker container ls -q`
sudo docker container rm -f `sudo docker container ls -q -a`

# get the IP Address of a container
sudo docker container inspect --format='{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
```

### Previlaged access inside container
```

# start container with host network
sudo docker container run -it --net=host alpine sh

# try following op, it's not permitted even if you are root user
ifconfig enp1s0:1 192.168.1.0 up

# run same container with privilaged mode
sudo docker container run -it --privileged --net=host alpine sh

# following ops should be allowed now
ifconfig enp1s0:1 192.168.1.0 up
ifconfig enp1s0:1 192.168.1.0 down
```


### Docker Containers & Images
```
# save a docker image
sudo docker image save modded:latest > /tmp/myimage.tar
```
- In order to share image over some repository, we need to push that image to docker registry.
- An image can be uniqely identified using image name that is formed by
  - registry/repository:tag
  - eg: https://index.docker.io/v1/nginx:1.19.6

```
# login to a registry
sudo docker login 

# push image | this will fail as this is an attempt to push image to docker hub's root
sudo docker image push modded:latest

# to resolve, use symlink as following, create a repository including your dockerhub username
sudo docker image tag modded:latest sahityakmr/modded:latest

# then push using
sudo docker image push sahityakmr/modded:latest

# to pull this image anywhere, use following
sudo docker image pull sahityakmr/modded:latest
```


### Working with Container Images using Docker:
```
# search image on registry
sudo docker search nginx

# digests
sudo docker image ls --digests

# inspect images
sudo docker image inspect nginx:alpine

# to prune unused images
sudo docker image prune

# to remove an image
sudo docker rm imagename
```

### [Dockerfile](./res/Dockerfile)
```
# create a dir for experimentation
sudo mkdir docker

# create a Dockerfile
sudo touch Dockerfile

# edit dockerfile and add the commands
sudo nano Dockerfile

# run build image from the same directory
sudo docker image build -t myimage:mytag .

# to list newly created image
sudo docker image ls

# run this container
sudo docker container run -it myimage:mytag sh

# run build image from the same directory again
sudo docker image build -t myimage:mytag .

# run the container again | date will come same as docker uses caching
sudo docker container run --rm myimage:mytag cat f1

# run build image from the same directory again
sudo docker image build -t myimage:mytag . --no-cache

# run the container again | date will come different as we passed no-cache flag
sudo docker container run -it myimage:mytag cat f1

# COPY command
# create two files in the source dir say `f1.py` and `f2.py`
# repeat the creation if image and container to see if these f1 and f2 are copied,

# ADD command | ADD srcURL /tmp
# Add can work as copy but also dump URL content to target dir

# CMD command
# default command
sudo docker image build -t myimage:cmd . --no-cache

# to avoid overriding default command
# use ENTRYPOINT command
```

- A Dockerfile may have multiple run instructions, but this will cause a larger image size, 
  as each layers metadata will need storage.
- So, a preferable option would be to combine multiple RUN ops using \&&

```
# EXPOSE command | port on which we can listen the container
EXPOSE 80
```

#### [ENTRTYPOINT with CMD](res/DockerfileEntryWithCMD)
If ENTRYPOINT and CMD are used together, the default command is specified and finalized by      ENTRYPOINT but CMD will tell the arguments which can be overriden at runtime

e.g:
```
sudo docker container run myimage:cmdentry /usr
```

### MultiStage Dockerfile
#### [Dockerfile](res/DockerfileMultiStage)

### Docker Networking
 [take screenshot from 04:36:30]

 ```
 # linux commands to show bridge interface
 brctl show

 # try bringign up a container and run this command, will show connected virtual interfaces

# port mapping
# sudo docker container run -d -p <host_port>:<container_port> nginx:alpine
sudo docker container run -d -p 8080:80 nginx:alpine

# try hitting: http://192.168.49.218:8080/

# random port mapping | will map a random port from host to an exposed port of container
sudo docker container run -d -P nginx:alpine

# list docker networks
sudo docker network ls

# bridge -> docker0
# host -> --net=host
# null -> --net=null
 ```

 This all is to support container communication within single host.
 But in order to make communication amongst containers accross the nodes, we use two other drivers
 1. MACvlan
 2. Overlay

#### Create User Defined Network Bridge
[take screenshot from 04:57:00]


```
# custom network creation
sudo docker network create mynet

# check it in docker networs
sudo docker network ls

# inspect
sudo docker network inspect mynet

# create two containers with this custom network
sudo docker container run -d --net=mynet --name backend nginx:alpine
sudo docker container run -d --net=mynet --name frontend nginx:alpine

# inspect | both containers should be listed
sudo docker network inspect mynet

# exec into one of the container
sudo docker container exec -it frontend sh

# try 
ping backend

# this will work, as a custom network provides dns for free :D
# docker0 network doesn't provide free DNS

```

### Container Storage & Volume Management
[take screenshot from 05:10:00]

```
# defining custom destination using -v
sudo docker container run -it --name volc -v /data alpine sh

# Note: this destination is mounnted somewhere on storage outside the docker
# this will be visible even after container is destroyed

# find the source of this volume using
sudo docker container inspect volc

# create a volume
sudo docker volume create myvol

# create container using this volume
sudo docker container run -it -v myvol:/data alpine sh

# this volume is persistant and will be available to other containers as well

mkdir /mnt/shared
echo "Docker Training" > /mnt/shared/index.html
sudo docker container run -d -v /mnt/shared:/usr/share/nginx/html/ nginx:alpine
```
`End TimeStamp: Video1 : 05:27:30`
 