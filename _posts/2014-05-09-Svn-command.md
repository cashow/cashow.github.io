---
layout: post
title: "Svn在linux下的指令"
date: 2014-05-09 17:41:20
tags: svn 常用工具
---

以下是svn在linux下常用的指令：  
1.将文件checkout到本地  
svn checkout svn://192.168.1.1/pro/domain  
2.添加文件进svn  
svn add *.txt  
3.将本地文件提交到svn  
svn commit -m "更新日志"    （提交本目录及子目录下的所有修改过的文件）  
svn commit -m "更新日志" abc.txt  
4.更新本地目录  
svn update  
svn update -r 1234 abc.txt    （更新a.txt到版本号为1234的版本）  
5.查看本地文件状态  
svn status abc      （查看abc文件夹里的文件状态）  
svn status -v abc   （第二列显示工作版本号，第三四列显示最后一次修改的版本号和修改人）  
6.查看日志  
svn log abc.txt （显示abc.txt的log）  
7.查看文件详细信息  
svn info abc.txt    （查看abc.txt的版本，最后修改的作者，版本，时间等详细信息）  
8.查看文件修改信息  
svn diff abc.txt  
svn diff -r 1234:1235 abc.txt   （对比1234版本和1235版本的修改信息）  
9.撤销文件的修改  
svn revert abc.txt
