---
layout: post
title: "vim常用配置"
date: 2014-05-09 10:41:05
---

<h4>在普通模式下的指令：</h4>
<table>
   <tr>
      <td>i</td>
      <td>在光标前插入</td>
   </tr>
   <tr>
      <td>a</td>
      <td>在光标后插入</td>
   </tr>
   <tr>
      <td>I</td>
      <td>在当前行首插入</td>
   </tr>
   <tr>
      <td>A</td>
      <td>在当前行尾插入</td>
   </tr>
   <tr>
      <td>o(小写字母o)</td>
      <td>在下方插入新行</td>
   </tr>
   <tr>
      <td>O(大学字母O)</td>
      <td>在上方插入新行</td>
   </tr>
   <tr>
      <td>x</td>
      <td>删除光标所在的字符</td>
   </tr>
   <tr>
      <td>dw</td>
      <td>删除光标所在处到下个单词起始处</td>
   </tr>
   <tr>
      <td>de</td>
      <td>删除光标所在处到单词末尾</td>
   </tr>
   <tr>
      <td>dd</td>
      <td>剪切光标所在的行</td>
   </tr>
   <tr>
      <td>d$</td>
      <td>删除从光标所在位置到行尾的字符</td>
   </tr>
   <tr>
      <td>cw或ce</td>
      <td>删除光标所在位置到单词末尾的字符并进入插入模式</td>
   </tr>
   <tr>
      <td>c$</td>
      <td>删除从光标所在位置到行尾的字符并进入插入模式</td>
   </tr>
   <tr>
      <td>p</td>
      <td>粘贴</td>
   </tr>
   <tr>
      <td>r+一个字符</td>
      <td>替换掉光标当前字符</td>
   </tr>
   <tr>
      <td>R</td>
      <td>进入替换模式，输入的字符会替换掉当前字符</td>
   </tr>
   <tr>
      <td>u</td>
      <td>撤销最后执行的命令</td>
   </tr>
   <tr>
      <td>U</td>
      <td>撤销对整行的修改</td>
   </tr>
   <tr>
      <td>Ctrl+r</td>
      <td>恢复上个操作</td>
   </tr>
   <tr>
      <td>hijk</td>
      <td>←↓↑→</td>
   </tr>
   <tr>
      <td>$</td>
      <td>移动光标到本行行尾</td>
   </tr>
   <tr>
      <td>^</td>
      <td>移动光标到本行第一个不为空的字符</td>
   </tr>
   <tr>
      <td>g_</td>
      <td>移动光标到本行最后一个不为空的字符</td>
   </tr>
   <tr>
      <td>0(数字0)</td>
      <td>移动光标到行首</td>
   </tr>
   <tr>
      <td>w</td>
      <td>移动光标到下个单词的开始</td>
   </tr>
   <tr>
      <td>e</td>
      <td>移动光标到下个单词的末尾</td>
   </tr>
   <tr>
      <td>b</td>
      <td>移动光标到上个单词</td>
   </tr>
   <tr>
      <td>G</td>
      <td>移动到文件最后一行</td>
   </tr>
   <tr>
      <td>数字n+G</td>
      <td>移动到第n行</td>
   </tr>
   <tr>
      <td>gg</td>
      <td>移动到文件第一行</td>
   </tr>
   <tr>
      <td>v</td>
      <td>进入选择模式，按hjkl移动光标，y键复制，d键删除。输入:w test.txt可把选取的文字保存在test.txt里面</td>
   </tr>
   <tr>
      <td>zc</td>
      <td>折叠代码</td>
   </tr>
   <tr>
      <td>zo</td>
      <td>打开折叠的代码</td>
   </tr>
   <tr>
      <td>gt</td>
      <td>移动到下一个标签页</td>
   </tr>
   <tr>
      <td>gT</td>
      <td>移动到上一个标签页</td>
   </tr>
   <tr>
      <td>q+字母x</td>
      <td>开始录制宏，宏的名字为x</td>
   </tr>
   <tr>
      <td>q</td>
      <td>开始录制宏后，按q结束录制宏</td>
   </tr>
   <tr>
      <td>@+字母x</td>
      <td>调用宏x</td>
   </tr>
   <tr>
      <td>/+字符串</td>
      <td>搜索字符串，按n键到下一个匹配的字符串，按N键到上一个匹配的字符串</td>
   </tr>
   <tr>
      <td>数字n+操作</td>
      <td>重复n次操作，例如3w使光标移动到第三个单词的末尾，d2w删除2个单词</td>
   </tr>
   <tr>
      <td>%</td>
      <td>跳转到匹配的括号处</td>
   </tr>
   <tr>
      <td>Shift+Insert</td>
      <td>调用系统剪贴板内的内容</td>
   </tr>
   <tr>
      <td>Ctrl+f</td>
      <td>下一页</td>
   </tr>
   <tr>
      <td>Ctrl+b</td>
      <td>上一页</td>
   </tr>
   <tr>
      <td>==</td>
      <td>对光标所在的行进行自动缩进</td>
   </tr>
