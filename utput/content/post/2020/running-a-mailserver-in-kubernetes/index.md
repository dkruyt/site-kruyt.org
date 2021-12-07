---
author: Dennis Kruyt
categories:
- kubernetes
- ingress
- proxy
- postfix
- mx
- email
- server
- mail
date: "2020-10-19T13:51:21Z"
description: ""
draft: false
image: /images/2020/10/photo-1527295699943-41f460f6f4c7.jpeg
slug: running-a-mailserver-in-kubernetes
tags:
- kubernetes
- ingress
- proxy
- postfix
- mx
- email
- server
- mail
title: Running a mailserver in Kubernetes
---


Running a webserver in Kubernetes is easy, but a mailserver is more challenging. Most of the challenging things has todo with your ip infrastructure, ingress and loadbalancer within Kubernetes.  Here are things I learned when I was deploying a mailserver based on Postfix on Kubernetes with metallb and nginx ingress. If you have a different setup, things could apply or not.

### Prevent mail loops on your secondary MX

Your postfix MX pod is running maybe on a private ip RFC1918, this is fine. But Postfix needs to know it's public ip. This is to prevent mail looping if yor primary MX is down.  You need to define your public incoming ip in the `main.cf` so if you use loadbalancing you need to define the public ip for the LoadBalancer that would be the same IP as defined in your DNS MX record, not your outgoing ip.

```yaml
proxy_interfaces = 178.242.244.145
```

### Ingress Proxy and external IP

If you want to run your mailserver behind for example a nginx or haproxy ingress proxy, because you don't have enough public ip's or want to use one ip for multiple services. Postfix wont know the Public IP of the incoming client connection. And that could be a problem for things like RBL lookups.

To let Postfix know what the external public ip is from the connecting client. We need to use the `externalTrafficPolicy` for the ingress service that is behind the LoadBalancer. If this is possible also depends on the type of LoadBalancer you use. I am using metallb and that is working.

```yaml
externalTrafficPolicy: Local
```

Now the ingress proxy knows the external ip of the connecting client. But this ingress proxy needs to pass this information down the the Postfix Pod. For this we can use in nginx of haproxy ingress the tcp-services configmap. This is brcuase ingress normaly does only http and not TCP.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  "25": "mail/mail-postfix:25::PROXY"
```

And in postfix we need to tell that connection from the ingress proxy is using the PROXY protocol. `haproxy` protocol definition here also works with nginx, haproxy is just a standard.

```yaml
postscreen_upstream_proxy_protocol=haproxy
```

### helo/ehlo Banners

Your incoming and outgoing mails doesn't need to be to or from same ip for the same pod, maybe ingress is via a LoadBalancer and ingress proxy, and outgoing will be from the node where the pod is running. So the A and PTR DNS records are different. Mail system can be strict for the correct usage with A and PTR records and whats is defined in the banners. So we need to define those in the `main.cf` from Postfix

```yaml
smtpd_banner = {{ LOADBALANCER_INGRESS_NAME }} ESMTP $mail_name
smtp_helo_name = {{ NODE_NAME }}
```

You can hardcode the names, better is to put in in the environment variable from the pod. So now when the pod starts, it get the hostname of the node (needs to be a FQDN). And this varible you can apply in your mail config. And if your DNS records are correctly setup the helo for outgoing connections should contain the hostname of the node running the pod.

```yaml
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
```

### RBL List

For fighting spam maybe you use RBL lists. The problem is that DNS servers will be limited in the amount of requests that they can do. So when you use your provider DNS and other people will also use this DNS server you will hit a max pretty easy. Better is to use your own resolver for this.

**Note: If you are running a commercial server you should be paying or donating to use this lists in general**.

One way is to install an Unbound resolver on your nodes (the nodes where CoreDNS is running) and point the config of CoreDNS in Kubernetes for the RBL lists to this Unbound resolver. Now the request for the RBL lists domains will go trough Unbound on the node.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |-
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
    zen.spamhaus.org:53 {
       errors
       cache 30
       forward . 172.17.0.1:5300
    }
    multi.uribl.com:53 {
       errors
       cache 30
       forward . 172.17.0.1:5300
    }
    dnswl.org:53 {
       errors
       cache 30
       forward . 172.17.0.1:5300
    }
```



