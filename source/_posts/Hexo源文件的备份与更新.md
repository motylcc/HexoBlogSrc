---
title: Hexo源文件的备份与更新
date: 2024-02-19 15:58:28
description: Hexo源文件的备份与更新
tags: [Github, Hexo]
---

## 一 备份本地Hexo源文件

1. 确保当前Hexo源文件为最新版本（与Github上比较）

2. 清除过程文件

   1. 删除Hexo源文件里的 ”.deploy_git”文件夹
   2. 删除过程文件（hexo clean）

3. 备份Hexo源文件到Github

   注意：如果使用git克隆过主题，需要显示隐藏文件并把主题文件中的.git文件夹删掉（git不能嵌套上传）。

   ```
   // 进入hexo文件夹，在命令行执行以下操作：
   
   git init	// git初始化
   
   //关联github仓库
   git remote add origin https://github.com/motylcc/HexoBlog.git	
   	// 如果出现错误“fatal: remote origin already exists”，执行以下：
   	// git remote rm origin
   	// 重新执行 git remote add origin 
   	
   git add .	//添加当前文件夹下的所有文件
   git commit -m "Update Backup"	//引号中的内容为对该文件的描述
   git push origin master	//上传
   ```

## 二 异地更新博客

1. 确保当前计算机已经安装[Git](https://git-scm.com/download/win)和[Node.js](https://nodejs.org/en/) （新版的Node.js已经集成了npm）

   ```
   // 查看所需软件的版本信息
   git -v
   node -v
   npm -v
   ```

2. 配置ssh-key

3. 确保当前计算机已经安装安装Hexo

   ```
   hexo -v   // 查看本地是否已经安装Hexo
   npm install -g hexo   // Hexo安装指定
   
   // 安装其他组件
   npm install --save hexo-deployer-git   // 为了使用hexo d来部署到git上 
   npm install hexo-generator-feed --save   // 为了建立RSS订阅
   npm install hexo-generator-sitemap --save   // 为了建立站点地图
   ```

4. 拉取Github上的hexo源文件（确保最新版本）

   1. 新建一个文件夹，通过Git拉取hexo源文件。

   2. 进入新建的文件夹，命令行执行以下操作：

      ```
      git init
      git remote add origin https://github.com/motylcc/HexoBlogSrc.git
      git fetch --all
      git reset --hard origin/master
      ```

5. 编写新的文章（hexo new）

6. 推送新文章（hexo deploy）

7. 备份新的Hexo源文件到Github

   1. 进入当前的hexo文件夹，删除”.deploy_git”文件夹

   2. 命令行执行：

      ```
      hexo clean
      git add .	//添加当前文件夹下的所有文件
      git commit -m "Update Backup"	//引号中的内容为对该文件的描述
      git push origin master	//上传
      ```

      



