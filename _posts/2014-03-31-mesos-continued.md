---
layout: post
title: Running Apache Spark on Apache Mesos (continued)
tags: apache mesos
date:   2014-03-31 06:00:00
---

Continuing my failed [attempt][attempt] to run spark on mesos.
[@pacoid][pacoid] pointed me to a nice [tutorial][tutorial] for this. Rest
of this post is just documentation of my efforts. To spare you the pain
of reading through all of this -- it ends well.

As of now I just want to run mesos properly. The test failures were due to the
/vagrant directory usage. Basically I had synced the mesos source tree into
/vagrant, which is shared between the host OS-X and the guest
ubuntu-in-vagrant. It turns out that `mmap(... MAP_SHARED ...)` throws `EINVAL`
when the file being mmaped is in /vagrant. This was causing leveldb's MANIFEST
file write to fail. This in turn caused all the mesos registrar tests and
leveldb state tests to fail. So just moving the source tree from /vagrant to
/home/vagrant solved the problem and allowed me to do a clean `make check`. :)

Now I am finding that the java and python test frameworks run fine but the C++
one does not. All the 5 tasks created are still lost in the C++ framework. But
the test never ends (not in 20 mins at least).

Before proceeding further a quick note on mesos terms. Mesos has the following
concepts:

- Master (co-ordinates slaves, part of mesos)
- Slave (owns resources and runs tasks, part of mesos)
- Framework (Executor + Scheduler, uses mesos)
 - Scheduler (talking to master and accepting resource offers, uses mesos)
 - Executor (talking to slave and running Tasks, uses mesos)
 - Task (smallest job unit of the framework, runs on slave)

In my case, the flow looked like this:

1. Framework started
2. Scheduler connects to master
3. Master gives resource offer (1 cpu, 1G ram, 35G disk, lots of ports)
4. Scheduler accepts resource offer (1 cpu, 32M ram)
5. Master asks slave to launch task
6. Slave launches executor
7. <font color=red>Executor exits with status 127</font>
8. Slave returns status to master
9. Master removes resource assignment
10. Steps 3-9 are repeated 5 times (since the test-framework's scheduler
    launches 5 tasks)
11. In the end, Slave cleans up the Executor
12. <font color=red>Master continues to send resource offers to Slave
and Slave rejects them</font>

I need to debug step 7 and step 12.

For step 12, it seems like the scheduler has a problem. I see the following
code snippet in `mesos-0.17.0/src/examples/test_framework.cpp`.
{% highlight C %}
    if (status.state() == TASK_FINISHED)
      tasksFinished++;
{% endhighlight %}

I changed it to:
{% highlight C %}
    if (status.state() == TASK_FINISHED || status.state() == TASK_LOST)
      tasksFinished++;
{% endhighlight %}

Predictably the test framework now exits reliably after all the 5 tasks are
LOST.

Now about Step 7. Looking at /tmp/mesos/slaves/.../stderr:

```
/home/vagrant/mesos-0.17.0/build/src/.libs/test-executor: error while loading
shared libraries: libmesos-0.17.0.so: cannot open shared object file: No such
file or directory
```

Since I had not installed mesos, the dynamic library was not found. Easy peasy
again. Changed the `LD_LIBRARY_PATH` for the slave invocation and obtained
nirvana. The new command lines are:

```
$ ./bin/mesos-master.sh --ip=10.0.2.15
$ LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/vagrant/mesos-0.17.0/build/src/.libs/
   ./bin/mesos-slave.sh --master=10.0.2.15:5050
$ ./src/test-framework --master=10.0.2.15:5050
```

Now back to running spark. I am retrying the spark-shell command and still
the tasks are getting lost. The slave error logs have:

```
Failed to copy from HDFS: hadoop fs -copyToLocal 'hdfs://10.0.2.15/user/hduser/s
park-1.0.0-SNAPSHOT-hadoop_1.0.4-bin.tar.gz' './spark-1.0.0-SNAPSHOT-hadoop_1.0.
4-bin.tar.gz'
sh: 1: hadoop: not found
```

Again I have not installed hadoop. So changing the slave invocation helps to
remove the `hadoop: not found` error.

```
$ HADOOP_HOME=/home/hduser/hadoop/
  LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/vagrant/mesos-0.17.0/build/src/.libs/
  ./bin/mesos-slave.sh --master=10.0.2.15:5050
```

