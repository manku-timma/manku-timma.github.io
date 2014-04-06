---
layout: post
title:  Understanding Write Ahead Logging
date:   2014-04-06 12:00:00
tags: wal, write ahead logging
---

I am trying to understand WAL (Write Ahead Logging) from the perspective of
sqlite3, HBase etc. Below is a summary of my understanding. Refer to these
links for more information: [sqlite3][sqlite3], [postgresql][postgresql],
[sqlserver][sqlserver], [hbase][hbase], [bigtable][bigtable].

###General stuff
- What is WAL
 - Write new content of a write at end of a WAL file instead overwriting
   content in the actual database file
- Advantages of WAL
 - Writes and reads are faster
 - Writes and reads can be concurrent (for a given data file)
 - More sequential disk I/O
 - Fewer `fsync()` calls
 - Allows crash recovery

###Sqlite3
- On Write
 - Append the new data at the end of the current WAL file
- On Read
 - Check wal-index for presence of the data page in WAL
 - If data is in WAL before this reader's end mark, read from WAL
 - Else read from database file
- On Checkpoint
 - Copy data from WAL to actual data file per transaction synchronously
 - Stop at the largest reader end mark
- Other considerations
 - Any number of simultaneous readers
 - One writer at a time
 - WAL size should be kept reasonably small
 - For durability WAL write should be synchronous
 - WAL writes are slower than normal writes
  - WAL must be synced to disk before writing to database file
  - Write to database file must be synced to disk before resetting WAL file
 - Checkpoint can be run in a separate thread or process to prevent certain
   transactions COMMITing very slowly (when they have to CHECKPOINT also)
 - There is a tradeoff between read and write performance. But this is probably
   tilted towards getting faster writes using larger WALs and fewer
   checkpoints. 
 - WAL databases need to be on read-write enabled storage
 - wal-index needs to be in shared memory

###Postgresql
 - WAL implementation seems to be similar to sqlite3
 - Async commit is when WAL is not synced to disk on commit
 - At checkpoint time all dirty pages are flushed to disk
 - Allows Point-in-time recovery and online backup to be implemented

###SQL server
 - Each WAL record corresponds to one update to the database file. So for each
   transaction multiple WAL records might be written. Also WAL records of
   concurrent transactions will be comingled.
 - Data files in memory are updated after the WAL record is written. These
   are the dirty pages which are synced to disk on a checkpoint. Both
   committed and uncommitted transactions will have their WAL written.

###Hbase
 - One WAL file per region server
 - WAL is written to HDFS (so that other servers can replay if a region server
   fails)
 - memstore is a sorted in-memory table to record writes after they are
   recorded in the WAL
 - One HFile per column family

###Bigtable
 - The way memtable and sstables interact is interesting. Reads happen on a
   merged view of all sstables and memtable. Once is a while a memtable is
   converted to sstable. Once in a longer while a few sstables and memtable are
   merged into one sstable. Once is an even longer while all sstables and
   memtable are merged into one sstable (major compaction).

[sqlite3]: https://www.sqlite.org/wal.html
[postgresql]: http://www.postgresql.org/docs/current/static/wal.html 
[sqlserver]: http://www.devintersection.com/updates/Randal-SQL-Logging.pdf
[hbase]: http://blog.cloudera.com/blog/2012/06/hbase-write-path/
[bigtable]: http://research.google.com/archive/bigtable.html
