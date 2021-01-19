---
layout: post
title: "MySQL触发器"
description: ""
category: Linux
tags: [Linux, CentOS]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### MySQL自动更新时间触发器

```
CREATE TRIGGER `trg_recruitment_source_insert` BEFORE INSERT ON `t_recruitment_source` FOR EACH ROW set new.InsertTime=current_date
;;

CREATE TRIGGER `trg_recruitment_source_update` BEFORE UPDATE ON `t_recruitment_source` FOR EACH ROW set new.UpdateTime=current_date

```

{% include JB/setup %}