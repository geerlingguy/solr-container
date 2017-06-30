# Apache Solr (Built with Ansible Container)

[![](https://images.microbadger.com/badges/image/geerlingguy/solr.svg)](https://microbadger.com/images/geerlingguy/solr "Get your own image badge on microbadger.com")

This project is in it's extremely early stages. _There will be bugs!_

## Prerequisites

Before using this project to build and maintain a Solr images for Docker, you need to have the following installed:

  - [Docker Community Edition](https://docs.docker.com/engine/installation/) (for Mac, Windows, or Linux)
  - [Ansible Container](https://docs.ansible.com/ansible-container/installation.html)

## Usage

  1. Build the Solr Docker image; run this command in the root directory:

      ansible-container --debug --var-file vars.yml build

  1. Once the image is built, you can run `docker images` to see the `acsolr-solr` image that was generated.
  1. To run the image, run `ansible-container run`

## Pushing to Docker Hub

Currently, the process for updating this image on Docker Hub is manual. Eventually this will be automated via Travis CI.

  1. Log into Docker Hub on the command line:

      docker login --username=geerlingguy

  1. Tag the latest version:

      docker tag [image id] geerlingguy/solr:latest

  1. Tag the Solr major version:

      docker tag [image id] geerlingguy/solr:6.x # or 5.x, 4.x...

  1. Push tags to Docker Hub:

      docker push geerlingguy/solr:latest
      docker push geerlingguy/solr:6.x # or 5.x, 4.x...


## License

MIT / BSD

## Author Information

This container build was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
