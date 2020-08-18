# About this repository
 This repository is contained Dockerfile and material for building images of Node on official Ubuntu docker images.

# How to use this image

## Create a `Dockerfile` in your Node.js app project

```dockerfile
# specify the node base image with your desired version node:<version>
FROM nmoeini/node:10-xenial
# replace this with your application's default port
EXPOSE 8888
```

## Without Docker Compose:
You can then build and run the Docker image:

```console
$ docker build -t my-nodejs-app .
$ docker run -it --rm --name my-running-app my-nodejs-app
```

## With Docker Compose:

```yml
version: "2"
services:
  node:
    image: "nmoeini/node:10-xenial"
    user: "node"
    working_dir: /home/node/app
    environment:
      - NODE_ENV=production
    volumes:
      - ./:/home/node/app
    expose:
      - "8081"
    command: "npm start"
```

You can then run using Docker Compose:

```console
$ docker-compose up -d
```

Docker Compose example copies your current directory (including node_modules) to the container.
It assumes that your application has a file named [`package.json`](https://docs.npmjs.com/files/package.json)
defining [start script](https://docs.npmjs.com/misc/scripts#default-values).

## Run a single Node.js script

For many simple, single file projects, you may find it inconvenient to write a
complete `Dockerfile`. In such cases, you can run a Node.js script by using the
Node.js Docker image directly:

```console
$ docker run -it --rm --name my-running-script -v "$PWD":/usr/src/app -w /usr/src/app nmoeini/node:10-xenial node your-daemon-or-script.js
```

## Environment Variables

Run with `NODE_ENV` set to `production`. This is the way you would pass in secrets and other runtime configurations to your application as well.

```
-e "NODE_ENV=production"
```

## Global npm dependencies

If you need to install global npm dependencies, it is recommended to place those dependencies in the [non-root user](#non-root-user) directory. To achieve this, add the following line to your `Dockerfile`

```Dockerfile
ENV NPM_CONFIG_PREFIX=/home/node/.npm-global

## optionally if you want to run npm global bin without specifying path
ENV PATH=$PATH:/home/node/.npm-global/bin
```

## Handling Kernel Signals

Node.js was not designed to run as PID 1 which leads to unexpected behaviour when running inside of Docker. For example, a Node.js process running as PID 1 will not respond to `SIGINT` (`CTRL-C`) and similar signals. As of Docker 1.13, you can use the `--init` flag to wrap your Node.js process with a [lightweight init system](https://github.com/krallin/tini) that properly handles running as PID 1.

```
docker run -it --init node
```

You can also include Tini [directly in your Dockerfile](https://github.com/krallin/tini#using-tini), ensuring your process is always started with an init wrapper.

## Non-root User

By default, Docker runs container as root which inside of the container can pose as a security issue. You would want to run the container as an unprivileged user wherever possible. The node images provide the `node` user for such purpose. The Docker Image can then be run with the `node` user in the following way:

```
-u "node"
```

Alternatively, the user can be activated in the `Dockerfile`:

```Dockerfile
FROM nmoeini/node:10-xenial
...
# At the end, set the user to use when running this image
USER node
```

Note that the `node` user is neither a build-time nor a run-time dependency and it can be removed or altered, as long as the functionality of the application you want to add to the container does not depend on it.

If you do not want nor need the user created in this image you can remove it with the following:

```Dockerfile
# For debian based images use:
RUN userdel -r node

# For alpine based images use:
RUN deluser --remove-home node
```

If you need to change the uid/gid of the user you can use:

```Dockerfile
RUN groupmod -g 999 node && usermod -u 999 -g 999 node
```

If you need another name for the user (ex. `myapp`) execute:

```Dockerfile
RUN usermod -d /home/myapp -l myapp node
```

For alpine based images, you do not have `groupmod` nor `usermod`, so to change the uid/gid you have to delete the previous user:

```Dockerfile
RUN deluser --remove-home node \
  && addgroup -S node -g 999 \
  && adduser -S -G node -u 999 node
```

## Memory

By default, any Docker Container may consume as much of the hardware such as CPU and RAM. If you are running multiple containers on the same host you should limit how much memory they can consume.

```
-m "300M" --memory-swap "1G"
```

## CMD

When creating an image, you can bypass the `package.json`'s `start` command and bake it directly into the image itself. First off this reduces the number of processes running inside of your container. Secondly it causes exit signals such as `SIGTERM` and `SIGINT` to be received by the Node.js process instead of npm swallowing them.

```Dockerfile
CMD ["node","index.js"]
```

## Docker Run

Here is an example of how you would run a default Node.JS Docker Containerized application:

```
$ docker run \
  -e "NODE_ENV=production" \
  -u "node" \
  -m "300M" --memory-swap "1G" \
  -w "/home/node/app" \
  --name "my-nodejs-app" \
  node [script]
```

## Security

The Docker team has provided a tool to analyze your running containers for potential security issues. You can download and run this tool from here: https://github.com/docker/docker-bench-security

