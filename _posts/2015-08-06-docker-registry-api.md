---
layout: post
title: "Docker库API"
description: ""
category: "Docker"
tags: ["Docker", "Docker registry"]
---
{% include JB/setup %}
<h3>查询库中镜像</h3>
{% highlight bash %}
root@docker2:~# curl -X GET docker3:5000/v1/search?q=
  
{
    "num_results": 2,
    "query": "",
    "results": [
        {
            "description": "",
            "name": "library/pause"
        },
        {
            "description": "",
            "name": "library/kube-ui"
        }
    ]
}
{% endhighlight %}

<h3>查询镜像版本</h3>
{% highlight bash %}
root@docker2:~# curl -X GET docker3:5000/v1/repositories/kube-ui/tags
  
{
    "dev": "9d098f8ba86d6e5477867ca67b9b30ec0754663d82eb33769e967fe8537c0214",
    "latest": "9d098f8ba86d6e5477867ca67b9b30ec0754663d82eb33769e967fe8537c0214"
}  
{% endhighlight %}
<p>
/v1/repositories/[XXXXXXX]/tags
</p>
