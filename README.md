# docker-demo

## Networking

### 1. Start MySQL

#### 1.1. Create the network

```
docker network create todo-app
```

#### 1.2. Start a MySQL container and attach it to the network

See the [“Environment Variables” section in the MySQL Docker Hub](https://hub.docker.com/_/mysql/) listing

```
docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

#### 1.3. To confirm we have the database up and running, connect to the database and verify it connects.

Check the _mysql_ container's ID and copy its value:

```
 docker ps
```

```
 docker exec -it <mysql-container-id> mysql -u root -p
```

When the password prompt comes up, type in **secret**. In the MySQL shell, list the databases and verify you see the todos database, by typing:

```
SHOW DATABASES;
```

### 2. Connect to MySQL

#### 2.1. Start a new container using the nicolaka/netshoot image and connect it to the same network.

```
 docker run -it --network todo-app nicolaka/netshoot
```

#### 2.2. Inside the container: look up the IP address for the hostname mysql:

```
 dig mysql
```

In the `ANSWER SECTION`, there is an `A` record for `mysql` that resolves to an IP address. `mysql` isn’t a valid hostname, Docker was able to resolve it to the IP address of the container that had that network alias.

### 3. Run the app with MySQL

#### 3.1. Specify the environment variables and connect the container to the app network

Navigate to the _Dockerfile_ first, then run the command:

```
 docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:12-alpine \
   sh -c "yarn install && yarn run dev"
```

#### 3.2. Check the logs of the container to be sure it's using the database

Check the _node:12-alpine_ container's ID and copy the value:

```
docker ps
```

Look at the logs:

```
docker logs <container-id>
```

#### 3.3. Open the app in the browser (localhost:3000) and add a few items to the todo list

#### 3.4. Connect to the mysql database and prove that the items are being written to the database

Check the _mysql_ container's ID and copy the value:

```
docker ps
```

Navigate to the _Dockerfile_ and run the command:

```
 docker exec -it <mysql-container-id> mysql -p todos
```

When the password prompt comes up, type in **secret**. And in the mysql shell, run the following:

```
 select * from todo_items;
```

See the added todos.

## Docker Compose

### 1. Check if it is installed

```sh
 docker-compose version
```

### 1. Create a compose file

#### 1.1. Create a docker-compose.yml file at the root of the project

#### 1.2. Define the schema version (the latest supported version)

```sh
 version: "3.7"
```

#### 1.3. Define the services (containers) to run as a part of the app

##### A. Define the `app` service

###### A1. Define the service entry (name) and the image for the container

Any name could be picked (in this case - it's used `app`). It will become a network alias automatically.

###### A2. Add the `command` definition

There is no ordering requirements, but it's often to be placed close to the `image` definition

###### A3. Add the `ports` definition

Long syntax:

```sh
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

Short syntax

```sh
ports:
  - "3000:3000"
```

###### A4. Add the `working_dir` and the `volumes` definitions

Long syntax for `volumes`:

```sh
volumes:
  - type: volume
  source: mydata
  target: /data
  volume:
   nocopy: true
  - type: bind
    source: ./static
    target: /opt/app/static
```

Where:

-   `type`: the mount type volume, bind, tmpfs or npipe
-   `source`: the source of the mount, a path on the host for a bind mount, or the name of a volume defined in the top-level volumes key. Not applicable for a tmpfs mount.
-   `target`: the path in the container where the volume is mounted
    read_only: flag to set the volume as read-only
-   `bind`: configure additional bind options
    -   `propagation`: the propagation mode used for the bind
-   `volume`: configure additional volume options
    -   `nocopy`: flag to disable copying of data from a container when a volume is created
-   `tmpfs`: configure additional tmpfs options - `size`: the size for the tmpfs mount in bytes

Short syntax for `volumes`:

It uses the generic `[SOURCE:]TARGET[:MODE]` format, where `SOURCE` can be either a host path or volume name. `TARGET` is the container path where the volume is mounted.

```sh
volumes:
  # Just specify a path and let the Engine create a volume
  - /var/lib/mysql

  # Specify an absolute path mapping
  - /opt/data:/var/lib/mysql

  # Path on the host, relative to the Compose file
  - ./cache:/tmp/cache

  # User-relative path
  - ~/configs:/etc/configs/:ro

  # Named volume
  - datavolume:/var/lib/mysql
```

###### A5. Add the `environment` definition

Specify the needed environment variables.

##### B. Define the `mysql` service

###### B1. Define the new service and specify the image to use

###### B2. Define the volume mapping

When running containers with Compose, named volumes are _not_ created automatically.
In the top-level of the file, there should be a `volumes` definition, where all of the needed volumes are specified.

###### B2. Add the `environment` definition

### 2. Run the application stack

### 2.1. Check the running containers

To be sure that nothing is running yet:

```sh
docker ps

docker rm -f <ids>
```

### 2.2. Start the application stack

Open a terminal where the `docker-compose.yaml` file is and run the command:

```sh
docker-compose up -d
```

-   `-d`: runs everything in the background

### 2.3. Check the logs

```sh
docker-compose logs -f
```

For a specific service:

```sh
docker-compose logs <service-name> -f
```

-   `-d`: 'follows', live output

### 2.4. Tear it down

```sh
docker-compose down --volumes
```

-   `--volumes`: removes the volumes as well

### 2.5. Check the running containers

To be sure that nothing is running anymore:

```sh
docker ps

docker rm -f <ids>
```
