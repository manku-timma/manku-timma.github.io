---
layout: post
title:  "Trying out spark streaming"
date:   2014-04-01 12:00:00
tags: apache spark, streaming
---

I am playing with simple examples of spark streaming [here][1] and [here][2].
The flexibility of creating any number of separate dstreams, and calling
`updateStateByKey` on any dstream is really nice.

Here is a [link][log4j] to configure logging in a standalone spark program.


[1]: https://github.com/manku-timma/spark/tree/master/5
[2]: https://github.com/manku-timma/spark/tree/master/6
[log4j]: http://apache-spark-user-list.1001560.n3.nabble.com/Suppress-Log4j-on-SBT-tt2746.html
