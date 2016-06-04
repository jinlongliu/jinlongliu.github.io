---
layout: post
title: "Python学习笔记（一）"
description: "Python学习笔记"
category: Python
tags: [Python]
---
{% include JB/setup %}

## 中文编码
Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。<br/>
解决方法为只要在文件开头加入 # -*- coding: UTF-8 -*- 或者 #coding=utf-8 就行了。<br/>
Python 2.0+ 实例
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*-

print "你好，世界";
{% endhighlight %}

Python 3.X源码文件默认使用utf-8编码，所以无需指定

## 基础语法
新增#!/usr/bin/python 后直接执行
{% highlight bash %}
$ chmod +x test.py     # 脚本文件添加可执行权限
$ ./test.py
{% endhighlight %}

### Python标识符
- 以下划线开头的标识符是有特殊意义的。以单下划线开头（_foo）的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用"from xxx import *"而导入；
- 以双下划线开头的（__foo）代表类的私有成员；以双下划线开头和结尾的（__foo__）代表python里特殊方法专用的标识，如__init__（）代表类的构造函数。

### 行和缩进
- 用缩写来写模块，不适用大括号{}，缩进的空白数量可变，但是所有模块必须一致。
- IndentationError: unexpected indent 错误是tab和空格没对齐
- IndentationError: unindent does not match any outer indentation level  错误是tab和空格缩进不一致

### 多行语句
- 使用 “\” 将一行分为多行
- 语句中包含[], {}, () 不需要使用多行连接符
{% highlight python %}
    total = item_one + \
            item_two + \
            item_three
        
    days = ['Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday']    
{% endhighlight %}

### Python 引号
- 单引号'双引号" 和三引号 ''' """ 表示字符串，三引号可以由多行组成
{% highlight python %}
    word = 'word'
    sentence = "这是一个句子。"
    paragraph = """这是一个段落。
    包含了多个语句"""    
{% endhighlight %}

### Python 注释
- 单行注释使用 # 开头
- 多行注释使用 ''' 或"""

### Python 空行
- 函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。
- 同一行显示多条语句，用分号;隔开

## Python 变量类型

### 变量赋值
- 变量不需要声明，赋值是声明和定义的过程

### 标准数据类型
- Numbers（数字）       不可改变的数据类型，改变意味着创建新对象
- String（字符串）
- List (列表)            用[] 标识, + 列表连接运算符， * 重复操作，有序的对象结合
- Tuple (元组)            用()标识，用逗号隔开，不能二次赋值，相当于只读列表
- Dictionary （字典）       用{}标识，通过键来存取，而非偏移存取，无序的对象集合

## Python 运算符

- 算术运算符 + - * / % ** //       ** 幂:x的y次幂  // 取整除：返回商的整数部分
- 比较运算符 ==    !=    <>  >   <   >=  <=
- 赋值运算符 =    +=    -=    *=    /=    %=    **=    //=
- 位运算符   &    |     ^    ~    <<    >>      ~按位取反
- 逻辑运算符 and    or    not    
- 成员运算符 in    not in
- 身份运算符 is    is not

### 运算符优先级
- **
- ~+-
- */%//
- +-
- \>\>  \<\<
- &
- ^  \|
- <= < > >=
- <> == !=
- =  %=  /= //=  -=  +=  *= **=
- is    is not
- in    not in  
- not or and  

## Python 条件语句 

### 形式
{% highlight python %}
if 判断条件：
    执行语句……
else：
    执行语句……    
    
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
    
#简单语句组
if ( var  == 100 ) : print "变量 var 的值为100"     
{% endhighlight %}

## Python 循环语句

### while循环
{% highlight python %}
while 判断条件：
    执行语句……

#循环使用else语句
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
   
#简单语句组 
while (flag): print 'Given flag is really true!'
{% endhighlight %}

### for循环
{% highlight python %}
for iterating_var in sequence:
    statements(s)
    
#通过序列索引迭代
fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print '当前水果 :', fruits[index]
   
#循环使用 else 语句
for num in range(10,20):  # 迭代 10 到 20 之间的数字
    for i in range(2,num): # 根据因子迭代
        if num%i == 0:      # 确定第一个因子
            j=num/i          # 计算第二个因子
            print '%d 等于 %d * %d' % (num,i,j)
            break            # 跳出当前循环
    else:                  # 循环的 else 部分
        print num, '是一个质数'
{% endhighlight %}

### 循环嵌套
#for语法
{% highlight python %}
for iterating_var in sequence:
    for iterating_var in sequence:
        statements(s)
    statements(s)
    
