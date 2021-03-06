[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-kibana/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-kibana/tree/master)
[![Slack](https://img.shields.io/badge/slack-join%20chat%20%E2%86%92-e01563.svg)](http://slack.oss.bitnami.com)
[![Kubectl](https://img.shields.io/badge/kubectl-Available-green.svg)](https://raw.githubusercontent.com/bitnami/bitnami-docker-kibana/master/kubernetes.yml)

# What is Kibana?

> Kibana is an open source, browser based analytics and search dashboard for Elasticsearch. Kibana is a snap to setup and start using. Kibana strives to be easy to get started with, while also being flexible and powerful, just like Elasticsearch

[elastic.co/products/kibana](https://www.elastic.co/products/kibana)

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-kibaba/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

## Kubernetes

> **WARNING:** This is a beta configuration, currently unsupported.

Get the raw URL pointing to the `kubernetes.yml` manifest and use `kubectl` to create the resources on your Kubernetes cluster like so:

```bash
$ kubectl create -f https://raw.githubusercontent.com/bitnami/bitnami-docker-kibana/master/kubernetes.yml
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Get this image

The recommended way to get the Bitnami Kibana Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/kibana).

```bash
$ docker pull bitnami/kibana:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/bitnami/kibana/tags/) in the Docker Hub Registry.

```bash
$ docker pull bitnami/kibana:[TAG]
```

If you wish, you can also build the image yourself.

```bash
$ docker build -t bitnami/kibana:latest https://github.com/bitnami/bitnami-docker-kibana.git
```

# How to use this image

## Run the application using Docker Compose

This is the recommended way to run Kibana. You can use the following docker compose template:

```yaml
version: '2'

services:
  kibana:
    image: 'bitnami/kibana:latest'
    ports:
      - 5601:5601
    environment:
      - KIBANA_ELASTICSEARCH_URL=elasticsearch
    volumes:
      - 'kibana_data:/bitnami'
  elasticsearch:
    image: 'bitnami/elasticsearc:latest'
    ports:
      - 9200:9200
    volumes:
      - 'elasticsearch_data:/bitnami'
volumes:
  kibana_data:
    driver:local
  elasticsearch_data:
    driver:local
```

## Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```
  $ docker network create kibana_network
  ```

2. Run the Elasticsearch container:

  ```
  $ docker run -d -p 9200:9200 --name elasticsearch --net=kibana_network bitnami/elasticsearch
  ```

3. Run the Kibana container:

  ```
  $ docker run -d -p 5601:5601 --name kibana --net=kibana_network \
    -e KIBANA_ELASTICSEARCH_URL=elasticsearch \
    bitnami/kibana
  ```

Then you can access your application at http://your-ip:5601/

# Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the Elasticsearch data](https://github.com/bitnami/bitnami-docker-elasticsearch#persisting-your-application).

The above examples define docker volumes namely `elasticsearch_data` and `kibana_data`. The Kibana application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

```bash
$ docker run -v /path/to/kibana-persistence:/bitnami bitnami/kibana:latest
```

or using Docker Compose:

```yaml
kibana:
  image: bitnami/kibana:latest
  volumes:
    - /path/to/kibana-persistence:/bitnami
```

# Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a Kibana server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

## Using the Command Line

### Step 1: Create a network

```bash
$ docker network create app-tier --driver bridge
```

### Step 2: Launch the Kibana server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Kibana container to the `app-tier` network.

```bash
$ docker run -d --name kibana-server \
    --network app-tier \
    bitnami/kibana:latest
```

### Step 3: Launch your application container

```bash
$ docker run -d --name myapp \
    --network app-tier \
    YOUR_APPLICATION_IMAGE
```

> **IMPORTANT**:
>
> 1. Please update the **YOUR_APPLICATION_IMAGE_** placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `kibana-server` to connect to the Kibana server

## Using Docker Compose

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the Kibana server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  kibana:
    image: 'bitnami/kibana:latest'
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the **YOUR_APPLICATION_IMAGE_** placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `kibana` to connect to the Kibana server

Launch the containers using:

```bash
$ docker-compose up -d
```

# Configuration

## Configuration file

The image looks for configurations in `/bitnami/kibana/conf/`. As mentioned in [Persisting your application](#persisting-your-application) you can mount a volume at `/bitnami` and copy/edit the configurations in the `/path/to/kibana-persistence/kibana/conf/`. The default configurations will be populated to the `conf/` directory if it's empty.

### Step 1: Run the Kibana image

Run the Kibana image, mounting a directory from your host.

```bash
$ docker run --name kibana -v /path/to/kibana-persistence:/bitnami bitnami/kibana:latest
```

or using Docker Compose:

```yaml
kibana:
  image: bitnami/kibana:latest
  volumes:
    - /path/to/kibana-persistence:/bitnami
```

### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```bash
$ vi /path/to/kibana-persistence/kibana/conf/kibana.conf
```

### Step 3: Restart Kibana

After changing the configuration, restart your Kibana container for changes to take effect.

```bash
$ docker restart kibana
```

or using Docker Compose:

```bash
$ docker-compose restart kibana
```

Refer to the [configuration](http://kibana.io/topics/config) manual for the complete list of configuration options.

# Logging

The Bitnami Kibana Docker image sends the container logs to the `stdout`. To view the logs:

```bash
$ docker logs kibana
```

or using Docker Compose:

```bash
$ docker-compose logs kibana
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

# Maintenance

## Upgrade this image

Bitnami provides up-to-date versions of Kibana, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
$ docker pull bitnami/kibana:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/kibana:latest`.

### Step 2: Stop and backup the currently running container

Stop the currently running container using the command

```bash
$ docker stop kibana
```

or using Docker Compose:

```bash
$ docker-compose stop kibana
```

Next, take a snapshot of the persistent volume `/path/to/kibana-persistence` using:

```bash
$ rsync -a /path/to/kibana-persistence /path/to/kibana-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the Elasticsearch data](https://github.com/bitnami/bitnami-docker-elasticsearch#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

### Step 3: Remove the currently running container

```bash
$ docker rm -v kibana
```

or using Docker Compose:

```bash
$ docker-compose rm -v kibana
```

### Step 4: Run the new image

Re-create your container from the new image, [restoring your backup](#restoring-a-backup) if necessary.

```bash
$ docker run --name kibana bitnami/kibana:latest
```

or using Docker Compose:

```bash
$ docker-compose start kibana
```

# Notable Changes

## 4.5.4-r1

- `ELASTICSEARCH_URL` parameter has been renamed to `KIBANA_ELASTICSEARCH_URL`.
- `ELASTICSEARCH_PORT` parameter has been renamed to `KIBANA_ELASTICSEARCH_PORT`.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-kibana/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-kibana/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-kibana/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright 2016-2017 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
