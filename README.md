# Apache Solr Container (Built with Ansible)

[![CI](https://github.com/geerlingguy/solr-container/workflows/Build/badge.svg?branch=master&event=push)](https://github.com/geerlingguy/solr-container/actions?query=workflow%3ABuild) [![Docker pulls](https://img.shields.io/docker/pulls/geerlingguy/solr)](https://hub.docker.com/r/geerlingguy/solr/)

This project is composed of three main parts:

  - **Ansible project**: This project is maintained on GitHub: [geerlingguy/solr-container](https://github.com/geerlingguy/solr-container). Please file issues, support requests, etc. against this GitHub repository.
  - **Docker Hub Image**: If you just want to use [the `geerlingguy/solr` Docker image](https://hub.docker.com/r/geerlingguy/solr/) in your project, you can pull it from Docker Hub.
  - **Ansible Role**: If you need a flexible Ansible role that's compatible with both traditional servers and containerized builds, check out [`geerlingguy.solr`](https://galaxy.ansible.com/geerlingguy/solr/) on Ansible Galaxy. (This is the Ansible role that does the bulk of the work in managing the Apache Solr container.)

## Versions

Currently maintained versions include:

  - `9.x`, `9.7.0`, `latest`: Apache Solr 9.x
  - `8.x`, `8.11.4`: Apache Solr 8.x
  - `7.x`, `7.7.3`: Apache Solr 7.x

## Standalone Usage

If you want to use the `geerlingguy/solr` image from Docker Hub, you don't need to install or use this project at all. You can quickly build a Solr container locally with:

    docker run -d --name=solr -p 8983:8983 geerlingguy/solr:latest /opt/solr/bin/solr start -p 8983 -f -force

You can also wrap up that configuration in a `Dockerfile` and/or a `docker-compose.yml` file if you want to keep things simple. For example:

    ```
    version: "3"
    
    services:
      solr:
        image: geerlingguy/solr:latest
        container_name: solr
        ports:
          - "8983:8983"
        restart: always
        # See 'Custom and Persistent Solr cores' for instructions for volumes.
        volumes: []
        command: ["/opt/solr/bin/solr", "start", "-p", "8983", "-f", "-force"]

Then run:

    docker-compose up -d

Now you should be able to access the Solr admin dashboard at `http://localhost:8983/`.

### Custom and Persistent Solr cores

The default installation includes a `collection1` core in the `SOLR_HOME` directory, `/var/solr`.

Apache Solr will autodiscover any Solr cores in `SOLR_HOME` by searching for `core.properties` files inside each subdirectory. A standard convention for a single Solr core is to to mount a host directory as a volume with the core directory, containing the core's `conf`, `data`, and `core.properties` files.

Here's an example minimal `core.properties` file, for a core named `mysearch`:

    name=mysearch
    config=solrconfig.xml
    schema=schema.xml
    dataDir=data

So, if you have a solr core directory named `mysearch` (with a `mysearch/core.properties` file inside, and a `conf` and `data` directory for storing Solr configuration and index data, respectively), which looks like this:

    mysearch_conf/
    ├── conf
    │   ├── elevate.xml
    │   ├── mapping-ISOLatin1Accent.txt
    │   ├── protwords.txt
    │   ├── _rest_managed.json
    │   ├── schema_extra_fields.xml
    │   ├── schema_extra_types.xml
    │   ├── schema.xml
    │   ├── solrconfig_extra.xml
    │   ├── solrconfig.xml
    │   ├── solrcore.properties
    │   ├── stopwords.txt
    │   └── synonyms.txt
    ├── core.properties
    └── data

Mount it as a volume like `-v ./mysearch:/var/solr/mysearch:rw`. If you have multiple solr cores (all defined inside a `cores` directory), mount them inside a `cores` directory like `-v ./cores:/var/solr/cores`.

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

## Management with Ansible

### Prerequisites

Before using this project to build and maintain a Solr images for Docker, you need to have the following installed:

  - [Docker Community Edition](https://docs.docker.com/engine/installation/) (for Mac, Windows, or Linux)
  - [Ansible](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### Build the image

First, install Ansible role requirements:

    ansible-galaxy install -r requirements.yml

Then, make sure Docker is running, and run the playbook to build the container:

    ansible-playbook --extra-vars="@vars/7.x.yml" main.yml

(Substitute whatever supported Solr version you desire in the vars path) Once the image is built, you can run `docker images` to see the `solr` image that was generated.

> Note: If you get an error like `Failed to import docker`, run `pip install docker`.

### Push the image to Docker Hub

See the `.github/workflows/build.yml` file in this repository for how it pushes all the tagged images automatically on any commit to the `master` branch.

## License

MIT / BSD

## Author Information

This container build was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
