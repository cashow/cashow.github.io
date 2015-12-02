---
layout: post
title: "Android开发工具 - android指令"
date: 2014-05-20 17:56:34 +0800
tags: Android Android开发工具
---

android指令可以用来在命令行创建、删除和查看虚拟设备，创建和更新android项目，查看已安装的Android SDK版本

android指令的全局选项：  
-h 显示帮助  
-s 静默模式，只打印错误信息  
-v 普通模式，打印所有的信息  

<pre class="mcode">
# 打开Android SDK Manager窗口  
android sdk  

# 打开Android AVD Manager窗口  
android avd  

# 列出已安装的Android SDK版本和AVD设备  
android list   

# 列出已安装的AVD设备  
android list avd   

# 列出已安装的Android SDK版本  
android list target   

# 创建名为myavd的AVD设备，SDK版本id为1，其中id是输入android list后得到的id。加上 -p path 可指定AVD生成的路径  
android create avd -n myavd -t 1   

# 删除名为myavd的AVD设备  
android delete avd -n myavd   

# 移动名为myavd的AVD设备，加上 -p path 可指定AVD移动的路径，加上-r newname指定移动后的重命名为newname  
android move avd -n myavd   

# 创建android项目，-n 指定项目名称为helloworld，-t 指定SDK版本id为1，-k 指定生成项目的包名，-a 指定入口activity的名称为Helloworld，-p 指定生成项目的路径  
android create project -n helloworld -t 1 -k com.example.test -a Helloworld -p .  

# 更新项目配置文件，可加 -l path指定项目依赖的库，加 -t 指定SDK版本  
android update project -p path  

# 创建lib工程  
android create lib-project -n helloworld -t 1 -k com.example.test -p .  

# 更新lib工程配置文件
android update lib-project -p path  
</pre>
官方文档：
<http://developer.android.com/tools/help/android.html>