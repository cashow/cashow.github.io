---
layout: post
title: "Android开发工具 - adb指令"
date: 2014-05-19 17:37:14
tags: Android Android开发工具 adb
---

官方文档：<http://developer.android.com/tools/help/adb.html>

1.adb devices  
显示已连接的设备，如下所示：  
<pre>
List of devices attached  
6e070d91        device  
</pre>
其中6e070d91是设备的id，device是设备的状态。  
设备状态有3种：offline表示设备离线，device表示设备连接正常，no device表示没有设备连接  
如果有多台手机连接到电脑，则需要用 -s 指定adb调用的手机，如adb -s 6e070d91 install helloWorld.apk  

2.adb get-serialno  
获取手机序列号  

3.adb get-state  
获取手机连接的状态即offline、device和no device  

4.adb wait-for-device install helloWorld.apk  
在手机状态变成device后执行install helloWorld.apk  

5.adb install helloWorld.apk  
安装helloWorld.apk到手机上，如果手机里已经安装该应用，可加 -r 重新安装并保留应用的数据  

6.adb uninstall com.example.test  
卸载包名为com.example.test的应用  

7.adb logcat  
显示logcat，可使用grep过滤log，如adb logcat | grep debug  

8.adb pull /sdcard/foo.txt foo.txt  
复制手机的/sdcard/foo.txt文件到本地并命名为foo.txt  

9.adb push foo.txt /sdcard/foo.txt  
将foo.txt文件复制到手机的/sdcard/文件夹并命名为foo.txt  

10.adb start-server  
开启adb服务  

11.adb kill-server  
结束adb服务  

12.adb shell  
进入shell模式，可在手机里执行shell命令  

13.adb shell shellCommand  
在手机里执行shellCommand指令，如adb shell ls  

14.adb shell am start -n com.example.test/.Helloworld  
启动包名为com.example.test的应用的入口activity即com.example.test.Helloworld  

15.adb shell am force-stop com.example.test  
强制关闭包名为com.example.test的应用

16.adb shell am kill com.example.test  
杀死和包名为com.example.test的应用进程  

17.adb shell am kill-all  
杀死所有的后台进程

18.adb shell pm list packages  
列出设备上安装的所有应用的包名  
-f 可显示应用对应的文件  
-d 只显示被禁用的应用  
-e 只显示可用的应用  
-s 只显示系统应用  
-3 只显示第三方应用  
-i 显示应用安装的方式

19.adb shell pm list features  
列出系统所支持的功能，如蓝牙、相机、定位等  

20.adb shell pm path com.example.test  
显示com.example.test的apk路径  

21.adb shell pm clear com.example.test  
删除和com.example.test相关的数据  

22.adb shell pm enable com.example.test  
启用应用com.example.test  

23.adb shell pm disable com.example.test  
禁用应用com.example.test  

24.adb shell pm get-install-location  
查看系统默认的安装方式，0 [auto]是系统自动决定安装位置，1 [internal]是安装在系统内部存储空间，2 [external]是安装在外置存储卡  

25.adb shell screenrecord /sdcard/demo.mp4
录制屏幕并保存为demo.mp4。该功能只能在Android 4.4(API level 19)或更高的版本运行。按Ctrl-C停止录制，否则在3分钟后自动停止录制。可通过--time-limit 30设置录制时间为30秒。  
<hr/>
有时手机已经连上电脑，但adb显示"failed to start daemon"，可通过下面的方法解决：  
1.执行命令adb nodaemon server，出现提示cannot bind 'tcp:5037'，表示端口绑定失败  
2.执行netstat -ano | findstr "5037"，查出正在使用5037端口的应用，结果如下：  
<pre>
  TCP    127.0.0.1:5037         0.0.0.0:0              LISTENING       6676
</pre>
3.在任务管理器里杀死PID为6676的进程。如果任务管理器没有显示PID可在"查看 - 选择列"里选中PID。