</table>

<table>
   <tr>
      <td>:q!</td>
      <td>放弃更改退出程序</td>
   </tr>
   <tr>
      <td>:w</td>
      <td>保存文件</td>
   </tr>
   <tr>
      <td>:w 文件名</td>
      <td>以指定的文件名保存文件(如输入:w text.txt并回车，会保存成text.txt的文本)</td>
   </tr>
   <tr>
      <td>:wq</td>
      <td>强制保存文件并退出，每次保存时刷新文件修改时间</td>
   </tr>
   <tr>
      <td>:x</td>
      <td>只有在文件修改了的情况下才会保存文件并刷新文件修改时间</td>
   </tr>
   <tr>
      <td>:r 文件名</td>
      <td>读取文件并把文件内容插入到光标所在处</td>
   </tr>
   <tr>
      <td>:r !命令</td>
      <td>执行命令并把命令的输出插入到光标所在处</td>
   </tr>
   <tr>
      <td>:e D:\1.java</td>
      <td>打开d盘1.java文件</td>
   </tr>
   <tr>
      <td>:saveas D:\2.java</td>
      <td>另存为2.java</td>
   </tr>
   <tr>
      <td>:s/字符串old/字符串new</td>
      <td>替换第一个字符串old为字符串new</td>
   </tr>
   <tr>
      <td>:s/字符串old/字符串new</td>
      <td>替换整行的字符串old为字符串new</td>
   </tr>
   <tr>
      <td>:数字m,数字ns/字符串old/字符串new/g</td>
      <td>替换第m行到第n行中的每个字符串old为字符串new</td>
   </tr>
   <tr>
      <td>:%s/字符串old/字符串new/g</td>
      <td>替换整个文件中的每个字符串old为字符串new</td>
   </tr>
   <tr>
      <td>:%s/字符串old/字符串new/gc</td>
      <td>替换整个文件中的每个字符串old为字符串new，并对每个匹配字符串提示是否进行替换</td>
   </tr>
   <tr>
      <td>:noh</td>
      <td>取消搜索后查询内容的高亮</td>
   </tr>
   <tr>
      <td>:!+命令</td>
      <td>执行外部命令，如:!ls列出目录内的所有文件</td>
   </tr>
   <tr>
      <td>:set ic</td>
      <td>查找时忽略大小写</td>
   </tr>
   <tr>
      <td>:set noic</td>
      <td>查找时不忽略大小写(在选项前加no是关闭选项)</td>
   </tr>
   <tr>
      <td>:tabc</td>
      <td>关闭当前标签页</td>
   </tr>
   <tr>
      <td>:tabo</td>
      <td>关闭所有标签页</td>
   </tr>
   <tr>
      <td>:tabn</td>
      <td>移动到下一个标签页</td>
   </tr>
   <tr>
      <td>:tabp</td>
      <td>移动到上一个标签页</td>
   </tr>
   <tr>
      <td>:tabnew</td>
      <td>新建标签页</td>
   </tr>
   <tr>
      <td>:tabfirst</td>
      <td>移动到第一个标签页</td>
   </tr>
   <tr>
      <td>:tablast</td>
      <td>移动到最后一个标签页</td>
   </tr>
