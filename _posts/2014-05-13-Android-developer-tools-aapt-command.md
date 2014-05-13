---
layout: post
title: "Android开发工具 - aapt"
date: 2014-05-13 11:42:02
tags: Android Android开发工具 aapt
---

aapt常用的命令如下：  

<table>
   <tr>
      <td>aapt d badging 123.apk</td>
      <td>获取123.apk的基本信息如packagename，versionCode，versionName，uses-permission，sdkVersion，launchable-activity等</td>
   </tr>
   <tr>
      <td>aapt d permissions 123.apk</td>
      <td>列出123.apk申请的权限</td>
   </tr>
   <tr>
      <td>aapt l 123.apk</td>
      <td>列出123.apk里的资源文件</td>
   </tr>
   <tr>
      <td>aapt l 123.apk >apkinfo.txt</td>
      <td>列出123.apk里的资源文件并将输出的信息存到apkinfo.txt文件</td>
   </tr>
</table>
<br/>
获取123.apk的packagename：
<pre>
packageName=`aapt d badging 123.apk | grep package`
packageName=${packageName#*"'"}
packageName=${packageName%%"'"*}
echo $packageName
</pre>
获取123.apk的versionCode：
<pre>
versionCode=`aapt d badging 123.apk | grep package`
versionCode=${versionCode#*"versionCode='"}
versionCode=${versionCode%%"'"*}
echo $versionCode
</pre>
获取123.apk的versionName：
<pre>
versionName=`aapt d badging 123.apk | grep package`
versionName=${versionName#*"versionName='"}
versionName=${versionName%%"'"*}
echo $versionName
</pre>
获取123.apk的launchable-activity：
<pre>
launchActivity=`aapt d badging 123.apk | grep launchable-activity`
launchActivity=${launchActivity#*"name='"}
launchActivity=${launchActivity%%"'"*}
echo $launchActivity
</pre>
通过以上脚本可实现安装并运行apk的功能：
<pre>
filename='123.apk'
packageName=`aapt d badging $filename | grep package`
packageName=${packageName#*"'"}
packageName=${packageName%%"'"*}
launchActivity=`aapt d badging $filename | grep launchable-activity`
launchActivity=${launchActivity#*"name='"}
launchActivity=${launchActivity%%"'"*}
adb install -r $filename
adb shell am start -n $packageName/$launchActivity
</pre>
<hr/>
以下是aapt的官方说明：
<pre>
 aapt d[ump] [--values] WHAT file.{apk} [asset [asset ...]]
   badging          Print the label and icon for the app declared in APK.
   permissions      Print the permissions from the APK.
   resources        Print the resource table from the APK.
   configurations   Print the configurations in the APK.
   xmltree          Print the compiled xmls in the given assets.
   xmlstrings       Print the strings of the given compiled xml assets.
</pre>
其余指令可通过aapt help查看