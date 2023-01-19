---
# User change
title: "Clair in the combo mode"

weight: 3 # 1 is first, 2 is second, etc.

# Do not modify these elements
layout: "learningpathall"
---

## Prerequisites

* An AWS account.
* [docker](https://docs.docker.com/engine/install/ubuntu/) and [docker-compose](https://docs.docker.com/compose/install/linux/) (latest version preferred).
* [go](https://go.dev/doc/install) (latest version preferred).

## Install and run Clair in the Combo Mode

In a combined deployment, all the Clair services run in a single OS process. This is by far the easiest deployment model to configure as it involves the least moving parts.

NOTE: The below mentioned steps are tested successfully with Clair v4.5.1.

Using your AWS Account, launch an Arm 64-bit instance running Ubuntu. Refer [this](https://github.com/zachlas/arm-software-developers-ads/blob/main/content/learning-paths/server-and-cloud/aws/gui.md) for more details.

Download Clair v4.5.1:

```console
wget https://github.com/quay/clair/releases/download/v4.5.1/clair-v4.5.1.tar.gz
tar -xvf clair-v4.5.1.tar.gz
```

We will setup a postgres database for Clair to store all the vulnerabilities specific to containers. `docker-compose.yaml` already has a target "clair-database" to setup a postgres database for Clair. For the combo mode, since postgres service will run inside a private container network and Clair service runs on localhost, we are required to expose postgres port 5432 to localhost. To do so, simply add the following to "clair-database" target in `docker-compose.yaml` file.

```console
ports:
      - "5432:5432"
```

Next, setup the postgres service as below:

```console
sudo docker-compose up -d clair-database
```

Clair uses a configuration file to configure indexer, matcher and notifier. In the combo mode, we will configure indexer, matcher and notifier to communicate to postgres service exposed to port 5432 on localhost. We will use the configuration file present at `clair/local-dev/clair/config.yaml`, and modify the connstring of indexer, matcher and notifier as below:

```console
indexer:
  connstring: host=localhost port=5432 user=clair dbname=indexer sslmode=disable

matcher:
  connstring: host=localhost port=5432 user=clair dbname=matcher sslmode=disable

notifier:
  connstring: host=localhost port=5432 user=clair dbname=notifier sslmode=disable
```

Next, generate the Clair binary with go, as below:

```console
sudo go build ./cmd/clair
```

This will generate a Clair binary in the root of the repo.

Now that the postgres service is running and Clair's configuration is ready, run Clair in the combo mode as below:

```console
./clair -conf "./local-dev/clair/config.yaml" -mode "combo"
```

The running logs on your screen confirms that Clair is running successfully in the combo mode. You can now open a new terminal and submit the manifest to generate the vulnerability report.