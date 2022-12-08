<img src="https://velero.io/docs/main/img/velero.png" width="50%" alt="velero logo">

## Overview

Velero (formerly Heptio Ark) gives you tools to back up and restore your Kubernetes cluster resources and persistent volumes. You can run Velero with a public cloud platform or on-premises. Velero lets you:

* Take backups of your cluster and restore in case of loss.
* Migrate cluster resources to other clusters.
* Replicate your production cluster to development and testing clusters.

Velero consists of:

* A server that runs on your cluster
* A command-line client that runs locally

## Documentation

[The documentation](https://velero.io/docs/) provides a getting started guide and information about building from source, architecture, extending Velero, and more.

Please use the version selector at the top of the site to ensure you are using the appropriate documentation for your version of Velero.

<hr>

### Install on cluster backed by openebs (Restic backups) with minio

```sh
velero install \
	--use-node-agent \
	--default-volumes-to-fs-backup \
	--provider aws \
	--plugins velero/velero-plugin-for-aws:v1.2.1 \
	--bucket backups \
	--secret-file ./credentials-velero \
	--use-volume-snapshots=false \
	--backup-location-config \
	region=minio,s3ForcePathStyle="true",s3Url=http://<minio_ip>:9000
```


### Create Backup On Schedule (CronTab)

```sh
velero schedule create "schedule_name" \
	--schedule="0 3 * * 0" \
	--include-namespaces "namespace_name"
```

### Create a backup now from a created schedule (On-Demand)

```sh
velero backup create \
	--from-schedule "schedule_name"
```

### Create a backup not from a schedule

```sh
velero backup create "backup_name" \
	--include-namespaces "namespace_name"
```

### Check backup details / progress

```sh
velero backup describe "backup_name"
```

### Restore a Backup

```sh
velero restore create "backup_name"
```

### List All Backups

```sh
velero backup get
```

### Delete a backup

```sh
velero backup delete "backup_name"
```

### Delete a schedule

```sh
velero schedule delete "schedule_name"
```

### Uninstall Velero from cluster

```sh
velero uninstall
```
