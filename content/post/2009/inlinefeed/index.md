---
author: Dennis Kruyt
categories:
- atom
- blog
- inlinefeed
- rss
- wordpress
date: "2009-04-28T14:35:00Z"
description: ""
draft: false
cover:
  image: /images/2019/10/photo-1560472355-109703aa3edc.jpeg
slug: inlinefeed
tags:
- atom
- blog
- inlinefeed
- rss
- wordpress
title: Inlinefeed a Wordpress plugin to display RSS feeds
---


**This is not working any more just for historical purpose.**

With the inlinefeed plugin you can display and embed RSS/ATOM feeds in your Wordpress posts and pages.

You can use the following shortcode within Wordpress:

`[inlinefeed rss_feed_url="http://feed.xml"]`

NOTE: From version 2.0 you can only use the shortcode `[inlinefeed]`, the old style `rss:[URL];` doesn’t work anymore! If you upgrade

from a 1.xx version, then you must change from `rss:[URL]` to `[inlinefeed rss_feed_url=”http://feed.xml”]` style..

Download at http://github.com/dkruyt/inlinefeed

Changelog:

2.01 truncate title accpets now a nummeric value. New option, displayfeedname.
2.00 Now uses the wp 2.5 shortcode.
1.62 Error handling and headers are giving descriptions and are clickable.
1.6 Added option newwindow. If you want to open in a new window.
1.5 Fixed a bug that the feed got bumped up to the top of the page.
1.4 XHTML compliant.
1.3 Truncate title.
1.2 minor fixes.
1.1 Initial release

If u want to use Inlinefeed to display rss feeds that are offered in gzip format, please take a look at the modified class-snoopy.php.



