---
layout: post
title: Running Apache Spark on Apache Mesos
tags: apache mesos
date:   2014-03-28 20:00:00
---

I am trying to run Apache Spark on Apache Mesos. I am following [this][mesos]
tutorial. One problem I hit when trying to save the spark distribution on
HDFS is:

```
put: org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot create
/user/hduser/spark-1.0.0-SNAPSHOT-hadoop_1.0.4-bin.tar.gz. Name node is in safe
mode.
```

This was solved with:

```
bin/hadoop dfsadmin -safemode leave
```

While trying to build mesos I hit the following error:

```
vagrant:/vagrant/mesos/build$ sudo apt-get install python-dev
Reading package lists... Done
Building dependency tree
Reading state information... Done
You might want to run 'apt-get -f install' to correct these:
The following packages have unmet dependencies:
 python-dev : Depends: python2.7-dev (>= 2.7.3) but it is not going to be instal
led
 scala : Depends: libjansi-java but it is not going to be installed
E: Unmet dependencies. Try 'apt-get -f install' with no packages (or specify a s
olution).
```

I was facing this error while doing different apt-get operations and I did not
care too much till now. Now I was stumped. So finally ran `apt-get -f install`
without any arguments. Then I could proceed with installing python-dev. Then
I proceeded with building mesos. Build went fine. `make check` failed with
the following error:

```
echo '../../src/'`tests/fault_tolerance_tests.cpp
g++: internal compiler error: Killed (program cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
See <file:///usr/share/doc/gcc-4.6/README.Bugs> for instructions.
make[3]: *** [tests/mesos_tests-fault_tolerance_tests.o] Error 4
make[3]: Leaving directory `/vagrant/mesos/build/src'
make[2]: *** [check-am] Error 2
make[2]: Leaving directory `/vagrant/mesos/build/src'
make[1]: *** [check] Error 2
make[1]: Leaving directory `/vagrant/mesos/build/src'
make: *** [check-recursive] Error 1
```

Then \#tillt from #mesos on IRC suggested increasing memory of the VM.
Increasing to 3GB allowed the make check to finish. But there were still some
failures:

```
[  PASSED  ] 263 tests.
[  FAILED  ] 17 tests, listed below:
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndFetch
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndStoreAndFetch
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndStoreFailAndFetch
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndExpungeAndFetch
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndExpungeAndExpunge
[  FAILED  ] LevelDBStateTest.FetchAndStoreAndExpungeAndStoreAndFetch
[  FAILED  ] LevelDBStateTest.Names
[  FAILED  ] Strict/RegistrarTest.recover/0, where GetParam() = false
[  FAILED  ] Strict/RegistrarTest.recover/1, where GetParam() = true
[  FAILED  ] Strict/RegistrarTest.admit/0, where GetParam() = false
[  FAILED  ] Strict/RegistrarTest.admit/1, where GetParam() = true
[  FAILED  ] Strict/RegistrarTest.readmit/0, where GetParam() = false
[  FAILED  ] Strict/RegistrarTest.readmit/1, where GetParam() = true
[  FAILED  ] Strict/RegistrarTest.remove/0, where GetParam() = false
[  FAILED  ] Strict/RegistrarTest.remove/1, where GetParam() = true
[  FAILED  ] Strict/RegistrarTest.bootstrap/0, where GetParam() = false
[  FAILED  ] Strict/RegistrarTest.bootstrap/1, where GetParam() = true
```

The failing test details looked like this:

```
    $ GLOG_v=2 ./bin/mesos-tests.sh --gtest_filter="LevelDBStateTest.FetchAndStoreAndFetch" --verbose
     
    ...
     
    [ RUN      ] LevelDBStateTest.FetchAndStoreAndFetch
    I0328 19:12:49.520093  4836 process.cpp:2533] Resuming (1)@10.0.2.15:59978 at 2014-03-28 19:12:49.520086016+00:00
    I0328 19:12:49.520884  4821 process.cpp:2523] Spawned process (1)@10.0.2.15:59978
    ../../src/tests/state_tests.cpp:70: Failure
    (future1).failure(): IO error: /vagrant/mesos/build/.state/MANIFEST-000001: Invalid argument
    I0328 19:12:49.538709  4841 process.cpp:2533] Resuming (1)@10.0.2.15:59978 at 2014-03-28 19:12:49.538699008+00:00
    I0328 19:12:49.539717  4841 process.cpp:2640] Cleaning up (1)@10.0.2.15:59978
    [  FAILED  ] LevelDBStateTest.FetchAndStoreAndFetch (28 ms)
    [----------] 1 test from LevelDBStateTest (29 ms total)
     
    [----------] Global test environment tear-down
    [==========] 1 test from 1 test case ran. (40 ms total)
    [  PASSED  ] 0 tests.
    [  FAILED  ] 1 test, listed below:
    [  FAILED  ] LevelDBStateTest.FetchAndStoreAndFetch
```

Then I gave up on the tests and tried running the master and slave. But the c++
test framework resulted in all 5 tasks lost. spark-shell also resulted in lost
task. So basically this is a dead end.

[mesos]: https://spark.apache.org/docs/latest/running-on-mesos.html
[testfail]: http://stackoverflow.com/questions/22619124/when-running-make-check-on-mesos-one-of-the-tests-fails-what-now
