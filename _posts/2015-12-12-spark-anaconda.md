---
layout: post
title: How to use spark with anaconda python
tags: spark, anaconda, python, yarn
date:   2015-12-12 08:00:00
---

The setup is spark 1.5.1 running on yarn (hadoop-2.6.0).

```bash
# Install anaconda python on all nodes of the cluster.
# Below we are installing anaconda's python-2.7 64-bit package.
wget https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda2-2.4.1-Linux-x86_64.sh
bash Anaconda2-2.4.1-Linux-x86_64.sh -b -f -p /usr/lib/anaconda2

# Run this on all nodes of the cluster and restart the ResourceManager/NodeManager daemons.
echo "VIRTUAL_ENV_DISABLE_PROMPT=1 source /usr/lib/anaconda2/bin/activate /usr/lib/anaconda2" >> /usr/lib/hadoop2/etc/hadoop/hadoop-env.sh

# Set some env vars and start pyspark.
export PYSPARK_PYTHON=/usr/lib/anaconda2/bin/python
export PYSPARK_DRIVER_PYTHON=/usr/lib/anaconda2/bin/python
/usr/lib/spark/bin/pyspark --master yarn-client --executor-memory 512m --executor-cores 1 --num-executors 2
```
