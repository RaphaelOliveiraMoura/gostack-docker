<h1 style="text-align: center">Executing redis database</h1>

```
  # docker run --name leaderboard-redis -e REDIS_PASSWORD=redis123 -d -p 6379:6379 bitnami/redis:latest
```
`--name` : container name
`-e` : environments variables
`-d` : detached mode (run in background)
`-p` : redirect port

All docker images are in *docker hub*, where there are a lot of *docker files / images*. 

<hr />

```
  # docker ps
```

List all containers.

<hr />

To put our application inside a container, we need to create a *docker file*.

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