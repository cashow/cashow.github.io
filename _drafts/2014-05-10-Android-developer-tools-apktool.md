---
layout: post
title: "Android开发工具 - apktool反编译apk"
date: 2014-05-09 16:58:06
tags: Android Android开发工具 apktool
---

官网：<https://code.google.com/p/android-apktool/>  
下载好apktool后，将apktool的路径加到环境变量里，即可对apk包进行反编译和重新编译  
<table>
   <tr>
      <td>apktool d 123.apk aaa</td>
      <td>把123.apk进行反编译并把反编译后的文件放在aaa文件夹里</td>
   </tr>
   <tr>
      <td>apktool b aaa</td>
      <td>将aaa文件夹里的文件重新编译生成apk文件，生成的文件在dist文件夹里</td>
   </tr>
</table>
