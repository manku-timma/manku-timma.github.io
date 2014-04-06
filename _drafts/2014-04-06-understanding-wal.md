---
layout: post
title:  Understand Write Ahead Logging
date:   2014-04-06 12:00:00
tags: wal, write ahead logging
---

I am trying to understand WAL (Write Ahead Logging) from the perspective of
sqlite3, HBase etc. Below is a summary of my understanding.

###Sqlite3
- Advantages of WAL
 - Writes and reads are faster
 - Writes and reads can be concurrent (for a given data file)
 - More sequential disk I/O
 - 
- On Write
 - Append the new data at the end of the current WAL file
- On Read
 - Check wal-index for presence of the data page in WAL
 - If data is in WAL before this reader's end mark, read from WAL
 - Else read from database file
- 
- Other considerations
 - Any number of simultaneous readers
 - One writer at a time
