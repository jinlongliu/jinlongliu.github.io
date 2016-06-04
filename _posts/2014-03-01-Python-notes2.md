---
layout: post
title: "Python学习笔记（二）"
description: "Python学习笔记"
category: Python
tags: [Python]
---
{% include JB/setup %}

## Python日期和时间
获取当前时间
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import time

localtime = time.localtime(time.time())
print "本地时间为 :", localtime
{% endhighlight %}

{% highlight python %}
#格式化日期
time.strftime(format[, t])
# 格式化成2016-03-20 11:45:39形式
print time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()) 

# 格式化成Sat Mar 28 22:24:24 2016形式
print time.strftime("%a %b %d %H:%M:%S %Y", time.localtime()) 
  
# 将格式字符串转换为时间戳
a = "Sat Mar 28 22:24:24 2016"
print time.mktime(time.strptime(a,"%a %b %d %H:%M:%S %Y"))
{% endhighlight %}

### 日期格式化符号
{% highlight python %}
%y 两位数的年份表示（00-99）
%Y 四位数的年份表示（000-9999）
%m 月份（01-12）
%d 月内中的一天（0-31）
%H 24小时制小时数（0-23）
%I 12小时制小时数（01-12）
%M 分钟数（00=59）
%S 秒（00-59）
%a 本地简化星期名称
%A 本地完整星期名称
%b 本地简化的月份名称
%B 本地完整的月份名称
%c 本地相应的日期表示和时间表示
%j 年内的一天（001-366）
%p 本地A.M.或P.M.的等价符
%U 一年中的星期数（00-53）星期天为星期的开始
%w 星期（0-6），星期天为星期的开始
%W 一年中的星期数（00-53）星期一为星期的开始
%x 本地相应的日期表示
%X 本地相应的时间表示
%Z 当前时区的名称
%% %号本身
{% endhighlight %}


### Time模块
{% highlight python %}
1	time.altzone
返回格林威治西部的夏令时地区的偏移秒数。如果该地区在格林威治东部会返回负值（如西欧，包括英国）。对夏令时启用地区才能使用。
2	time.asctime([tupletime])
接受时间元组并返回一个可读的形式为"Tue Dec 11 18:07:14 2008"（2008年12月11日 周二18时07分14秒）的24个字符的字符串。
3	time.clock( )
用以浮点数计算的秒数返回当前的CPU时间。用来衡量不同程序的耗时，比time.time()更有用。
4	time.ctime([secs])
作用相当于asctime(localtime(secs))，未给参数相当于asctime()
5	time.gmtime([secs])
接收时间辍（1970纪元后经过的浮点秒数）并返回格林威治天文时间下的时间元组t。注：t.tm_isdst始终为0
6	time.localtime([secs])
接收时间辍（1970纪元后经过的浮点秒数）并返回当地时间下的时间元组t（t.tm_isdst可取0或1，取决于当地当时是不是夏令时）。
7	time.mktime(tupletime)
接受时间元组并返回时间辍（1970纪元后经过的浮点秒数）。
8	time.sleep(secs)
推迟调用线程的运行，secs指秒数。
9	time.strftime(fmt[,tupletime])
接收以时间元组，并返回以可读字符串表示的当地时间，格式由fmt决定。
10	time.strptime(str,fmt='%a %b %d %H:%M:%S %Y')
根据fmt的格式把一个时间字符串解析为时间元组。
11	time.time( )
返回当前时间的时间戳（1970纪元后经过的浮点秒数）。
12	time.tzset()
根据环境变量TZ重新初始化时间相关设置。
{% endhighlight %}


