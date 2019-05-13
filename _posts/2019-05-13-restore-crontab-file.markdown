---
layout: post
title:  "ubuntu restore crontab task file"
date:   2019-05-13 10:27:02 +0800
comments: true
tags:
- ubuntu
- extundelete
- restore
- crontab
---

```
#### Intro
This Saturday, I lost my crontab task file because of input wrong command by mistake, use crontab -r to instead crontab -e.
that's a big trouble， that means this mistake may cause all tasks stop.

#### Use log to restore tasks
Firstly, I can get the task execution result and log from log files, e.g., locate at /var/log/syslog(this may be different for different configuration), and use shell script to filter and clarify the task info:

`cat /var/log/syslog* | grep "(develop) CMD" | awk '{$1=null;$2=null;$4=null;$5=null;$6=null;$7=null;print $0}' | sort -u >~/duke_backup/2019-5-9/crontab.txt`, and I can get the result like:

```
01:01:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:02:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:03:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:04:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:05:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:06:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:07:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:08:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:09:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:10:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:11:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:12:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:13:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:14:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:15:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:16:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:17:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:18:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:19:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:20:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:21:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:22:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:23:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:24:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:25:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:26:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:27:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:28:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:29:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:30:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
  01:31:01     (cd $SRC && $PYDIR/python appannie_app_details_bulk.py Cleanup --auto >> $LOGDIR/cleanup__appannie_app_details_bulk.log 2>&1)
```

But I find that it's very difficult to calculate the Periodic execution time， and I tried another way to get the crontab file, that is use file system itself.

#### use extundelete to restore 
I find the file system type is:ext4

```
develop@etl2:~/duke_backup/2019-5-9$ df -T -h
Filesystem          Type      Size  Used Avail Use% Mounted on
udev                devtmpfs   13G   12K   13G   1% /dev
tmpfs               tmpfs     2.6G  408K  2.6G   1% /run
/dev/sda1           ext4      9.9G  5.4G  4.0G  58% /
none                tmpfs     4.0K     0  4.0K   0% /sys/fs/cgroup
none                tmpfs     5.0M     0  5.0M   0% /run/lock
none                tmpfs      13G  548K   13G   1% /run/shm
none                tmpfs     100M     0  100M   0% /run/user
/dev/mapper/vg-data ext4      296G   73G  208G  26% /data
```

And extundelete is the tool  for file restore.
As below, if you want to restore one file:

`extundelete --restore-file /var/spool/cron/crontabs/develop /dev/sda1`

And then copy the file from directory: RECOVERED_FILES.

You can also find the related command for resotring directory, or all files(--restore-directory， --restore-all).

For ubuntu, you can install this tools by apt-get tool easily.

```
