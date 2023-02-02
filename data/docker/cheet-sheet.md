# run container in interactive mode
docker container run -it alpine sh

# run container in background 
docker container run -d --name web nginx

# inspect container 
docker container inspect container_name

# list running containers
docker container ls
docker ps

# list running & stopped containers
docker container ls -a

# show diffs to container
docker container diff container_name

# export current state of container
docker container commit old_name new_name

# list local images
docker image ls

# ctrl + p, ctrl + 1 will detach from container

# pull image to local
docker image pull alpine

# create container without starting
docker container create --name image1 -it alpine sh

# start a container
docker container start image1

