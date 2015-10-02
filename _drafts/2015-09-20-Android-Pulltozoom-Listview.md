---
layout: post
title: "Android - 实现Listview下拉时图片缩放的功能"
date: 2015-09-20 11:57:06 +0800
tags: Android Listview
---

现在有很多app在展示个人资料时，都是在一张背景图上展示用户的信息，在用户信息的下方则是用户最近发布的内容列表。部分app还会实现下拉时背景图片放大的效果。这种效果看起来很复杂，实现起来其实很简单。  
先看效果图：
![效果图](http://7xjvhq.com1.z0.glb.clouddn.com/pulltozoom.gif)  
***
###布局文件：
ListView顶部的用户信息栏分为两部分，一个是显示背景图片的image_background，一个是展示用户信息的layout_userinfo。两个控件放在FrameLayout里面。  
###实现方法：  
还记得ImageView的centerCrop有什么用吗？在给ImageView的ScaleType属性设置成centerCrop后，ImageView里面的图片会等比例缩放，使得图片填充满整个控件，并显示图片中间部分。比如在100x200的imageview里显示400x400的图片，会将图片缩放成200x200并显示图片中间部分。  
想象一下，假如展示用户信息的layout宽度是100像素，高度是200像素，那么我们可以生成一个100x200的bitmap，并放入设置了centerCrop的