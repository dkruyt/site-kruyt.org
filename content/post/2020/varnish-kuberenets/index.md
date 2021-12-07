---
author: Dennis Kruyt
categories:
- varnish
- cache
- kubernetes
date: "2020-09-15T18:50:45Z"
description: ""
draft: false
cover:
  image: /images/2020/09/Webp.net-resizeimage.jpg
slug: varnish-kuberenets
tags:
- varnish
- cache
- kubernetes
title: Varnish in Kubernetes
---


{{< figure src="/images/2020/09/image-1.png" >}}

This is a simple minimal but highly configurable Varnish caching service for Kubernetes. This should be placed between your ingress and your application service.

{{< figure src="/images/2020/09/image-4.png" caption="simple setup" >}}

It can be used in combination with multiple ingresses and application services at the same time.

{{< figure src="/images/2020/09/image-6.png" caption="multiple ingress and services" >}}

### Setup

Apply the following yaml file, replicas and environment variables can be adjusted to your need. This will deploy the Varnish service and Varnish proxy pods. The container is now based up on Alpine Linux and Varnish 6.4

```yaml
apiVersion: v1
kind: Service
metadata:
  name: varnish-svc
  namespace: default
spec:
  ports:
  - name: "http"
    port: 80
  selector:
    app: varnish-proxy
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: varnish-proxy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: varnish-proxy
  template:
    metadata:
      name: varnish-proxy
      labels:
        app: varnish-proxy
    spec:
      volumes:
        - name: varnish-config
          configMap:
            name: varnish-vcl
            items:
              - key: default.vcl
                path: default.vcl
        - name: varnish-secret
          secret:
            secretName: varnish-secret
      containers:
      - name: varnish
        image: dkruyt/varnish:alpine
        imagePullPolicy: Always
        env:
        - name: CACHE_SIZE
          value: 128m
        - name: VCL_CONFIG
          value: /etc/varnish/configmap/default.vcl
        - name: SECRET_FILE
          value: /etc/varnish/k8s-secret/secret
        volumeMounts:
          - name: varnish-config
            mountPath: /etc/varnish/configmap
          - name: varnish-secret
            mountPath: /etc/varnish/k8s-secret
        ports:
        - containerPort: 80

```

Create a secret for Varnish cli admin operations

```bash
kubectl create secret generic varnish-secret --from-literal=secret=$(head -c32 /dev/urandom  | base64)

```

### Varnish vcl config

You will need to provide your custom vcl file, you can put this in a configmap the following way.

```bash
kubectl create configmap varnish-vcl --from-file=default.vcl

```

A very simple vcl example config file, here it wil take the request hostname and transfer it to a backend, in the backend we define the Kubernetes service name. No need for multiple backend, because Kubernetes does the load balancing if there are multiple pods defined.

```
vcl 4.0;

import directors;
import std;


backend site1 {
        .host = "service-name1";
        .port = "80";

}

backend site2 {
        .host = "service-name2";
        .port = "80";
}

sub vcl_recv {

   if (req.http.host == "www.site1.com") {
        set req.backend_hint = site1;
    }
    if (req.http.host == "www.site2.org") {
        set req.backend_hint = site2;
    }
}

```

### Setup Ingress point service to Varnish

In your ingress, point to the varnish service. Multiple ingresses can be all pointing the the same varnish service, you will need to apply the proper routing in the varnish config to point everything to the correct application services.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: site1
spec:
  rules:
  - host: www.site1.com
    http:
      paths:
      - path:
        backend:
          serviceName: varnish-svc
          servicePort: 80
  - host: www.site2.org
    http:
      paths:
      - path:
        backend:
          serviceName: varnish-svc
          servicePort: 80

```

### Optional Metrics: telegraf sidecar

I am using Telegraf, InfluxDB and Grafana everywhere for metrics.

If you want metrics from Varnish, I created a sidecar container that will pull the metrics from Varnish and send it to InfluxDB. Don't forget to adjust the hostAlias in the yaml file to point it to your InfluxDB instance.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: varnish-svc
  namespace: default
spec:
  ports:
  - name: "http"
    port: 80
  selector:
    app: varnish-proxy
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: varnish-proxy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: varnish-proxy
  template:
    metadata:
      name: varnish-proxy
      labels:
        app: varnish-proxy
    spec:
      hostAliases:
       - ip: "192.168.66.168"
         hostnames:
         - "influxdb"
      volumes:
        - name: varnish-config
          configMap:
            name: varnish-vcl
            items:
              - key: default.vcl
                path: default.vcl
        - name: varnish-secret
          secret:
            secretName: varnish-secret
        - name: shared-data
          emptyDir: {}
      containers:
      - name: varnish
        image: dkruyt/varnish:alpine
        imagePullPolicy: Always
        env:
        - name: CACHE_SIZE
          value: 128m
        - name: VCL_CONFIG
          value: /etc/varnish/configmap/default.vcl
        - name: SECRET_FILE
          value: /etc/varnish/k8s-secret/secret
        - name: VARNISHD_PARAMS
          value: -p default_ttl=3600 -p default_grace=3600 -T localhost:6082
        volumeMounts:
          - name: varnish-config
            mountPath: /etc/varnish/configmap
          - name: varnish-secret
            mountPath: /etc/varnish/k8s-secret
          - name: shared-data
            mountPath: /var/lib/varnish
        ports:
        - containerPort: 80
      - name: telegraf
        image: dkruyt/telegraf-varnish:alpine
        imagePullPolicy: Always
        volumeMounts:
          - name: varnish-secret
            mountPath: /etc/varnish/k8s-secret
          - name: shared-data
            mountPath: /var/lib/varnish
            readOnly: true
```



