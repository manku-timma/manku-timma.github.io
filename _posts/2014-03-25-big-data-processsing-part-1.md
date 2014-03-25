---
layout: post
title: Big data processing toy example - Part 1
tags: apache spark
date:   2014-03-25 20:00:00
---

This is the first part of the blog series on big data processing. This will
focus on generating a large dataset. The idea is to generate a large dataset in
a distributed way and write it to HDFS. Below is scala-spark snippet to
generate lots of random integers and store them on HDFS.

{% highlight scala %}
import util.Random

val rdd = sc.parallelize(Seq.fill(100)(Random.nextInt))
rdd.saveAsTextFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random")
{% endhighlight %}

<br/>

###Problems encountered

The hostname was not really matching and spark-shell
was throwing an EOFException.

```
java.io.IOException: Failed on local exception: java.io.IOException: Broken
pipe ; Host Details : local host is: "vagrant-ubuntu-precise-64/10.0.2.15";
destination host is: "vagrant-ubuntu-precise-64":8020;
```

This was happening because I was using spark-0.9.0 which was linked against an
older hadoop version but my installed hadoop was newer.

Then I switched to spark synced to git and presto - it could speak to my local
HDFS. But then I hit this error.

```
org.apache.hadoop.security.AccessControlException:
org.apache.hadoop.security.Ac cessControlException: Permission denied:
user=vagrant, access=WRITE, inode="hduser":hduser:supergroup:rwxr-xr-x
```

Solving this required adding a property into conf/hdfs-site.xml.

{% highlight xml %}
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
{% endhighlight %}

<br/>

###Using multiple spark slaves

I then started 3 spark slaves and increased the problem size to 30000.
The operation was done in 5 seconds.

{% highlight scala %}
import util.Random

val rdd = sc.parallelize(Seq.fill(30000)(Random.nextInt))
rdd.saveAsTextFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random2")
{% endhighlight %}

The output in HDFS shows up as below:

```
$ bin/hadoop dfs -ls /user/hduser/random2
Found 4 items
-rw-r--r--   3 vagrant supergroup          0 2014-03-25 06:12 /user/hduser/random2/_SUCCESS
-rw-r--r--   3 vagrant supergroup     109914 2014-03-25 06:12 /user/hduser/random2/part-00000
-rw-r--r--   3 vagrant supergroup     109915 2014-03-25 06:12 /user/hduser/random2/part-00001
-rw-r--r--   3 vagrant supergroup     109821 2014-03-25 06:12 /user/hduser/random2/part-00002
```

Next I increased the problem size to 300000. The task was done in around 10 s.

{% highlight scala %}
import util.Random

val rdd = sc.parallelize(Seq.fill(300000)(Random.nextInt))
rdd.saveAsTextFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random3")
{% endhighlight %}

Then I used just one slave and kept the problem size at 300000. This time the
task was done in 1 s. Increasing the problem size to 3 million allowed the task
to finish in 2 s.

{% highlight scala %}
import util.Random

val rdd = sc.parallelize(Seq.fill(300000)(Random.nextInt))
rdd.saveAsTextFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random4")
{% endhighlight %}

<br/>

###Problems encountered:
I am guessing that the memory pressure due to running HDFS namenode, datanode,
task trackers, job trackers, and 1 spark master and 3 spark workers -- all
inside a vagrant instance on a Macbook Air is responsible for allowing one
worker to finish much faster than 3 for a cpu bound task.

Another issue I noticed is that the JVM never releases memory. Each of these
tasks sucks up about 120MB of resident memory with the spark shell taking up
about 520MB of resident memory. I don't understand why the spark-shell should
take up so much memory. Is it because when the spark-shell is run without
connecting to an external master it spawns a master within itself?

Overall JVM not releasing memory back to the OS seems like a really bad idea.
Probably there are some configuration params to control this.

Also the performance characteristics of spark is not very clear in the single
machine case where all HDFS and spark processes are running on the same
machine. This will need more investigation.
