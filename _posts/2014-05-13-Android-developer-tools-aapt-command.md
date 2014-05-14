---
layout: post
title: "Android开发工具 - aapt获取apk信息"
date: 2014-05-13 11:42:02
tags: Android Android开发工具 aapt
---

####aapt常用命令  

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
******
####aapt的应用
获取123.apk的packagename：
<pre>
aapt d badging 123.apk |grep package |awk '{print $2}' | awk -F[\'] '{print $2}'
</pre>
获取123.apk的versionCode：
<pre>
aapt d badging 123.apk |grep versionCode |awk '{print $3}' | awk -F[\'] '{print $2}'
</pre>
获取123.apk的versionName：
<pre>
aapt d badging 123.apk |grep versionName |awk '{print $4}' | awk -F[\'] '{print $2}'
</pre>
获取123.apk的launchable-activity：
<pre>
launchActivity=`aapt d badging 123.apk | grep launchable-activity |awk '{print $2}' | awk -F[\'] '{print $2}'`
echo $launchActivity
</pre>
通过以上脚本可实现安装并运行apk的功能：
<pre>
filename='123.apk'
packageName=`aapt d badging $filename |grep package |awk '{print $2}' | awk -F[\'] '{print $2}'`
launchActivity=`aapt d badging $filename | grep launchable-activity |awk '{print $2}' | awk -F[\'] '{print $2}'`
adb install -r $filename
adb shell am start -n $packageName/$launchActivity
</pre>
******
####aapt的部分官方说明：
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