---

title: 如何使用Github+Hexo搭建个人博客
date: 2019-01-01 09:27:00
tags: 
 - Github
---

[TOC]


##  1. 需要安装的软件
1. 安装 Node.js [下载Node.js](https://nodejs.org/en/)
2. 安装 Git [下载Git](https://github.com/waylau/git-for-win)
3. 安装Hexo，1和2装完成后，即可使用 npm 安装 Hexo。``$ npm install -g hexo-cli``

## 2. Git配置
1. 配置Github的账号和密码<br>
```git
$ git config --global user.name "jackychen88"
$ git config --global user.email "chenlongxxxx@126.com"
```
![Git Bash配置账号](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101093448.png)

## 3. 配置Github的repositories 
   1. 新建Github网站资源库
![创建Github的网站资源库](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101101148.png)
   2. 重命名资源库名，以后次名字可以直接通过url来访问
![配置Github的资源库名](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101101352.png)
   3. 配置Github Pages
![配置Github Pages](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101101619.png)

4. 获取网页部署url,下面步骤的配置文件\_config.yml文件中配置的网址
![取网页部署url](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101101817.png)

## 4. 设定本地电脑Blog文件源码仓库
1. 在任意磁盘处新建一个空文件夹作为我们Blog的代码仓库，例如G:\GitHub\Blog

2. 在G:\GitHub\Blog里右键选择Git Bash here进入Git 命令输入界面

3. 初始化Blog代码``$ hexo init blog``,稍微等待一会，加载速度比较慢，提示成功后会在G:\GitHub\Blog下生成几个文件夹和文件<br>
     ![hexo初始化](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101094722.png)

4. 配置Blog的配置文件\_config.yml文件，修改参数信息，以便以后每次新建内容的时候带出一些预设值，和能mapping上我们的github（**特别提醒，在每个参数的：后都要加一个空格**）

   1. 修改Blog的标题和作者信息

   ```
   # Site
   title: Jacky's Blog
   subtitle: Programer for ABAP
   description: 
   keywords:
   author: Jacky chen
   language: zh-CN
   timezone: Asia/Shanghai
   ```
   2. new_post_name每次新建时的md文件名，带出年月日一边管理

   ```
   new_post_name: :year-:month-:day-:title.md # File name of new posts
   default_layout: post
   ```
   3. deploy 用来关联hexo deploy时推送到github的资源仓链接（上面创建github的网址）

   ``` 
   deploy:
     type: git
     repo: https://github.com/Jackychen88/jackychen88.github.io.git
     branch: master
   ```
5. 发表博客文章，使用下面命令后会在G:\GitHub\Blog\source\_posts下自动生产一个md文件,生成后可自行修改md的内容，其实就是在更新Blog内容
```
$ hexo new "如何使用Github+Hexo搭建个人博客"
```
6. 编译并测试Blog是否能在本地运行,运行完下面命令后可以在http://localhost:4000/ 中查看Blog是否正常运行
```
$ hexo clean
$ hexo generate
$ hexo server
```
## 5. 推送至Github的资源仓库
1. 本地编译成功后推送到Github的资源仓库中去 （**如果在执行 hexo deploy 后,出现 error deployer not found:github 的错误``npm install hexo-deployer-git --save``**）

```
$ hexo deploy
```
![推送内容到github](https://raw.githubusercontent.com/Jackychen88/Picture/master/20190101102411.png)
至此，就可以直接用https://jackychen88.github.io 这个url访问blog了
## 5. 推送Blog源码到Github资源库
1. 为了能在不同电脑上都能方便的去发布Blog，可以将Blog的源码也推送到Github上去，主要有以下几个步骤
   1. github上创建blog源码的资源库
   2. 在本地使用``git init``来跟踪源码文件夹下的所有文件
   3. ``git add .``增加文件
   4. ``git commit -m "提交描述"``来commit修改
   5. ``git remote add blog https://github.com/Jackychen88/Blog.git``来添加访问资源库的链接，注意blog可以自己命名
   6. ``git push blog master`` 推送次文件夹内容到github对应仓库，注意若推送失败尝试先获取罪行github的资源仓内容``git pull blog master``
