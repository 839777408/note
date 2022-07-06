---
title: Git
tags:
  - Git
  - GitHub
categories:
  - Git
top_img: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp2.jpg'
cover: 'https://cdn.jsdelivr.net/gh/839777408/tupian/img/wp2.jpg'
abbrlink: 629e
date: 2020-10-8 15:51:41
updated: 2020-10-8 15:51:46
---

>参考视频：【尚硅谷】Git与GitHub基础全套完整版教程 https://www.bilibili.com/video/BV1pW411A7a5

# 版本控制

**版本控制** ：工程设计领域中使用版本控制管理工程蓝图的设计过程。在 IT 开发过程中也可以使用版本控制思想管理代码的版本迭代。

---

**版本控制工具应该具备的功能：**

>- **协同修改：**多人并行不悖的修改服务器端的同一个文件。 
>
>- **数据备份：**不仅保存目录和文件的当前状态，还能够保存每一个提交过的历史状态。 
>
>- **版本管理：**在保存每一个版本的文件信息的时候要做到不保存重复数据，以节约存储空间，提高运行效率。这方面 SVN 采用的是**增量式管理**的方式，而 Git 采取了**文件系统快照**的方式。  
>
>- **权限控制：**对团队中参与开发的人员进行权限控制。 对团队外开发者贡献的代码进行审核（Git 独有）。 
>
>- **历史记录：**查看修改人、修改时间、修改内容、日志信息。 将本地文件恢复到某一个历史状态。 
>
>- **分支管理：**允许开发团队在工作过程中多条生产线同时推进任务，进一步提高效率。

---

**版本控制工具**

集中式版本控制工具： CVS、**SVN**、VSS……

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/微信截图_20201005192430.png)

分布式版本控制工具： **Git**、Mercurial、Bazaar、Darcs……

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/微信截图_20201005192508.png)



# Git

## 了解和准备

### 简史

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201006143119.jpg)

### 优势

- 大部分操作在本地完成，不需要联网

- 完整性保证

- 尽可能添加数据而不是删除或修改数据

- 分支操作非常快捷流畅

- 与 Linux 命令全面兼容

### Git结构

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20201006144816.png)

### 代码托管中心

代码托管中心的任务：维护远程库 

局域网环境下： GitLab 服务器 

外网环境下 ： GitHub 、码云 

### 跨团队协作

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20201006174847.png)

### 安装

1. 安装到一个非中文没有空格的目录下
2. 使用Vim编辑器，选择【Use  Vim(the  ubiquitous  text  editor)  as  Git's  default  editor】
3. 完全不修改PATH环境变量，仅在Git Bash中使用Git，选择【Use  Git  from  Git  Bah  only】
4. 选择【Use  the  OpenSSL  library 】
5. 行末换行符转换方式，使用默认值，选择【Checkout  Windows-style , commit  Unix-style  line  endings】
6. 执行Git命令的默认终端，使用默认值，选择【Use  MinTTY(the  default  terminal  of  MSYS2)】
7. 选择【Enable  file  system  caching !】和【Enable  Git  Credential  Manager】，最后点击 install



## 命令行基本操作

### 本地库初始化

**命令：**`git init`

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/202010061755.jpg)

注意： .git 目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡乱修改。

---

### 设置签名 

>形式：
>
>- 用户名：Nan
>
>- Email 地址：839777408@qq.com
>
>作用：区分不同开发人员的身份 
>
>辨析：这里设置的签名和登录远程库(代码托管中心)的账号、密码没有任何关系。

**命令：**

**项目级别/仓库级别：**仅在当前本地库范围内有效 

- `git config user.name Nan`

- `git config user.email 839777408@qq.com`

信息保存位置： .git 目录下的 config 文件中

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007103825.png)

**系统用户级别：**登录当前操作系统的用户范围 

- `git config --global user.name 阿楠`

- `git config --global user.email 839777408@qq.com`

信息保存位置：~/.gitconfig （根目录下的 .gitconfig文件中）

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007160012.png)

> 级别优先级：
>
> - 就近原则：项目级别优先于系统用户级别，二者都有时采用项目级别的签名 
> - 如果只有系统用户级别的签名，就以系统用户级别的签名为准 
> - 二者都没有不允许

---

### **状态查看**

查看工作区、暂存区状态

`git status`

>modified为红色，说明工作区和暂存区内容不一致
>
>modified为绿色，说明暂存区和本地库内容不一致

---

### 添加到暂存区和取消添加到暂存区

 将工作区的“新建/修改”添加到暂存区：

`git add [file name]`

取消添加到暂存区命令：

`git rm --cached [file name]`

---

### 提交到本地库

将暂存区的内容提交到本地库：

`git commit -m "commit message" [file name]`

---

### 查看历史版本记录

`git log`

`git log --pretty=oneline`

`git log --oneline`

`git reflog` 

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007160950.png)

>- 多屏显示控制方式： 
>  - 空格向下翻页 
>  - b 向上翻页 
>  - q 退出
>
>- HEAD@{移动到当前版本需要多少步} 
>
>- vim模式（显示行号）： `:set nu`

---

### 版本的前进和后退

**本质就是HEAD指针的移动**

- 基于索引值操作 【推荐】
  - `git reset --hard [局部索引值]`
  - 例如：git reset --hard 31364f7 

