---
author: Dennis Kruyt
categories:
- SPF
- email
- instagram
- fail
- helo
- smtp
- facebook
date: "2019-09-13T03:36:21Z"
description: ""
draft: false
image: /images/2019/10/photo-1473158912295-779ef17fc94b.jpeg
slug: instragram-mail-server
tags:
- SPF
- email
- instagram
- fail
- helo
- smtp
- facebook
title: Instagram/Facebook HELO/SPF fail
---


I am trying to get an email from instagram, but I wasn't receiving it. Upon checking my mail logs I see the following log message.

```
Sep 13 03:50:51 mailserver postfix/smtpd[28105]: NOQUEUE: reject: RCPT from 66-220-155-156.mail-mail.facebook.com[66.220.155.156]: 550 5.7.1 <dennis@kruyt.org>: Recipient address rejected: Message rejected due to: domain owner discourages use of this host. Please see http://www.openspf.net/Why?s=helo;id=mx-out.facebook.com;ip=66.220.155.156;r=dennis@kruyt.org; from=<security@mail.instagram.com> to=<dennis@kruyt.org> proto=ESMTP helo=<mx-out.facebook.com>
```

So the mail is SPF (Sender Policy Framework) failing. Email domains use this protocol to specify which mail hosts are authorised to use this domain in the SMTP HELO and MAIL FROM commands.

{{< figure src="/images/2019/10/spf_flow--1-.jpg" >}}

There is a URL in the log message, for a quick check. But the domain openspf.com doesn't exist any more. So we need to do some manual checking.

```bash
1 dennis@colossus:~> host -t TXT mail.instagram.com
mail.instagram.com descriptive text "v=spf1 include:facebookmail.com -all"
1 dennis@colossus:~> host -t TXT facebookmail.com
facebookmail.com descriptive text "v=spf1 ip4:66.220.144.128/25 ip4:66.220.155.0/24 ip4:66.220.157.0/25 ip4:69.63.178.128/25 ip4:69.63.181.0/24 ip4:69.63.184.0/25 ip4:69.171.232.0/24 ip4:69.171.244.0/23 -all"
```

So ip **66.220.155.156** is in the allowed list of the SPF record. So it shouldn't fail on SPF.

But on a second look it is not failing on the SPF for the from domain, but on the HELO from the sending mailserver. The HELO is mx-out.facebook.com.

```bash
1 dennis@colossus:~> host -t TXT mx-out.facebook.com
mx-out.facebook.com descriptive text "v=spf1 a ~all"
1 dennis@colossus:~> host -t A mx-out.facebook.com
mx-out.facebook.com has address 69.63.179.2
```

So the HELO mx-out.facebook.com and the A record is resolving to 69.63.179.2, and that is not in the SPF record for mx-out.facebook.com. So it is soft failing.

I am using postfix as a mail server, that is using policyd-spf-python framework for SPF checking. So lets see the config file _/etc/postfix-policyd-spf-python/policyd-spf.conf_

There is a option for HELO_reject

```bash
HELO_reject = SPF_Not_Pass
```

So what From the policyd-spf.conf manual

> **SPF_Not_Pass** (default) - Reject if result not Pass, None, or Temperror (alternatively put,        reject if the SPF result is Fail, Softfail, Neutral, PermError). Unlike Mail From        checking, there are no standard e-mail use cases where a HELO check should not Pass if        there is an SPF record for the HELO name (transparent forwarding, for example, is not an        issue). Technically this option is not fully RFC 4408 compliant since the SPF check for        the Mail From identity is mandatory and Neutral and None results must be treated the same.        HELO/EHLO is known first in the SMTP dialogue and there is no practical reason to waste        resources on Mail From checks if the HELO check will already cause the message to be        rejected. These deviations should not cause interoperability problems when used for HELO.

> **Null** - Only reject HELO Fail for Null sender (SPF Classic).  This is the approach used by        the pre-RFC 4408 reference implementation and many of the pre- RFC specifications.  Use of        at least this option (SPF_Not_Pass or Fail) are preferred) is highly recommended.

The manual refers to RFC 4408, but that one is already replace by RFC 7208. The part of HELO in the RFC see [https://tools.ietf.org/html/rfc7208#section-2.3](https://tools.ietf.org/html/rfc7208#section-2.3)

> **[2.3](https://tools.ietf.org/html/rfc7208#section-2.3).  The "HELO" Identity** It is RECOMMENDED that SPF verifiers not only check the "MAIL FROM"    identity but also separately check the "HELO" identity by applying    the check_host() function ([Section 4](https://tools.ietf.org/html/rfc7208#section-4)) to the "HELO" identity as the    <sender>.  Checking "HELO" promotes consistency of results and can    reduce DNS resource usage.  If a conclusive determination about the    message can be made based on a check of "HELO", then the use of DNS    resources to process the typically more complex "MAIL FROM" can be    avoided.  Additionally, since SPF records published for "HELO"    identities refer to a single host, when available, they are a very    reliable source of host authorization status.  Checking "HELO" before    "MAIL FROM" is the RECOMMENDED sequence if both are checked.

So after changing the option to **Null** I received the emails again. I could also add the facebook mailserver to a whitelist. But al that is just a work around, instagram/facebook should add the HELO for outbound mailservers to their SPF records.

An other site where you can do SPF checks that will show this [https://www.kitterman.com/spf/validate.html](https://www.kitterman.com/spf/validate.html?)

```
Mail sent from this IP address: 66.220.155.156
Mail from (Sender): security@mail.instagram.com
Mail checked using this SPF policy: v=spf1 include:facebookmail.com -all
Results - PASS sender SPF authorized


Mail sent from this IP address: 66.220.155.156
Mail Server HELO/EHLO identity: mx-out.facebook.com

HELO/EHLO Results - softfail domain owner discourages use of this host
```

So what does a correct implementation looks like? Microsoft, Outlook has a correct HELO/SPF setup.

```bash
1 dennis@colossus:~> host -t txt eur01-db5-obe.outbound.protection.outlook.com
eur01-db5-obe.outbound.protection.outlook.com descriptive text "v=spf1 include:spf.protection.outlook.com -all"
1 dennis@colossus:~> host -t txt spf.protection.outlook.com
spf.protection.outlook.com descriptive text "v=spf1 ip4:207.46.100.0/24 ip4:207.46.163.0/24 ip4:65.55.169.0/24 ip4:157.56.110.0/23 ip4:157.55.234.0/24 ip4:213.199.154.0/24 ip4:213.199.180.128/26 ip4:52.100.0.0/14 include:spfa.protection.outlook.com -all"
1 dennis@colossus:~> host eur01-db5-obe.outbound.protection.outlook.com
eur01-db5-obe.outbound.protection.outlook.com has address 213.199.154.175
eur01-db5-obe.outbound.protection.outlook.com has IPv6 address 2a01:111:f400:7e02::200
```



