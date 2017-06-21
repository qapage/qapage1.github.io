---
layout: post
title: My top linux commands
categories: [linux]
tags: [linux]
bigimg: /img/smyrna.jpg
---
I’ve mostly worked on linux/unix machines for the most part of my career. These are some of the linux commands that have been very useful to me in different situations on various projects, testing related or otherwise.

#### top

Find load on a box, get a list of different processes running (sortable by a number of fields), find number of processes running/sleeping/stopped/zombie etc.

image

#### netstat

When I need to find out when anything is listening at a certain port, I use

`netstat -aln | grep <port>`

For example if I’m checking on port 80

`netstat -aln | grep 8080`

netstat can also give you a lot of packet level statistics. For example,
the below command gives you a summary of packet level statistics about
your network interfaces. You can see if you’re having packet loss, how
many packets came in, how many got sent out etc.

```
netstat -s | grep packet
   4324774 total packets received
   0 incoming packets discarded
   4324756 incoming packets delivered
   3369 packets received
   44 packets to unknown port received.
   0 packet receive errors
   1136526 packets sent
   4285 packets directly queued to recvmsg prequeue.
   413474 packets directly received from prequeue
   1993168 packets header predicted
   381 packets header predicted and directly queued to user
   90 DSACKs sent for old packets
```

#### lsof

When I need to find out what process is listening at a certain port, I use 

`lsof -i :port`

For example, if I’m checking on port 8080

```
lsof -i :80
COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   23951    root   28u  IPv4 220407      0t0  TCP *:http (LISTEN)
nginx   24040 duo_www   28u  IPv4 220407      0t0  TCP *:http (LISTEN)
```

#### find

When I need to find a file or directory that matches a certain criteria or search string, i use the find command. 

`find <path you want to search in> <how do you want to search> <search criteria>`

For example, if I wanted to find all the files that have the .log extension in my current directory, I would do, 

```
find . -name ‘*.log’

./test.log
./anothertest.log
./onemoretest.log
```

#### df

When I need to know file system utilization, I use the df command. For example, I suspect that one of my file systems is nearing 100% and I need to go clear it. How would I find out what the disk util is?  

`df -k <whichever directory/file system you want to know the utilization of>`

Now, if I wanted to know how /home was doing, I’d do

```
df -k /home
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/VolGroup-lv_root
                     17938864 5064080  11956872  30% /

```
That tells me the /home (mounted on /) is 30% used. 

#### ifconfig

When I need to find out what interfaces are defined on a box or what IP addresses are assigned to a machine I am logged in on, I use ifconfig.

```
ifconfig

eth0      Link encap:Ethernet  HWaddr 08:00:27:9F:0A:01
         inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
         inet6 addr: fe80::a00:27ff:fe9f:a01/64 Scope:Link
         UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
         RX packets:1225086 errors:0 dropped:0 overruns:0 frame:0
         TX packets:269646 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:1000
         RX bytes:1394615163 (1.2 GiB)  TX bytes:26194169 (24.9 MiB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:01:63:D1
         inet addr:10.50.0.2  Bcast:10.50.0.255  Mask:255.255.255.0
         inet6 addr: fe80::a00:27ff:fe01:63d1/64 Scope:Link
         UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
         RX packets:49869 errors:0 dropped:0 overruns:0 frame:0
         TX packets:155433 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:1000
         RX bytes:7887126 (7.5 MiB)  TX bytes:42488260 (40.5 MiB)

lo        Link encap:Local Loopback
         inet addr:127.0.0.1  Mask:255.0.0.0
         inet6 addr: ::1/128 Scope:Host
         UP LOOPBACK RUNNING  MTU:65536  Metric:1
         RX packets:3050889 errors:0 dropped:0 overruns:0 frame:0
         TX packets:3050889 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:0
         RX bytes:1496552588 (1.3 GiB)  TX bytes:1496552588 (1.3 GiB)
```

Based on this output, I know that there are three interfaces on my machine - eth0, eth1 and lo. I also know what IP addresses are assigned to each of them.

#### tee

I use the tee command when I need the output of stdout written into a file in addition to being displayed on the screen. For example, if you’re monitoring the output of a tail on a log file and you also want this content  written into a separate log file for later reference, then tee is the tool for you.

`tail -f logfile.log | tee test.log`

In this case, you’d see the output of running tail -f on the logfile.log and you’ll also have those same contents written into the file name you provided as input to tee (test.log in this case).
