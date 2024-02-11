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
cover:
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


Running a web server in Kubernetes is easy, but a mail server is more challenging. Most of the challenges have to do with your IP infrastructure, ingress, and load balancer within Kubernetes. Here are things I learned when deploying a mail server based on Postfix on Kubernetes with MetalLB and NGINX ingress. If you have a different setup, things might or might not apply.

### Prevent mail loops on your secondary MX

Your Postfix MX pod is maybe running on a private IP RFC1918, which is fine. But Postfix needs to know its public IP to prevent mail looping if your primary MX is down. You need to define your public incoming IP in the `main.cf`. So, if you use load balancing, you need to define the public IP for the LoadBalancer that would be the same IP as defined in your DNS MX record, not your outgoing IP.

```yaml
proxy_interfaces = 178.242.244.145
```

### Ingress Proxy and external IP

If you want to run your mail server behind, for example, an NGINX or HAProxy ingress proxy because you don't have enough public IPs or want to use one IP for multiple services, Postfix won't know the public IP of the incoming client connection. And that could be a problem for things like RBL lookups.

To let Postfix know the external public IP from the connecting client, we need to use the `externalTrafficPolicy` for the ingress service that is behind the LoadBalancer. Whether this is possible also depends on the type of LoadBalancer you use. I am using MetalLB, and that is working.

```yaml
externalTrafficPolicy: Local
```

Now, the ingress proxy knows the external IP of the connecting client. But this ingress proxy needs to pass this information down to the Postfix Pod. For this, we can use in NGINX or HAProxy ingress the tcp-services config map. This is because ingress normally does only HTTP and not TCP.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  "25": "mail/mail-postfix:25::PROXY"
```

And in Postfix, we need to tell that connection from the ingress proxy is using the PROXY protocol. The `haproxy` protocol definition here also works with NGINX; HAProxy is just a standard.

```yaml
postscreen_upstream_proxy_protocol=haproxy
```

### helo/ehlo Banners

Your incoming and outgoing mails don't need to be to or from the same IP for the same pod; maybe ingress is via a LoadBalancer and ingress proxy, and outgoing will be from the node where the pod is running. So the A and PTR DNS records are different. Mail systems can be strict about the correct usage with A and PTR records and what is defined in the banners. So we need to define those in the `main.cf` from Postfix.

```yaml
smtpd_banner = {{ LOADBALANCER_INGRESS_NAME }} ESMTP $mail_name
smtp_helo_name = {{ NODE_NAME }}
```

You can hardcode the names; better is to put them in the environment variable from the pod. So now, when the pod starts, it gets the hostname of the node (needs to be a FQDN). And this variable you can apply in your mail config. And if your DNS records are correctly set up, the HELO for outgoing connections should contain the hostname of the node running the pod.

```yaml
        env:
          - name: NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
```

### RBL List

For fighting spam, maybe you use RBL lists. The problem is that DNS servers will be limited in the amount of requests they can do. So when you use your provider's DNS and other people will also use this DNS server, you will hit a max pretty easily. It's better to use your own resolver for this.

**Note: If you are running a commercial server you should be paying or donating to use this lists in general**.

One way is to install an Unbound resolver on your nodes (the nodes where CoreDNS is running) and point the config of CoreDNS in Kubernetes for the RBL lists to this Unbound resolver. Now the request for the RBL lists domains will go through Unbound on the node.

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

### Monitoring and Logging

An often-overlooked aspect of deploying a mail server in Kubernetes is the need for comprehensive monitoring and logging. Implementing a robust monitoring solution helps in identifying issues proactively and ensuring the mail server operates at its optimal capacity. Tools like Prometheus for monitoring and Grafana for visualization can be integrated with Kubernetes to keep a close eye on the server's performance, resource usage, and operational metrics.

For logging, consider setting up Fluentd or a similar log management tool to aggregate logs from Postfix and other components of your mail server setup. This is crucial for troubleshooting and maintaining the security of your mail server infrastructure.

### Conclusion
Deploying a mail server on Kubernetes is undeniably more diffecult than setting up a basic web server. I hope I have given some tips and pointers By tackling challenges such as mail loops, ingress configuration, and RBL lists.



