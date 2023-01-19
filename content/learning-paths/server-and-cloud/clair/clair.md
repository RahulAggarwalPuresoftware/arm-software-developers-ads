---
# User change
title: "Introduction to Clair and it's deployment models."

weight: 2 # 1 is first, 2 is second, etc.

# Do not modify these elements
layout: "learningpathall"
---

## What is Clair

[Clair](https://github.com/quay/clair) is an application for parsing image contents and reporting vulnerabilities affecting the contents. This is done via static analysis and not at runtime.

Clair supports the extraction of contents and assignment of vulnerabilities from the following official base containers:

* Ubuntu
* Debian
* RHEL
* Suse
* Oracle
* Alpine
* AWS Linux
* VMWare Photon
* Python

The above list defines Clair's current support matrix.

## How Clair Works

Clair can run in several modes - Indexer, matcher, notifier or combo mode. In combo mode, everything runs in a single OS process.
Clair's analysis is broken into three distinct parts.

### Indexing

Indexing starts with submitting a Manifest to Clair. On receipt, Clair will fetch layers, scan their contents, and return an intermediate representation called an IndexReport.
Manifests are Clair's representation of a container image. Clair leverages the fact that OCI Manifests and Layers are content-addressed to reduce duplicated work.
Once a Manifest is indexed, the IndexReport is persisted for later retrieval.

### Matching

Matching is taking an IndexReport and correlating vulnerabilities affecting the manifest the report represents.
Clair is continually ingesting new security data and a request to the matcher will always provide you with the most up to date vulnerability analysis of an IndexReport.

### Notifications

Clair implements a notification service.
When new vulnerabilities are discovered, the notifier service will determine if these vulnerabilities affect any indexed Manifests. The notifier will then take action according to its configuration.

## Postgres

Clair uses PostgreSQL for its data persistence. Migrations are supported so you should only need to point Clair to a fresh database and have it do the setup for you.

## Deploying Clair

### Combined Deployment

![combo_mode_clair_pics](https://user-images.githubusercontent.com/87687089/213428835-6e54ee7e-885c-4114-9123-348e162924b2.PNG)

In a combined deployment, all the Clair processes run in a single OS process. This is by far the easiest deployment model to configure as it involves the least moving parts. To configure this model you will provide all node types the same database and start Clair in combo mode.
In this mode, any configuration informing Clair how to talk to other nodes is ignored, it is not needed as all intra-process communication is done directly.
For added flexibility, it's also supported to split the databases while in combo mode.
Since Clair is conceptually a set of micro-services, its processes do not share database tables even when combined into the same OS process.

### Distributed Deployment

![distributive_mode_clair_pic](https://user-images.githubusercontent.com/87687089/213429015-2a574d77-cf44-4310-a003-99e7afacded2.PNG)

If your application needs to asymmetrically scale or you expect high load you may want to consider a distributed deployment.
In a distributed deployment, each Clair process runs in its own OS process. Typically this will be a Kubernetes or OpenShift Pod.
A load balancer must be setup in this deployment model. The load balancer will route traffic between Clair nodes along with routing API requests via path based routing to the correct services.Keep in mind a config file per process is not need. Processes only use the values necessary for their configured mode.

