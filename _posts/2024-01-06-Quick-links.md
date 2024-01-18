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



