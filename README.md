# Multi-Container Applications with Docker (Docker Compose included) (section 4)

## Without Docker compose

1. Create a network

```bash
docker network create goals-net
```

2. Run MongoDB Container

```shell
docker run -d --rm -v mongo-vol:/data/db --name mongodb -e MONGO_INITDB_ROOT_USERNAME=anand -e MONGO_INITDB_ROOT_PASSWORD=secret --network goals-net mongo:7.0
```

```python
docker run --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=anand \
  -e MONGO_INITDB_ROOT_PASSWORD=secret \
  -v data:/data/db \
  --rm \
  -d \
  --network goals-net \
  mongo
```

3. Build Node API Image (backend)
   
  - Navigate to `backend` folder where `Dockerfile` exists and build the image

```bash
docker build . -t actionanand/docker_multi-container_node
```

4. Run Node API Container

```shell
docker run -d --rm --name docker_multi-container_node -e MONGODB_USERNAME=anand -e MONGODB_PASSWORD=secret -v logs:/app/logs -v D:\AR_extra\rnd\docker\docker_multi-container\backend:/app:ro -v /app/node_modules --network goals-net -p 3002:80 actionanand/docker_multi-container_node
```

below command for mac, linux and wsl2

```shell
docker run -d --rm --name docker_multi-container_node -e MONGODB_USERNAME=anand -e MONGODB_PASSWORD=secret -v logs:/app/logs -v $(pwd)/backend:/app:ro -v /app/node_modules --network goals-net -p 3002:80 actionanand/docker_multi-container_node
```

```python
docker run --name docker_multi-container_node \
  -e MONGODB_USERNAME=anand \
  -e MONGODB_PASSWORD=secret \
  -v logs:/app/logs \
  -v /Users/actionanand/development/docker/docker_multi-container/backend:/app:ro \
  -v /app/node_modules \
  --rm \
  -d \
  --network goals-net \
  -p 3002:80 \
  actionanand/docker_multi-container_node
```

5. Build React SPA Image

   - Navigate to `frontend` folder where `Dockerfile` exists and build the image

```bash
docker build . -t actionanand/docker_multi-container_react
```

6. Run React SPA Container

```shell
docker run -d --rm -it --name docker_multi-container_react -p 3001:3000 -v D:\AR_extra\rnd\docker\docker_multi-container\frontend\src:/app/src:ro actionanand/docker_multi-container_react
```

below command for mac, linux and wsl2

```shell
docker run -d --rm -it --name docker_multi-container_react -p 3001:3000 -v $(pwd)/frontend/src:/app/src:ro actionanand/docker_multi-container_react
```

```python
docker run --name docker_multi-container_react \
  -v /Users/actionanand/development/docker/docker_multi-container/frontend/src:/app/src:ro \
  --rm \
  -d \
  -p 3001:3000 \
  -it \
  actionanand/docker_multi-container_react
```

- `-it` should be added for frontend apps. They're for **interactive tty**. If not added, process will exit.

7. You can visit at `http://localhost:3001/` to view the app.

8. Stopping all Containers

```shell
docker stop mongodb docker_multi-container_node docker_multi-container_react
```

## Docker compose

Docker Compose is an **additional tool**, offered by the Docker ecosystem, which helps with **orchestration / management** of multiple Containers. It can also be used for single Containers to simplify building and launching.

### Why?

Consider this example:

```shell
docker network create shop
docker build -t shop-node .
docker run -v logs:/app/logs --network shop --name shope-web shop-node
docker build -t shop-database
docker run -v data:/data/db --network shop --name shop-db shop-database
```

This is a very simple (made-up) example - yet you got **quite a lot of commands** to execute and memorize to bring up all Containers required by this application.

And you have to run (most of) these commands whenever you change something in your code or you need to bring up your Containers again for some other reason.

With Docker Compose, this gets much easier.

You can put your Container configuration into a `docker-compose.yaml` or `docker-compose.yml` file and then use just one command to bring up the entire environment: `docker-compose up`.