</table>
<br/>
<h4>_vimrc文件的配置：</h4>
<table>
   <tr>
      <td>set nofoldenable</td>
      <td>关闭自动折叠代码功能</td>
   </tr>
   <tr>
      <td>set guifont=Consolas:h11</td>
      <td>设置字体为11号Consolas体</td>
   </tr>
   <tr>
      <td>colorscheme molokai</td>
      <td>设置配色方案为molokai</td>
   </tr>
   <tr>
      <td>set lines=40 columns=160</td>
      <td>启动时窗口宽度为160行，高度为40行</td>
   </tr>
   <tr>
      <td>winpos 255 100</td>
      <td>启动时窗口位置，左上角是0 0</td>
   </tr>
   <tr>
      <td>set number</td>
      <td>显示行号</td>
   </tr>
   <tr>
      <td>set showmatch</td>
      <td>光标移动到括号时高亮显示匹配的括号</td>
   </tr>
   <tr>
      <td>set expandtab</td>
      <td>输入tab时用空格替代</td>
   </tr>
   <tr>
      <td>set shiftwidth=4</td>
      <td>设置系统自动缩进所使用的空白长度为4个空格</td>
   </tr>
   <tr>
      <td>set tabshop=4</td>
      <td>设置tab宽度为4个空格</td>
   </tr>
   <tr>
      <td>set autoindent</td>
      <td>换行时自动缩进，每一行和上一行有同样的缩进量。autoindent的缩进规则是最简单的。</td>
   </tr>
   <tr>
      <td>set smartindent</td>
      <td>换行时智能缩进，每一行和上一行有同样的缩进量，同时在遇到右花括号时取消缩进形式。如果这一行是以#开头，则不采用缩进形式。</td>
   </tr>
   <tr>
      <td>set cindent</td>
      <td>专门用于c语言的缩进。cindent可设置不同的缩进风格，设置项包括cinkeys、cinwords和cinoptions。 </td>
   </tr>
   <tr>
      <td>set nofoldenable</td>
      <td>禁用代码折叠功能</td>
   </tr>
   <tr>
      <td>set foldlevelstart=99</td>
      <td>不禁用代码折叠，但在开启文件时不自动折叠代码</td>
   </tr>
   <tr>
      <td>set foldmethod=indent</td>
      <td>设置折叠规则为按文件的缩进进行折叠 </td>
   </tr>
   <tr>
      <td>set fileencodings=ucs-bom,utf-8,chinese </td>
      <td>让vim在使用ansi编码前尝试ucs-born,utf-8,chinese的编码方案</td>
   </tr>
   <tr>
      <td>set clipboard+=unnamed</td>
      <td>将vim的默认缓冲挂到Windows的剪贴版上。这样按y和p键就可以直接实现“复制”和“粘贴”的功能。</td>
   </tr>
   <tr>
      <td>autocmd FileType text setlocal textwidth=0</td>
      <td>修改vimrc_example.vim的这段代码，禁用vim的自动换行功能（在一行输入太多文字后会自动换行）</td>
   </tr>
   <tr>
      <td>au FileType php setlocal dict+=$VIMRUNTIME/php_funclist.txt</td>
      <td>为php文件加入php_funclist.txt的字典，用于自动补全代码</td>
   </tr>
   <tr>
      <td>:inoremap ( ()&#60;ESC&#62;i</td>
      <td>输入左括号"("时自动补上右括号，"()&#60;ESC&#62;i"表示在输入"("时改成"()"并按ESC键，再按i键进行插入，这时光标会在括号里面</td>
   </tr>
   <tr>
      <td>:inoremap ) &#60;c-r&#62;=ClosePair(')')<CR></td>
      <td>输入右括号")"如果有匹配的左括号则不插入右括号，把光标右移一位</td>
   </tr>
</table>

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