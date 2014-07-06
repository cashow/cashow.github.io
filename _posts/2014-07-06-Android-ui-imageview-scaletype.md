---
layout: post
title: "Android基础控件 - ImageView的scaleType属性"
date: 2014-07-06 22:21:41
---

官方文档：<http://developer.android.com/reference/android/widget/ImageView.ScaleType.html>  

ImageView的scaleType属性：  

<table>
   <tr>
	   <td>center</td>
	   <td>图片居中显示，不对图片进行任何处理。如果图片高度或宽度大于控件的高度或宽度，则只显示图片中间部分</td>  
   </tr>
   <tr>
		<td>centerCrop</td>
		<td>图片居中显示。等比例缩放图片使得图片填充满整个控件，并显示图片中间部分</td>  
   </tr>
   <tr>
		<td>centerInside</td>
		<td>图片居中显示。如果图片高度大于控件高度或图片宽度大于控件宽度，则等比例缩放图片使得图片能在控件中完整显示</td>  
   </tr>
   <tr>
		<td>fitCenter</td>
		<td>图片居中显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件</td>  
   </tr> 
   <tr>
		<td>fitStart</td>
		<td>图片在左上角显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件</td>  
   </tr>
   <tr>
		<td>fitEnd</td>
		<td>图片在右下角显示，等比例缩放图片,使得图片在控件内完整显示并尽可能填充满控件</td>  
   </tr>
   <tr>
		<td>fitXY</td>
		<td>不按图片比例进行缩放，使图片完全填充满整个控件</td>  
   </tr>
   <tr>
		<td>matrix</td>
		<td>绘制图片时使用自定义的matrix</td>
 </tr>
</table>

其中，centerInside和fitCenter的区别在于，当图片比控件小时，centerInside不会对图片进行拉伸，而fitCenter会将图片等比例拉伸到高度或宽度大于控件的高度或宽度。  
***
scaleType的各个属性效果如下（imageview的背景是黑色）：  
1.图片比控件小，图片宽度小于图片高度
<table>
  <tr>
	   <td>原始图片</td>  
	   <td>center</td>
		 <td>centerCrop</td>
			<td>centerInside</td>
  </tr>
  <tr>
	   <td><img src="/images/origin_80x92.jpg"/></td>
	   <td><img src="/images/center_80x92.png"/></td>  
		<td><img src="/images/centerCrop_80x92.png"/></td>  
		<td><img src="/images/centerInside_80x92.png"/></td>  
  </tr>
  </tr>
		<td>fitCenter</td>
		<td>fitStart</td>
		<td>fitEnd</td>
		<td>fitXY</td>
  </tr>
  <tr>
	   <td><img src="/images/fitCenter_80x92.png"/></td>
	   <td><img src="/images/fitStart_80x92.png"/></td>  
		<td><img src="/images/fitEnd_80x92.png"/></td>  
		<td><img src="/images/fitXY_80x92.png"/></td>  
  </tr>
</table>
</br>
2.图片比控件小，图片宽度大于图片高度
<table>
  <tr>
	   <td>原始图片</td>  
	   <td>center</td>
		 <td>centerCrop</td>
			<td>centerInside</td>
  </tr>
  <tr>
	   <td><img src="/images/origin_105x70.jpg"/></td>
	   <td><img src="/images/center_105x70.png"/></td>  
		<td><img src="/images/centerCrop_105x70.png"/></td>  
		<td><img src="/images/centerInside_105x70.png"/></td>  
  </tr>
  </tr>
		<td>fitCenter</td>
		<td>fitStart</td>
		<td>fitEnd</td>
		<td>fitXY</td>
  </tr>
  <tr>
	   <td><img src="/images/fitCenter_105x70.png"/></td>
	   <td><img src="/images/fitStart_105x70.png"/></td>  
		<td><img src="/images/fitEnd_105x70.png"/></td>  
		<td><img src="/images/fitXY_105x70.png"/></td>  
  </tr>
</table>
</br>
3.图片比控件大
<table>
  <tr>
	   <td>原始图片</td>  
	   <td>center</td>
		 <td>centerCrop</td>
			<td>centerInside</td>
  </tr>
  <tr>
	   <td><img src="/images/origin_320x240.jpg"/></td>
	   <td><img src="/images/center_320x240.png"/></td>  
		<td><img src="/images/centerCrop_320x240.png"/></td>  
		<td><img src="/images/centerInside_320x240.png"/></td>  
  </tr>
  </tr>
		<td>fitCenter</td>
		<td>fitStart</td>
		<td>fitEnd</td>
		<td>fitXY</td>
  </tr>
  <tr>
	   <td><img src="/images/fitCenter_320x240.png"/></td>
	   <td><img src="/images/fitStart_320x240.png"/></td>  
		<td><img src="/images/fitEnd_320x240.png"/></td>  
		<td><img src="/images/fitXY_320x240.png"/></td>  
  </tr>
</table>