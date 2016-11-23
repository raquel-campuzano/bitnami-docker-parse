[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-parse/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-parse/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/parse)](https://hub.docker.com/r/bitnami/parse/)
# What is Parse?

> Parse Server is an open source version of the Parse backend that can be deployed to any infrastructure that can run Node.js.

http://parse.com/

# Looking for Parse + Parse Dashboard?

We also provide a Docker Image for Parse Dashboard. Parse Dashboard is a standalone dashboard for managing your Parse apps. You can find it at:

[Bitnami Parse Dashboard](https://github.com/bitnami/bitnami-docker-parse-dashboard)

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run Parse with a Database Container

Running Parse with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run Parse. You can use the following docker compose template:

```yaml
version: '2'

services:
  mongodb:
    image: 'bitnami/mongodb:latest'
    volumes:
      - 'mongodb_data:/bitnami/mongodb'
  application:
    image: 'bitnami/parse:latest'
    environment:
      PARSE_SERVER_HOST: your_host
    ports:
      - '1337:1337'
    volumes:
      - 'parse_data:/bitnami/parse'
    depends_on:
      - mongodb
volumes:
  mongodb_data:
    driver: local
  parse_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create parse-tier
  ```

2. Start a MongoDB database in the network generated:

  ```bash
  $ docker run -d --name mongodb --net=parse-tier bitnami/mongodb
  ```

  *Note:* You need to give the container a name in order to Parse to resolve the host

3. Run the Parse container:

  ```bash
  $ docker run -d -p 1337:1337 --name parse --net=parse-tier bitnami/parse
  ```

Then you can access your application at http://your-ip/parse

## Persisting your application

If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence of the Parse deployment, the above examples define docker volumes namely `mongodb_data` and `parse_data`. The Parse application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:
```yaml
version: '2'

  mongodb:
    image: 'bitnami/mongodb:latest'
    volumes:
      - '/path/to/mongodb-persistence:/bitnami/mongodb'
  parse:
    image: bitnami/parse:latest
    depends_on:
      - mongodb
    ports:
      - 1337:1337
    volumes:
      - '/path/to/parse-persistence:/bitnami/parse'

```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create parse-tier
  ```

2. Create a MongoDB container with host volume:

  ```bash
  $ docker run -d --name mongodb \
    --net parse-tier \
    --volume /path/to/mongodb-persistence:/bitnami/mongodb \
    bitnami/mongodb:latest
  ```

  *Note:* You need to give the container a name in order to Parse to resolve the host

3. Run the Parse container:

  ```bash
  $ docker run -d --name parse -p 1337:1337 \
    --net parse-tier \
    --volume /path/to/parse-persistence:/bitnami/parse \
     bitnami/parse:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of Mongodb and Parse, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Parse container. For the Mongodb upgrade see https://github.com/bitnami/bitnami-docker-mongodb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/parse:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop parse`
 * For manual execution: `$ docker stop parse`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the parse folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm parse`
 * For manual execution: `$ docker rm parse`

5. Run the new image

 * For docker-compose: `$ docker-compose start parse`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name parse bitnami/parse:latest`

# Configuration
## Environment variables
 When you start the parse image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:
```yaml
application:
  image: bitnami/parse:latest
  ports:
    - 1337:1337
  environment:
    - PARSE_SERVER_HOST=my_host
  volumes:
    - 'parse_data:/bitnami/parse'
  depends_on:
    - mongodb
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -e PARSE_SERVER_HOST=my_host -p 1337:1337 --name parse -v /your/local/path/bitnami/parse:/bitnami/parse --network=parse-tier bitnami/parse
```

Available variables:
 - `PARSE_SERVER_HOST`: Parse server host. Default: **127.0.0.1**
 - `PARSE_SERVER_PORT`: Parse server port. Default: **1337**
 - `PARSE_SERVER_MOUNT_PATH`: Parse server mount path. Default: **/parse**
 - `PARSE_SERVER_APP_ID`: Parse server App ID. Default: **myappID**
 - `PARSE_SERVER_MASTER_KEY`: Parse server Master Key: **mymasterKey**
 - `MONGODB_HOST`: Hostname for Mongodb server. Default: **mongodb**
 - `MONGODB_PORT`: Port used by Mongodb server. Default: **27017**

# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop parse`
  * For manual execution: `$ docker stop parse`

2. Copy the Parse data folder in the host:

  ```bash
  $ docker cp /your/local/path/bitnami:/bitnami/parse
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with Parse data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-parse/issues), or submit a
[pull request](https://github.com/bitnami/bitnami-docker-parse/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-parse/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
