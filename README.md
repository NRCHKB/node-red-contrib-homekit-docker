# Node-RED-homekit-docker
[![Build Status](https://travis-ci.org/NRCHKB/node-red-contrib-homekit-docker.svg?branch=master)](https://travis-ci.org/NRCHKB/node-red-contrib-homekit-docker)
[![DockerHub Pull](https://img.shields.io/docker/pulls/nrchkb/node-red-homekit.svg)](https://hub.docker.com/r/nrchkb/node-red-homekit)
[![DockerHub Star](https://img.shields.io/docker/stars/nrchkb/node-red-homekit.svg?maxAge=2592000)](https://hub.docker.com/r/nrchkb/node-red-homekit)
 
Node-RED-homekit-docker is a Node-RED based project with support for homekit. It is based on the [official Node-RED Docker](https://hub.docker.com/r/nodered/node-red) images with the necessary tools and npm module [node-red-contrib-homekit-bridged](https://www.npmjs.com/package/node-red-contrib-homekit-bridged) installed to run homekit within a docker container. 

## Architecture
Node-RED-homekit-docker is supported by manifest list, which means one doesn't need to specify the tag for a specific architecture. Using the image without any tag or the latest tag, will pull the right image for the architecture required.

Currently Node-RED-homekit has support for multiple architectures:
- `amd64`   : based on linux Alpine - for most desktop computer (e.g. x64, x86-64, x86_64)
- `arm32v6` : based on linux Alpine - (i.e. Raspberry Pi 1 & Zero)
- `arm32v7` : based on linux Alpine - (i.e. Raspberry Pi 2, 3, 4)
- `arm64v8` : based on linux Alpine - (i.e. Pine64)

**Note**: Currently there is a bug in Docker's architecture detection that fails for arm32v6 - eg Raspberry Pi Zero or 1. For these devices you currently need to specify the full image tag for arm32v6.

### Quick Start (for those already running Docker)

```bash
docker run -d --net=host -v <path_on_host>:/data --name=node-red-homekit nrchkb/node-red-homekit
```

Let's dissect that command:

    docker run                  - Run this container.
    -d                          - Run container in background and print container ID.
    --net=host                  - Connect to the host network, which is required to work with homekit.
    -v <path_on_host>:/data     - Persist container data
    --name node-red-homekit     - Give this machine a friendly local name.
    nrchkb/node-red-homekit     - The image to base it.

### Raspberry Pi (including install Docker)

Following these commands will install Docker, add user `pi` to Docker group, then set the docker container to always run. Update commands will be available soon (work in progress July, 2020).

```bash
sudo apt update && sudo apt upgrade
cd ~
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker pi
sudo reboot
docker run -d --net=host -v ~/:/data --restart always --name node-red-homekit nrchkb/node-red-homekit
```

### Synology

Synology users need to add the environment variable DSM_HOSTNAME.

Click the Environment tab and add a new environment variable named DSM_HOSTNAME. The value of the DSM_HOSTNAME environment variable should exactly match the server name as shown under Synology DSM Control Panel -> Info Center -> Server name, it should contain no spaces or special characters.

```bash
docker run it --net=host -v <path_on_host>:/data -e DSM_HOSTNAME=<synology_hostname> --name=homekit nrchkb/node-red-homekit:<tag>
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

To debug NRCHKB you have to run node-red in docker in debug mode by adding `-e` argument:

```-e "DEBUG=NRCHKB*,HAP-NodeJS*"```

To do that modify starting script like below:

```bash
docker run it -e "DEBUG=NRCHKB*,HAP-NodeJS*" --net=host -v <path_on_host>:/data -e DSM_HOSTNAME=<synology_hostname> --name=homekit nrchkb/node-red-homekit:<tag>
```

### Node-RED Docker official
For more detailed info refer to the [Node-RED Docker official](https://github.com/node-red/node-red-docker) pages.