- 使用^符号：只能后退 
  - `git reset --hard HEAD^`

  - 注：一个^表示后退一步，n 个表示后退 n 步 

- 使用~符号：只能后退 
  - `git reset --hard HEAD~n` 
  - 注：表示后退 n 步

---

### reset 命令的三个参数对比

- --soft 参数 
  - 仅仅在本地库移动 HEAD 指针 
  - 暂存区和工作区没有修改

- --mixed 参数 
  - 在本地库移动 HEAD 指针 
  - 重置暂存区 ，使其与本地库一致

- --hard 参数 [常用]
  - 在本地库移动 HEAD 指针 
  - 重置暂存区，使其与本地库一致
  - 重置工作区，使其与本地库一致

---

### 比较文件差异

- 将工作区中的文件和暂存区进行比较：
  - `git diff [文件名] `

- 将工作区中的文件和本地库历史记录比较：
  - `git diff [本地库中历史版本] [文件名]` 

- 不带文件名比较多个文件



## 分支管理

### 什么是分支 

在版本控制过程中，使用多条线同时推进多个任务。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007172431.png)

**好处：**

- 同时并行推进多个功能开发，提高开发效率 。

- 各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响。失败的分支删除重新开始即可。

---

### 分支操作

**创建分支：** 

`git branch [分支名] `

**查看分支 :**

`git branch -v`

**切换分支 ：**

`git checkout [分支名]`

**合并分支 :**

1. 切换到接受修改的分支（被合并，增加新内容）上 

   `git checkout [被合并分支名]`

2. 执行 merge 命令 

   `git merge [有新内容分支名]`

**解决冲突 ：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007180054.png)

1. 编辑文件，删除特殊符号 

2. 把文件修改到满意的程度，保存退出 

3. git add [文件名] 

4. git commit -m "日志信息" 

**注意：此时 commit 一定不能带具体文件名。**



## 基本原理

### 哈希

哈希是一个系列的加密算法，各个不同的哈希算法虽然加密强度不同，但是有以下几个共同点： 

①不管输入数据的数据量有多大，输入同一个哈希算法，得到的加密结果长度固定。 

②哈希算法确定，输入数据确定，输出数据能够保证不变 

③哈希算法确定，输入数据有变化，输出数据一定有变化，而且通常变化很大 

④哈希算法不可逆 

**Git 底层采用的是 SHA-1 算法。** 

哈希算法可以被用来验证文件。原理如下图所示：

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201007193359.png)

Git 就是靠这种机制来从根本上保证数据完整性的。 



### Git保存版本的机制

**集中式版本控制工具的文件管理机制** 

以文件变更列表的方式存储信息。这类系统将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107193820.png)

---

**Git** **的文件管理机制** 

Git 把数据看作是小型文件系统的一组快照。每次提交更新时 Git 都会对当前的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。所以 Git 的工作方式可以称之为快照流。

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107193900.png)



### Git文件管理机制细节

- Git 的”提交对象“

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107195524.png)



- 提交对象及其父对象形成的链条

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107195538.png)



### Git分支管理机制

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107200212.png)

---

**由master分支切换到testing分支：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107200223.png)

---

**testing分支提交到本地库：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107200235.png)

---

**切换回master分支：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107200251.png)

---

**master分支提交到本地库：**

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/2020107200304.png)



## Github

### 创建远程仓库

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201008140047.png)

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201008140332.png)

### 查看远程仓库地址

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201008140720.png)

### 查看当前所有远程地址别名

`git remote -v`

### 创建远程库地址别名

`git remote add [别名] [远程地址]`

### 推送

`git push [别名] [分支名]`

### 克隆

`git clone <-b 分支名> [远程地址] <本地目录名>`

<>表示可选，默认克隆master分支，本地目录名称与版本库同名。

> 效果：完整的把远程库下载到本地，创建 origin 远程地址别名，初始化本地库

### 团队成员邀请

![](https://cdn.jsdelivr.net/gh/839777408/tupian/blog/20201008143446.png)

### 拉取

- pull=fetch+merge 

- git fetch [远程库地址别名] [远程分支名] 

- git merge [远程库地址别名/远程分支名] 

- git pull [远程库地址别名] [远程分支名] 

### 解决冲突

要点：如果不是基于 GitHub 远程库的最新版所做的修改，不能推送，必须先拉取。 拉取下来后如果进入冲突状态，则按照“分支冲突解决”操作解决即可。 

类比：

债权人：老王 

债务人：小刘 

老王说：10 天后归还。小刘接受，双方达成一致。 

老王媳妇说：5 天后归还。小刘不能接受。老王媳妇需要找老王确认没有冲突后再执行。

### SSH免密登录

- 进入当前用户的家目录 

  `$ cd ~`

- 删除.ssh 目录 

  `$ rm -rvf .ssh`

- 运行命令生成.ssh 密钥目录 

  `$ ssh-keygen -t rsa -C 839777408@qq.com` 

- 进入.ssh 目录查看文件列表 

  `$ cd .ssh` 

  `$ ls -lF `

- 查看 id_rsa.pub 文件内容

  `$ cat id_rsa.pub` 

- 复制 id_rsa.pub 文件内容，登录 GitHub，点击用户头像→Settings→SSH and GPG keys→New SSH Key 

- 输入复制的密钥信息 