---
author: Dennis Kruyt
categories:
- kuberenetes
- volume
- sidecar
- backup
- container
- rclone
cover:
  image: /images/2021/09/CE-Cheap-Vespa-Motor-Tricycle-Adult-Tricycle-Scooter-3-Wheel-Scooter.webp
date: "2021-09-29T07:40:52Z"
description: ""
draft: false
slug: kubernetes-sidecar-backup
summary: kscb is a sidecar container, that is based upon rclone with a crontab. The
  sidecar container in the pod has access to the PV mount and the data on it, Rclone
  can then easily copy/sync to a public/private storage provider
tags:
- kuberenetes
- volume
- sidecar
- backup
- container
- rclone
title: Kubernetes SideCar Backup
---


When you run containers in Kubernetes, you can have also persistent data in those containers on a _PersistentVolume_ (PV). So have I running Bitwarden and Ghost blog and the persistent data on a couple of PV's. So how can we backup the persistent data? Well that depends on your setup and access. So if you have limited access to Kubernetes, for example only to your namespace. And you just want to copy/sync your data to a remote location.

{{< figure src="/images/2021/09/kscb--1-.png" caption="overview" >}}

That's why I created a sidecar container named kscb, that is based upon [rclone](https://rclone.org/) with a crontab. The sidecar container in the pod has access to the PV mount and the data on it, we can mount it read only, so no danger there. Rclone can then easily copy/sync to a public/private storage provider, just take a look at the [supported provider](https://rclone.org/#providers) list on the rclone site.

{{< bookmark url="https://github.com/dkruyt/kscb" title="GitHub - dkruyt/kscb: kscb is a sidecar container, that is based upon rclone with a crontab. The sidecar container in the pod has access to the PV mount and the data on it, we can mount it read only, so no danger there. Rclone can then easily copy/sync to a public/private storage provider" description="kscb is a sidecar container, that is based upon rclone with a crontab. The sidecar container in the pod has access to the PV mount and the data on it, we can mount it read only, so no danger there...." author="dkruyt" publisher="GitHub" thumbnail="https://opengraph.githubassets.com/22da4133c352819969e8aa7102acf052870d592371d068817a1fdb23dee5bb67/dkruyt/kscb" caption="" >}}

The container is very light weight, it is only running the busybox crontab, so it will use only about 1-2 mb of memory when not running rclone backups.

## Setup

First you need to create a rclone config for your storage provider.  You can use the [interactive](https://rclone.org/commands/rclone_config/) method of create a manual file. See the rclone site on how to do this.

```ini
[aws-s3]
type = s3
env_auth = false
access_key_id = <key_id>
region = eu-amsterdam-1
secret_access_key = <secret>
endpoint = <endpoint>
```

Because there are credentials in it we create a Kubernetes secret from the rclone config.

```bash
kubectl create secret generic rclone --from-file rclone.conf
```

Then we create a crontab, this example crontab has 2 entries. one for the full backup, once a month, and one for the incremental backup daily. And this would result in something like this on your storage provider.

```
bucketname/<podname> #latest full
bucketname/<podname>-01 # incremental day 1
bucketname/<podname>-02 # ...
bucketname/<podname>-03 # ...
...
```

If your rclone or sync is more elaborate you can also put it in a shell script and put the shell script in a config map and mount it in the sidecar container. Or you can put multiple storage provider targets in here. It's fully customizable this way.

```yaml
apiVersion: v1
data:
  crontab: |
    0 3 * * * rclone sync "$KSCB_SRC" "$KSCB_DST/`hostname | cut -d'-' -f1`" -v --backup-dir="$KSCB_DST/`hostname | cut -d'-' -f1`-`date +%d`"
    0 1 1 * * rclone sync "$KSCB_SRC" "$KSCB_DST/`hostname | cut -d'-' -f1`" -v
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: crontab
```

Then apply this yml file with kubectl apply -f cron.yml

And finally we add the sidecar to the deployment. In this example deployment  I am using bitwarden (a password manager), the persistent data is on a pv bitwardenrs-data, I mount this data volume read-only in the sidecar container on /backup/data. The crontab and rclone config are mounted also in the sidecar. You can adjust KSCB_SRC and KSCB_DST to your mount point and storage provider in your rclone config. Even you can use multiple destinations or sources in rclone config and cron.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bitwardenrs
  labels:
    app: bitwardenrs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bitwardenrs
  template:
    metadata:
      labels:
        app: bitwardenrs
    spec:
      containers:
      - name: bitwardenrs
        image: bitwardenrs/server:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /data
          name: data
        env:
        - name: SIGNUPS_ALLOWED
          value: "false"
      - name: kscb
        image: dkruyt/kscb:latest
        env:
        - name: KSCB_SRC
          value: "/backup/data"
        - name: KSCB_DST
          value: "aws-s3:backup"
        volumeMounts:
        - name: data
          mountPath: /backup/data
          readOnly: true
        - name: crontab
          mountPath: /crontab
          subPath: crontab
        - name: rclone
          mountPath: /config/rclone/rclone.conf
          subPath: rclone.conf
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: bitwardenrs-data
      - name: crontab
        configMap:
          name: crontab
      - name: rclone
        secret:
          secretName: rclone
```

Also apply this yml file with kubectl apply -f deployment-sidecar.yml

After this your deployment will create a pod with 2 containers in it. The sidecar will only run crond and scheduled execute rclone for backup/sync.

```bash
 $ kubectl get po -lapp=bitwardenrs -o=custom-columns=NAME:.metadata.name,CONTAINERS:.spec.containers[*].name
NAME                           CONTAINERS
bitwardenrs-57d6768556-s5ppn   bitwardenrs,kscb
```



