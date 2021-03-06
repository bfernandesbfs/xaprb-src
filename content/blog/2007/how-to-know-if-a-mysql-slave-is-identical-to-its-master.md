---
title: How to know if a MySQL replica is identical to its master
date: "2007-04-07"
url: /blog/2007/04/07/how-to-know-if-a-mysql-slave-is-identical-to-its-master/
categories:
  - Databases
---
A [frequently asked question about MySQL replication](http://onlinesolutionsmysql.blogspot.com/2007/03/questions-and-answers-during-first.html) is "how do I know whether my replica is identical to the master?" Until recently there hasn't been a good way to know, but now you can compare all the data in your master to the data in the replicas and get a reliable yes-or-no answer. And you can do this online, efficiently, across many servers simultaneously. Read on to find out how.

### Checksum your data

The best way I know to find out whether two servers are the same is to compare every table, and the best way I know to do that is to [checksum every table](/blog/2007/01/25/how-to-calculate-table-checksums-in-mysql/). If the checksums match, it's a pretty strong guarantee the tables are identical, and checksums are only a few dozen bytes of network traffic no matter how large the tables are. There are built-in checksums for MyISAM, but you can use cryptographic hash functions for other storage engines. [MySQL Table Checksum](http://code.google.com/p/maatkit) makes this process simple and efficient with fast, parallel checksums across many servers at once.

### Get a consistent read

For a strong guarantee of consistency, you need to checksum your tables at the same point in the replication sequence. One way to do this is to lock the table on the master and wait on the replicas until they reach the master's position, checksum everything, and then unlock on the master.

This can be expensive in terms of blocking updates on that table on the master, so I prefer to take a low-fidelity checksum to begin with, and then come back and re-examine any tables that look suspicious.

### Putting it all together

Here's how I verify all my replicas are in sync with the master:

1.  Do a low-impact, fast checksum across all the replicas, measuring replica lag but not locking or waiting for the master. Most tables are probably not being updated during this process, so even if I don't measure them at the same position in the binlog, they're probably still identical.
2.  From this list of tables I eliminate ones that checked out okay, and special-case tables I don't want to be identical on the replica.
3.  I re-checksum the remaining list and see if they're truly different. This time I force them to be at the same point in the binlog.

MySQL Table Checksum makes this very easy. Here's how:

```
# Take a fast initial checksum and save to a file
$ mysql-table-checksum --slavelag master replica1 replica2 \
   | tee checksums

# See which tables are different
$ mysql-checksum-filter --master master checksums

# Run again on the bad table(s), this time forcing
# replicas to checksum at the same binlog position
$ mysql-table-checksum --databases db1 --tables tbl1,tbl2 \
  --wait 600 master replica1 replica2 | tee bad_checksums
```

### I found differences. What now?

The traditional advice is "you have to re-initialize your replicas with a fresh snapshot of data from the master." That's expensive, and **you might not have to do it**. I wrote [MySQL Table Sync](http://code.google.com/p/maatkit) to do fast, network-efficient, consistent data comparision and syncing *over replication*. It's explicitly designed to bring tables back into sync on replicas.

### Should I check even if there are no replication errors?

When there's no noticeable error, most people assume the replica has the same data as the master. That sounds reasonable, but it turns out to be a bad assumption. In practice, I have replication running rock-solid, and the replicas still drift out of sync. You'll never know unless you measure, of course.


