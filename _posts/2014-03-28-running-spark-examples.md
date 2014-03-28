---
layout: post
title: Running Apache Spark examples
tags: apache spark
date:   2014-03-28 10:00:00
---

Apache Spark has a good set of examples to play with. But I could not figure
out how to rerun the test by recompiling only that test. An entire `sbt/sbt
assembly` will take ages and will preclude understanding modified examples
within this lifetime. So a shorter route was necessary. The obvious `sbt/sbt
compile org.apache.spark.streaming.examples` failed.

Digging into the (slow as in molasses) output of `sbt/sbt help` showed `sbt/sbt project` to set the current project. *Sidenote*: Leaving a JVM running
could not hurt the speed of sbt's startup. *End of Sidenote*.  Trying to set
the current project to examples did not succeed. Wading through several
error messages which mentioned '/' as a special character did not help.

Then I gave up on compiling only one test and ran `sbt/sbt assembly` followed
by `run-example` -- which worked fine.

Then I tried to run `NetworkWordCount` example using spark-shell. Of course it
did not work on the first attempt. A couple of tries later the following
snippet worked:
{% highlight scala%}
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.streaming.StreamingContext._
import org.apache.spark.storage.StorageLevel

val ssc = new StreamingContext(sc, Seconds(5))

val lines = ssc.socketTextStream("localhost", 9999, StorageLevel.MEMORY_ONLY_SER)
val words = lines.flatMap(_.split(" "))
val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
wordCounts.print()

ssc.start()
{% endhighlight %}

The spark-shell was started as:
`SPARK_JAVA_OPTS=-Dspark.cleaner.ttl=10000 bin/spark-shell local[2]`.

In another shell I had `nc -lk 9999` running as advised by the help page.
Typing in several lines did not help. Ctrl-D sent that batch of words and
I could see them getting printed as dstreams. But nothing after that
got printed (probably because netcat was not sending the chars over the
network). Also restarting netcat did not help.
