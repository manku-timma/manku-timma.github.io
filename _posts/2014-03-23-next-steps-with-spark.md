---
layout: post
title: Basic usage of Apache Spark
tags: apache spark
date:   2014-03-23 10:00:00
---

Played around with [Apache Spark][spark] today. Ran the spark-shell and did
some text processing as shown in the quickstart documentation. I tried to run
the examples but that fails complaining that the examples are not built. Trying
to run `sbt/sbt assembly` again but there is a SSL certificate error on
repo.maven.apache.org -- there is a domain mismatch (hmm...). Curl complains
that the security chain is invalid.

I tried out the master-slave mode of running spark. Feels really cool counting
119 lines of a small text file using 3 workers and taking 2-3 seconds :)

[spark]: https://spark.apache.org
