---
layout: post
title: "Mysql数据库各表统计"
description: ""
category: "运维"
tags: [Mysql]
---
{% include JB/setup %}
<h3>Mysql数据库各表统计</h3>
{% highlight bash %}
use information_schema;
SELECT 
   TABLE_NAME,
	(DATA_LENGTH/1024/1024) as DataM ,
	(INDEX_LENGTH/1024/1024) as IndexM, 
	((DATA_LENGTH+INDEX_LENGTH)/1024/1024) as AllM,
	TABLE_ROWS
FROM
    TABLES
WHERE
    TABLE_SCHEMA = 'ceilometer';
{% endhighlight %}
<p>将'ceilometer'修改为对应数据库名称即可</p>