### 日历（Calendar）模块
{% highlight python %}
1	calendar.calendar(year,w=2,l=1,c=6)
返回一个多行字符串格式的year年年历，3个月一行，间隔距离为c。 每日宽度间隔为w字符。每行长度为21* W+18+2* C。l是每星期行数。
2	calendar.firstweekday( )
返回当前每周起始日期的设置。默认情况下，首次载入caendar模块时返回0，即星期一。
3	calendar.isleap(year)
是闰年返回True，否则为false。
4	calendar.leapdays(y1,y2)
返回在Y1，Y2两年之间的闰年总数。
5	calendar.month(year,month,w=2,l=1)
返回一个多行字符串格式的year年month月日历，两行标题，一周一行。每日宽度间隔为w字符。每行的长度为7* w+6。l是每星期的行数。
6	calendar.monthcalendar(year,month)
返回一个整数的单层嵌套列表。每个子列表装载代表一个星期的整数。Year年month月外的日期都设为0;范围内的日子都由该月第几日表示，从1开始。
7	calendar.monthrange(year,month)
返回两个整数。第一个是该月的星期几的日期码，第二个是该月的日期码。日从0（星期一）到6（星期日）;月从1到12。
8	calendar.prcal(year,w=2,l=1,c=6)
相当于 print calendar.calendar(year,w,l,c).
9	calendar.prmonth(year,month,w=2,l=1)
相当于 print calendar.calendar（year，w，l，c）。
10	calendar.setfirstweekday(weekday)
设置每周的起始日期码。0（星期一）到6（星期日）。
11	calendar.timegm(tupletime)
和time.gmtime相反：接受一个时间元组形式，返回该时刻的时间辍（1970纪元后经过的浮点秒数）。
12	calendar.weekday(year,month,day)
返回给定日期的日期码。0（星期一）到6（星期日）。月份为 1（一月） 到 12（12月）。
{% endhighlight %}


## Python函数

### 定义函数规则
- 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()。
- 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
- 函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
- 函数内容以冒号起始，并且缩进。
- return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

### 语法
{% highlight python %}
def functionname( parameters ):
    "函数_文档字符串"
    function_suite
    return [expression]
{% endhighlight %}

### 按值传递参数和按引用传递参数
- 所有参数（自变量）在Python中都是按引用传递，在函数内修改参数，调用函数后，原始参数被改变
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def changeme( mylist ):
   "修改传入的列表"
   mylist.append([1,2,3,4]);
   print "函数内取值: ", mylist
   return
 
# 调用changeme函数
mylist = [10,20,30];
changeme( mylist );
print "函数外取值: ", mylist
{% endhighlight %}

结果：
{% highlight python %}
函数内取值:  [10, 20, 30, [1, 2, 3, 4]]
函数外取值:  [10, 20, 30, [1, 2, 3, 4]]
{% endhighlight %}


###　参数
- 必备参数     必备参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样。
- 关键字参数   关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。调用顺序可和声明不一致
- 默认参数     调用函数时，缺省参数的值如果没有传入，则被认为是默认值。
- 不定长参数

关键字参数
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printme( str ):
    "打印任何传入的字符串"
    print str;
    return;
 
#调用printme函数
printme( str = "My string");
{% endhighlight %}

