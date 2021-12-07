---
author: Dennis Kruyt
categories:
- snort
- ids
- barnyard2
- mysql
date: "2017-07-24T19:33:52Z"
description: ""
draft: false
image: /images/2017/08/1337925082_da43cdc3e6_b.jpg
slug: barnyard2_data_to_long
tags:
- snort
- ids
- barnyard2
- mysql
title: 'Barnyard2: Data too long'
---


After I logged into Snorby to check some events on our IDS. I notice that one of my sensors didn't return any events any more into the database.

I log in the sensors and I notice that Barnyard2 wasn't running any more. Barnyard2 parses the Snort logs and inserts the min the database. So I started the service again and after a few minutes it stoped running again. 

Upon inspection of the logs I fount this warning. So the data doesn't fit in the column.


```sql
Jul 24 14:50:35 barnyard2[23145]: FATAL ERROR: database mysql_error: Data too long for column 'data_payload' at row 1#012#011SQL=[INSERT INTO data (sid,cid,data_payload) VALUES (5,5,'0500000310000000C5FF0000000000000000000000000C00E0FEFEFEFFFFFFFEFEE0FEFFFFFFFEFEFEFFE0FFFFFEFEFEFFFFFFE0FEFEFEFFFFFFFEFEE0FEFFFFFFFEFEFEFFE0FFFFFEFEFEFFFFFFE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFEFEFEFCFEFEE0FEFEFEFCFEFEFEFEE0FEFCFEFEFEFEFEFCE0FEFEFCFEFEFCFEFEE0FCFEFEFCFEFEFCFEE0FEFCFEFEFCFEFEFCE0FEFEFCFEFEFCFEFEE0FCFEFEFCFEFEFCFEE0FEFCFEFEFCFDFEFE40F9FCFC29A76722D5CA22DFFEE9F7F6FBF7F4F5F9F340EFF7F221FBFD41F9FCFD5FFFFFCDF1F2F446F2F3F5E3D2D4D6566482102DE0596F90C6DFEBF0EFE1F6E6ECECE7E8E5F3E0F1F6F4EFF5FDFAFC01FCA1EFF1E6F6F9EBE1F7F2CEE0E35C7A9AE008233F54638CD7DA00EC43FDFCFD65F8EAEBE05F5AFEFBFB61F1FBFDFCE3CBD2E4848DB36A74E09EA1AACEE9EAF2FB20FAFCE7FBEFF
Jul 24 14:50:35 barnyard2[23145]: Barnyard2 exiting
```

Lets see what type the column is.

```sql
mysql> use snorby;
Database changed
mysql> explain data;
+--------------+------------------+------+-----+---------+-------+
| Field        | Type             | Null | Key | Default | Extra |
+--------------+------------------+------+-----+---------+-------+
| sid          | int(10) unsigned | NO   | PRI | NULL    |       |
| cid          | int(10) unsigned | NO   | PRI | NULL    |       |
| data_payload | text             | YES  |     | NULL    |       |
+--------------+------------------+------+-----+---------+-------+
3 rows in set (0.01 sec)
```

The type of data_payload is text, text only hold 64KB, it seems that barnyard2 is trying to store more then 64KB. So I change the field to `mediumtext`, this can holds unto 16MB. Only disadvantage is that this filed type has a 3 bytes overhead instead of 2 bytes for type `text`  

```sql
mysql> alter table data modify data_payload mediumtext;
Query OK, 2924 rows affected (0.96 sec)
Records: 2924  Duplicates: 0  Warnings: 0
mysql> explain data;
+--------------+------------------+------+-----+---------+-------+
| Field        | Type             | Null | Key | Default | Extra |
+--------------+------------------+------+-----+---------+-------+
| sid          | int(10) unsigned | NO   | PRI | NULL    |       |
| cid          | int(10) unsigned | NO   | PRI | NULL    |       |
| data_payload | mediumtext       | YES  |     | NULL    |       |
+--------------+------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

After this change I started barnyard2 again and it didn't crash as before and is logging the data.

Created a pull request at GitHub for this issue: https://github.com/firnsy/barnyard2/pull/222

