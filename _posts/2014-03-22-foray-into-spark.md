---
layout: post
title: About Apache Spark, jekyll, github pages and rubygems
tags: apache spark, jekyll, github pages, rubygems
---

I tried out [Apache Spark][spark] for the first time today. It was a bit of a
roundabout journey. I downloaded `spark-0.9.0-incubating-bin-hadoop2.tgz` and
tried `sbt/sbt assembly`. java kept failing trying to build the scala compiler.
That is when I realized the binaries are already there since I downloaded a
binary package (ouch). I played around with spark-shell and tried a few
basic things.

**Update 1**: Increasing the vagrant memory limit from 512MB to 1024MB
allows the scala compiler to be built without overrunning available
memory. I see around 780MB peak ram usage.

I created my github Page and tried to make it a [jekyll][jekyll] site. That
involved quite a bit of pain. jekyll mysteriously fails on all commands unless
json and commander gems are not installed. That is not clear in any help page
related to github pages or to jekyll. Once [this page][gemerror] rescued me
from this conundrum, I was able to create my website and update it.

One thing I noticed is that `jekyll serve -P 8000 -w` does not pick up
the changes I make on my OS-X host. Basically the updates to the posts
(which are stored in /vagrant) are not propagated to the guest OS
(ubuntu) where my jekyll server is watching for updates. It appears to
be a problem with either [vagrant][vagrant] or virtualbox where OS-X
file update events don't propagate to the guest OS.

[Ruby Gems][rubygems] must be the lowest common denominator among package
managers. If A depends on B, then deletion of B does not trigger any action. If
A is installed before B, there is no complaint. I am new to rubygems so most
likely I am missing something here.

[spark]: https://spark.apache.org
[jekyll]: http://jekyllrb.com/
[gemerror]: http://dhakshinamoorthy.wordpress.com/2014/03/19/error-setting-up-jekyll-in-ubuntu/
[rubygems]: http://rubygems.org/
[vagrant]: http://www.vagrantup.com/
