# Helm Chart for Pipelinewise

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## Introduction

This [Helm](https://github.com/kubernetes/helm) chart installs a CronJob in a Kubernetes cluster. This CronJob takes a Pipelinewise configuration and runs a sync job between a tap and a target

## Prerequisites

- Kubernetes cluster 1.10+
- Helm 3.0.0+
- PV provisioner support in the underlying infrastructure.
- You have created Pipelinewise configuration files as described [here](https://transferwise.github.io/pipelinewise/installation_guide/creating_pipelines.html)
- Have have created and pushed a Docker image with the connectors you need according to the [docs](https://transferwise.github.io/pipelinewise/installation_guide/installation.html#running-in-docker) 

## Installation

### Create Kubernetes secret
First, we need to create a kubernetes secret from the configuration files so we can mount them later on as a volume.

Assuming we have a `tap_mysql.yml` and a `target_snowflake.yml` config files, this would be the command to generate the secret: 
```bash
`kubectl create secret generic mysql-sf-secret --from-file=./tap_mysql.yml --from-file=./target_snowflake.yml`
```

### Add Helm repository

```bash
helm repo add pipelinewise https://github.com/fairfly/helm-pipelinewise
helm repo update
```

OR Clone this repo locally

### Configure the chart

The following items can be set via `--set` flag during installation or configured by editing the `values.yaml` directly (need to download the chart first).

#### Configure the pipeline

- **image.repository**: The name of the docker image you built as part of the pre-requisites 
- **pipelinewiseConfigSecretName**: The name of the kubernetes secret with the pipelinewise configuration files
- **tapName**: The name of the tap to run
- **targetName**: The name the target to insert data into
- **schedule**: The CronJob schedule

### Install the chart

Install the Pipelinewise helm chart with a release name `my-sync-job`:

```bash
helm install --name my-sync-job pipelinewise \
	--set image.repository="imagename" \
	--set pipelinewiseConfigSecretName="pplwise-secret" \
	--set tapName=lol \
	--set targetName=yay
```

### Install from local clone

```bash
git clone https://github.com/fairfly/helm-pipelinewise
cd helm-pipelinewise
helm install --name my-sync-job .
```

We recommend naming your release to describe the tap and target you are running. E.g `mysql-to-snowflake`.

## Uninstallation

To uninstall/delete the `my-release` deployment:

```bash
helm delete --purge my-release
```

## Configuration

The following table lists the configurable parameters of the chart and the default values.

| Parameter                                                                   | Description                                                                                                        | Default                         |
| --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------| ------------------------------- |
| **Image**                                                                   |
| `image.repository`                                                          | Image name                                                                                                    | ``                   |
| `image.tag`                                                                 | Image tag                                                                                                     | `latest`                        |
| `imagePullSecrets`                                                          | Image pull secret                                                                                             | `nil`                           |
| **Pipeline**
| `pipelinewiseConfigSecretName`                                                                   | The name of the secret with all the pipelinewise configuration files                  | `nil`                           |
| `tapName`                                                                | The name of the tap to pull data from             | `nil`                           |
| `targetName`                                                                | The name of the target to sync data into | `nil`                           |
| `schedule`                                                                | The CronJob schedule | `nil`                           |
| `env`                                                                | Any environment variables you want to be injected into the cronjob pod | `nil`                           |
| **Persistence**                                                             |
| `persistence.storageClass`                                                  | Storage class name of PVCs (use the default type if unset)                                                         | `gp2`                           |
| `persistence.storage`                                                    | ReadWriteOnce or ReadOnly                                                                                          | `[ReadWriteOnce]`               |
| **Resources**                                                               |
| `resources`                                                                 | Pod resource requests and limits for logs                                                                          | `{}`                            |
| **nodeSelector**                                                            |
| `nodeSelector`                                                              | Node labels for pod assignment                                                                                     | `{}`                            |
| **tolerations**                                                             |
| `tolerations`                                                               | Tolerations for pod assignment                                                                                     | `[]`                            |
| **affinity**                                                          |
| `affinity`                                                            | Affinities for pod assignment | `[]`                            |

## Contributing

Feel free to contribute by making a [pull request](https://github.com/fairfly/helm-pipelinewise/pull/new/master).

## License

[Apache License 2.0](/LICENSE)