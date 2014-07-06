---
layout: post
title: "Android开发工具 - android指令"
date: 2014-05-20 17:56:34
tags: Android Android开发工具
---

android指令的全局选项：  
-h 显示帮助  
-s 静默模式，只打印错误信息  
-v 普通模式，打印所有的信息  

1.android sdk  
打开Android SDK Manager窗口  

2.android avd  
打开Android AVD Manager窗口  

3.android list   
列出已安装的Android SDK版本和AVD设备  

4.android list avd   
列出已安装的AVD设备  

5.android list target   
列出已安装的Android SDK版本  

6.android create avd -n myavd -t 3   
创建名为myavd的AVD设备，SDK版本id为3，其中id是输入android list后得到的id。加上 -p path 可指定AVD生成的路径  

7.android delete avd -n myavd   
删除名为myavd的AVD设备  

8.android move avd -n myavd   
移动名为myavd的AVD设备，加上 -p path 可指定AVD移动的路径，加上-r newname指定移动后的重命名为newname  

9.android create project -n helloworld -t 3 -k com.example.test -a Helloworld -p .  
-n 指定项目名称为helloworld，-t 指定SDK版本id为3，-k 指定生成项目的包名，-a 指定入口activity的名称为Helloworld，-p 指定生成项目的路径  

10.android update project -p path  
更新项目配置，可加 -l path指定项目依赖的库，加 -t 指定SDK版本  

11.android create lib-project -n helloworld -t 3 -k com.example.test -p .  
创建lib工程  

12.android update lib-project -p path  
更新lib工程