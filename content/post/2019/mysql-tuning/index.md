+++
author = "Dennis Kruyt"
categories = ["mysql", "mariadb", "performance", "tuning"]
date = 2019-09-24T03:36:00Z
description = ""
draft = false
image = "/content/images/2019/10/photo-1532393950032-b666e39c29b3.jpeg"
slug = "mysql-tuning"
summary = "At my work in the past I needed to trouble shoot and tune MySQL installations. Here are some best practises I have used."
tags = ["mysql", "mariadb", "performance", "tuning"]
title = "MySQL tuning"

+++


### innodb_buffer_pool_size

The default innodb_buffer_pool_size after a MySQL installation is too small, maybe  10M, this value is standard and too small for a dedicated large production environment.

{{< figure src="https://images.unsplash.com/photo-1504309092620-4d0ec726efa4?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=2000&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ" caption="Photo by <a href="https://unsplash.com/@etiennegirardet?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Etienne Girardet</a> / <a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Unsplash</a>" >}}

To determine the maximum minimum value, the InnoDB data and indexes must be looked at. The following query gives the RIBPS, Recommended InnoDB Buffer Pool Size with an additional 30% growth. This value may be set to 80% of the memory. But this is only allowed if InnoDB is the only storage engine being used. I the same database is also using for example MyISAM, then this value must not be too high of the available memory and there must also be memory left for the MyISAM engine.

```sql
mysql>     SELECT CEILING(Total_InnoDB_Bytes*1.3/POWER(1024,3)) RIBPS FROM
    ->     (SELECT SUM(data_length+index_length) Total_InnoDB_Bytes
    ->     FROM information_schema.tables WHERE engine='InnoDB') A;
+-------+
| RIBPS |
+-------+
|     5 |
+-------+
1 row in set (4.31 sec)

mysql>

```

With this output we can make the following adjustment in /etc/my.cnf

```
[mysqld]
innodb_buffer_pool_size=5G
```

MySQL needs to be restarted after this

After a few weeks run the following query:

```sql
SELECT (PagesData*PageSize)/POWER(1024,3) DataGB FROM (SELECT variable_value PagesData FROM information_schema.global_status HERE variable_name='Innodb_buffer_pool_pages_data') A,(SELECT variable_value PageSize FROM information_schema.global_status WHERE variable_name='Innodb_page_size') B;
```

This gives the number of pages of InnoDB data that is located in the InnoDB Buffer Pool, to which the innodb_buffer_pool_size can possibly be adjusted after this.

The following query can be used to investigate the performance of the InnoDB buffer.

```sql
SHOW INNODB STATUS;
```

To view the Buffer pool hit ratio, it is best to get as close to 100% as possible. For larger databases this will be approximately 95 ~ 99%. Note that if the value is too small, it will almost always be 100%.

```
BUFFER POOL AND MEMORY
----------------------
Total memory allocated 47662558; in additional pool allocated 1048576
Dictionary memory allocated 22312480
Buffer pool size   640
Free buffers       0
Database pages     610
Modified db pages  0
Pending reads 0
Pending writes: LRU 0, flush list 0, single page 0
Pages read 528736601, created 1244044, written 19482608
0.00 reads/s, 0.00 creates/s, 8.00 writes/s
Buffer pool hit rate 1000 / 1000
```

### MyISAM Key Buffer

The MyISAM storage engine uses a key buffer for caching index data from disk. A correct value can improve performance.

{{< figure src="https://images.unsplash.com/photo-1557133285-a2b6b21f6e13?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=2000&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ" caption="Photo by <a href="https://unsplash.com/@mangofantasy?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Tim Johnson</a> / <a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Unsplash</a>" >}}

The key counters can be read with the following query.

```sql
mysql> show status like 'key%';
+------------------------+-------------+
| Variable_name          | Value       |
+------------------------+-------------+
| Key_blocks_not_flushed | 0           |
| Key_blocks_unused      | 346749      |
| Key_blocks_used        | 346980      |
| Key_read_requests      | 43530411625 |
| Key_reads              | 3947698     |
| Key_write_requests     | 16170948756 |
| Key_writes             | 15316984    |
+------------------------+-------------+
7 rows in set (0.00 sec)

```

Here the following values are important for the key_buffer.

• **Key_read_requests** The number of requests to read a key block from the cache.• **Key_reads** The number of physical reads or a key block from disk.

With these values you can determine the miss ratio. However, these values are the counters from the moment the database runs. It is better to collect and store metrics over time and put them in a graph.

The current key_buffer is 384M, this seems to be sufficient for now, possibly it can go to 512M. Raising it too high has no adverse effect because MySQL will allocate the memory only with actual use, with InnoDB buffer pool size it will allocate everything immediately.

**MySQLTuner**

To generate an overview of recommendations for a running database, MySQLTuner-perl can be used. However, it is advisable to first test these recommendations on an acceptance database and todo some performance test on them before taking into production.

{{< figure src="https://images.unsplash.com/photo-1421878512040-134f5e04e971?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=2000&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ" caption="Photo by <a href="https://unsplash.com/@marcin?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Marcin Nowak</a> / <a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Unsplash</a>" >}}

MysqlTunner-perl can be downloaded from the following website[http://mysqltuner.com/](https://mysqltuner.com/)

### Slow query logging

To gain insight into which queries last longer than X seconds, you can activate a slow query. In the case of a production database, this should only be used for troubleshooting, as this affects the performance of the database. So by default on production slow query logging must be switched off!

{{< figure src="https://images.unsplash.com/photo-1558896548-cbdf8e7f12e8?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=2000&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ" caption="Photo by <a href="https://unsplash.com/@yuli_superson?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Yuliya Kosolapova</a> / <a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Unsplash</a>" >}}

You can turn this on / off without restarting the database.

First check where the logging will be placed.

```sql
SELECT @@global.slow_query_log_file;
```

You can adjust this to another location.

```sql
SET @@global.slow_query_log_file = '/var/log/mysql-slow.log';
```

See which queries will be logged (> X seconds).

```sql
SELECT @@global.long_query_time;
```

You can adjust this to a different number (at least 1).

```sql
SET @@global.long_query_time = 5;
```

Then turn on the slow query logging (and later off).

```sql
SET GLOBAL slow_query_log = 'ON';

SET GLOBAL slow_query_log = 'OFF';
```

Optionally, you can do a flush of the logs.

```sql
FLUSH LOGS;
```

## Performance monitoring

Monitoring is key, always collect, store and analyse your metrics. Without this you don't know what impact your changes will have on your database server, linux system and eventually on your application.

{{< figure src="https://images.unsplash.com/photo-1509041172795-d4869fe4a9eb?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=2000&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ" caption="Photo by <a href="https://unsplash.com/@amutiomi?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Miguel A. Amutio</a> / <a href="https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit">Unsplash</a>" >}}

There are many opensource tooling that can do this. Some tools I have used in the past that can mesure MySQL metrics are Observium and the TIG stack, TeleGraf, InfluxDB and Grafana.

{{< gallery caption="Grafana MySQL dashboard" >}}
{{< galleryImg  src="/content/images/2019/09/image--2-.png" width="2846" height="1594" >}}{{< galleryImg  src="/content/images/2019/09/image--1-.png" width="2850" height="1294" >}}{{< /gallery >}}

{{< gallery caption="Observium graphs" >}}
{{< galleryImg  src="/content/images/2019/09/Picture-1.png" width="472" height="308" >}}{{< galleryImg  src="/content/images/2019/09/Picture-4.png" width="472" height="289" >}}{{< galleryImg  src="/content/images/2019/09/Picture-3.png" width="472" height="308" >}}{{< galleryImg  src="/content/images/2019/09/Picture-2.png" width="472" height="308" >}}{{< /gallery >}}



