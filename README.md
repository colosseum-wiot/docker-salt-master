# SaltStack Master v2018.3.2

Dockerfile to build a [SaltStack](https://www.saltstack.com) Master image for the Docker opensource container platform.

SaltStack Master is set up in the Docker image using the install from git source method as documented
in the the [official bootstrap](https://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html) documentation.

For other methods to install SaltStack please refer to the
[Official SaltStack Installation Guide](https://docs.saltstack.com/en/latest/topics/installation/index.html).

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [Custom Recipes](#custom-recipes)
  - [Minion Keys](#minion-keys)
  - [Available Configuration Parameters](#available-configuration-parameters)
- [Usage](#usage)
- [Shell Access](#shell-access)
- [References](#references)

## Installation

Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/cdalvaro/saltstack-master/)
and is the recommended method of installation.

```sh
docker pull cdalvaro/saltstack-master:2018.3.2
```

You can also pull the latest tag which is built from the repository HEAD

```sh
docker pull cdalvaro/saltstack-master:latest
```

Alternatively you can build the image locally.

```sh
docker build -t cdalvaro/saltstack-master github.com/cdalvaro/saltstack-master
```

## Quick Start

The quickest way to get started is using [docker-compose](https://docs.docker.com/compose/).

```sh
wget https://gitlab.com/cdalvaro/saltstack-master/raw/master/docker-compose.yml
```

Start SaltStack master using:

```sh
docker-compose up --detach
```

Alternatively, you can manually launch the `saltstack-master`  container:

```sh
docker run --name salt_master --detach \
    --publish 4505:4505/tcp --publish 4506:4506/tcp \
    --env 'SALT_LOG_LEVEL=info' \
    --read-only --volume ./srv/:/srv/ \
    cdalvaro/saltstack-master:2018.3.2
```

## Configuration

### Custom Recipes

This image does not require storing data out of the container.

But it is necessary to mount the `/srv/` volume ir order to provide your custom recipes.

### Minion Keys

Minion keys can be added automatically on startup to SaltStack master by mounting the volume `/etc/salt-docker/keys`
and copying the minion keys inside `keys/minions/` directory:

```sh
mkdir -p keys/minions
cp -v /etc/salt/pki/minion/minion.pub keys/minions/minion1

docker run --name salt_master -d \
    --publish 4505:4505/tcp --publish 4506:4506/tcp \
    --env 'SALT_LOG_LEVEL=info' \
    --read-only --volume ./srv/:/srv/ \
    --volume ./keys/:/etc/salt-docker/keys/ \
    cdalvaro/saltstack-master:2018.3.2
```

## Available Configuration Parameters

Please refer the docker run command options for the `--env-file` flag where you can specify all required environment variables in a single file.
This will save you from writing a potentially long docker run command. Alternatively you can use docker-compose.

Below is the list of available options that can be used to customize your SaltStack master installation.

| Parameter | Description |
|-----------|-------------|
| `SALT_LOG_LEVEL` | The level of messages to send to the console. One of 'garbage', 'trace', 'debug', info', 'warning', 'error', 'critical'. Default: 'warning' |
| `SALT_LEVEL_LOGFILE` | The level of messages to send to the log file. One of 'garbage', 'trace', 'debug', info', 'warning', 'error', 'critical'. Default: 'warning' |

Any parameter not listed in the table and available in the next
[link](https://docs.saltstack.com/en/latest/ref/configuration/examples.html#configuration-examples-master)
can be set by creating the directory `confs` and adding into it configurations files with the desired parameters:

```sh
mkdir confs
cat > confs/ports.conf << EOF
# The tcp port used by the publisher:
publish_port: 3505
# The port used by the communication interface.
ret_port: 3506
EOF

docker run --name salt_master -d \
    --publish 3505:3505/tcp --publish 3506:3506/tcp \
    --env 'SALT_LOG_LEVEL=info' \
    --read-only --volume ./srv/:/srv/ \
    --volume ./confs/:/etc/salt-docker/confs/ \
    cdalvaro/saltstack-master:2018.3.2
```

## Usage

To test which salt minions are listening the following command can be executed from the master service:

```sh
docker-compose exec master salt '*' test.ping
```

Then, you can apply salt states to your minions:

```sh
docker-compose exec master salt '*' state.apply
```

## Shell Access

For debugging and maintenance purposes you may want access the container shell.
If you are using docker version 1.3.0 or higher you can access a running container shell using docker exec command.

```sh
docker exec -it salt_master bash
```

## References

- https://docs.saltstack.com/en/latest/topics/installation/index.html
- https://docs.saltstack.com/en/latest/topics/tutorials/salt_bootstrap.html
- https://github.com/saltstack/salt/releases