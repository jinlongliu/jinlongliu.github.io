---
layout: post
title: "Elasticsearch 1.x升级到5.x"
description: ""
category: Elasticsearch 
tags: [Elasticsearch]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### 添加白名单
{% highlight bash %}
#vim elasticsearch.yml
reindex.remote.whitelist: "172.18.60.121:9200"

reindex.remote.whitelist: "172.18.60.121:9200,172.18.60.122:9200,172.18.60.123:9200"
{% endhighlight %}


### 5.x创建相同索引
{% highlight bash %}
curl -XPUT http://172.18.60.133:9200/ilog-test3 -d '
{
	"settings": {
		"refresh_interval": -1,
		"number_of_replicas": 0
	}
}
'

#禁用refresh，导入时不需要搜索，完成后恢复副本数
{% endhighlight %}

### 执行API reindex
{% highlight bash %}
curl -XPOST http://172.18.60.133:9200/_reindex -d '
{
  "source": {
    "remote": {
      "host": "http://172.18.60.121:9200"
    },
    "index": "filebeat-2017.07.27",
    "query": {
      "match_all": {}
    }
  },
  "dest": {
    "index": "ilog-test3"
  }
}
'


#Respond
{
    "took": 1000475,
    "timed_out": false,
    "total": 6429992,
    "updated": 0,
    "created": 6429992,
    "deleted": 0,
    "batches": 1287,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1,
    "throttled_until_millis": 0,
    "failures": []
}
{% endhighlight %}

### 后台运行
{% highlight bash %}
$curl -XPOST http://172.18.60.133:9200/_reindex?wait_for_completion=false -d '...'

#Respond
{
    "task": "GPxXJYbGSt-a_avTbwbFCA:62419"
}

$curl -XGET http://172.18.60.133:9200/_tasks/GPxXJYbGSt-a_avTbwbFCA:62419

#Respond
{
    "completed": false,
    "task": {
        "node": "GPxXJYbGSt-a_avTbwbFCA",
        "id": 62419,
        "type": "transport",
        "action": "indices:data/write/reindex",
        "status": {
            "total": 6429992,
            "updated": 0,
            "created": 5000,
            "deleted": 0,
            "batches": 1,
            "version_conflicts": 0,
            "noops": 0,
            "retries": {
                "bulk": 0,
                "search": 0
            },
            "throttled_millis": 0,
            "requests_per_second": -1,
            "throttled_until_millis": 0
        },
        "description": "reindex from [host=172.18.60.121 port=9200 query={\n  \"match_all\" : { }\n}][filebeat-2017.07.27] to [ilog-test4]",
        "start_time_in_millis": 1501234679363,
        "running_time_in_nanos": 14076089976,
        "cancellable": true
    }
}
{% endhighlight %}

{% include JB/setup %}


