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

# Get all the data
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

# Get all states, districts, sub-districts which
# have a rural population of 10000 exactly :)
hbase> import org.apache.hadoop.hbase.filter.CompareFilter
hbase> import org.apache.hadoop.hbase.filter.SingleColumnValueFilter
hbase> import org.apache.hadoop.hbase.filter.SubstringComparator
hbase> import org.apache.hadoop.hbase.util.Bytes

# EQUAL works. But any other kind of numeric
# compare does not work.
hbase> scan 'villages', {LIMIT => 10, FILTER => SingleColumnValueFilter.new(Bytes.toBytes('population'), Bytes.toBytes('persons'), CompareFilter::CompareOp.valueOf('EQUAL'), Bytes.toBytes('10000')), COLUMNS => 'population:persons' }

{% endhighlight %}

I tried a simple numeric comparison and fought
long and hard to get it right but it seems nigh
impossible for today. My brain is also shutting
down.  Documentation seems really sparse. This
[link][hbasehelp] was the one ray of hope in the
dark expanse of the internet. In terms of filters
hbase is probably really immature and
documentation is worse.  Getting a few values
can't be this hard. A SQL layer on top of hbase
seems necessary. The hbase shell syntax is too
much of a pain and is too idiosyncratic.

[pig]: http://pig.apache.org/docs/r0.9.1/basic.html
[hbasehelp]: http://stackoverflow.com/questions/11013197/how-to-scan-table-for-a-column-having-particular-value-in-hbase
