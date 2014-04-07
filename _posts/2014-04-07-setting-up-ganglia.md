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

[ganglia]: http://sourceforge.net/apps/trac/ganglia/wiki/ganglia_quick_start
