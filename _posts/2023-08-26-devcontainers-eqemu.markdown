---
layout: post
title:  "VSCode Devcontainers for an MMO Emulator"
date:   2023-08-26 13:22:32 -0400
---

VSCode's devcontainers are a great way to isolate an entire environment through the use of Docker. I chose to utilize devcontainers to create a development environment which could build and host a complex system for a popular MMO emulator, EQEmu. [EverQuest Emulator](https://docs.eqemu.io/){:target="_blank"} is a fantastic open source reveresed engineered server implementation for EverQuest. For this case I'll be standing up a classic version of the server called [The Al`Kabor Project](https://takproject.net/){:target="_blank"} 

The system's architectural design comprises several autonomous `C++` processes responsible for managing global actions, along with additional dynamically initiated C++ processes, all seamlessly integrated with a backend `MySQL` database which encompasses the game engine. `Lua` scripts are used to control AI entities within the world. 

What makes this setup great is that `Windows` is used, yet the entire environment is isolated within containers, which makes it easy to pick back up where you left off. I was also concerned about the number of dependencies required to build the project on Windows. By mounting the source code within the container, you can build within the container (`Linux`), and all required dependencies are easily documented and managed through the Dockerfiles. The entry point of the application server can execute a build with `cmake`, which has great caching abilities. Furthermore, the mounted source/build directory will survive container reboots.

A simple rebuild of the container executes an entire build and deploy of the system.

## Environment
- Windows
- Docker Desktop
- WSL2
- VSCode


![VSCode devcontainer EQEmu TAKP](/assets/20230826-devcontainer.png)

## Setting up devcontainers
Fork the repository and fetch the main git branch locally. Within the source code create a folder called `.devcontainer` which is the hook into VSCode. Within this folder our minimal `devcontainer.json` file mounts our workspace (our local directory) to a location within the container called `/workspaces` and links our `docker-compose.yml` file

```
{
    "name": "TAKP-EQMac",
    "dockerComposeFile": "docker-compose.yml",
    "overrideCommand": false,
    "service": "app",
    "workspaceFolder": "/workspaces"

}
```

## Docker and docker-compose
Our `docker-compose.yml` contains two extended images, one for the application server and another for the database server. The configuration mounts a few directories and publishes ports used by the different servers. A `.env` file within this same direcotry is also created to store environment variables.


```
version: '3.8'

volumes:
  mariadb-data:

services:
  app:
    build:
      context: .
      dockerfile: app.Dockerfile
    env_file:
      - .env

    volumes:
      - ..:/workspaces:cached

    ports:
      # loginserver
      - 6000:6000/udp
      - 5998:5998
      # world
      - 9000:9000/udp
      - 9000:9000
      # ucs
      - 7778:7778/udp
      # boat zones
      - 7375-7400:7375-7400/tcp
      - 7375-7400:7375-7400/udp
      # dynamic zones
      - 7000-7374:7000-7374/tcp
      - 7000-7374:7000-7374/udp

  db:
    build:
      context: .
      dockerfile: db.Dockerfile
    restart: unless-stopped
    volumes:
      - mariadb-data:/var/lib/MARIADB
    env_file:
      - .env
    ports:
      - 3306:3306
```

# Dockerfiles
Here's our app server which will need build and run-time dependencies. The entry point can build the server and launch the entire process. The final command is a sleep as our entrypoint's script launches many background processes, and this will force the container to remain up.


```
FROM debian:12-slim

ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt install -y \
    bash \
    build-essential \
    cmake \
    cpp \
    debconf-utils \
    g++ \
    gcc \
    libboost-dev \
    libcurl4-openssl-dev \
    liblua5.1-dev \
    libmariadb-dev \
    libsodium-dev \
    libssl-dev \
    lua5.1 \
    lua-bitop \
    make \
    mariadb-client \
    uuid-dev \
    dos2unix


COPY app-context/takp-init.sh /
RUN chmod +x /takp-init.sh
RUN dos2unix /takp-init.sh
ENTRYPOINT [ "/takp-init.sh" ]
CMD [ "sleep", "infinity" ]
```


The database server is a standard MariaDB dockerfile that adds a few initialization scripts the server needs on a fresh install.
```
FROM mariadb:10.3.32-focal

# additional mysql conf properties
COPY db-context/mariadb.cnf /etc/mysql/mariadb.conf.d

# base database
ADD db-context/alkabor_2023-08-01-14_55.tar.gz /docker-entrypoint-initdb.d
ADD db-context/tblloginserversettings.sql /docker-entrypoint-initdb.d
# takp .tar contains this, dont want it to run
RUN rm /docker-entrypoint-initdb.d/drop_system.sql

# extra sql scripts
ADD db-context/auto_create_login.sql /docker-entrypoint-initdb.d/z_auto_create_login.sql
ADD db-context/launcher_boats.sql /docker-entrypoint-initdb.d/z_launcher_boats.sql
```


Our bash script to bootstrap the build, configuration, env var substitution, deploy, and execution of the entire system.

```
#!/bin/bash

# build
cd /workspaces
mkdir -p build
cd build
cmake -G 'Unix Makefiles' ..
make -j `grep -P '^core id\t' /proc/cpuinfo | sort -u | wc -l`


# runtime
# create takp folder on the root of container for runtime files
mkdir -p /takp/logs
mkdir -p /takp/shared

# symlink maps and quests (note submodules name)
ln -s /workspaces/TAKP-Maps /takp/Maps
ln -s /workspaces/TAKP-quests /takp/quests

# copy static opcodes
cp /workspaces/loginserver/login_util/*.conf /takp
cp /workspaces/utils/patches/*.conf /takp

# copy env files
envsubst < /workspaces/.devcontainer/app-context/template-eqemu_config.json > /takp/eqemu_config.json
envsubst < /workspaces/.devcontainer/app-context/template-login.ini > /takp/login.ini

# symlink binaries, build/bin/* as individual files in /takp
cp --symbolic-link /workspaces/build/bin/* /takp


# wait for db?
sleep 1
cd /takp
./shared_memory &> logs/shared_memory.log
./loginserver &> logs/login.log &
./world &> logs/world.log &
./eqlaunch 'dynzone1' &> logs/eqlaunch.log &
./eqlaunch 'boats' &> logs/eqlaunch-boats.log &
./queryserv &> logs/queryserv.log &
./ucs &> logs/ucs.log &


exec "$@"
```


Using VSCode's rebuild container task will begin the entire process and at the end, we have a working local development environment for a massive multiplayer online game emulator!


