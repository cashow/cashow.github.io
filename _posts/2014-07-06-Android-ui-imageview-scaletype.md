---
layout: post
title: "Android基础控件 - ImageView的ScaleType属性"
date: 2014-07-06 22:21:41 +0800
tags: Android Android基础控件 ImageView 原创
---
<style type="text/css">
tr td:first-child{
  white-space: nowrap;
}
tr th:first-child{
  white-space: nowrap;
}
</style>
ImageView的ScaleType属性是用来设置图片的缩放方式


center 			| 图片居中显示，不对图片进行任何处理。如果图片高度或宽度大于控件的高度或宽度，则只显示图片中间部分
centerCrop 		| 图片居中显示。等比例缩放图片使得图片填充满整个控件，并显示图片中间部分。可以想象成图片正好包裹住了控件。比如在100x200的imageview里显示400x400的图片，会将图片缩放成200x200并显示图片中间部分
centerInside 	| 图片居中显示。如果图片高度大于控件高度或图片宽度大于控件宽度，则等比例缩放图片使得图片所有内容都能在控件中显示。比如在100x200的imageview里显示400x400的图片，会将图片缩放成100x100
fitCenter 		| 图片居中显示，等比例缩放图片，使得图片所有内容都能在控件内显示并尽可能填充满控件。可以想象成控件正好包裹住了图片
fitStart 		| 图片在左上角显示，等比例缩放图片，使得图片所有内容都能在控件内显示并尽可能填充满控件
fitEnd 			| 图片在右下角显示，等比例缩放图片，使得图片所有内容都能在控件内显示并尽可能填充满控件
fitXY 			| 不按图片比例进行缩放，将图片缩放成图片高度等于控件高度，图片宽度等于控件宽度
matrix 			| 绘制图片时使用自定义的matrix

其中，centerInside和fitCenter的区别在于，当图片比控件小时，centerInside不会对图片进行拉伸，而fitCenter会将图片等比例拉伸到高度或宽度大于控件的高度或宽度。  

***
ScaleType的各个属性效果如下（imageview的背景是黑色）：

1.图片比控件小，图片宽度小于图片高度  

原始图片 			| center 		| centerCrop | centerInside
![origin](http://7xjvhq.com1.z0.glb.clouddn.com/origin_80x92.jpg) | ![center](http://7xjvhq.com1.z0.glb.clouddn.com/center_80x92.png) | ![centerCrop](http://7xjvhq.com1.z0.glb.clouddn.com/centerCrop_80x92.png) | ![centerInside](http://7xjvhq.com1.z0.glb.clouddn.com/centerInside_80x92.png)
fitCenter | fitStart | fitEnd | fitXY
![fitCenter](http://7xjvhq.com1.z0.glb.clouddn.com/fitCenter_80x92.png) | ![fitStart](http://7xjvhq.com1.z0.glb.clouddn.com/fitStart_80x92.png) | ![fitEnd](http://7xjvhq.com1.z0.glb.clouddn.com/fitEnd_80x92.png) | ![fitXY](http://7xjvhq.com1.z0.glb.clouddn.com/fitXY_80x92.png)


2.图片比控件小，图片宽度大于图片高度

原始图片 			| center 		| centerCrop | centerInside
![origin](http://7xjvhq.com1.z0.glb.clouddn.com/origin_105x70.jpg) | ![center](http://7xjvhq.com1.z0.glb.clouddn.com/center_105x70.png) | ![centerCrop](http://7xjvhq.com1.z0.glb.clouddn.com/centerCrop_105x70.png) | ![centerInside](http://7xjvhq.com1.z0.glb.clouddn.com/centerInside_105x70.png)
fitCenter | fitStart | fitEnd | fitXY
![fitCenter](http://7xjvhq.com1.z0.glb.clouddn.com/fitCenter_105x70.png) | ![fitStart](http://7xjvhq.com1.z0.glb.clouddn.com/fitEnd_105x70.png) | ![fitEnd](http://7xjvhq.com1.z0.glb.clouddn.com/fitStart_105x70.png) | ![fitXY](http://7xjvhq.com1.z0.glb.clouddn.com/fitXY_105x70.png)

3.图片比控件大

原始图片 			| center 		| centerCrop | centerInside
![origin](http://7xjvhq.com1.z0.glb.clouddn.com/origin_320x240.jpg) | ![center](http://7xjvhq.com1.z0.glb.clouddn.com/center_320x240.png) | ![centerCrop](http://7xjvhq.com1.z0.glb.clouddn.com/centerCrop_320x240.png) | ![centerInside](http://7xjvhq.com1.z0.glb.clouddn.com/centerInside_320x240.png)
fitCenter | fitStart | fitEnd | fitXY
![fitCenter](http://7xjvhq.com1.z0.glb.clouddn.com/fitCenter_320x240.png) | ![fitStart](http://7xjvhq.com1.z0.glb.clouddn.com/fitStart_320x240.png) | ![fitEnd](http://7xjvhq.com1.z0.glb.clouddn.com/fitEnd_320x240.png) | ![fitXY](http://7xjvhq.com1.z0.glb.clouddn.com/fitXY_320x240.png)


***
官方文档：
<http://developer.android.com/reference/android/widget/ImageView.ScaleType.html>  
