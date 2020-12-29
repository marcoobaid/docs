# Docker Cheatsheet
***

## Basic Commands

`docker ps` --> (list containers)

`docker ps -a` --> (list all containers running or not)

`docker stop` --> [container ID or name]

`docker rm` --> (stop and remove a container)

`docker images` --> (list of on our host and their status)

`docker rmi [image]` --> removes an image (*Stop and delete all dependent containers first*)

`docker pull [image]` --> only pull and not run the image

`docker run ubuntu sleep 5`

`docker exec [container name] cat /etc/hosts`

`docker run kodekloud/simple-webapp`

`docker run -d kodekloud/simple-webapp` --> runs in background

`docker run --name webapp nginx:1.14-alpine`

`docker attach [name or id of container]`

`docker run -it kodekloud/simple-prompt-docker`

`docker run -p 80:5000 kodekloud/simple-webapp` 

`docker run -v /opt/datadir:/var/lib/mysql mysql`

`docker run -e APP_COLOR=blue simple-webapp-color`

`docker inspect simple-webapp-color`

`docker build Dockerfile -t marcoobaid/my-custom-app`

`docker push marcoobaid/my-custom-app`

`docker history marcoobaid/my-custom-app`


## Run

#### TAGS
- `docker run redis:4.0` -> redis:4.0 is called a "tag". If we don't specify a tag, docker will default to "latest" tag.
- docker hub has all the tags associated with all the images.

#### STDIN
- By default, docker runs in an non-interactive mode. To enter interactive, use: `docker run -i kodekloud/simple-prompt-docker`.
- `docker run -it kodekloud/simple-prompt-docker` -> enter interactive mode and attach to pseudo terminal.

#### PORT-FORWARDING
- `docker run -p 80:5000 kodekloud/simple-webapp` -> -p allows to map external port 80 (docker host) to internal port 5000 (docker container) for "simple-webapp" container app.

#### INSPECT
- To presist data, we need to map a directory outside the docker container on a docker host to a directory inside a container. `docker run -v /opt/datadir:/var/lib/mysql mysql`
- docker inspect blissful_hopper -> returns a jason file with more details about blissful_hopper container (used when required to find details about the container).

#### LOGS
- `docker logs blissful_hopper` -> displays stout logs written for the container blissful_hopper.


## Environment Variables
- The docker image is run  with a set of environment variable. To run multiple instances of a container with different values for a specific environment variable (i.e APP_COLOR), run:
        `docker run -e APP_COLOR=blue simple-webapp-color` 
- To find the value of an environment variable on a container that is already running, run:
        `docker inspect simple-webapp-color`  (look under section "Env").

## Images
- Create Dockerfile (series of commands, instructions and arguments, executed sequentionally to layer our container)
- Build the image (local) --> `docker build Dockerfile -t marcoobaid/my-custom-app`
- Make the image available on Docker Registry (public) --> `docker push marcoobaid/my-custom-app`
- Docker images are typically based on other base images (i.e. ubuntu, alpine, ... )
- `docker history marcoobaid/my-custom-app` --> displays information about the layers of the image we built 
- Build steps are cached by Docker. In case a step fails and we rebuild, Docker will resume starting with the failed step.
- We can containerize everything, including web browsers, spotify, text editors, ... , etc.

## CMD vs ENTRYPOINT
- Container runs as long as the process within it runs. If no process is running, it exits.
- CMD in Dockerfile defines the process that runs when we start the image.
- Or, # docker run ubuntu [COMMAND] --> This will override the default command within the image (if one exists) 
- To permanently make th above command part of the image, we need to add a CMD line in Dockerfile and rebuild the image.
- Format can be:  CMD sleep 5 OR CMD ["sleep","5"] --> jason format.
- ENTRYPOINT is used to allow us to pass a value to variable when we start a container. For example, put the following in Dockerfile and buid the image:
        `ENTRYPOINT ["sleep"]`
        Now, run the command below. This will assign value 10 to sleep, which instructs the image to sleep for 10 seconds when it is started.
        `docker run ubuntu-sleeper 10`
- To assing a default value for ENTRYPOINT, in case one is not passed when image is run, add the following to Dockerfile and build image (must be in json format):
        `ENTRYPOINT ["sleep"]`
        `CMD ["5"]`
