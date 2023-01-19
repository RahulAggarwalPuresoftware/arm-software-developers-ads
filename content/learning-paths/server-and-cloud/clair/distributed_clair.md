---
# User change
title: "Clair in the distributed mode"

weight: 4 # 1 is first, 2 is second, etc.

# Do not modify these elements
layout: "learningpathall"
---

## Prerequisites

* An AWS account.
* [docker](https://docs.docker.com/engine/install/ubuntu/) and [docker-compose](https://docs.docker.com/compose/install/linux/) (latest version preferred).
* [go](https://go.dev/doc/install) (latest version preferred).

## Install and run Clair in the Distributed Mode

In a distributed deployment, each Clair service i.e. indexer, matcher and notifier, runs in it's own OS process. `docker-compose.yaml` already has targets defined to run these 3 services. Unlike the Combo mode, all 3 services will run inside containers. So, there is no need to expose postgres port 5432, as all 3 services of Clair are in same container network as postgres.

NOTE: The below mentioned steps are tested successfully with Clair v4.5.1.

Using your AWS Account, launch an Arm 64-bit instance running Ubuntu. Refer [this](https://github.com/zachlas/arm-software-developers-ads/blob/main/content/learning-paths/server-and-cloud/aws/gui.md) for more details.

Download Clair v4.5.1:

```console
wget https://github.com/quay/clair/releases/download/v4.5.1/clair-v4.5.1.tar.gz
tar -xvf clair-v4.5.1.tar.gz
```

Execute below command to setup postgres database:

```console
sudo docker-compose up -d clair-database
```

We will need a load balancer to divert traffic to the correct service i.e. indexer, matcher and notifier as requested. We will use "Traefik" here, which will run on port 6060. To setup traefik, execute the below command:

```console
sudo docker-compose up -d traefik
```

Next, run indexer, matcher and notifier as 3 separate processes. `docker-compose.yaml` already has targets defined for the same.

```console
sudo docker-compose up -d indexer matcher notifier
```

You can verify if all 5 containers, i.e. clair-database, traefik, indexer, matcher, notifier, are running, using docker CLI:

```console
sudo docker ps
```

To check logs in each of the Clair's service:

```console
sudo docker logs clair-indexer
sudo docker logs clair-matcher
sudo docker logs clair-notifier
```

Now that everything is running, you can submit manifest for generating the vulnerability report.