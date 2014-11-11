---
layout: post
title: "备份最近7份"
description: ""
category: "运维"
tags: [Shell, 备份]
---
{% include JB/setup %}
<p>
Linux系统备份最近7天的脚本
</p>
{% highlight bash %}
#!/bin/bash
time_str=`date "+%F-%H%M%S"`
echo $time_str
  
/*生成当前时间字符串*/  
function get_time(){
    time_str=`date "+%F-%H%M%S"`
}
  
/*更改文件名，依次往前推*/  
function change_name(){
    rm -rf 1-*
    name_tail=`ls 2-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 2-* 1-$name_tail
    name_tail=`ls 3-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 3-* 2-$name_tail
    name_tail=`ls 4-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 4-* 3-$name_tail
    name_tail=`ls 5-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 5-* 4-$name_tail
    name_tail=`ls 6-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 6-* 5-$name_tail
    name_tail=`ls 7-* | awk -F '-' '{print $2"-"$3"-"$4"-"$5}'`
    mv 7-* 6-$name_tail
}
   
/*主备份程序*/ 
function backup(){
   count=`ls |egrep -c "^$1-"`		/*检测参数1的文件是否存在	*/
   if [ $count -eq 1 ]; then		/*判断检测结构			*/
        echo "Find file, not backup"	/*文件存在，不备份		*/
	if [ $1 -eq 7 ]; then		/*检测参数文件是否是最新	*/
	    echo "The number is 7"
            change_name			/*如果为参数为7，则修改之前版本	*/
            get_time
            touch 7-$time_str.bak	/*生成最新版本			*/
	fi
   else
        echo "Not Find"			/*参数名文件不存在，进行备份	*/
        get_time
        touch $1-$time_str.bak
	exit 0				/*备份完毕，退出脚本，即main中的程序只会备份一个	*/
   fi
}
  
function main(){
   backup 1				/*主程序，从第一个版本备份,backup内部会控制如果备份就退出脚本，如果找到就不操作	*/
   backup 2
   backup 3
   backup 4
   backup 5
   backup 6
   backup 7				/*首先查找有7，就修改文件名递推，在备份，没有7就直接备份			*/
   ls -al
}
  
main
{% endhighlight %}

