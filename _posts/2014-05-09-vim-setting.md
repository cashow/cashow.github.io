---
layout: post
title: "Vim笔记"
date: 2014-05-09 10:41:05 +0800
tags: vim 常用工具
---

以下是vim常用的快捷键和配置  

i | 在光标前插入
a | 在光标后插入
I | 在当前行首插入
A | 在当前行尾插入
o(小写字母o)    | 在下方插入新行
O(大学字母O)    | 在上方插入新行
x | 删除光标所在的字符
dw | 删除光标所在处到下个单词起始处
de | 删除光标所在处到单词末尾
dd | 剪切光标所在的行
d$ | 删除从光标所在位置到行尾的字符
cw或ce | 删除光标所在位置到单词末尾的字符并进入插入模式
c$ | 删除从光标所在位置到行尾的字符并进入插入模式
 p | 粘贴
 r+一个字符 | 替换掉光标当前字符
 R | 进入替换模式，输入的字符会替换掉当前字符
 u | 撤销最后执行的命令
 U | 撤销对整行的修改
 Ctrl+r | 恢复上个操作
 hijk | ←↓↑→
 $ | 移动光标到本行行尾
 ^ | 移动光标到本行第一个不为空的字符
 g_ | 移动光标到本行最后一个不为空的字符
 0(数字0) | 移动光标到行首
 w | 移动光标到下个单词的开始
 e | 移动光标到下个单词的末尾
 b | 移动光标到上个单词
 G | 移动到文件最后一行
 数字n+G | 移动到第n行
 gg | 移动到文件第一行
 v | 进入选择模式，按hjkl移动光标，y键复制，d键删除。输入:w test.txt可把选取的文字保存在test.txt里面
 zc | 折叠代码
 zo | 打开折叠的代码
 gt | 移动到下一个标签页
 gT | 移动到上一个标签页
 q+字母x | 开始录制宏，宏的名字为x
 q | 开始录制宏后，按q结束录制宏
 @+字母x | 调用宏x
 /+字符串 | 搜索字符串，按n键到下一个匹配的字符串，按N键到上一个匹配的字符串
 数字n+操作 | 重复n次操作，例如3w使光标移动到第三个单词的末尾，d2w删除2个单词
 % | 跳转到匹配的括号处
 Shift+Insert | 调用系统剪贴板内的内容
 Ctrl+f | 下一页
 Ctrl+b | 上一页
 == | 对光标所在的行进行自动缩进  


:q! | 放弃更改退出程序
:w | 保存文件
:w 文件名 | 以指定的文件名保存文件(如输入:w text.txt并回车，会保存成text.txt的文本)
:wq | 强制保存文件并退出，每次保存时刷新文件修改时间
:x | 只有在文件修改了的情况下才会保存文件并刷新文件修改时间
:r 文件名 | 读取文件并把文件内容插入到光标所在处
:r !命令 | 执行命令并把命令的输出插入到光标所在处
:e D:\1.java | 打开d盘1.java文件
:saveas D:\2.java | 另存为2.java
:s/字符串old/字符串new | 替换第一个字符串old为字符串new
:s/字符串old/字符串new | 替换整行的字符串old为字符串new
:数字m,数字ns/字符串old/字符串new/g | 替换第m行到第n行中的每个字符串old为字符串new
:%s/字符串old/字符串new/g | 替换整个文件中的每个字符串old为字符串new
:%s/字符串old/字符串new/gc | 替换整个文件中的每个字符串old为字符串new，并对每个匹配字符串提示是否进行替换
:noh | 取消搜索后查询内容的高亮
:!+命令 | 执行外部命令，如:!ls列出目录内的所有文件
:set ic | 查找时忽略大小写
:set noic | 查找时不忽略大小写(在选项前加no是关闭选项)
:tabc | 关闭当前标签页
:tabo | 关闭所有标签页
:tabn | 移动到下一个标签页
:tabp | 移动到上一个标签页
:tabnew | 新建标签页
:tabfirst | 移动到第一个标签页
:tablast | 移动到最后一个标签页


***
<h4>vimrc_example.vim文件的配置：</h4>
<table>
   <tr>
      <td>autocmd FileType text setlocal textwidth=0</td>
      <td>修改这段代码，禁用vim的自动换行功能（在一行输入太多文字后会自动换行）</td>
   </tr>
</table>

***
<h4>_vimrc文件的配置：</h4>

set nofoldenable | 关闭自动折叠代码功能
set guifont=Consolas:h11 | 设置字体为11号Consolas体
colorscheme molokai | 设置配色方案为molokai
set lines=40 columns=160 | 启动时窗口宽度为160行，高度为40行
winpos 255 100 | 启动时窗口位置，左上角是0 0
set number | 显示行号
set showmatch | 光标移动到括号时高亮显示匹配的括号
set expandtab | 输入tab时用空格替代
set shiftwidth=4 | 设置系统自动缩进所使用的空白长度为4个空格
set tabshop=4 | 设置tab宽度为4个空格
set autoindent | 换行时自动缩进，每一行和上一行有同样的缩进量。autoindent的缩进规则是最简单的。
set smartindent | 换行时智能缩进，每一行和上一行有同样的缩进量，同时在遇到右花括号时取消缩进形式。如果这一行是以#开头，则不采用缩进形式。
set cindent | 专门用于c语言的缩进。cindent可设置不同的缩进风格，设置项包括cinkeys、cinwords和cinoptions。 
set nofoldenable | 禁用代码折叠功能
set foldlevelstart=99 | 不禁用代码折叠，但在开启文件时不自动折叠代码
set foldmethod=indent | 设置折叠规则为按文件的缩进进行折叠 
set fileencodings=ucs-bom,utf-8,chinese  | 让vim在使用ansi编码前尝试ucs-born,utf-8,chinese的编码方案
set clipboard+=unnamed | 将vim的默认缓冲挂到Windows的剪贴版上。这样按y和p键就可以直接实现“复制”和“粘贴”的功能。
:inoremap ( ()&#60;ESC&#62;i | 输入左括号"("时自动补上右括号，"()&#60;ESC&#62;i"表示在输入"("时改成"()"并按ESC键，再按i键进行插入，这时光标会在括号里面
:inoremap ) &#60;c-r&#62;=ClosePair(')')&lt;CR&gt; | 输入右括号")"如果有匹配的左括号则不插入右括号，把光标右移一位

au FileType php setlocal dict+=$VIMRUNTIME/php_funclist.txt | 为php文件加入php_funclist.txt的字典，用于自动补全代码

ClosePair的定义：  
<pre>
function ClosePair(char)
if getline('.')[col('.') - 1] == a:char
return "\<Right>"
else
return a:char
endif
endf
</pre>