---
author: Dennis Kruyt
categories:
- etag
- server
- header
- nginx
- web
date: "2018-06-29T18:28:29Z"
description: ""
draft: false
image: /images/2018/06/837-imported-1443570304-837-imported-1443554777-default-headers-pages.png
slug: etag
tags:
- etag
- server
- header
- nginx
- web
title: ETAG headers in a load balanced farm
---


When I was looking at some Nginx caching settings to improve them on a load balanced static content farm. I noticed that the etag headers difference between servers for the same file when they should be the same.

So first what is a etag header? 

*From WikipediA* 
> The ETag or entity tag is part of HTTP, the protocol for the World Wide Web. It is one of several mechanisms that HTTP provides for web cache validation, which allows a client to make conditional requests. This allows caches to be more efficient, and saves bandwidth, as a web server does not need to send a full response if the content has not changed.

When I ask the headers of the same file from different servers, note the x-server. I see the etag header is not the same.

```bash
dennis@colossus:~> curl 'https://server/5.jpg' -I
HTTP/2 200
server: nginx
content-length: 23715
last-modified: Mon, 14 May 2018 08:50:26 GMT
etag: "5af94dd2-5ca3"
x-server: a-stat01
```

```bash
dennis@colossus:~> curl 'https://server/5.jpg' -I
HTTP/2 200
server: nginx
content-length: 23715
last-modified: Mon, 14 May 2018 12:16:14 GMT
etag: "5af97e0e-5ca3"
x-server: a-stat02
```

A quick md5sum learns that both files are the same on both server.
```bash
root@a-stat01:# md5sum 5.jpg
d968155f06057025f4cc8723ac2c764f  5.jpg
```
```bash
root@a-stat02:# md5sum 5.jpg
d968155f06057025f4cc8723ac2c764f  5.jpg
```

So why the etag is different on both servers then?

I am here using a Nginx server. And every webserver generates a different kind of etag header. So we need to look in the Nginx source code. There we can see how the etag value is generated. 
It uses the last modification time and the content length.

```javascript
etag->value.len = ngx_sprintf(etag->value.data, "\"%xT-%xO\"",
                            r->headers_out.last_modified_time,
                            r->headers_out.content_length_n)
```

So, when we look at the headers earlier we see the modification times differs. This is because the files are uploaded to each server separately.

So uploading the files to each server wont work unless you preserve modification times while uploading but this not the best approach. A better approach is to upload to one server and the use rsync with -t or something maybe like a glusterfs setup, so that all servers have the same content and modification times and thus the same etag value.

