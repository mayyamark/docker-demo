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