### Docker Compose Files

A `docker-compose.yaml` file looks like this:

```yaml
version: "3.8" # version of the Docker Compose spec which is being used
services: # "Services" are in the end the Containers that your app needs
  web:
    build: # Define the path to your Dockerfile for the image of this container
      context: .
      dockerfile: Dockerfile-web
    volumes: # Define any required volumes / bind mounts
      - logs:/app/logs # will create volume as `docker_multi-container_logs`
  db: # will create container name as `docker_multi-container_db_1`
    build:
      context: ./db
      dockerfile: Dockerfile-web
    container_name: containerName # custom name
    volumes:
      - data:/data/db
    environment: 
      - MONGO_INITDB_ROOT_PASSWORD=secret
      - MONGO_INITDB_ROOT_USERNAME=anand
    networks:
      - networkName # automatically ceated network name will be `docker_multi-container_default`, if not mentioned here
volumes: 
  data: # named valume should be declared at the end separately
  logs:
```

docker compose will automatically create `network` for all the services whichever defined under one file. We can also manually create if needed the deferent one.

You can conveniently edit this file at any time and you just have a short, simple command which you can use to bring up your Containers:

```bash
docker-compose up
```

```bash
docker-compose up -d
```

You can find the full (possibly intimidating - you'll only need a small set of the available options though) **list of configurations** [here](https://docs.docker.com/compose/compose-file/)

**Important to keep in mind**: When using Docker Compose, you **automatically get a Network for all your Containers** - so you don't need to add your own Network unless you need multiple Networks!

### Docker Compose Key Commands

There are two key commands:

* `docker-compose up` : **Start all containers / services** mentioned in the Docker Compose file
    * `-d` : Start in **detached mode**
    * `--build` : **Force Docker Compose** to re-evaluate / **rebuild all images** (otherwise, it only does that if an image is missing. That means it'll serve the cached image only)

* `docker-compose build` is used to just build the docker image

* `docker-compose down` : **Stop and remove** all containers / services
    * `-v` : **Remove all Volumes** used for the Containers - otherwise they stay around, even if the Containers are removed

  So, to remove all generated named volums along with container, below command will help:

  ```shell
  docker-compose down -v
  ```

Of course, there are **more commands**. We see more commands in other sections (e.g. the "Utility Containers" and "Laravel Demo" sections) but you can of course also already dive into the [official command reference](https://docs.docker.com/compose/reference/)

### Docker Images

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/b34b01c2-c3ba-4bb5-b634-c41c4db740fe)

### Docker Containers

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/1d5a3965-50e0-43f1-bf58-9a4f8373c00f)

### Docker Volumes

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/9eb1b2b7-229a-4141-a27f-3db0bd5c1e53)

### Docker Networks

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/b0d12cc5-e1de-474b-8155-f6320c7c3f28)

### Docker-compose - Shell commands

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/395db00c-36e0-4288-a68f-f934d7114bb2)

> The displayed localhost port `3000` is to access it inside the docker. This port will be exposed at port `3001` for our host machine. Please refer the `docker-compose.yaml` file for more details.

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/f710a030-7264-453e-9333-afe391dea013)

## Output

Point your browser to `http://localhost:3001/`

![image](https://github.com/actionanand/docker_multi-container/assets/46064269/702739dd-96d3-4736-9abe-1379e2238b11)

### Docker Image's public URL

* [actionanand/docker_multi-container_node](https://hub.docker.com/r/actionanand/docker_multi-container_node)
* [actionanand/docker_multi-container_react](https://hub.docker.com/r/actionanand/docker_multi-container_react)

## Associated repos:

1. [Docker Basics](https://github.com/actionanand/docker_playground)
2. [Managing Data and working with volumes](https://github.com/actionanand/docker_data_volume)
3. [Docker Communication](https://github.com/actionanand/docker_communication)
4. [Docker Multi-container with docker-compose](https://github.com/actionanand/docker_multi-container)
