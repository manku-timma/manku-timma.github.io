---
layout: post
title: Playing with HBase (part 2)
date:   2014-04-07 20:00:00
tags: hbase
---

I used [Pig][pig] to load data into hbase.

{% highlight bash %}
# Download and extract pig

# Download population data of India's villages from http://www.censusindia.gov.in/2011census/A-3_Vill/A-3%20MDDS_Release.xls

# Use Libreoffice to convert the .xls to a .csv

# Create a table called villages with a column family called population in HBase
hbase> create 'villages', 'population'

bash> export PIG_CLASSPATH=$PIG_/home/vagrant/hbase-0.94.18/hbase-0.94.18.jar:/home/vagrant/hbase-0.94.18/lib/*:pig-0.12.0-withouthadoop.jar:/home/hduser/hadoop/lib/*:$PIG_CLASSPATH

# Load the csv file in Pig
grunt> A = load 'file:///vagrant/village-population.csv' using PigStorage(',');

# Compute (Name of village, Level of village in hierarchy)
grunt> B = foreach A generate $4, $3;

# Clean up data
grunt> C = filter B $0 != '' and $1 != '';

# Save to HBase using name of the village as key and the level as value into population:level column
grunt> store C into 'hbase://villages' using org.apache.pig.backend.hadoop.hbase.HBaseStorage('population:level');

# Compute (Name of village, Number of persons in village)
grunt> B = foreach A generate $4, $6;

# Clean up data
grunt> C = filter B BY $0 != '' and $1 != '';

# Save to HBase using name of village as key and the number of persons as the value into population:persons column
grunt> store C into 'hbase://villages' using org.apache.pig.backend.hadoop.hbase.HBaseStorage('population:persons');

hbase> scan 'villages'
ROW                   COLUMN+CELL
  'N' Thingdawl    column=population:level, timestamp=1396879365983,
                    value=SUB-DISTRICT
  'N' Thingdawl    column=population:persons, timestamp=139687960225
                   8, value=12108

...

 Zunheboto Sadar   column=population:level, timestamp=1396879365963,
                    value=SUB-DISTRICT
 Zunheboto Sadar   column=population:persons, timestamp=139687960225
                   6, value=13344

{% endhighlight %}

[pig]: http://pig.apache.org/docs/r0.9.1/basic.html