{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printinfo( name, age ):
    "打印任何传入的字符串"
    print "Name: ", name;
    print "Age ", age;
    return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
{% endhighlight %}

默认参数
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printinfo( name, age = 35 ):
    "打印任何传入的字符串"
    print "Name: ", name;
    print "Age ", age;
    return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
printinfo( name="miki" );
{% endhighlight %}

不定长参数
{% highlight python %}
def functionname([formal_args,] *var_args_tuple ):
    "函数_文档字符串"
    function_suite
    return [expression]
    
    
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def printinfo( arg1, *vartuple ):
    "打印任何传入的参数"
    print "输出: "
    print arg1
    for var in vartuple:
      print var
    return;
 
# 调用printinfo 函数
printinfo( 10 );
printinfo( 70, 60, 50 );
{% endhighlight %}

### 匿名函数
- lambda只是一个表达式，函数体比def简单很多。
- lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
- lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。
- 虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。

语法
{% highlight python %}
lambda [arg1 [,arg2,.....argn]]:expression
{% endhighlight %}

{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2;
 
# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )
print "相加后的值为 : ", sum( 20, 20 )
{% endhighlight %}


### 变量作用域
定义在函数内部的变量拥有一个局部作用域，定义在函数外的拥有全局作用域

- 全局变量    
- 局部变量    

## Python 模块
模块就是一个保存了Python代码的文件，如一个叫做aname的模块里的Python代码一般都能在一个叫aname.py的文件中找到。

### 语法
{% highlight python %}
#import语句   如果模块在当前搜索路径就会被导入
import module1[, module2[,... moduleN]

#From…import语句   从模块中导入一个指定的部分到当前的命名空间
from modname import name1[, name2[, ... nameN]]

From…import*语句
from modname import *
{% endhighlight %}

### 定位模块
Python解释器对模块的搜索顺序：

- 当前目录
- 如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录。
- 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/。

## Python文件/IO

### 读取键盘输入
- raw_input([prompt]) 函数从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符）
- input([prompt]) 函数和 raw_input([prompt]) 函数基本类似，但是 input 可以接收一个Python表达式作为输入，并将运算结果返回。

### 打开关闭文件 

### 打开文件
{% highlight python %}
file object = open(file_name [, access_mode][, buffering])
{% endhighlight %}

- file_name：file_name变量是一个包含了你要访问的文件名称的字符串值。
- access_mode：access_mode决定了打开文件的模式：只读，写入，追加等。
- buffering:如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。

{% highlight python %}
模式	描述
r	以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
rb	以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
r+	打开一个文件用于读写。文件指针将会放在文件的开头。
rb+	以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
w	打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
wb	以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
w+	打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
wb+	以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
a	打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
ab	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
a+	打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。
{% endhighlight %}

一个文件被打开后，你有一个file对象，你可以得到有关该文件的各种信息。
{% highlight python %}
属性	描述
file.closed	返回true如果文件已被关闭，否则返回false。
file.mode	返回被打开文件的访问模式。
file.name	返回文件的名称。
file.softspace	如果用print输出后，必须跟一个空格符，则返回false。否则返回true。
{% endhighlight %}

### 关闭文件
File 对象的 close（）方法刷新缓冲区里任何还没写入的信息，并关闭该文件，这之后便不能再进行写入。
{% highlight python %}
fileObject.close();
{% endhighlight %}

### 写入
{% highlight python %}
fileObject.write(string);
{% endhighlight %}

### 读出
{% highlight python %}
fileObject.read([count]);
{% endhighlight %}

注意：Python字符串可以是二进制数据，而不是仅仅是文字。

### 文件定位
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 打开一个文件
fo = open("foo.txt", "r+")
str = fo.read(10);
print "读取的字符串是 : ", str
 
# 查找当前位置
position = fo.tell();
print "当前文件位置 : ", position
 
# 把指针再次重新定位到文件开头
position = fo.seek(0, 0);
str = fo.read(10);
print "重新读取字符串 : ", str
# 关闭打开的文件
fo.close()
{% endhighlight %}

### 重命名和删除文件
{% highlight python %}
#语法
os.rename(current_file_name, new_file_name)

#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 重命名文件test1.txt到test2.txt。
os.rename( "test1.txt", "test2.txt" )



#语法
os.remove(file_name)

#!/usr/bin/python
# -*- coding: UTF-8 -*-

import os
 
# 删除一个已经存在的文件test2.txt
os.remove("test2.txt")
{% endhighlight %}

### 目录操作
{% highlight python %}
#创建
os.mkdir("newdir")

#改变当前目录
os.chdir("newdir")

#显示当前的工作目录
os.getcwd()

#删除目录
os.rmdir('dirname')
{% endhighlight %}

### Python File方法
{% highlight python %}
序号	方法及描述
1	
file.close()
关闭文件。关闭后文件不能再进行读写操作。
2	
file.flush()
刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入。
3	
file.fileno()
返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。
4	
file.isatty()
如果文件连接到一个终端设备返回 True，否则返回 False。
5	
file.next()
返回文件下一行。
6	
file.read([size])
从文件读取指定的字节数，如果未给定或为负则读取所有。
7	
file.readline([size])
读取整行，包括 "\n" 字符。
8	
file.readlines([sizehint])
读取所有行并返回列表，若给定sizeint>0，返回总和大约为sizeint字节的行, 实际读取值可能比sizhint较大, 因为需要填充缓冲区。
9	
file.seek(offset[, whence])
设置文件当前位置
10	
file.tell()
返回文件当前位置。
11	
file.truncate([size])
截取文件，截取的字节通过size指定，默认为当前文件位置。
12	
file.write(str)
将字符串写入文件，没有返回值。
13	
file.writelines(sequence)
向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。
{% endhighlight %}

## Python异常处理
语法
{% highlight python %}
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
{% endhighlight %}

实例
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-

try:
    fh = open("testfile", "w")
    fh.write("这是一个测试文件，用于测试异常!!")
except IOError:
    print "Error: 没有找到文件或读取文件失败"
else:
    print "内容写入文件成功"
    fh.close()
{% endhighlight %}

try-finally 语句
{% highlight python %}
try:
<语句>
finally:
<语句>    #退出try时总会执行
raise
{% endhighlight %}

触发异常
{% highlight python %}
raise [Exception [, args [, traceback]]]
{% endhighlight %}








