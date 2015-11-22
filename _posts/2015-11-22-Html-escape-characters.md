---
layout: post
title: "网页工具 - 尖括号转义"
date: 2015-11-22 15:25:15 +0800
tags: 常用工具
---

要在markdown里显示html代码，需要把小于号 < 和大于号 > 进行转义
###转义前
<textarea id="text_before" class="minput" style="width:100%" rows="20"></textarea>
<button class="btn btn-primary btn-lg pagebtn" id="copy_before" data-container="body" data-toggle="manual" data-placement="right" data-content="已复制">复制到剪贴板</button>
###转义后
<textarea id="text_after" class="minput" style="width:100%" rows="20"></textarea>
<button class="btn btn-primary btn-lg pagebtn" id="copy_after" data-container="body"data-toggle="manual" data-placement="right" data-content="已复制">复制到剪贴板</button>

<script type="text/javascript" src="{{ '/assets/js/clipboard.min.js' | prepend: site.baseurl }}"></script>
<script type="text/javascript">
	$(function () {
	  $('[data-toggle="popover"]').popover()
	})

	$("#text_before").keyup(function(){
		var text = $("#text_before").val();
		var escaped_text = text.replace(/</g, '&lt;').replace(/>/g, '&gt;')
		$("#text_after").val(escaped_text)
	});
	
	$("#text_after").keyup(function(){
		var text = $("#text_after").val();
		var unescaped_text = text.replace(/&lt;/g, '<').replace(/&gt;/g, '>')
		$("#text_before").val(unescaped_text)
	});
	
	$("#copy_before").click(function(){
		var text = $("#text_before").val();
		clipboard.copy(text).then(function(){
			$('#copy_before').popover('show')
		  }, function(err){
		 });
	})
	
	$('#copy_before').on('shown.bs.popover', function() {
    	setTimeout(function() {
        	$('#copy_before').popover('hide');
    	}, 1000);
	});
	
	$("#copy_after").click(function(){
		var text = $("#text_after").val();
		clipboard.copy(text).then(function(){
			$('#copy_after').popover('show')
		  }, function(err){
		 });
	})
	
	$('#copy_after').on('shown.bs.popover', function() {
    	setTimeout(function() {
        	$('#copy_after').popover('hide');
    	}, 1000);
	});
</script>