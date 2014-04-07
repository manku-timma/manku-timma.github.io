---
layout: post
title: Playing with HBase
date:   2014-04-07 14:00:00
tags: hbase
---

Below are the steps I followed to get HBase up and running.

{% highlight bash %}
# Download and extract hbase-0.94.18 into /home/vagrant/hbase-0.94.18

# Edit hbase-0.94.18/conf/hbase-site.xml to save data in /home/vagrant/hbase
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>/home/vagrant/hbase/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/vagrant/hbase/zookeeper</value>
  </property>
</configuration>

# Edit hadoop-metrics.properties for monitoring with ganglia
dfs.class=org.apache.hadoop.metrics.ganglia.GangliaContext31
dfs.period=10
dfs.servers=239.2.11.71:8649

mapred.class=org.apache.hadoop.metrics.ganglia.GangliaContext31
mapred.period=10
mapred.servers=239.2.11.71:8649

jvm.class=org.apache.hadoop.metrics.ganglia.GangliaContext31
jvm.period=10
jvm.servers=239.2.11.71:8649

rpc.class=org.apache.hadoop.metrics.ganglia.GangliaContext31
rpc.period=10
rpc.servers=239.2.11.71:8649

# Add a column and some data
hbase> create 'test', 'columnfamily'
hbase> put 'test', 'rowkey1', 'columnfamily:column1', 'value1'
hbase> put 'test', 'rowkey1', 'columnfamily:column2', 'value2'
hbase> put 'test', 'rowkey2', 'columnfamily:column1', 'value3'
hbase> put 'test', 'rowkey2', 'columnfamily:column2', 'value4'
hbase> scan 'test'

# Try out HBase-on-HDFS. HDFS is running locally listening on IP 10.0.2.15
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://10.0.2.15/user/hduser/hbase/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>hdfs://10.0.2.15/user/hduser/hbase/zookeeper</value>
  </property>
</configuration>
{% endhighlight %}

