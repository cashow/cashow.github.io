---
layout: post
title: "Android开发工具 - apktool反编译apk"
date: 2014-05-14 14:55:33
tags: Android Android开发工具 apktool
---

apktool是反编译apk的工具，通过apktool可以查看和修改apk里的文件。  
官网链接：<https://ibotpeaches.github.io/Apktool/>  
下载好apktool后，将apktool的路径加到环境变量里，即可对apk包进行反编译和重新编译  
<table>
   <tr>
      <td>apktool d helloWorld.apk</td>
      <td>对helloWorld.apk进行反编译</td>
   </tr>
   <tr>
      <td>apktool b aaa</td>
      <td>将aaa文件夹里的文件重新编译生成apk文件，生成的apk文件在dist文件夹里，需要注意的是这个apk未签名，不能直接安装在手机里</td>
   </tr>
</table>
******
####修改反编译后的文件
将apk反编译后，可以任意修改AndroidManifest.xml文件和资源文件  
1.通过修改AndroidManifest.xml里的广告activity和service，可删去应用里的广告，如给activity添加属性
<pre class="prettyprint">
android:layout_width="0dp"
android:layout_height="0dp"
</pre>
2.通过修改strings.xml和smali里面标为"const-string"的字符串，可将应用汉化  
3.如果能看懂smali语法，可通过修改smali代码破解应用的内购  
******
####重新签名
在对反编译后的文件进行修改后，可通过[Auto-sign](http://7xjvhq.com1.z0.glb.clouddn.com/Auto-sign.rar)重新签名  
将dist文件夹里的apk放到Auto-sign的文件夹里，重命名为1.zip，双击Sign.bat，会生成update_signed.zip，重命名成update_signed.apk即可  
也可以执行以下命令，其中1.zip是未签名的包，update_signed.zip是签名后的文件名字
<pre class="prettyprint">
java -jar signapk.jar testkey.x509.pem testkey.pk8 1.zip update_signed.zip
</pre>