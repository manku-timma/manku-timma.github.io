---
layout: post
title: Big data processing toy example - Part 2
tags: apache spark
date:   2014-03-25 12:00:00
---

This is the second part of the blog series on big data processing. This will
focus on doing some simple processing on a large dataset stored in HDFS.

I have 300000 random integers stored in HDFS as a single shard in
`/user/hduser/random4` directory.

{% highlight scala %}
val rdd4 = sc.hadoopFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random4")
rdd4.partitions.size
{% endhighlight %}

<br/>

###Problems encountered:
The above snippet throws an exception:

```
java.lang.RuntimeException: java.lang.InstantiationException
        at org.apache.hadoop.util.ReflectionUtils.newInstance
          (ReflectionUtils.java:115)
        at org.apache.spark.rdd.HadoopRDD.getInputFormat
          (HadoopRDD.scala:141)
        at org.apache.spark.rdd.HadoopRDD.getPartitions
          (HadoopRDD.scala:154)
```
Checking the api reference for hadoopFile shows that some more arguments
might be needed. Switching to sc.textFile solved the problem.

So the modified snippet is:
{% highlight scala %}
val rdd4 = sc.textFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random4")
rdd4.partitions.size
{% endhighlight %}

In the spark source code, core/src/main/scala/org/apache/spark/SparkContext.scala
shows the following which indicates the right extra arguments to pass.

{% highlight scala %}
 368   def textFile(path: String, minSplits: Int = defaultMinSplits): RDD[String] = {
 369     hadoopFile(path, classOf[TextInputFormat], classOf[LongWritable], classOf[Text],
 370       minSplits).map(pair => pair._2.toString)
 371   }
{% endhighlight %}

Also all ssh connections from the macbook air to my vagrant instance were
list en-masse -- not really sure why this happened.

-----

The next thing to do is to add up all the numbers to find the total.

{% highlight scala %}
val total = sc.accumulator(0)
rdd4.foreach(total += _.toInt)
total.value
{% endhighlight %}

This finished in half a second.

Then I found the total number of non-negative and negative integers.

{% highlight scala %}
rdd4.map(x => if (x.toInt >= 0) (1, 1) else (-1, 1)).reduceByKey(_+_).toArray
{% endhighlight %}

This returns `Array[(Int, Int)] = Array((1,150260), (-1,149740))` indicating my
vagrant instance is looking on the brighter side of today's meandering.

Then I tried k-means clustering. I don't really understand what it stands for
but I was curious. I followed the sample code on the apache spark website.

{% highlight scala %}
val data = rdd4.map(_.split(' ')map(_.toDouble))
// Two clusters needed (using 20 iterations).
val clusters = KMeans.train(data, 2, 20)
clusters2.clusterCenters

// Returns
Array[Array[Double]] = Array(Array(1.0725365163773994E9),
                             Array(-1.0719118797539531E9))
{% endhighlight %}

<br/>

-----

An interesting find was:

```
scala> sc.getConf.getAll
res43: Array[(String, String)] = Array(
           (spark.home,/vagrant/spark),
           (spark.driver.host,vagrant-ubuntu-precise-64),
           (spark.repl.class.uri,http://10.0.2.15:42272),
           (spark.app.name,Spark shell),
           (spark.fileserver.uri,http://10.0.2.15:50572),
           (spark.jars,""),
           (spark.driver.port,49381),
           (spark.master,local),
           (spark.httpBroadcast.uri,http://10.0.2.15:34214))
```

I could not find out the resource consumption of the spark shell anywhere yet.

-----

Next thing I checked was how things work when the HDFS data has more than
one shard. I have 300000 random integers stored in 3 shards on HDFS in
`/user/hduser/random3`.

{% highlight scala %}
val rdd3 = sc.textFile("hdfs://vagrant-ubuntu-precise-64/user/hduser/random3")
rdd3.partitions.size

// Returns
Int = 3
{% endhighlight %}

Then I tested out adding all the numbers.

-----

Last thing to test was to use 3 spark workers on data stored in 3 shards.

This showed a strange warning message

```
scala> rdd3.partitions.size
14/03/25 09:12:42 WARN NativeCodeLoader: Unable to load native-hadoop
                  library for your platform... using builtin-java
                  classes where applicable
14/03/25 09:12:42 WARN LoadSnappy: Snappy native library not loaded
14/03/25 09:12:42 INFO FileInputFormat: Total input paths to process : 3
res0: Int = 3
```

Need to lookup this particular warning and what it means.

-----

One thing that is missing in spark-shell is the ability to show the resource
usage of all its previous commands. This would have been really helpful.
