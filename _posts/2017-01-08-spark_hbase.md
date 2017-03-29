---
layout: post
title:  "spark访问hbase"
date:   2017-01-08 21:15:11 +0800
categories: spark
---

Spark访问HBase
===

Spark中简单访问HBase的代码
---
{% highlight shell linenos %}
val conf = HBaseConfiguration.create()
conf.set("hbase.zookeeper.property.clientPort", "2181")
conf.set("hbase.zookeeper.quorum", "webserver,namenode01,namenode02")
//Get操作
val table = new HTable(conf, "tbl_zhaogj")
val get = new Get(Bytes.toBytes("133"))
val result = table.get(get)
println("rowkey:" + new String(result.getRow))
println("getValue name:" + Bytes.toString(result.getValue(Bytes.toBytes("nothing"), Bytes.toBytes("name"))))
{% endhighlight %}

并发访问HBase的代码
---
直接用以上代码，程序会报错说HTable不能序列化，所以需要foreachPartition的方式
{% highlight shell linenos %}
val sparkConf = new SparkConf().setAppName("SparkHBaseGetSerializableTest")
val sc = new SparkContext(sparkConf)
val dpi = sc.textFile("/user/zhaogj/input/dpiB.txt")
dpi.foreachPartition { iter =>
  val conf = HBaseConfiguration.create()
  conf.set("hbase.zookeeper.property.clientPort", "2181")
  conf.set("hbase.zookeeper.quorum", "webserver,namenode01,namenode02")
  //Get操作
  val table = new HTable(conf, "tbl_zhaogj")
  iter.foreach { x =>
    val get = new Get(Bytes.toBytes("" + x.length() % 10))
    val result = table.get(get)
    //do something about result
    println("rowkey:" + new String(result.getRow))
    println("getValue name:" + Bytes.toString(result.getValue(Bytes.toBytes("nothing"), Bytes.toBytes("name"))))
  }
  table.close()
}
sc.stop()
{% endhighlight %}
注：这种并发访问单服务器提供Request的性能不是很好，大概6万次／秒

BTW
===
生日快乐，xq