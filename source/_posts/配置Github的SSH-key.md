---
title: 配置Github的SSH key
description: 配置Github的SSH key
date: 2024-02-19 16:42:09
tags: [Github, Hexo]
---

**SSH** (Secure Shell，安全外壳协议) 可以连接远程服务器和服务并向它们验证。利用 SSH 密钥可以连接到 GitHub，而无需在每次访问时都提供用户名和 personal access token。还可以使用 SSH 密钥对提交进行签名。

### 配置步骤：

1. 确保本地电脑已经安装[Git](https://git-scm.com/download/win)。

2. 检查SSH keys。(需要寻找一对以 id_dsa 或 id_rsa 命名的文件，其中一个带有 .pub 扩展名。 .pub 文件是公钥)

   ```
   // 运行Git Bash 然后输入
   ls -al ~/.ssh
   
   // 如果没有类似以下输出结果，则需要进入下一步生成SSH Key
   // -rw-r--r-- 1 UserName 197121 2610 Feb 18 11:28 id_rsa
   // -rw-r--r-- 1 UserName 197121  575 Feb 18 11:28 id_rsa.pub
   ```

3. 如果本地不存在SSH Key，需要[生成SSH Key](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。

   ```
   // 运行Git Bash 然后输入生成指令
   ssh-keygen -t rsa -C "jxzhangms@hotmail.com"
   // 最后一项参数为GitHub的邮件地址，这将以提供的电子邮件为标签创建新的SSH Key
   
   // 提示输入保存地址时，使用默认位置（直接按回车），提示如下：
   // Enter file in which to save the key (/Users/you/.ssh/id_rsa):[直接输入回车]
   
   // 提示输入密码（可以直接按回车跳过），如下：
   // Enter passphrase (empty for no passphrase): [Type a passphrase]
   // Enter same passphrase again: [Type passphrase again]
   ```

4. [将 SSH 密钥添加至 GitHub 帐户](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

   1. 复制SSH密钥数据：

      密钥文件路径（默认）为 "%UserProfile%/.ssh/id_rsa.pub"，可以通过指令将SSH密钥添加到剪贴板。

      ```
      // 运行Git Bash 然后输入
      clip < ~/.ssh/id_rsa.pub
      // 如果剪切板没有数据，检查SSH密钥的文件路径是否有效
      ```

   2. 单击Github页面右上角的个人头像，找到 "Setting"。

   3. 在左边栏的“Access”中，选择“SSH and GPG keys”。

   4. 点击“New SSH 密钥” ：

      1. 在 "Title"字段中，为新密钥添加描述性标签（可随意填写）。
      2. 在 "Key Type" 中选择"Authentication Key"（默认选项）。
      3. 在“密钥”字段中，粘贴公钥，点击“Add SSH Key”。

   

   

