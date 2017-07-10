# Apache Solr (Built with Ansible Container)

[![Build Status](https://travis-ci.org/geerlingguy/ac-solr.svg?branch=master)](https://travis-ci.org/geerlingguy/ac-solr) [![](https://images.microbadger.com/badges/image/geerlingguy/solr.svg)](https://microbadger.com/images/geerlingguy/solr "Get your own image badge on microbadger.com")

This project is in it's extremely early stages. _There will be bugs!_ You may be better served using the [official Solr Docker image](https://hub.docker.com/_/solr/) if it meets your requirements.

This project is composed of three main parts:

  - **Ansible Container project**: This project is maintained on GitHub: [geerlingguy/ac-solr](https://hub.docker.com/r/geerlingguy/solr/). Please file issues, support requests, etc. against this GitHub repository.
  - **Docker Hub Image**: If you just want to use [the `geerlingguy/solr` Docker image](https://hub.docker.com/r/geerlingguy/solr/) in your project, you can pull it from Docker Hub.
  - **Ansible Role**: If you need a flexible Ansible role that's compatible with both traditional servers and containerized builds, check out [`geerlingguy.solr`](https://galaxy.ansible.com/geerlingguy/solr/) on Ansible Galaxy. (This is the Ansible role that does the bulk of the work in managing the Apache Solr container.)

## Versions

Currently maintained versions include:

  - `6.x`, `6.6.0`, `latest`: Apache Solr 6.x
  - `5.x`, `5.5.4`: Apache Solr 5.x
  - `4.x`, `4.10.4`: Apache Solr 4.x

## Standalone Usage

If you want to use the `geerlingguy/solr` image from Docker Hub, you don't need to install or use this project at all. You can quickly build a Solr container locally with:

    docker run -d --name=solr -p 8983:8983 geerlingguy/solr:latest /opt/solr/bin/solr start -p 8983 -s /var/solr -f -force

(For Solr 4.x, drop the final `-force` argument.)

You can also wrap up that configuration in a `Dockerfile` and/or a `docker-compose.yml` file if you want to keep things simple. For example:

    version: "3"
    
    services:
      solr:
        image: geerlingguy/solr:latest
        container_name: solr
        ports:
          - "8983:8983"
        restart: always
        # See 'Custom and Persisten Solr cores' for instructions for volumes.
        volumes: []
        command: ["/opt/solr/bin/solr", "start", "-p", "8983", "-s", "/var/solr", "-f", "-force"]

Then run:

    docker-compose up -d

Now you should be able to access the Solr admin dashboard at `http://localhost:8983/`.

### Custom and Persistent Solr cores

The default installation includes a `collection1` core in the `SOLR_HOME` directory, `/var/solr`.

Apache Solr will autodiscover any Solr cores in `SOLR_HOME` by searching for `core.properties` files inside each subdirectory. A standard convention for a single Solr core is to to mount a host directory as a volume with the core directory, containing the core's `conf`, `data`, and `core.properties` files.

As an example, if you have a solr core directory named `mysearch` (with a `mysearch/core.properties` file inside), mount it as a volume like `-v ./mysearch:/var/solr/mysearch:rw`. If you have multiple solr cores (all defined inside a `cores` directory), mount them inside a `cores` directory like `-v ./cores:/var/solr/cores`.

Or, if using a Docker Compose file:

    services:
      solr:
        ...
        volumes:
          # If you have one core:
          - ./mysearch:/var/solr/mysearch:rw
          # If you have multiple cores:
          - ./cores:/var/solr/cores:rw

You can also mount volumes from a data container or elsewhere; the key is you will be able to both provide custom Solr configuration (`schema.xml`, `solrconfig.xml`, etc.), and also have a persistent `data` directory that lives outside the container.

## Management with Ansible Container

### Prerequisites

Before using this project to build and maintain a Solr images for Docker, you need to have the following installed:

  - [Docker Community Edition](https://docs.docker.com/engine/installation/) (for Mac, Windows, or Linux)
  - [Ansible Container](https://docs.ansible.com/ansible-container/installation.html)

### Build the image

  1. Build the Solr Docker image; run this command in the root directory:

         ansible-container --var-file vars-6.x.yml build

  1. Once the image is built, you can run `docker images` to see the `acsolr-solr` image that was generated.

Older Solr versions are also supported—specify the vars file for the version you would like to install to switch to that version of Solr. (**Note**: Until 0.9.2+ is released, you must [install and use Ansible Container from source](https://docs.ansible.com/ansible-container/installation.html#running-from-source) due to [this PR only being in devel so far](https://github.com/ansible/ansible-container/pull/609).)

### Run the image as a container

    ansible-container --var_file vars-6.x.yml run

**Note**: The `--var_file` option currently seems to not work with `run`, so you may need to just use `run` alone and paste any relevant variables into the `container.yml` file, at least until [this issue](https://github.com/ansible/ansible-container/issues/643) is resolved.

You should be able to reach the Solr dashboard by accessing [http://localhost:8983/](http://localhost:8983/) in your browser.

(Use `stop` to stop the container, and `destroy` to reset the containers and _all_ images.)

### Push the image to Docker Hub

Currently, the process for updating this image on Docker Hub is manual. Eventually this will be automated via Travis CI using `ansible-container push` (currently, this is waiting on [this issue](https://github.com/ansible/ansible-container/issues/630) to be resolved).

  1. Log into Docker Hub on the command line:

         docker login --username=geerlingguy

  1. Tag the latest version (only if this is the latest/default version):

         docker tag [image id] geerlingguy/solr:latest

  1. Tag the Solr major version:

         docker tag [image id] geerlingguy/solr:6.x # or 5.x, 4.x...
         docker tag [image id] geerlingguy/solr:6.6.0 # the specific version

  1. Push tags to Docker Hub:

         docker push geerlingguy/solr:latest # (if this was just tagged)
         docker push geerlingguy/solr:6.x # or 5.x, 4.x...
         docker push geerlingguy/solr:6.6.0 # the specific version


## License

MIT / BSD

## Author Information

This container build was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
