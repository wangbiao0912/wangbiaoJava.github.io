---
layout: post
title:  "ElasticSearch Result Window"
date:   2016-12-15 10:46:06 +0800
categories: es
---
ES默认允许查询的数据量是10000条，当大于这个值时会报错。

假设索引中有10001条数据，当你设置参数setFrom大于10000时会报下面的错误

{% highlight shell %}
org.elasticsearch.action.search.SearchPhaseExecutionException: all shards failed
        at org.elasticsearch.action.search.AbstractSearchAsyncAction.onFirstPhaseResult(AbstractSearchAsyncAction.java:206) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.action.search.AbstractSearchAsyncAction$1.onFailure(AbstractSearchAsyncAction.java:152) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.action.ActionListenerResponseHandler.handleException(ActionListenerResponseHandler.java:46) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.TransportService$DirectResponseChannel.processException(TransportService.java:855) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.TransportService$DirectResponseChannel.sendResponse(TransportService.java:833) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.TransportService$4.onFailure(TransportService.java:387) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:39) ~[elasticsearch-2.3.4.jar:2.3.4]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) ~[?:1.8.0_91]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) ~[?:1.8.0_91]
        at java.lang.Thread.run(Thread.java:745) ~[?:1.8.0_91]
Caused by: org.elasticsearch.search.query.QueryPhaseExecutionException: Result window is too large, from + size must be less than or equal to: [10000] but was [11000]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level parameter.
        at org.elasticsearch.search.internal.DefaultSearchContext.preProcess(DefaultSearchContext.java:212) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.query.QueryPhase.preProcess(QueryPhase.java:103) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.SearchService.createContext(SearchService.java:676) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.SearchService.createAndPutContext(SearchService.java:620) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.SearchService.executeDfsPhase(SearchService.java:264) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.action.SearchServiceTransportAction$SearchDfsTransportHandler.messageReceived(SearchServiceTransportAction.java:360) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.search.action.SearchServiceTransportAction$SearchDfsTransportHandler.messageReceived(SearchServiceTransportAction.java:357) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.TransportRequestHandler.messageReceived(TransportRequestHandler.java:33) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:75) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.transport.TransportService$4.doRun(TransportService.java:376) ~[elasticsearch-2.3.4.jar:2.3.4]
        at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) ~[elasticsearch-2.3.4.jar:2.3.4]
        ... 3 more
{% endhighlight %}

解决办法：
通过调整index的setting，将index.max_resultwindow调大，java代码如下：

{% highlight shell %}
client.admin().indices().prepareUpdateSettings(PubConf.strIdIndex).setSettings(Settings.builder().put("index.max_result_window", 2147483647)).get();
{% endhighlight %}