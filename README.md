# Node-RED-homekit-docker
[![Docker Build](https://github.com/NRCHKB/node-red-contrib-homekit-docker/actions/workflows/docker-build.yml/badge.svg)](https://github.com/NRCHKB/node-red-contrib-homekit-docker/actions/workflows/docker-build.yml)
[![DockerHub Pull](https://img.shields.io/docker/pulls/nrchkb/node-red-homekit.svg)](https://hub.docker.com/r/nrchkb/node-red-homekit)
[![DockerHub Star](https://img.shields.io/docker/stars/nrchkb/node-red-homekit.svg?maxAge=2592000)](https://hub.docker.com/r/nrchkb/node-red-homekit)
 
Node-RED-homekit-docker is a Node-RED based project with support for homekit. It is based on the [official Node-RED Docker](https://hub.docker.com/r/nodered/node-red) images with the necessary tools and npm module [node-red-contrib-homekit-bridged](https://www.npmjs.com/package/node-red-contrib-homekit-bridged) installed to run homekit within a docker container. 

## Architecture
Node-RED-homekit-docker is supported by manifest list, which means one doesn't need to specify the tag for a specific architecture. Using the image without any tag or the latest tag, will pull the right image for the architecture required.

Currently, Node-RED-homekit has support for multiple architectures:
- `amd64`   : based on linux Alpine - for most desktop computer (e.g. x64, x86-64, x86_64)
- `arm32v6` : based on linux Alpine - (i.e. Raspberry Pi 1 & Zero)
- `arm32v7` : based on linux Alpine - (i.e. Raspberry Pi 2, 3, 4)
- `arm64v8` : based on linux Alpine - (i.e. Pine64)

**Note**: Currently there is a bug in Docker's architecture detection that fails for arm32v6 - e.g. Raspberry Pi Zero or 1. For these devices you currently need to specify the full image tag for arm32v6.

**Note**: As of Node-RED 3.0.0 release, we are no longer building docker image for version 1. At the same time, images with NodeJS 10 and 12 are dropped, Node-RED 2 will be shipped with NodeJS 14. Next major NRCHKB release will require NodeJS >= 16.

### Quick Start (for those already running Docker)

```bash
docker run -d --net=host -v <path_on_host>:/data -e TZ=Europe/Amsterdam -e DEBUG=NRCHKB:* --name=node-red-homekit nrchkb/node-red-homekit
```

Let's dissect that command:

    docker run                  - Run this container.
    -d                          - Run container in background and print container ID.
    --net=host                  - Connect to the host network, which is required to work with homekit.
    -v <path_on_host>:/data     - Persist container data
    -e TZ=Europe/Amsterdam      - Set timezone, see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
    -e DEBUG=NRCHKB:*           - Print basic debug logs for NRCHKB
    --name node-red-homekit     - Give this machine a friendly local name.
    nrchkb/node-red-homekit     - The image to base it.

### Raspberry Pi (including install Docker)

Following these commands will install Docker, add user `pi` to Docker group, then set the docker container to always run.

We assume you have some basic knowledge of Linux and you are logged in as `pi` user.  

1) Make sure we are in the home directory of the pi user:

```bash
cd ~
```

2) Make sure we have the latest packages available and upgrade to the latest versions (reboot if needed!):

```bash
sudo apt update && sudo apt upgrade -y
```

3) Download the docker install script and execute it to install Docker on your system.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

4) As the Docker script explains, add the pi user to the Docker group so that the pi user has the permissions to execute docker commands:

```bash
sudo usermod -aG docker pi
```

5) Reboot the Raspberry PI or just log out and back in:

```bash
sudo reboot
```

6) To test if your Docker install went well: 

```bash
docker run --rm hello-world
```

The above command should say 'Hello from Docker':

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

If the above steps went as expected, you are ready to run the `nrchkb/node-red-homekit` image as a container. But before that we create a directory on the pi host so that all node-red / node-red-homekit files are stored outside the container on your raspberry pi.

7) Make directory in your pi user's home directory:

```bash
mkdir node-red-homekit
```

8) Run the Docker run command to deploy the `nrchkb/node-red-homekit` image as a container and where the container `/data` directory is bound to the `/home/pi/node-red-homekit` directory on your Raspberry PI.

```bash
docker run -d --net=host -v ~/node-red-homekit:/data --restart always -e TZ=Europe/Amsterdam -e DEBUG=NRCHKB:* --name node-red-homekit nrchkb/node-red-homekit
```

