# Glossary

## Image

- The application and its dependencies, packed together.
- Usually, an image is derived from multiple base images that are layer
  stacked on top of each other, to form the container's filesystem.
- Images are **immutable** once created.

## Container

- A container is a runtime **instance** of a docker image.
- It consists of: A Docker image; An execution environment; A standard
  set of instructions

## Volumes

- Since images are **immutable**, a containerized application **cannot
  write to it’s own image!** That is why we need volumes.
- Volumes allow the container to **write to the host's filesystem**.
- Volumes are designed to **persist data**, independent of the
  container’s life cycle.
- Docker therefore **never** automatically deletes volumes when you
  remove a container, nor will it “garbage collect” volumes that are no
  longer referenced by a container.

## Dockerfile

- A text file that defines how to build a Docker image.
- Defines which images to use, which programs to install and which files
  to copy to get the environment working as needed.

## Compose
Does what "docker run" does, but instead of passing a bunch of arguments on the 
command line, you create a YAML file with the arguments, and run "docker-compose"

- Command-line tool and YAML file format for defining and running containers 
- Can deploy multiple containers/images with just one command.

