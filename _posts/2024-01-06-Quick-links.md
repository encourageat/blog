---
layout: post
title:  "Quick Links.."
published: false
date:   2024-01-06 09:41:42 +0530
categories: General
---

Quick references..

This is just a blog with some links and commands. (mainly created for my reference)

**Jekyll commands**

Command to start server..
```
bundle exec jekyll serve
```

**Docker commands**

Docker hub URL [here](https://hub.docker.com/)

Pull the image of keycloak 23.0.1
```
docker pull quay.io/keycloak/keycloak:23.0.1
```
View the images
```
docker images
```
Run Keycloak in developer mode..
```
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=<your_password> quay.io/keycloak/keycloak:23.0.1 start-dev
```
Run Keycloak as detached in developer mode..
```
docker run -p 8080:8080 -d -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=<your_password> quay.io/keycloak/keycloak:23.0.1 start-dev
```
Check logs in detached mode
```
docker logs <returned id from previous command(the container ID or name xan be specified)>
```

Running containers information..
```
docker ps
```

Docker ps output ..
```
C:\>docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS          PORTS                              NAMES
ffac561456f0   quay.io/keycloak/keycloak:23.0.1   "/opt/keycloak/bin/k…"   31 minutes ago   Up 31 minutes   0.0.0.0:8080->8080/tcp, 8443/tcp   objective_jackson
C:\>
```

To view even running and stopped containers..
```
docker ps -a
```

To inspect a container
```
docker exec -it <CONTAINER_NAME_OR_ID> /bin/bash
```

Example..
```
C:\>docker exec -it objective_jackson /bin/bash
bash-5.1$ ls
afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
bash-5.1$ cd /opt/keycloak/
bash-5.1$ ls
bin  conf  data  lib  LICENSE.txt  providers  README.md  themes  version.txt
bash-5.1$ exit
exit

C:\>
```
In the above example, /bin/bash is used as the command to start an interactive shell inside the container. Other command based based on the tool available in your container are /bin/sh, /bin/zsh, etc.

Docker stop- to stop a running container
```
docker stop <CONTAINER_ID_or_NAME>
```
Docker start - to restart a stopped container
```
docker start <CONTAINER_ID_or_NAME>
```

Docker stop and start examples..
```
C:\>docker stop objective_jackson
objective_jackson

C:\>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

C:\>docker start objective_jackson
objective_jackson

C:\>docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED             STATUS         PORTS                              NAMES
ffac561456f0   quay.io/keycloak/keycloak:23.0.1   "/opt/keycloak/bin/k…"   About an hour ago   Up 5 seconds   0.0.0.0:8080->8080/tcp, 8443/tcp   objective_jackson

C:\
```

Docker compose- Allows you to define a multi-container application in a single file (usually named docker-compose.yml) and then use that configuration to start all the containers with a single command.
```
version: '3'

volumes:
  keycloak:

services:
  kc-encourageat:
    image: quay.io/keycloak/keycloak
    ports:
      - "8081:8080"
    environment:
      KC_HOSTNAME: localhost
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_LOG_LEVEL: debug
    volumes:
      - keycloak:/opt/keycloak/data/
    networks:
      - keycloak-network
    command: start-dev


networks:
  keycloak-network:
    driver: bridge

```
The above docker compose file starts Keycloak in developer mode. It uses H2 database (not recommended for production). The volume entry is to persist the data.

Command to start using docker-compose
```
docker-compose -f docker-compose.yml up
```

Command to start using docker-compose in detached mode
```
docker-compose -f docker-compose.yml up -d
```

Command to stop..
```
docker-compose -f docker-compose.yml down
```

Details on configuring keycloak database and details on tested versions is [here](https://www.keycloak.org/server/db)

Keycloak documentation on creating docker image is [here](https://www.keycloak.org/server/containers)

In Docker, a network is a way for containers to communicate with each other 

Docker copmose file to start two keycloak instances (accessible on port 9090 and 9091) and a postgres database which both the keycloak uses. All these are part of same network.
```
version: '3'

networks:
  keycloak_postgres_net:

volumes:
  db_data:

services:
  postgres-container:
    image: postgres:15
    container_name: postgres-container
    restart: always
    networks:
      - keycloak_postgres_net
    environment:
      POSTGRES_DB: pgDb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: changeme  
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

  keycloak-instance1:
    image: quay.io/keycloak/keycloak:23.0.3
    container_name: keycloak-instance1
    restart: always
    networks:
      - keycloak_postgres_net
    environment:
      KC_HOSTNAME: localhost
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_LOG_LEVEL: info
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres-container:5432/pgDb
      KC_DB_USERNAME: admin
      KC_DB_PASSWORD: changeme
    depends_on:
      - postgres-container
    ports:
      - "9090:8080"
    command: start-dev

  keycloak-instance2:
    image: quay.io/keycloak/keycloak:23.0.3
    container_name: keycloak-instance2
    restart: always
    networks:
      - keycloak_postgres_net
    environment:
      KC_HOSTNAME: localhost
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_LOG_LEVEL: info
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres-container:5432/pgDb
      KC_DB_USERNAME: admin
      KC_DB_PASSWORD: changeme
    depends_on:
      - postgres-container
    ports:
      - "9091:8080"
    #command: start-dev #command: ["-v", "start-dev"] if verbose log is needed.
```
The above also is not good for production since https is not enabled and since keycloak is started in developer mode.  


```
C:\Docker>docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS          PORTS                              NAMES
810d1813d34f   quay.io/keycloak/keycloak:23.0.3   "/opt/keycloak/bin/k…"   57 minutes ago   Up 57 minutes   8443/tcp, 0.0.0.0:9090->8080/tcp   keycloak-instance1
2c15ebdfad38   quay.io/keycloak/keycloak:23.0.3   "/opt/keycloak/bin/k…"   57 minutes ago   Up 57 minutes   8443/tcp, 0.0.0.0:9091->8080/tcp   keycloak-instance2
fdfcd98f8519   postgres:15                        "docker-entrypoint.s…"   57 minutes ago   Up 57 minutes   0.0.0.0:5432->5432/tcp             postgres-container
```

Lists the network..
```
docker network ls
```
```
C:\Docker>docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
eb6830bee7d2   bridge                         bridge    local
a052a0dbf3cb   docker_keycloak_postgres_net   bridge    local
73697a1e425d   host                           host      local
a5eab27fdd8d   none                           null      local

C:\Docker>
```

Docker swarm

```
C:\dockerswarm>docker swarm init
Swarm initialized: current node (ssgpmk7yrbs2ipt09hsgf4ciu) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2k9we4mbyie8ebd4qd07er1b0nnq5ykkd12kqpo047pqhhapd5-7yk03er9azb5vf6s3hsjiohky 192.168.65.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```
Above creates a node as master node and worker node

Below is for verifying swarm status

```
docker info
```

List all the nodes   

```
C:\dockerswarm>docker node ls
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ssgpmk7yrbs2ipt09hsgf4ciu *   docker-desktop   Ready     Active         Leader           24.0.7

C:\dockerswarm>
```

Leave the single running node (that is the master node)

```
C:\dockerswarm>docker swarm leave --force
Node left the swarm.
```

Single master node and multiple worker nodes (need more VMs)

docker swarm init --advertise-addr <MANAGER_IP>   

```
C:\dockerswarm>docker swarm init --advertise-addr 127.0.0.1
Swarm initialized: current node (ncs8npk4pozztm1k73t1gl91t) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-47m4namcbhlmq5oxvkzr0ma5mufwzux4udh0w1m71fl15n8ivr-9ylo5zyuuyrzxdfagqhkyzhlz 127.0.0.1:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


C:\dockerswarm>docker node ls
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ncs8npk4pozztm1k73t1gl91t *   docker-desktop   Ready     Active         Leader           24.0.7
```

Extract from "docker info" command

```
 Swarm: active
  NodeID: ncs8npk4pozztm1k73t1gl91t
  Is Manager: true
  ClusterID: oflh8yvznrlu0uh2grk4kals2
  Managers: 1
  Nodes: 1
  Default Address Pool: 10.0.0.0/8
  SubnetSize: 24
  Data Path Port: 4789
  Orchestration:
   Task History Retention Limit: 5
  Raft:
  ```

  To get the worker node add details   

  ```
  C:\dockerswarm>docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-47m4namcbhlmq5oxvkzr0ma5mufwzux4udh0w1m71fl15n8ivr-9ylo5zyuuyrzxdfagqhkyzhlz 127.0.0.1:2377
C:\dockerswarm>
```

Running service...

```
C:\>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

C:\>docker service ls
ID        NAME      MODE      REPLICAS   IMAGE     PORTS

C:\>docker service create --name web-site --replicas 1 --publish 80:80 nginx:latest
xo6j8qm0lac9hy5yhmkqjzjmc
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged

C:\>docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
cdaaae5cb8d8   nginx:latest   "/docker-entrypoint.…"   22 seconds ago   Up 16 seconds   80/tcp    web-site.1.dnp10nmv3abr3osjypusujlm3

C:\>docker service ls
ID             NAME       MODE         REPLICAS   IMAGE          PORTS
xo6j8qm0lac9   web-site   replicated   1/1        nginx:latest   *:80->80/tcp

C:\>
```

Docker swarm starts a new container even though forcefully stopped one as shown below

```
C:\>docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
cdaaae5cb8d8   nginx:latest   "/docker-entrypoint.…"   13 minutes ago   Up 13 minutes   80/tcp    web-site.1.dnp10nmv3abr3osjypusujlm3

C:\>docker rm -f cdaaae5cb8d8
cdaaae5cb8d8

C:\>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

C:\>docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS     NAMES
cfbdaccde7e3   nginx:latest   "/docker-entrypoint.…"   10 seconds ago   Up 2 seconds   80/tcp    web-site.1.44j0r4k56t6s4xrerg7i7380r

C:\>
```

Now trying to stop service..
All containers in replicas will be stopped

```
C:\>docker node ls
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ncs8npk4pozztm1k73t1gl91t *   docker-desktop   Ready     Active         Leader           24.0.7

C:\>docker service ls
ID             NAME       MODE         REPLICAS   IMAGE          PORTS
xo6j8qm0lac9   web-site   replicated   1/1        nginx:latest   *:80->80/tcp

C:\>docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
cfbdaccde7e3   nginx:latest   "/docker-entrypoint.…"   51 minutes ago   Up 51 minutes   80/tcp    web-site.1.44j0r4k56t6s4xrerg7i7380r

C:\>docker service rm web-site
web-site

C:\>docker service ls
ID        NAME      MODE      REPLICAS   IMAGE     PORTS

C:\>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

C:\>docker node ls
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ncs8npk4pozztm1k73t1gl91t *   docker-desktop   Ready     Active         Leader           24.0.7

C:\>
```

Inspect...

```
C:\>docker service ls
ID             NAME       MODE         REPLICAS   IMAGE          PORTS
n9nx7zsg9hw4   web-site   replicated   1/1        nginx:latest   *:80->80/tcp4

C:\>docker service inspect --pretty n9nx7zsg9hw4

ID:             n9nx7zsg9hw4fg4r8wxn6qzz6
Name:           web-site
Service Mode:   Replicated
 Replicas:      1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:         nginx:latest@sha256:4c0fdaa8b6341bfdeca5f18f7837462c80cff90527ee35ef185571e1c327beac
 Init:          false
Resources:
Endpoint Mode:  vip
Ports:
 PublishedPort = 80
  Protocol = tcp
  TargetPort = 80
  PublishMode = ingress


C:\>

```

Docker service ps id/name => Give details of all replicas

```

C:\>docker service ls
ID             NAME       MODE         REPLICAS   IMAGE          PORTS
n9nx7zsg9hw4   web-site   replicated   1/1        nginx:latest   *:80->80/tcp

C:\>docker service ps web-site
ID             NAME         IMAGE          NODE             DESIRED STATE   CURRENT STATE           ERROR     PORTS
yu8fohg4hxg8   web-site.1   nginx:latest   docker-desktop   Running         Running 9 minutes ago

C:\>
```