You don't need to explicit map ports, since all ports are opened on the hosts network! This is required for homekit to work well.

### Upgrade to the latest image
Suppose there is a new `nrchkb/node-red-homekit` image available? How do I make use of this new image?

1) Find the id of your current deployed container:

```bash
docker container ls
```
The above command lists all running container and in the first column it displays the id of the container and in the last column it's name.

2) Stop the current container:

```
docker stop <container_id or container_name>
```

3) Remove the current container:
```
docker rm  <container_id or container_name>
```

4) Pull the latest `nrchkb/node-red-homekit` image from [Docker Hub](https://hub.docker.com/r/nrchkb/node-red-homekit):

```bash
docker pull nrchkb/node-red-homekit
```

5) Deploy the container again:

```bash
docker run -d --net=host -v ~/node-red-homekit:/data --restart always -e TZ=Europe/Amsterdam -e DEBUG=NRCHKB:* --name node-red-homekit nrchkb/node-red-homekit
```

This runs the container based on the latest `nrchkb/node-red-homekit` image and retains your flows!

### Docker Compose (including install Docker Compose)
For demo purpose we use a Raspberry Pi with Docker installed (see )

1) Install required packages for Docker Compose

```bash
sudo apt update && sudo apt install -y libffi-dev libssl-dev python3 python3-pip
```

2) Install Docker Compose using pip3

```
sudo pip3 -v install docker-compose
```

3) Create a Docker compose yml for `nrchkb/node-red-homekit`:

Let's assume you have a directory `/home/pi/node-red-homekit` with a `data` sub-directory:

```bash
node-red-homekit/
`-- data

1 directory, 0 files
```

Use your favorite editor like nano to create a file named `docker-compose.yml` with the content below and save it in `/home/pi/node-red-homekit`.

```yaml
version: '2'
services:
  node-red-homekit:
    image: nrchkb/node-red-homekit
    restart: always
    network_mode: host
    environment:
      - TZ=Europe/Amsterdam
      - DEBUG=NRCHKB:*
    volumes:
      - /home/pi/node-red-homekit/data:/data
```

note 1: `/home/pi/node-red-homekit/data` is the persistence directory where the Docker container stores its files.
note 2: there is no port mapping defined, since the container is attached / uses the host network. 

4) Deploy the service as define in `docker-compose.yml`

From the `/home/pi/node-red-homekit` directory executed the command below to deploy the service and therefore the container:

```bash
docker-compose up -d
```

5) Verify your deployed container:

```bash
docker container ls
```

6) To update to the latest image:

The command below stops the current running container, removes it and removes the image.

```bash
docker-compose down --rmi  all
```

Run the command in step 4 to redeploy the service. It pulls the (latest) image, since now is available locally. 

### Synology

Synology users need to add the environment variable DSM_HOSTNAME.

Click the Environment tab and add a new environment variable named DSM_HOSTNAME. The value of the DSM_HOSTNAME environment variable should exactly match the server name as shown under Synology DSM Control Panel -> Info Center -> Server name, it should contain no spaces or special characters.

```bash
docker run -it --net=host -v <path_on_host>:/data -e DSM_HOSTNAME=<synology_hostname> -e TZ=Europe/Amsterdam -e DEBUG=NRCHKB:* --name=homekit nrchkb/node-red-homekit:<tag>
```

### Permissions

Since Node-RED 1.0 the container user is `node-red` and has uid `1000` and gid `1000`, make sure your <path_on_host> has the same uid and gid:

Verify command:

```bash
ls -nal <path_on_host>
```

Modify command:

```bash
chown -R  1000:1000 <path_on_host>
```

### Debug

To debug NRCHKB you have to add environment variable to command running node-red in docker by adding `-e` argument:

```-e "DEBUG=NRCHKB*,HAP-NodeJS*"```

To do that modify starting script like below:

```bash
docker run -it -e "DEBUG=NRCHKB*,HAP-NodeJS*" --net=host -v <path_on_host>:/data -e DSM_HOSTNAME=<synology_hostname> -e TZ=Europe/Amsterdam -e DEBUG=NRCHKB:* --name=homekit nrchkb/node-red-homekit:<tag>
```

### Node-RED Docker official
For more detailed info refer to the [Node-RED Docker official](https://github.com/node-red/node-red-docker) pages.

### NRCHKB Support
For more info visit our [Website](https://nrchkb.github.io) or [Discord](https://discord.gg/uvYac5u).
