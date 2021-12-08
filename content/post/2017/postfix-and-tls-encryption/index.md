---
author: Dennis Kruyt
categories:
- tls
- ssl
- postfix
- email
- mail
- startssl
- mta-sts
- Renegotiation
date: "2017-08-04T19:05:26Z"
description: ""
draft: false
cover:
  image: /images/2019/09/photo-1509822929063-6b6cfc9b42f2.jpeg
slug: postfix-and-tls-encryption
tags:
- tls
- ssl
- postfix
- email
- mail
- startssl
- mta-sts
- Renegotiation
title: Postfix and TLS encryption
---


With hackers around every corner, governments wants to read your emails, now a days encryption is a necessity. Now most major sites are only available on https, and more and more IM are using encryption. But what about and old protocol email that is still very popular and we cant go without it any more. How can we increase security for this?

Of course you can use S/MIME or PGP and have end to end encryption, but the problem that in transit between mail servers the from, to, cc, and subject fields are not encrypted. For this we can use Transport Layer Security (TLS) encryption between the smtp servers. Now in June 2018 from [Google's perspective](https://transparencyreport.google.com/safer-email/overview?hl=en) 89% outbound mails and 88% inbound mails are using encryption.

In this post I will show how I setup a smtp server running Postfix with TLS encryption and with the correct cyphers. So that email between smtp servers where possible is using strong email encryption.   
 
## Postfix mail daemon

First you need to know that postfix has separate mail daemons for handling different flow of mail. And each daemon is configured separately. So it is possible to accept weak ciphers but you only use strong ciphers when delivering mail to the out side.

The two that are responsible for handling mail in and out from the world are:

**smtpd** - The SMTP daemon process for handling incoming mail and delivering to the appropriate internal location.

**smtp** - The SMTP daemon process for delivering mail out to the world.

### Default config test

Lets see first how good this default config is for incoming mail to the **smtpd** daemon. Normally I would test it with [SSLLABS](https://ssllabs.com) sadly this only can check _https_ can't check smtp/STARTTLS. A alternative is [immuniweb](https://www.immuniweb.com/ssl/) but we will use this later. There is also a shellscript self hosted tool on https://testssl.sh/ which can check your SSL/TLS settings and vulnerabilities of your mail server. 

To test with testssl run the following after installing.

```bash
~>./testssl.sh -t smtp localhost:25
```

Below is a summery of the issues with the default postfix config on Ubuntu 16.04.

![](/images/2017/08/testssh-default-1.jpg)

Lets see how we can fix these issues.

### Trusted certificate

While it is not mandatory for mailserver to have a trusted certificate, now a day's it is easy and free to get one from LetsEncrypt. So request one and use it for Postfix. Make sure you use the *fullchain*, so that intermediates in the chain are also sent.  

```bash
smtpd_tls_cert_file=/etc/letsencrypt/live/domain.nl/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/domain.nl/privkey.pem
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
```

What about a client certificate for the _smtp_ daemon? For this the readme from postfix is very clear http://www.postfix.org/TLS_README.html

> Do not configure Postfix SMTP client certificates unless you must present client TLS certificates to one or more servers. Client certificates are not usually needed, and can cause problems in configurations that work well without them. The recommended setting is to let the defaults stand:

### Disable SSL,TLSv1
After that we disable all SSL and TLSv1, allow only high ciphers for both **smtp** and **smtpd**. This will mitigate BEAST. And allow **only** high ciphers. And we want to negotiate the strongest available cipher available with the remote server.

```bash
smtpd_tls_protocols = TLSv1.2, TLSv1.1, !TLSv1, !SSLv2, !SSLv3
smtp_tls_protocols = TLSv1.2, TLSv1.1, !TLSv1, !SSLv2, !SSLv3
smtp_tls_ciphers = high
smtpd_tls_ciphers = high
smtpd_tls_mandatory_protocols = TLSv1.2, TLSv1.1, !TLSv1, !SSLv2, !SSLv3
smtp_tls_mandatory_protocols = TLSv1.2, TLSv1.1, !TLSv1, !SSLv2, !SSLv3
smtp_tls_mandatory_ciphers = high
smtpd_tls_mandatory_ciphers = high
```
###Disable deprecated ciphers
And exclude some deprecated not so secure ciphers.

```bash
smtpd_tls_mandatory_exclude_ciphers = MD5, DES, ADH, RC4, PSD, SRP, 3DES, eNULL, aNULL
smtpd_tls_exclude_ciphers = MD5, DES, ADH, RC4, PSD, SRP, 3DES, eNULL, aNULL
smtp_tls_mandatory_exclude_ciphers = MD5, DES, ADH, RC4, PSD, SRP, 3DES, eNULL, aNULL
smtp_tls_exclude_ciphers = MD5, DES, ADH, RC4, PSD, SRP, 3DES, eNULL, aNULL
tls_preempt_cipherlist = yes
```

###Use opportunistic encryption 
With mailserver we want to use _opportunistic_ encryption. We don't force encryption, If we do so this sounds secure but not all other mail servers supports encryption. So worse case scenario you will not receive from some other mail servers or be able to sent to some other mail server that dont support TLS. 

This is also stated in RFC2487

> A publicly-referenced SMTP server **MUST NOT** require use of the STARTTLS extension in order to deliver mail locally. This rule prevents the STARTTLS extension from damaging the interoperability of the Internet's SMTP infrastructure.

To set _oppertunistic_ encryption enable the following settings.

```bash
smtpd_tls_security_level = may
smtp_tls_security_level = may
``` 

Quoted from http://www.postfix.org/TLS_README.html

 > At the "may" TLS security level, TLS encryption is opportunistic. The SMTP transaction is encrypted if the STARTTLS ESMTP feature is supported by the server. Otherwise, messages are sent in the clear. Opportunistic TLS can be configured by setting "smtp_tls_security_level = may". 

 > With this, the Postfix SMTP server announces STARTTLS support to remote SMTP clients, but does not require that clients use TLS encryption.

 > You can ENFORCE the use of TLS, so that the Postfix SMTP server announces STARTTLS and accepts no mail without TLS encryption, by setting "smtpd_tls_security_level = encrypt". According to RFC 2487 this MUST NOT be applied in case of a publicly-referenced Postfix SMTP server. This option is off by default and should only seldom be used.

We could also validate remote certificates for the _smtp_ daemon with `verify` or `secure` like you would normal do with _https_ in a browser, but with a mail server this is not common since the majority of them will fail validation due to poor setup. So you will lose emails if you use this.

With these settings we are still susceptible for a downgrade attack. This because is email has been invented in 1982 and SSL/TLS in 1999 so security needed to be adopted to an existing protocol to maintain backward compatibility. And we still want to be able to send and receive emails from legacy or poorly configured servers.

There are some options coming to solve these issues. Like [mta-sts](https://datatracker.ietf.org/doc/draft-ietf-uta-mta-sts/) or a [startssl policy list](https://starttls-everywhere.org/). But these are still very new and not broadly supported yet (June 2018). Maybe an other time I make a post about these.

### Secure Client-Initiated Renegotiation

On a Ubuntu 16.04 with postfix version 3.1.0-3ubuntu0.3 and openssl version 1.0.2g-1ubuntu4.15 you can't solve this issue.

`Secure Client-Initiated Renegotiation     VULNERABLE (NOT ok), potential DoS threat`

But this is not a major issue.

> The impact of TLS-based attacks on SMTP should not be over-stated. Presently, most SMTP clients don't verify the TLS certificates of SMTP servers. Such clients are already vulnerable to ordinary man-in-the-middle attacks, and TLS renegotiation introduces no new threats for them.

> The Postfix SMTP server with OpenSSL is not affected by the TLS renegotiation attack that redirects and modifies SMTP mail, due to accidental details of the Postfix and OpenSSL implementations.

On Ubuntu 18.04 we have postfix 3.3.0-1ubuntu0.2 and openssl 1.1.1-1ubuntu2.1~18.04.4, with openssl 1.1.1 we can disable the renegotiation via a tls_ssl_option. Then the vulnerability is gone. 

postfix 3.3
```
# NO_RENEGOTIATION postfix 3.3 openssl >1.1.1
tls_ssl_options = 0x40000000
```
And in [postfix 3.4](http://www.postfix.org/postconf.5.html) you can set it also without the hex code.
```
# NO_RENEGOTIATION postfix 3.4 and openssl >1.1.1
tls_ssl_options = NO_RENEGOTIATION
```

## Test new config

After these settings, and you restart postfix and check again with `testssl.sh`, all issue's reported earlier are gone. Wit maybe the eception of the enegotiation issue depending of you postfix/openssl version.  

### immuniweb

An other tool you can test online with is from https://www.immuniweb.com/ssl/ these can also check mailservers, make sure you use `:25` at the end of your hostname. With the above SSL/TLS settings for postfix you get an A or an A+, depending on your postfix and openssl version. If you are using this site, make sure to hit refresh after you changed your postfix config or else you will get cached results.

![SSL_Security](/images/2019/09/SSL_Security.jpg)

immuniweb will give the following warning. This is due to the fact that postfix doesn't support OCSP stapling. So we can not do anything about this, so we will ignore this. That is also one of the issues found in HIPAA COMPLIANT or NIST.

`SERVER DOES NOT SUPPORT OCSP STAPLING`

### Check the logs / headers if it is working
How do I know when mail is delivered over TLS, you can ook in the 
 mail logs. But before this is logged, you need to enable the tls log level.

```bash
smtpd_tls_loglevel = 2
smtp_tls_loglevel = 2
```


After that you will see something like this for an incoming mail daemon _smtpd_, in this case from google/gmail.

```sql
Aug  4 18:58:41 mail postfix/smtpd[18454]: setting up TLS connection from mail-oi0-f43.google.com[209.85.218.43]
Aug  4 18:58:41 mail postfix/smtpd[18454]: mail-oi0-f43.google.com[209.85.218.43]: TLS cipher list "aNULL:-aNULL:HIGH:@STRENGTH:!MD5:!DES:!ADH:!RC4:!PSD:!SRP:!3DES:!eNULL:!aNULL"
Aug  4 18:58:41 mail postfix/smtpd[18454]: mail-oi0-f43.google.com[209.85.218.43]: Issuing session ticket, key expiration: 1501874441
Aug  4 18:58:41 mail postfix/smtpd[18454]: Anonymous TLS connection established from mail-oi0-f43.google.com[209.85.218.43]: TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits)
```


And for an outgoing mail using the _smtp_ daemon.

```sql
Aug  4 19:19:02 mail postfix/smtp[18987]: setting up TLS connection to gmail-smtp-in.l.google.com[173.194.69.27]:25
Aug  4 19:19:02 mail postfix/smtp[18987]: gmail-smtp-in.l.google.com[173.194.69.27]:25: TLS cipher list "aNULL:-aNULL:HIGH:@STRENGTH:!MD5:!DES:!ADH:!RC4:!PSD:!SRP:!3DES:!eNULL:!aNULL"
Aug  4 19:19:02 mail postfix/smtp[18987]: gmail-smtp-in.l.google.com[173.194.69.27]:25: depth=2 verify=0 subject=/C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
Aug  4 19:19:02 mail postfix/smtp[18987]: gmail-smtp-in.l.google.com[173.194.69.27]:25: depth=1 verify=1 subject=/C=US/O=Google Inc/CN=Google Internet Authority G2
Aug  4 19:19:02 mail postfix/smtp[18987]: gmail-smtp-in.l.google.com[173.194.69.27]:25: depth=0 verify=1 subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=mx.google.com
Aug  4 19:19:02 mail postfix/smtp[18987]: gmail-smtp-in.l.google.com[173.194.69.27]:25: subject_CN=mx.google.com, issuer_CN=Google Internet Authority G2, fingerprint=05:DF:99:DD:61:DA:84:5C:77:EF:5B:72:5D:05:B1:52, pkey_fingerprint=EC:A8:0A:1E:05:F2:51:48:B5:DC:1C:A2:78:5E:ED:DE
Aug  4 19:19:02 mail postfix/smtp[18987]: Untrusted TLS connection established to gmail-smtp-in.l.google.com[173.194.69.27]:25: TLSv1.2 with cipher ECDHE-RSA-AES128-GCM-SHA256 (128/128 bits)
```


If you want the information in the email headers you can enable the following option.

```markup
smtpd_tls_received_header = yes
```


Then if you view the headers of the email message you will see wich ciphers was used for this connection.

```sql
Received: from mail-oi0-f47.google.com (mail-oi0-f47.google.com [209.85.218.47])
	(using TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits))
	(No client certificate requested)
```

A oneliner to check how many connections with wich cyphers are made.

```bash
dennis@mailserver:/var/log> grep "TLS connection established" mail.log | sed 's/.*: //g' | sort | uniq -c | sort -rn
   1730 TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits)
   1522 TLSv1.2 with cipher ECDHE-RSA-AES128-GCM-SHA256 (128/128 bits)
    145 TLSv1.2 with cipher ECDHE-RSA-AES256-SHA384 (256/256 bits)
     20 TLSv1.2 with cipher ECDHE-RSA-AES256-SHA (256/256 bits)
     15 TLSv1.1 with cipher ECDHE-RSA-AES256-SHA (256/256 bits)
      7 TLSv1.2 with cipher AES256-GCM-SHA384 (256/256 bits)
      5 TLSv1.2 with cipher AES128-SHA256 (128/128 bits)
      4 TLSv1.2 with cipher DHE-RSA-AES256-GCM-SHA384 (256/256 bits)
      4 TLSv1.2 with cipher AES256-SHA256 (256/256 bits)
      4 TLSv1.2 with cipher AES256-SHA (256/256 bits)
      4 TLSv1.2 with cipher AES128-SHA (128/128 bits)
      4 TLSv1.2 with cipher AES128-GCM-SHA256 (128/128 bits)
      4 TLSv1.1 with cipher DHE-RSA-AES256-SHA (256/256 bits)
      3 TLSv1.1 with cipher AES256-SHA (256/256 bits)
      3 TLSv1.1 with cipher AES128-SHA (128/128 bits)
      2 TLSv1.2 with cipher ECDHE-RSA-AES128-SHA256 (128/128 bits)
      2 TLSv1.2 with cipher ECDHE-RSA-AES128-SHA (128/128 bits)
      2 TLSv1.2 with cipher DHE-RSA-CAMELLIA256-SHA (256/256 bits)
      2 TLSv1.2 with cipher DHE-RSA-CAMELLIA128-SHA (128/128 bits)
      2 TLSv1.2 with cipher DHE-RSA-AES256-SHA256 (256/256 bits)
      2 TLSv1.2 with cipher DHE-RSA-AES256-SHA (256/256 bits)
      2 TLSv1.2 with cipher DHE-RSA-AES128-SHA256 (128/128 bits)
      2 TLSv1.2 with cipher DHE-RSA-AES128-SHA (128/128 bits)
      2 TLSv1.2 with cipher DHE-RSA-AES128-GCM-SHA256 (128/128 bits)
      2 TLSv1.2 with cipher CAMELLIA256-SHA (256/256 bits)
      2 TLSv1.2 with cipher CAMELLIA128-SHA (128/128 bits)
      2 TLSv1.1 with cipher ECDHE-RSA-AES128-SHA (128/128 bits)
      2 TLSv1.1 with cipher DHE-RSA-CAMELLIA256-SHA (256/256 bits)
      2 TLSv1.1 with cipher DHE-RSA-CAMELLIA128-SHA (128/128 bits)
      2 TLSv1.1 with cipher DHE-RSA-AES128-SHA (128/128 bits)
      2 TLSv1.1 with cipher CAMELLIA256-SHA (256/256 bits)
      2 TLSv1.1 with cipher CAMELLIA128-SHA (128/128 bits)
```

But if you want more real connections we need to filter out some internet search/scanning engine. So we filter shodan, immuniweb and internet-census. After that we notice that we only had TLSv1.2 connection!

```bash
dennis@mailserver:/var/log> grep "TLS connection established" mail.log egrep -v "(shodan|immuniweb|internet-census)" | sed 's/.*: //g' | sort | uniq -c | sort -rn
   1700 TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits)
   1520 TLSv1.2 with cipher ECDHE-RSA-AES128-GCM-SHA256 (128/128 bits)
    143 TLSv1.2 with cipher ECDHE-RSA-AES256-SHA384 (256/256 bits)
     10 TLSv1.2 with cipher ECDHE-RSA-AES256-SHA (256/256 bits)
      3 TLSv1.2 with cipher AES256-GCM-SHA384 (256/256 bits)
      1 TLSv1.2 with cipher AES128-SHA256 (128/128 bits)
```


After implementing these settings, your mailserver will exchange emails with other email server using high 'secure' encryption if possible.

## Changelog

*Update June 2018*
Added mta-sts and startssl policy list info and links. 

*Update September 2019*
Added oneliner.

*Update September 2019*
When I wrote this guide I had used Ubuntu 16.04 for testing. With Ubuntu 18.04 and Postfix, TLS 1.3 is also supported.

*Update September 2019*
Added /rewrite Secure Client-Initiated Renegotiation.

