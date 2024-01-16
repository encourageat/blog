---
layout: post
title:  "Use docker image of Keycloak"
date:   2024-01-15 19:41:42 +0530
categories: IAM Keycloak
---

When I have started using Keycloak, I started with zip distribution of version 21.1.1 (the latest at that time) and had its dependency OpenJDK 17 (minimum required version) already installed. Just three months over and now the latest version is 23.0.3. I wanted to use Docker image in my further learning on Keycloak and this blog relates to that..

My personal laptop is Windows 11 and virtualisation was by default enabled. I guess , running Dokcer require minimum Windows 10 and if we are on prior versions you may find some other tools as pre-reqisites for Docker.

I have downloaded Docker Desktop Installer for Windows and the installation of it did not create any issues. Started Docker Desktop from the icon installer created  on taskbar. I prefer the command line options of Docker, but I guess the Docker Desktop window still should be launched for smooth working with docker commands. I faced an error when it was not running.

Unlike virtual machines, docker run on the same OS kernel but have separate filesystems, processes, and network stacks. We will just jump into starting Keycloak quickly.

Keycloak image can be obtained from container registry quay.io owned by RHEL.

To pull the Keycloak version 23.0.3, we can use the following docker command from command prompt.  

```
docker pull quay.io/keycloak/keycloak:23.0.3
```
To pull a different version specify replace the version 23.0.3 with the desired value in the above command. Omiiting version will always pull the latest image. 

Example:
```
docker pull quay.io/keycloak/keycloak
```

The following command shows the pulled images in the local system  

```
docker images
```

We will be running Keycloak in developer mode only in this blog. This is not good for production usage. The intention in starting in developer mode is to quickly get started with Keycloak..  

We will explore three options in this blog 

- Keycloak in developer mode started using docker command
- Keycloak in developer mode started using docker-compose command and uses H2 database (default)
- Keycloak in developer moded started using docker-compose and connected to a postgres database running in the same network

**Keycloak in developer mode started using docker command**

Following is the command to run and access Keycloak in http://localhost:9090 and the admin UI credentials are mentioned as environment variables on startup

```
docker run -p 9090:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=changeme quay.io/keycloak/keycloak:23.0.3 start-dev
```

It takes a while to start keycloak and once its started try accessing the admin UI with the credentials we specified in the previous step. The URL for the admin UI is http://localhost:9090/admin. The port in the local system is 9090 since we did the mapping of container port 8080 as follows -p 9090:8080 in the docker command arguments.  

The following "docker ps" shows the running container.
```
C:\Docker>docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS
                     NAMES
6d14f62dd413   quay.io/keycloak/keycloak:23.0.3   "/opt/keycloak/bin/k…"   4 minutes ago   Up 4 minutes   8443/tcp, 0.0.0.0:9090->8080/tcp   amazing_blackwell

C:\Docker>
```
Now let us stop this container, and go to the next option

Commands to stop and further check the status of running containers. (amazing_blackwell is my container name. I could also have specified the conatainer ID also, both are displayed in "docker ps" output as seen in the previous command)  

```
C:\Docker>docker stop amazing_blackwell
amazing_blackwell

C:\Docker>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**Keycloak in developer mode started using docker-compose command and uses H2 database (default)**

The command docker-compose is available when Docker Desktop Installer is installed. It supports specifying one or more container images in a YAML file and helps for easy start. The YAML file u can create in Visual Studio Code or some other text editor.

Contents of docker-compose.yml (In my case I have saved under C:\Docker directory)  

```
version: '3'

volumes:
  keycloak:

services:
  kc-encourageat:
    image: quay.io/keycloak/keycloak
    ports:
      - "9095:8080"
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

In the above I had specified mapped port as 9095 and gave a admin password as 
admin. I am pulling the latest Keycloak image (although its advisable to specify version to be sure that you are getting the version you want)

For running in detached mode use the command

```
docker-compose -f docker-compose.yml up -d
```

This time try accessing the admin UI http://localhost:9095/admin again (the port and credetials I have used is different in above than the first one. The volume entry is for data persistence)

If you are interested in veryifying the data persistence, create a relam or users..

Removing -d from above, runs in interactive mode

docker ps output and after stopping "docker ps" is again invoked for demonstration

```
C:\Docker>docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS          PORTS
                NAMES
86ed796afa16   quay.io/keycloak/keycloak   "/opt/keycloak/bin/k…"   10 minutes ago   Up 10 minutes   8443/tcp, 0.0.0.0:9095->8080/tcp   docker-kc-encourageat-1

C:\Docker>docker-compose -f docker-compose.yml down
[+] Running 2/2
 ✔ Container docker-kc-encourageat-1  Removed                                                                      4.9s
 ✔ Network docker_keycloak-network    Removed                                                                      0.3s

C:\Docker>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

C:\Docker>
```

With the below command we actually stopped the container in docker-compose.yml
```
docker-compose -f docker-compose.yml down
```
If you are interested in checking data persistence after a shutdown, restart the container specified in docker-compose.yml file. You should be finding your custom created realms and users if you had created it on previous startup.

Now we will go to the 3rd step

**Keycloak in developer moded started using docker-compose and connected to a postgres database running in the same network**

This time Keycloak will be connected to postgres running in the same network. We should have volumes entry for postgres to have data persistence since all data is stored there.

docker-compose.yml sample for this
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
```
The entry depends_on was to make sure postgres is started before Keycloak. As done before access the admin URL with the correct port and credentials after running docker-compose as done in previous steps.

Otutput of "docker ps"

```
C:\Docker>docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS
                     NAMES
3f61974eb29e   quay.io/keycloak/keycloak:23.0.3   "/opt/keycloak/bin/k…"   4 minutes ago   Up 4 minutes   8443/tcp, 0.0.0.0:9090->8080/tcp   keycloak-instance1
873a47f108e6   postgres:15                        "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   0.0.0.0:5432->5432/tcp             postgres-container

C:\Docker>
```





