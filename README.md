<h1 style="margin: auto;text-align: center">Using a created image</h1>

Executing redis database:

```
  # docker run --name leaderboard-redis -e REDIS_PASSWORD=redis123 -d -p 6379:6379 bitnami/redis:latest
```

`--name` : container name
`-e` : environments variables
`-d` : detached mode (run in background)
`-p` : redirect port

All docker images are in **docker hub**, where there are a lot of **docker files / images**.

<hr />

```
  # docker ps
```

List all containers.

<hr />

<h1 style="margin: auto;text-align: center">Creating our Dockerfile</h1>

To put our application inside a container, we need to create a **docker file**.

```dockerfile
  FROM node:lts-alpine    # base image

  RUN mkdir -p /home/node/api/node_modules && chmod -R node:node /home/node/api  # node is a default user exported from base image

  WORKDIR /home/node/api # when execute some command the base dir will be this

  COPY package.json yarn.* ./ # copy into default dir the files `package.json` and if exists `yarn.lock`

  USER node # all thing will be made with node user, because this dont has higth permissions (more security)

  RUN yarn

  COPY --chown=node:node . . ## copy all files using node user

  EXPOSE 3333

  CMD ["yarn", "start"]

```

Now, we need to build our image:

```
  # docker build -t leaderboard-image .
```

To see all images:

```
  # docker images
```

To run our created image, just run:

```
  # docker run -p 3333:3333 leaderboard-image
```

At this point, the application up in the container but it does not have any network with redis, so will be error in the connection.

To solve this, we will create a **docker-compose.yml**

<hr />

<h1 style="margin: auto;text-align: center">Creating our docker-compose</h1>

Docker compose is a serie of docker commands wrote in yml formmat.

```yml
version: '3'

services: # container configurations (docker run, docker start)
  leaderboard-api:
    build: . # find a locally image
    # image: ${image in docker hub}
    container_name: leaderboard-api
    volumes: # to development in container (nodemoon)
      - .:/home/node/api
      - /home/node/api/node_modules
    ports:
      - '3333:3333'
    depends_on:
      - leaderboard-redis
    networks:
      - leadboard-network
  leaderboard-redis:
    image: bitnami/redis:latest
    container_name: leaderboard-redis
    # env_file: .env (you can specify your env file)
    environment:
      - ALLOW_EMPTY_PASSWORD=no
      - REDIS_PASSWORD=${REDIS_PASSWORD}
    volumes:
      - leaderboard-redis-data:/data
    networks:
      - leadboard-network

volumes: # storage, save data after restart
  leaderboard-redis-data:

networks: # comunication between containers
  leadboard-network:
    driver: bridge
```

To execute our docker compose, just execute:

```
  docker-compose up
```

**Tip** Script to reset all containers and images:

```shell
  # Stop all containers
  docker stop $(docker ps -a -q)

  # Destroy all containers
  docker rm $(docker ps -a -q)

  # Destroy all images
  docker rmi $(docker images -q)
```
