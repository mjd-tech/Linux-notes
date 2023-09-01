# Docker
Run applications in containers

## Examples

Create and run a container

    docker run image_name

Run a container in the background (detached)

    docker run -d image_name

Open a shell inside a running container:

    docker exec -it container_name bash

Run a container and publish a container’s port(s) to the host.

    docker run -p host_port:container_port image_name

List currently running containers:

    docker ps

List all docker containers (running and stopped):

    docker ps --all

Start or stop an existing container:

    docker start|stop container_name (or container-id)

Remove a stopped container:

    docker rm container_name

Fetch and follow the logs of a container:

    docker logs -f container_name

Inspect a running container:

    docker inspect container_name (or container_id)

List local images

    docker images

Pull an image from a Docker Hub

    docker pull image_name

Delete an Image

    docker rmi image_name


## Other commands

View resource usage stats

    docker container stats

Build an Image from a Dockerfile

    docker build -t image_name

Build an Image from a Dockerfile without the cache

    docker build -t image_name . –no-cache

Remove all unused images

    docker image prune

Search Docker Hub for an image

    docker search image_name


