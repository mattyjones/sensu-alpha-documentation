# Installing and configuring Sensu 2.0

The Sensu 2.0 binaries are statically linked and can be deployed to any Linux or Windows operating system.

## Backend

The Sensu Backend (sensu-backend) is a single statically linked binary that can be deployed via packages (.deb or .rpm) or Docker image.

## Agent

The Sensu Agent (sensu-agent) is a single statically linked binary that can be deployed via packages (.deb or .rpm) or Docker image.

## Linux - Package Repositories

### Debian / Ubuntu

Add the Sensu prerelease repository.

```sh
curl -s https://packagecloud.io/install/repositories/sensu/prerelease/script.deb.sh | sudo bash
```

Install the packages from the Sensu prerelease repository.

```sh
sudo apt-get install sensu-backend sensu-agent
```

### RHEL / CentOS

Add the Sensu prerelease repository.

```sh
curl -s https://packagecloud.io/install/repositories/sensu/prerelease/script.rpm.sh | sudo bash
```

Install the Sensu backend and agent packages.

```sh
sudo yum install sensu-backend sensu-agent
```

## Linux - Configuration

The example config files list all of the configurable options for each service.

#### Sensu Backend

Copy the example backend config file to the default config path.

```sh
sudo cp /etc/sensu/backend.yml.example /etc/sensu/backend.yml
```

The backend config requires `state-dir` to be set. The example config sets `state-dir` to `/var/lib/sensu` by
default.

#### Sensu Agent

Copy the example agent config file to the default config path.

```sh
sudo cp /etc/sensu/agent.yml.example /etc/sensu/agent.yml
```

In order for the agent to function it will need to have a list of one or more backends to point to. This can be set
by setting `backend-url`.

```yaml
backend-url:
  - "ws://127.0.0.1:8081"
```

## Linux - Starting the services

#### Ubuntu 14.04 / CentOS 6 / RHEL 6

Start the services using the sysvinit scripts.

```sh
sudo /etc/init.d/sensu-backend start
sudo /etc/init.d/sensu-agent start
```

#### Ubuntu 16.04 / CentOS 7 / RHEL 7

Start the services using systemd.

```sh
sudo systemctl start sensu-backend
sudo systemctl start sensu-agent
```

## Windows

Coming soon.

## Docker

Sensu 2.0 can be run via [Docker](https://www.docker.com/) or [rkt](https://coreos.com/rkt) using the [sensuapp/sensu](https://hub.docker.com/r/sensuapp/sensu/) image. When running Sensu from Docker there are a couple of things to take into consideration.

The backend requires 3 exposed ports and persistent storage. This example uses a shared filesystem. Sensu 2.0 is backed by a distributed database, and its storage should be provisioned accordingly.  We recommend local storage or something like Throughtput Optimized or Provisioned IOPS EBS if local storage is unavailable.  The 3 exposed ports are:

- 2380: Sensu storage peer listener (only other sensu-backends need access to this port)
- 3000: Sensu dashboard (not yet complete)
- 8080: Sensu API (all users need access to this port)
- 8081: Agent API (all agents need access to this port)

We suggest, but do not require, persistent storage for sensu-agents. The Sensu Agent will cache runtime assets locally for each check (see [Checks and Assets](06-checks-and-assets.md) for more details). This storage should be unique per sensu-agent process.

### Dashboards and Visualizations

Currently Uchiwa is not yet supported, but a new dashboard is under active development.

### How To

1. Start the sensu-backend process

`docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-backend -p 2380:2380 -p 3000:3000 -p 8080:8080 -p 8081:8081 sensuapp/sensu-go:2.0.0-alpha sensu-backend start`

2. Start an agent

In this case, we're starting an agent whose ID is the hostname with the webserver and system subscriptions. This assumes that sensu-backend is running on another host named sensu.yourdomain.com. If you are running these locally on the same system, be sure to add `--link sensu-backend` to your Docker arguments and change the backend URL `--backend-url ws://sensu-backend:8081`.

`docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-agent sensuapp/sensu-go:2.0.0-alpha sensu-agent start --backend-url ws://sensu.yourdomain.com:8081 --subscriptions webserver,system --cache-dir /var/lib/sensu`

A note about sensuctl and Docker:

It's possible to use sensuctl via Docker, but sensuctl stores configuration locally in the current user's home directory. If you choose to use sensuctl from Docker, make sure to mount the config directory. You can set a script or alias to call sensuctl like so:

`alias sensuctl="docker run -v $HOME/.config/sensu:/var/lib/sensu sensuapp/sensu-go:2.0.0-alpha sensuctl --config-dir /var/lib/sensu`
