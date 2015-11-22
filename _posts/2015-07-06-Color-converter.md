---
layout: post
title: "颜色工具 - RGB颜色与十六进制转换"
date: 2015-07-06 22:10:23 +0800
tags: 常用工具
---

RGB颜色与十六进制转换  
<table>
	<tr>
		<td>
			RGB值
		</td>
		<td>
			十六进制值
		</td>
		<td>
			颜色
		</td>
	</tr>
	<tr>
		<td>
			<div>
				R : <input class="minput input-color" id="color-r" placeholder="0" onkeypress='return event.charCode >= 48 && event.charCode <= 57'/>
				G : <input class="minput input-color" id="color-g" placeholder="0" onkeypress='return event.charCode >= 48 && event.charCode <= 57'/>
				B : <input class="minput input-color" id="color-b" placeholder="0" onkeypress='return event.charCode >= 48 && event.charCode <= 57'/>
			</div>
		</td>
		<td>
			<input class="minput input-dex" id="color-dex" value="000000" />
		</td>
		<td>
			<div class="color-block" />
		</td>
	</tr>
</table>
***
Material design推荐的颜色：  
<https://www.google.com/design/spec/style/color.html>

<script type="text/javascript">
	function updateFromRGB(){
		rString = new Number($("#color-r").val()).toString(16);
		if(rString.length == 0){
			rString = "00";
		}
		if(rString.length == 1){
			rString = "0" + rString;
		}
		gString = new Number($("#color-g").val()).toString(16);
		if(gString.length == 0){
			gString = "00";
		}
		if(gString.length == 1){
			gString = "0" + gString;
		}
		bString = new Number($("#color-b").val()).toString(16);
		if(bString.length == 0){
			bString = "00";
		}
		if(bString.length == 1){
			bString = "0" + bString;
		}
		dexString = rString + gString + bString;
		$("#color-dex").val(dexString);
		$(".color-block").css("background-color","#" + dexString);
	}
	function updateFromDEX(){
		dex = $("#color-dex").val();
		if(dex.length == 6){
			r = dex.substring(0, 2);
			g = dex.substring(2, 4);
			b = dex.substring(4, 6);
		}else{
			r = dex.substring(0, 1);
			g = dex.substring(1, 2);
			b = dex.substring(2, 3);
		}
		
		rNum = parseInt(r, 16);
		gNum = parseInt(g, 16);
		bNum = parseInt(b, 16);

		$("#color-r").val(rNum);
		$("#color-g").val(gNum);
		$("#color-b").val(bNum);

		$(".color-block").css("background-color","#" + dex);
	}
	$("#color-r").keyup(function(){
		var value = $("#color-r").val();
		if(value.length == 0){
			value = "0";
		}
		var num = new Number(value);
		if(num >=0 && num <= 255){
			updateFromRGB();
			$("#color-r").removeClass("incorrect");
		}else{
			$("#color-r").addClass("incorrect");
		}
	});
	$("#color-g").keyup(function(){
		var value = $("#color-g").val();
		if(value.length == 0){
			value = "0";
		}
		var num = new Number(value);
		if(num >=0 && num <= 255){
			updateFromRGB();
			$("#color-g").removeClass("incorrect");
		}else{
			$("#color-g").addClass("incorrect");
		}
	});
	$("#color-b").keyup(function(){
		var value = $("#color-b").val();
		if(value.length == 0){
			value = "0";
		}
		var num = new Number(value);
		if(num >=0 && num <= 255){
			updateFromRGB();
			$("#color-b").removeClass("incorrect");
		}else{
			$("#color-b").addClass("incorrect");
		}
	});
	$("#color-dex").keyup(function(){
		var dex = $("#color-dex").val();
		if(dex.length ==6){
			var isValid = true;
			for (var i = dex.length - 1; i >= 0; i--) {
				var x = dex[i];
				if((x <= '9' && x >= '0') || (x <= 'z' && x >= 'a') || (x <= 'Z' && x >= 'A')){
					continue;
				}
				isValid = false;
			};
			if(isValid == true){
				updateFromDEX();
				$("#color-dex").removeClass("incorrect");
			}else{
				$("#color-dex").addClass("incorrect");
			}
		}else if(dex.length ==3){
			var isValid = true;
			for (var i = dex.length - 1; i >= 0; i--) {
				var x = dex[i];
				if((x <= '9' && x >= '0') || (x <= 'z' && x >= 'a') || (x <= 'Z' && x >= 'A')){
					continue;
				}
				isValid = false;
			};
			if(isValid == true){
				updateFromDEX();
				$("#color-dex").removeClass("incorrect");
			}else{
				$("#color-dex").addClass("incorrect");
			}
		}else{
			$("#color-dex").addClass("incorrect");
		}
	});
</script>