#while语法
while expression:
    while expression:
        statement(s)
    statement(s)
{% endhighlight %}

### Pass语句
Python pass是空语句，是为了保持程序结构的完整性。
{% highlight python %}
#!/usr/bin/python
# -*- coding: UTF-8 -*- 

# 输出 Python 的每个字母
for letter in 'Python':
    if letter == 'h':
        pass
        print '这是 pass 块'
    print '当前字母 :', letter

print "Good bye!"
{% endhighlight %}

## Python Numbers（数字）
{% highlight python %}
#类型转换
int(x [,base ])         将x转换为一个整数  
long(x [,base ])        将x转换为一个长整数  
float(x )               将x转换到一个浮点数  
complex(real [,imag ])  创建一个复数  
str(x )                 将对象 x 转换为字符串  
repr(x )                将对象 x 转换为表达式字符串  
eval(str )              用来计算在字符串中的有效Python表达式,并返回一个对象  
tuple(s )               将序列 s 转换为一个元组  
list(s )                将序列 s 转换为一个列表  
chr(x )                 将一个整数转换为一个字符  
unichr(x )              将一个整数转换为Unicode字符  
ord(x )                 将一个字符转换为它的整数值  
hex(x )                 将一个整数转换为一个十六进制字符串  
oct(x )                 将一个整数转换为一个八进制字符串  


#数学函数
abs(x)	返回数字的绝对值，如abs(-10) 返回 10
ceil(x)	返回数字的上入整数，如math.ceil(4.1) 返回 5
cmp(x, y)	如果 x < y 返回 -1, 如果 x == y 返回 0, 如果 x > y 返回 1
exp(x)	返回e的x次幂(ex),如math.exp(1) 返回2.718281828459045
fabs(x)	返回数字的绝对值，如math.fabs(-10) 返回10.0
floor(x)	返回数字的下舍整数，如math.floor(4.9)返回 4
log(x)	如math.log(math.e)返回1.0,math.log(100,10)返回2.0
log10(x)	返回以10为基数的x的对数，如math.log10(100)返回 2.0
max(x1, x2,...)	返回给定参数的最大值，参数可以为序列。
min(x1, x2,...)	返回给定参数的最小值，参数可以为序列。
modf(x)	返回x的整数部分与小数部分，两部分的数值符号与x相同，整数部分以浮点型表示。
pow(x, y)	x**y 运算后的值。
round(x [,n])	返回浮点数x的四舍五入值，如给出n值，则代表舍入到小数点后的位数。
sqrt(x)	返回数字x的平方根，数字可以为负数，返回类型为实数，如math.sqrt(4)返回 2+0j


#随机函数
choice(seq)	从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数。
randrange ([start,] stop [,step])	从指定范围内，按指定基数递增的集合中获取一个随机数，基数缺省值为1
random()	随机生成下一个实数，它在[0,1)范围内。
seed([x])	改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。
shuffle(lst)	将序列的所有元素随机排序
uniform(x, y)	随机生成下一个实数，它在[x,y]范围内。


#三角函数
acos(x)	返回x的反余弦弧度值。
asin(x)	返回x的反正弦弧度值。	
atan(x)	返回x的反正切弧度值。
atan2(y, x)	返回给定的 X 及 Y 坐标值的反正切值。
cos(x)	返回x的弧度的余弦值。
hypot(x, y)	返回欧几里德范数 sqrt(x*x + y*y)。
sin(x)	返回的x弧度的正弦值。
tan(x)	返回x弧度的正切值。
degrees(x)	将弧度转换为角度,如degrees(math.pi/2) ， 返回90.0
radians(x)	将角度转换为弧度


#数学常量
pi	数学常量 pi（圆周率，一般以π来表示）
e	数学常量 e，e即自然常数（自然常数）。
{% endhighlight %}

## Python字符串
{% highlight python %}
#转义字符
转义字符	描述
\(在行尾时)	续行符
\\	反斜杠符号
\'	单引号
\"	双引号
\a	响铃
\b	退格(Backspace)
\e	转义
\000	空
\n	换行
\v	纵向制表符
\t	横向制表符
\r	回车
\f	换页
\oyy	八进制数，yy代表的字符，例如：\o12代表换行
\xyy	十六进制数，yy代表的字符，例如：\x0a代表换行
\other	其它的字符以普通格式输出


