---
layout: post
title: "Elasticsearch 1.x升级到5.x"
description: ""
category: Elasticsearch 
tags: [Elasticsearch]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，未经博主授权不得转载。</font>**

### 添加白名单
{% highlight bash %}
#vim elasticsearch.yml
reindex.remote.whitelist: "172.18.60.121:9200"

reindex.remote.whitelist: "172.18.60.121:9200,172.18.60.122:9200,172.18.60.123:9200"
{% endhighlight %}


### 5.x创建相同索引

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
{% endhighlight %}




{% include JB/setup %}


