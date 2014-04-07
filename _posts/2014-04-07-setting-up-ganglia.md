---
layout: post
title:  Setting up Ganglia
date:   2014-04-07 10:00:00
tags: ganglia, monitoring
---

Just for my reference, this is how I setup [ganglia][ganglia] on my ubuntu
12.04.

{% highlight bash %}
# Install relevant packages
$ sudo apt-get install ganglia-monitor gmetad ganglia-webfrontend

# Edit /etc/ganglia/gmond.conf by setting the cluster attribute as below
cluster {
  name = "local-cluster"
  owner = "manku-timma"
  latlong = "world"
  url = "http://127.0.0.1:9001/ganglia"
}

# List statistics being monitored
$ telnet localhost 8649

# Modification to /etc/apache2/sites-available/default
    Alias /ganglia/ "/usr/share/ganglia-webfrontend/"
    <Directory "/usr/share/ganglia-webfrontend/">
        Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
        AllowOverride None
        Order allow,deny
        Allow from all
    </Directory>

# Restart apache after config change
$ sudo service apache2 restart

# List statistics being stored
$ telnet localhost 8651

# Location of stored statistics
$ ls /var/lib/ganglia/rrds/unspecified/__SummaryInfo__

# In a browser access http://localhost/ganglia to view graphs.
{% endhighlight %}

To [configure][configure] [hbase][hbase] to report metrics:

{% highlight bash %}
# Edit hbase/conf/hadoop-metrics.properties with the snippet below.
# For the IP use the mcast_join value from /etc/ganglia/gmond.conf
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
{% endhighlight %}

Hbase reports a ton of metrics, so beware.

[ganglia]: http://sourceforge.net/apps/trac/ganglia/wiki/ganglia_quick_start
[configure]: http://wiki.apache.org/hadoop/GangliaMetrics
[hbase]: http://hbase.apache.org/book/quickstart.html
