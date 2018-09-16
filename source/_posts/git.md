---
title: git
date: 2018-09-16 19:27:27
tags: [git]
---

### git 初始化
```
  git init 
```

### 将本地文件推送到远端
```
  git init
  git add .
  git commit -m "first commit"
  git remote add origin git@github.com:13717506327/git_test.git // 配置远端库地址
  git push -u origin master // 第一次推送加参数 -u
```

### 将本地库文件推送到远端
```
  git remote add origin git@github.com:13717506327/git_test.git
  git push -u origin master
```

### 远端库同步
```
  git remote -v // 列出远端库的详细情况
  git pull [origin] [branch名] //  取回远程仓库的变化，并与本地分支合并,
  //格式：git pull  <远程主机名> <远程分支名>:<本地分支名>
  git fetch [origin] // 下载远程仓库的所有变动
  git push [origin] [branch名] // 上传本地指定分支到远程仓库  
```
### 查看信息
```
  git status // 查看文件状态变更
  git log -5 --pretty --oneline // 显示 最近 5 次 的commit 记录, 并以 简写形式展示
```

### git stash 暂存
```
 // 暂时将未提交的变化移除，稍后再移入
  git stash // 保存工作现场
  git stash pop // 回到工作现场
```

### 分支管理
```
  git checkout -b dev  // 创建 分支 并切换
  // 相当于 git branch dev 然后 git checkout dev 
  git branch // 查看当前分支
  git branch -r // 查看远端分支
  git branch -a // 查看本地和远端的所有分支
  git branch 分支名 // 创建分支
  git checkout 分支名 // 切换分支
  git branch -d // 删除本地分支
  git push origin -d BranchName  // 删除远端分支
  git merge // 合并到当前分支
  git merge master // 合并主干代码到当前分支
  git push –all // 将本地的所有分支都推送到远端
```
**注：git pull = git fetch + git merge**