- To modify ENTRYPOINT at runtime, use: `docker run --entrypoint sleep1 ubuntu-sleeper 10`

## Networking
- Types:    
    - **Default** - default networks when first *installed*
    - **Bridge** - private (internal) 172.17 range 
                 - Used by containers when they are first created
                 - We use port-forwarding to give access to containers from external network
    - **None**   - Containers are not associated with any networks (internal/external)
    - **Host**   - Containters are associated with host network. This means that all exposed ports are immediately accessible externally.
- **User-Defined**
        - `docker network create --driver bridge --subnet 182.18.0.0/16` custom-isolated-network
        - docker network ls
- **Identify network settings for a particular container**
        - `docker inspect blissful_hopper`  *(check "Networks" section)*
- **Embedded DNS** 
        - docker has a built-in DNS that allows containers to reach each other using container names
        - Built-in DNS server always runs as **127.0.0.11**
- **Isolation**
        - Docker uses "namespace" technology that guarantees isolation within the same node or host.
        - Docker uses virtual ethernet pairs to connect containers together.
        

## Storage
- Docker stores its filesystem on the host under **/var/lib/docker/volumes**
- Built docker image is **"read-only"** --> containers are created from read-only images (image layers) and docker adds a writeable "container layer".
- When a container is destroyed, the container layer is removed (which also removes any changes and/or saved data within the container)
- The image layer is shared by all containers created using this image.
- Modifying an app file baked into a container image, will cause docker to copy the file to the container layer (copy-on-write). All changes to this file will remain on the cotnainer layer.
- **VOLUME MOUNTING**
        To presist data/changes on the container layer, we would need to add a presistant volume to the container.
            `docker volume create data_volume`
            `docker run -v data_volume:/var/lib/mysql mysql`  *(if data_volume has not been created, this command will cause docker to create the volume)*
- **BIND MOUNTING**
        To mount a volume that "binds" to an external storage to the docker host, we would need to this:
            `docker run -v /data/mysql:/var/lib/mysql mysql`  *(/data/mysql is the volume mounted on the docker host and not on the default docker volumes path)*
- There is a new (preferred) syntax to mount Docker volumes:
            `docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`
- Common storage drivers used in Docker are: aufs, zfs, btrfs, device mapper, overlay, overlay2

## Compose
- https://docs.docker.com/compose/compose-file/
- There are v1, v2, and v3.x versions of composer. Check the documentation above for details.

## Registry
- Essential repository of Docker images.
- Image: `[user or org name]:[image name]`
- If only one name is provided, then it is assumend that the user/org name is the same as the image name. For example, `image: nginx = image: nginx:nginx`
- (If location not specified) Default location of registery is "Docker Hub" at docker.io.
- As images are updated, they are pushed to Docker Hub and users will pull the latest.
- Other registeries are, for example, Google Registery at gcr.io
- Private or internal registery is used to host internal images.
- To access Docker private registery: `docker run private-registery.io/app/internal-app`  *(will require username/password)*
- Deploy your own registery using docker image "registry"
        `docker run -d -p 5000:5000 --name registery registery:2`
        `docker image tag my-image localhost:5000/my-image`
        `docker push localhost:5000/my-image`
        `docker pull localhost:5000/my-image`
        `docker pull 192.168.56.100:5000/my-image`


## Engine

- Docker Daemon - process managing Docker objects (containers, images, volumes, networks).
- Rest API - Interface for programs to talk to the daemon and provide instructions.
- Docker CLI - commandline interface that uses Rest API to talk to the Docker daemon. It can run on a remote machine or the docker host.
    `docker -H=10.123.2.1:23275 run nginx`
- Docker uses "namespaces" to isolate workspaces process ID, Network, Moun, interProcess, and Unix timesharing.
*For example, PID:5 on Linux System (host) is also given PID:1 in the container namespace that only container prcoesses can see.*
- Docker host and containers share the same system resources (cpu and memory). *By default, there is no resitriction on a container resource utilization.*
 - Using cgroups (Control Groups), docker can restrict the amount of resources utilized by a container.
            `docker run --cpus=.5 ubuntu`
            `docker run --memory=100m ubuntu`
    