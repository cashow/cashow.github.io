---
layout: post
title: "Sublime text 3常用配置、插件及快捷键"
date: 2014-05-09 16:58:06
---

<h4>安装package control：</h4>
<pre>
import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
</pre>

<h4>配置文件：</h4>
<pre>
Preferences.sublime-settings文件（Preferences -> Settings - User）：
{
   //设置配色方案为Flatland Monokai
   "color_scheme": "Packages/Theme - Flatland/Flatland Monokai.tmTheme",
   //设置字体大小为13
   "font_size": 13,
   //开启vi模式，在插入状态下按ESC即可进入
   "ignored_packages":[],
   //设置主题为Flatland Dark
   "theme": "Flatland Dark.sublime-theme",
   //开启自动换行
   "word_wrap": true
}

Default(windows).sublime-keymap文件（Preferences -> Key Bindings - User）：
[
	//将reindent的快捷键设置成ctrl+alt+f
	{ "keys": ["ctrl+alt+f"], "command": "reindent"}
]
</pre>

<h4>插件：</h4>
[Emmet](http://docs.emmet.io/) (原名zen coding)  
[Trimmer](http://jonlabelle.github.io/Trimmer/)  
[git](https://github.com/kemayo/sublime-text-git)  
[livereload](https://sublime.wbond.net/packages/LiveReload)  
[ColorPicker](http://weslly.github.io/ColorPicker/)  
[JsFormat](https://github.com/jdc0589/JsFormat)  
[ConvertToUTF8](https://github.com/seanliang/ConvertToUTF8)  
