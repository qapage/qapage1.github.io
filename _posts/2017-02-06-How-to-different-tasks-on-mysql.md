---
layout: post
title: How to do different tasks on mysql
categories: [mysql]
tags: [mysql]
---
Here’s a list of things that I’ve had to do on mysql from time to time.  You dont always need these but when you do, its good to have them put together in one place.

#### Enable logging and get all sql queries hitting a mysql DB written to a log file

```
mysql> show variables like “general_log%”; This shows you two values usually. One ‘’ is a flag for setting general logging on and off. The other ‘’ is where to actually log to.

mysql> SET GLOBAL general_log = ‘ON’;

mysql> show variables like “general_log%”;
+——————+————————-+
| Variable_name    | Value                   |
+——————+————————-+
| general_log      | ON                      |
| general_log_file | /var/lib/mysql/logname.log |
+——————+————————-+
2 rows in set (0.00 sec)

tail -f /var/log/mysqld/logname.log
```

When you’ve gotten what you’re looking for, go back and turn off the
logging.

```
mysql> SET GLOBAL general_log = 'OFF’;
```

#### Enabling slow query logging

You notice a web page loading slowly and you suspect it might be because its taking a long time to fetch the results from the DB. Or you’re running performance tests against your system and you want to find out which of your queries are running slowly and hurting your system’s response time? Well - mysql has a slow query logging facility that can help you narrow down the issue.

ssh into your mysql server as a user that has superuser privileges etc and add the below lines to /etc/my.cnf and restart mysql service on that box. 

```
[mysqld]
slow_query_log=1
slow_query_log_file=/tmp/slow.log
long_query_time=2
```

The ‘slow_query_log’ flag turns on or off slow query logging. The ‘slow_query_log_file’ setting specifies the actual file to log to and the ‘long_query_time’ is the setting where you specify what is to be considered a slow query. Here, 2 stands for anything longer than 2 seconds. 

And you’re all set! Run your tests, your UI scripts etc and then check in on the /tmp/slow.log file and you should see the queries you’re interested in.

#### Listing out all the currently executing queries on a mysql server

The `show processlist` command is really useful in that, it shows you whats currently executing on your mysql server, what state they are in etc etc. Sort of like the `ps -ef` for mysql. 

```
mysql> show processlist;
+—-+——+———–+——-+———+——-+———-+——————+
| Id | User | Host      | db    | Command | Time  | State    | Info
|
+—-+——+———–+——-+———+——-+———-+——————+
| 29 | test  | localhost | test   | Sleep   |   281 |          | NULL
|
| 30 | test  | localhost | test   | Sleep   |   281 |          | NULL
|
| 31 | test  | localhost | test   | Sleep   |   281 |          | NULL
|
| 32 | test  | localhost | test   | Sleep   | 15280 |          | NULL
|
| 35 | test  | localhost | dbmap | Sleep   | 13481 |          | NULL
|
| 41 | test  | localhost | dbmap | Sleep   |  3033 |          | NULL
|
| 42 | test  | localhost | test   | Sleep   |  3033 |          | NULL
|
| 43 | root | localhost | NULL  | Query   |     0 | starting | show
processlist |
+—-+——+———–+——-+———+——-+———-+——————+
8 rows in set (0.00 sec)
```

#### Getting mysql server status from the command line

After you’ve logged into your mysql server, issue the `status` command to see various bits of information about your mysql server and your current session, including current user, current database, is SSL being used or not, mysql version, Server characterset, uptime of the server, number of slow queries, no of open tables, average number of queries per second etc. Here’s a sample output,

```
mysql> status
————–
mysql  Ver 14.14 Distrib 5.7.17, for Linux (x86_64) using  EditLine
wrapper

Connection id:          59
Current database:       test
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ’’
Using delimiter:        ;
Server version:         5.7.17 MySQL Community Server (GPL)
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    latin1
Db     characterset:    utf8
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /var/lib/mysql/mysql.sock
Uptime:                 4 hours 39 min 28 sec

Threads: 8  Questions: 7605  Slow queries: 0  Opens: 626  Flush tables:
1  Open tables: 588  Queries per second avg: 0.453
```

#### Finding simple things about the DB like - what are the different
databases available, what are the different tables in current DB etc

```
show databases;

show tables;
```

#### The easier way to get deadlock data to find deadlocks in your DB

Install innotop (yum install innotop).

Once you’ve got that installed, you can connect innotop to your DB from
the command line like this

```
innotop –user root –askpass
```

and you will be prompted for the password on the command line. Then you
will be placed in the Dashboard view of innotop. Hit ? there and you
will see a list of different information screens that you can then
navigate to depending on what you’re trying to find out. L for locks, U
for user statistics, D for InnoDB deadlocks, M for replication status, K
for InnoDB Lock Waits etc. Try it out!

This article has a whole lot more information on [using
innotop](https://www.tecmint.com/install-innotop-to-monitor-mysql-server-performance/)

#### Finding deadlocks with queries running against your DB (The harder way)

The `show engine innodb status;` gives you a whole lot of data about
your database. One of the things that I find useful is the data on
deadlocks.

Here’s a sample output from this command, (Warning - wall of text coming
your way, my apologies!)

```
| InnoDB |      |
=====================================
2017-02-06 21:32:28 0x7f5326922700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 2 seconds
—————–
BACKGROUND THREAD
—————–
srv_master_thread loops: 32 srv_active, 0 srv_shutdown, 15943 srv_idle
srv_master_thread log flush and writes: 15975
———-
SEMAPHORES
———-
OS WAIT ARRAY INFO: reservation count 1101
OS WAIT ARRAY INFO: signal count 1033
RW-shared spins 0, rounds 399, OS waits 218
RW-excl spins 0, rounds 127, OS waits 3
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 399.00 RW-shared, 127.00 RW-excl, 0.00 RW-sx
————
TRANSACTIONS
————
Trx id counter 4075
Purge done for trx’s n:o < 3940 undo n:o < 0 state: running but idle
History list length 17
LIST OF TRANSACTIONS FOR EACH SESSION:
—TRANSACTION 421470258491456, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258489632, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258488720, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258490544, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258487808, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258486896, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258485984, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
—TRANSACTION 421470258485072, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
LOG
—
Log sequence number 5145450
Log flushed up to   5145450
Pages flushed up to 5145450
Last checkpoint at  5145441
0 pending log flushes, 0 pending chkp writes
1490 log i/o’s done, 0.00 log i/o’s/second
———————-
BUFFER POOL AND MEMORY
———————-
Total large memory allocated 137428992
Dictionary memory allocated 2551617
Buffer pool size   8191
Free buffers       7084
Database pages     1096
Old database pages 384
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 231, created 865, written 1002
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read
ahead 0.00/s
LRU len: 1096, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
————–
ROW OPERATIONS
————–
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=5270, Main thread ID=139994953017088, state: sleeping
Number of rows inserted 5124, updated 362, deleted 17, read 21639
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
```
