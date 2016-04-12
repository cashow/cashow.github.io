---
layout: post
title: "Android开发工具 - ant实现自动化编译"
date: 2014-05-13 15:05:18 +0800
tags: Android Android开发工具 ant
---
<style type="text/css">
tr td:first-child{
  white-space: nowrap;
}
tr th:first-child{
  white-space: nowrap;
}
</style>
ant可以用来在命令行里自动编译、打包android项目  
ant下载地址：<http://ant.apache.org/bindownload.cgi>  
安装好后在系统变量里添加变量ANT_HOME，值为ant的安装路径，再将%ANT_HOME%\bin添加到path变量里面  
输入android list并回车，得到电脑上已经安装的sdk版本号和AVD设备，如下所示：  
<pre>
id: 3 or "android-17"
	Name: Android 4.2.2
	Type: Platform
	API level: 17
	...
</pre>
切换到需要编译的项目目录里，输入以下代码即可添加build.xml等ant的配置文件
<pre>
android update project -p . -t 3
</pre>
-p指定的是所要处理的路径，-t指定的是项目所依赖的sdk版本，填入android list里获取到的id即可  
如果要生成签名包，需要在项目的根目录下加入ant.properties文件，文件内容如下：
<pre>
key.store=../keystore/ReleaseKey
key.store.password=123456
key.alias=release
key.alias.password=123456
</pre>
key.store是密钥库文件的地址，key.store.password是密钥库文件的密码，key.alias是签名所使用的私钥，key.alias.password是私钥的密码  

******

#### ant的常用命令：

ant debug | 生成debug包
ant release | 生成签名的apk包
ant debug install | 生成debug包并安装在手机上
ant release install | 生成release包并安装在手机上
ant clean | 清理项目，删除bin目录及gen目录，具体操作可见${sdk.dir}/tools/ant/build.xml里的clean部分