#字符串运算符
操作符	描述	实例
+	字符串连接	a + b 输出结果： HelloPython
*	重复输出字符串	a*2 输出结果：HelloHello
[]	通过索引获取字符串中字符	a[1] 输出结果 e
[ : ]	截取字符串中的一部分	a[1:4] 输出结果 ell
in	成员运算符 - 如果字符串中包含给定的字符返回 True	H in a 输出结果 1
not in	成员运算符 - 如果字符串中不包含给定的字符返回 True	M not in a 输出结果 1
r/R	原始字符串 - 原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法。	print r'\n' 输出 \n 和 print R'\n' 输出 \n
%	格式字符串	请看下一章节


#字符串格式化
#!/usr/bin/python

print "My name is %s and weight is %d kg!" % ('Zara', 21)
输出结果
My name is Zara and weight is 21 kg!


#字符串格式化符号
 符   号	描述
%c	 格式化字符及其ASCII码
%s	 格式化字符串
%d	 格式化整数
%u	 格式化无符号整型
%o	 格式化无符号八进制数
%x	 格式化无符号十六进制数
%X	 格式化无符号十六进制数（大写）
%f	 格式化浮点数字，可指定小数点后的精度
%e	 用科学计数法格式化浮点数
%E	 作用同%e，用科学计数法格式化浮点数
%g	 %f和%e的简写
%G	 %f 和 %E 的简写
%p	 用十六进制数格式化变量的地址


#Unicode字符串
>>> u'Hello World !'
u'Hello World !'
{% endhighlight %}

## Python列表
{% highlight python %}
#脚本操作符
Python 表达式			结果				描述
len([1, 2, 3])			3				长度
[1, 2, 3] + [4, 5, 6]		[1, 2, 3, 4, 5, 6]		组合
['Hi!'] * 4			['Hi!', 'Hi!', 'Hi!', 'Hi!']	重复
3 in [1, 2, 3]			True				元素是否存在于列表中
for x in [1, 2, 3]: print x,	1 2 3				迭代


#函数
1	cmp(list1, list2)
比较两个列表的元素
2	len(list)
列表元素个数
3	max(list)
返回列表元素最大值
4	min(list)
返回列表元素最小值
5	list(seq)
将元组转换为列表

#方法
1	list.append(obj)
在列表末尾添加新的对象
2	list.count(obj)
统计某个元素在列表中出现的次数
3	list.extend(seq)
在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
4	list.index(obj)
从列表中找出某个值第一个匹配项的索引位置
5	list.insert(index, obj)
将对象插入列表
6	list.pop(obj=list[-1])
移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
7	list.remove(obj)
移除列表中某个值的第一个匹配项
8	list.reverse()
反向列表中元素
9	list.sort([func])
对原列表进行排序
{% endhighlight %}

## Python元组
{% highlight python %}
#无关闭分隔符  任意无符号的对象，以逗号隔开，默认为元组
#!/usr/bin/python

print 'abc', -4.24e93, 18+6.6j, 'xyz';
x, y = 1, 2;
print "Value of x , y : ", x,y;

输出：
abc -4.24e+93 (18+6.6j) xyz
Value of x , y : 1 2

#内置函数
1	cmp(tuple1, tuple2)
比较两个元组元素。
2	len(tuple)
计算元组元素个数。
3	max(tuple)
返回元组中元素最大值。
4	min(tuple)
返回元组中元素最小值。
5	tuple(seq)
将列表转换为元组。
{% endhighlight %}


## Python字典
- 字典值可以没有限制地取任何python对象，既可以是标准的对象，也可以是用户定义的，但键不行。
- 1）不允许同一个键出现两次。创建时如果同一个键被赋值两次，后一个值会被记住;
- 2）键必须不可变，所以可以用数字，字符串或元组充当，所以用列表就不行;
{% highlight python %}
#内置函数
1	cmp(dict1, dict2)
比较两个字典元素。
2	len(dict)
计算字典元素个数，即键的总数。
3	str(dict)
输出字典可打印的字符串表示。
4	type(variable)
返回输入的变量类型，如果变量是字典就返回字典类型。


#内置方法
1	radiansdict.clear()
删除字典内所有元素
2	radiansdict.copy()
返回一个字典的浅复制
3	radiansdict.fromkeys()
创建一个新字典，以序列seq中元素做字典的键，val为字典所有键对应的初始值
4	radiansdict.get(key, default=None)
返回指定键的值，如果值不在字典中返回default值
5	radiansdict.has_key(key)
如果键在字典dict里返回true，否则返回false
6	radiansdict.items()
以列表返回可遍历的(键, 值) 元组数组
7	radiansdict.keys()
以列表返回一个字典所有的键
8	radiansdict.setdefault(key, default=None)
和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default
9	radiansdict.update(dict2)
把字典dict2的键/值对更新到dict里
10	radiansdict.values()
以列表返回字典中的所有值
{% endhighlight %}

