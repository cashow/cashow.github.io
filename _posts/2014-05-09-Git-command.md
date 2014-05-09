---
layout: post
title: "Git指令"
date: 2014-05-09 16:32:08
---

<table>
   <tr>
      <td>git init</td>
      <td>创建新的git仓库</td>
   </tr>
   <tr>
      <td>git add a.txt</td>
      <td>将a.txt的改动添加到缓存区</td>
   </tr>
   <tr>
      <td>git add -A</td>
      <td>将所有文件的改动添加到缓存区</td>
   </tr>
   <tr>
      <td>git commit -m "更新"</td>
      <td>将缓存区的改动提交到HEAD，改动信息为"更新"</td>
   </tr>
   <tr>
      <td>git push origin master</td>
      <td>将HEAD的改动提交到远程仓库的master主分支上</td>
   </tr>
   <tr>
      <td>git checkout -b branchx </td>
      <td>创建名为branchx的分支</td>
   </tr>
   <tr>
      <td>git checkout -- a.txt</td>
      <td>用HEAD里的a.txt替换掉工作目录中的文件</td>
   </tr>
   <tr>
      <td>git branch -d branchx</td>
      <td>删除名为branchx的分支</td>
   </tr>
   <tr>
      <td>git checkout master</td>
      <td>切换回主分支</td>
   </tr>
   <tr>
      <td>git pull</td>
      <td>更新本地仓库至最新，git会尝试自动合并</td>
   </tr>
   <tr>
      <td>git merge branchx</td>
      <td>合并branchx分支到当前分支，git会尝试自动合并</td>
   </tr>
   <tr>
      <td>git rm -r --cached a.txt</td>
      <td>在提交之前撤销本地的add操作</td>
   </tr>
</table>