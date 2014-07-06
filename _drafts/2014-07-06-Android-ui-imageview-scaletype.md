---
layout: post
title: "Android基础控件 - ImageView的scaleType属性"
date: 2014-07-06 16:37:37
---

官方文档：<http://developer.android.com/reference/android/widget/ImageView.ScaleType.html>

ImageView的scaleType属性：
center：图片居中显示，不对图片进行任何处理
centerCrop：图片居中显示。如果图片高度小于控件高度或图片宽度小于控件宽度，则等比例缩放图片使得图片的高度和宽度都大于或等于控件的高度和宽度
centerInside：图片居中显示。如果图片高度大于控件高度或图片宽度大于控件宽度，则等比例缩放图片使得图片能在控件中完整显示
fitCenter：图片居中显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件
fitStart：图片在左上角显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件
fitEnd：图片在右下角显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件
fitXY：不按图片比例进行缩放，是图片完全填充满整个控件
matrix：绘制图片时使用自定义的matrix