Now I verified that mesos is able to download the executor .tar.gz and expand
it. But trying to execute spark on the slave throws the following error:

```
java.lang.ClassNotFoundException: org/apache/spark/serializer/JavaSerializer
        at java.lang.Class.forName0(Native Method)
        at java.lang.Class.forName(Class.java:249)
        at org.apache.spark.SparkEnv$.instantiateClass$1(SparkEnv.scala:151)
        at org.apache.spark.SparkEnv$.create(SparkEnv.scala:162)
        at org.apache.spark.executor.Executor.<init>(Executor.scala:105)
        at org.apache.spark.executor.MesosExecutorBackend.registered(MesosExecutorBackend.scala:56)
```

I then created spark-env.sh in /tmp/mesos/.../spark/conf/ with the following
contents.

```
export MESOS_NATIVE_LIBRARY='/home/vagrant/mesos-0.17.0/build/src/.libs/libmesos.so'
export SPARK_EXECUTOR_URI='hdfs://10.0.2.15/user/hduser/spark-1.0.0-SNAPSHOT-hadoop_1.0.4-bin.tar.gz'
export MASTER='mesos://10.0.2.15:5050/'
export HADOOP_HOME='/home/hduser/hadoop/'
```

Even this gave the `ClassNotFoundException`. Then I reran spark's
`make-distribution.sh` to recreate the spark distribution with my spark-env.sh
changes in it. Nada.

Then I tried to extract the jar created by `make-distribution.sh`. I hit
the following error:

```
java.io.IOException: META-INF/license : could not create directory
       at sun.tools.jar.Main.extractFile(Main.java:907)
       at sun.tools.jar.Main.extract(Main.java:850)
       at sun.tools.jar.Main.run(Main.java:240)
       at sun.tools.jar.Main.main(Main.java:1147)
```

I found that depending on when this error was hit, there was a different class
which was the victim of the ClassNotFoundException. So if JavaSerializer was
not already extracted from the java archive at the time of this exception, then
mesos slaves would report this as the reason of the abrupt checking-out of the
spark framework's executor. In different spark version the victim class was
MesosExecutorBackend. Sweeping the web for these names got me nought. An email
to [spark-dev][sparkdev] got Timothy St. Clair to give me the pom.xml bump
suggestion. I modified pom.xml as follows:

```
-    <mesos.version>0.13.0</mesos.version>
+    <mesos.version>0.17.0</mesos.version>
     <akka.group>org.spark-project.akka</akka.group>
     <akka.version>2.2.3-shaded-protobuf</akka.version>
     <slf4j.version>1.7.5</slf4j.version>
     <log4j.version>1.2.17</log4j.version>
     <hadoop.version>1.0.4</hadoop.version>
-    <protobuf.version>2.4.1</protobuf.version>
+    <protobuf.version>2.5</protobuf.version>
```

So my jar extraction hunch was going in a really wrong direction. The new
spark framework tgz executed past the ClassNotFoundException and into an
entirely new area of the cosmos filled with dark energy:

```
14/03/31 19:41:18 INFO SparkEnv: Connecting to BlockManagerMaster: akka.tcp://sp
ark@localhost:7077/user/BlockManagerMaster
akka.actor.ActorNotFound: Actor not found for: ActorSelection[Actor[akka.tcp://s
park@localhost:7077/]/user/BlockManagerMaster]
        at akka.actor.ActorSelection$$anonfun$resolveOne$1.apply(ActorSelection.
scala:66)
```

Scavenging for this error got me [this][akkaproblem] and [this][akkasolution].
Patching in this PR into my spark 0.9.0 codebase gave me success. Basically I
have a mesos master, mesos slave, hdfs namenode, hdfs datanode, hdfs secondary
namenode, and spark shell and I can now add a few numbers using these
processes.


[attempt]: http://manku-timma.github.io/2014/03/28/running-mesos.html
[pacoid]: https://twitter.com/pacoid
[tutorial]: http://mesosphere.io/learn/run-spark-on-mesos/
[sparkdev]: http://apache-spark-user-list.1001560.n3.nabble.com/java-lang-ClassNotFoundException-spark-on-mesos-tt3510.html
[akkaproblem]: https://spark-project.atlassian.net/browse/SPARK-1052
[akkasolution]: https://github.com/apache/incubator-spark/pull/568/files
