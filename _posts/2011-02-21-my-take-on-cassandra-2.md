--- 
layout: post
title: My take on Cassandra
location: Manipal
---

For all  those who have just stumbled across this post thinking it to be a
guide on scoring chicks named Cassandra, then I suggest you get a life!

Over the past couple of weeks, we have been trying to shift our backend cache
store from memcachedb to cassandra. The coolest thing about cassandra is its
auto sort feature. As soon as you insert a value in the cassandra data store,
it sorts it for you based on the schema specified and updates it across all the
nodes.  Sorting can be done based on a lot of parameters. Heres a brief list


{% highlight bash %}
BytesType - Simple sort by byte value. No validation is performed. 
UTF8Type - A string encoded as UTF8 
TimeUUIDType - a 128bit version 1 UUID, compared by timestamp 
{% endhighlight %}

Getting cassandra up and running on my Ubuntu Machine was as easy as getting a
Micheal Jordon shoot a 2 pointer. You can download the source from
[here](http://cassandra.apache.org/download/). I would recommend going with the
oldstable release of 0.6.11 if you intend to use PHP as a thrift client. We ran
into a [couple of
problems](http://groups.google.com/group/pandra-user/t/f9bc92bf77ffc986) trying
to make [Pandra](http://github.com/mjpearson/Pandra/) work with the 0.7
version. Thrift is a client interface which talks to cassandra. You need to
download that from [here](http://incubator.apache.org/thrift/download)


To run an instance of cassandra on your machine, untar cassandra-bin.

```tar xvfz apache-cassandra-version-bin.tar.gz```

And run cassandra as root.

```sudo bin/cassandra -f```

-f (foreground) gives you a line by line output of what cassandra is doing.

Before we get started on inserting and pulling out values from Cassandra, heres something you should know.

##Keyspace
Much like your database in RDBMS, these are known as keyspaces in Cassandra.


##Column
A column is a collection of 3 items. A name, value and a timestamp.

{% highlight xml %}
<Column1>
    <name>City</name>
    <value>Mumbai</value>
    <timestamp>123456789</timestamp>
</Column1>
{% endhighlight %}

##Super Column
A super column is just a collection of columns.

{% highlight xml %}
<SuperColumn1>
   <Column1>
      <name>City</name>
      <value>Mumbai</value>
      <timestamp>123456789</timestamp>
   </Column1>
   <Column2>
      <name>City</name>
      <value>Delhi</value>
      <timestamp>123456790</timestamp>
   </Column2>
</SuperColumn1>
{% endhighlight %}


To define your own SuperColumn Schema, you might want to take a look at the
```conf/storage-conf.xml``` file (cassandra.yaml in 0.7) in the conf directory. It
has a pretty good explanation on how to define keyspaces.

Lets get the ball rolling and hook up the cassandra-cli ```bin/cassandra-cli```

Since I'm using the keyspace Keyspace1 (check out the Keyspace1 schema in
storage-conf.xml file), I need to specify that. To insert in cassandra, this is
what you need to do from the command line.

{% highlight bash %}
cassandra>use Keyspace1
cassandra>set Keyspace1.Standard2['id1']['fname'] = 'Jack'
Value Inserted
cassandra>set Keyspace1.Standard2['id1']['lname'] = 'Bauer'
Value Inserted
{% endhighlight %}

To retrieve data, heres what you do.

{% highlight bash %}
cassandra>get Keyspace1.['id1']
=> (column=lname, value=Bauer, timestamp=123456789)
=> (column=fname, value=Jack, timestamp=123456788)
Returned 2 results.
{% endhighlight %}

Thats about it. There are a lot of thrift clients out there in different
languages. I largely code in python, if you do too, go with [pycassa]
(http://github.com/pycassa/pycassa">pycassa)


 
