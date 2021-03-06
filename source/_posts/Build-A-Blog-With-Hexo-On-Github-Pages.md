---
title: 使用Github Pages+Hexo建立博客
date: 2018-11-30 15:40:07
tags: ["Blog","Hexo"]
categories: "博客"
---

![github pages](https://i.loli.net/2018/11/30/5c00ed59d64fe.jpg)

经常看到身边越来越多人购买服务器建自己的博客，写着自己学习过程中的心得体会。而正好近日了解到Github Pages这种静态页面托管的服务，了解实现方法后，决定使用Github Pages+Hexo建立一个自己的博客。

<!--more-->

## 环境

### Github Pages

> GitHub Pages is a static site hosting service designed to host your personal, organization, or project pages directly from a GitHub repository.

Github Pages是github上提供的一项类似网页空间的功能，本意是通过在文件夹中放静态的网页文件，让别人通过官方给定的一个专属域名来访问这个网页查看你展示的一些项目。但是现在很多人用其来建立自己的个人博客。

### Hexo

> Hexo is a fast, simple and powerful blog framework. You write posts in [Markdown](http://daringfireball.net/projects/markdown/) (or other languages) and Hexo generates static files with a beautiful theme in seconds.

Hexo是一种快速、简洁高效的博客框架，用它可以使得Markdown文章在极短的时间内快速的转变为静态网页，并且可以修改配置文件和下载开源的主题方案，更加方便地自定义和管理自己的博客。

## 准备工作

由于使用的是Hexo框架，所以先要安装以下程序，搭建环境。

- Git
- node.js

## 安装步骤

### Git部分

#### 创建Github仓库

新建一个仓库，仓库的命名按照官网要求的格式为 **你的用户名.github.io**，例如github上用户名为vwonx，则仓库名为vwonx.github.io，而仓库名之后就是你的网站博客地址（当然如果嫌这个太丑，可以绑定自己购买的域名），建立成功后就可以通过vwonx.github.io直接访问。

#### 配置SSH KEY

因为我们的项目是保存在github上面的，所以当你上传文件到远程仓库的时候，肯定是需要验证你的用户名和密码的，配置了ssh后，可以免用户名和密码进行上传，当然不用担心这样会不安全，使用ssh方而比传统账户密码验证更加安全。

1. 检查本机是否以及有配置过ssh密钥

   ```bash
   $ cd ~/.ssh
   $ ls
   ```

   如果已经配置过则会存在该文件夹，并且存在 id_rsa 和 id_rsa.pub 文件，其中 id_rsa.pub是公钥，id_rsa 则是密钥。如果无该文件夹，或者文件夹下无类似配对的文件说明你还未配置过，则按接下来的步骤继续配置。

2. 生成本机的ssh密钥

   ```bash
   $ ssh-keygen -t rsa -C "邮箱地址"
   ```

   之后直接按三个回车就好，第一个回车表示在默认位置存储密钥文件，后面两个回车表示不设置密码，当然你也可以设置密码，区别就是push的时候需不需要输入该密码。

3. 配置你的 ssh 至 github

   首先复制上一步创建的公钥文件 id_rsa.pub 中的内容，登陆到你的github账号上，在页面右上角头像框的下拉选项宗进入Settings，点击左侧的 SSH and GPG kys，点击 new SSH key 进行添加。于 Title 处添加该key的名字，下面key中粘贴你刚才复制的公钥内容。

4. 测试

   在git Bash中输入以下代码。

   ```bash
   $ ssh -T git@github.com
   ```

   会出现一段警告，键入yes即可，当你看到类似下方的提示的时候，说明配置成功。

   ```bash
   "Hi username! You've successfully authenticated, but GitHub does not provide shell access."
   ```

   注意如果你之前生成 ssh 密钥的时候有键入密码，那么当你输入yes后会要求你输入密码，当你输入正确的密码后才会出现上面的提示。

   注意：配置完成后，当你需要clone你的项目的时候，要将其https的方式改为ssh方式。

#### 配置Git信息

配置Git你的提交的账号信息和邮箱信息，下面的配置是全局配置。

```bash
$ git config --global user.name "username"    // 你的github用户名
$ git config --global user.email "xxxx@xx.xx" // 填写你的github注册邮箱
```

注意邮箱要使用你的github上的邮箱，不让push后你的账号首页不会有绿色的方格。

### Hexo部分

#### 安装Hexo

```bash
$ npm install -g hexo
```

由于所用的系统是Ubuntu，并且node.js采取压缩包的方式进行安装，所以可能会找不到上面的命令，解决方法是当安装完成后可以将node.js的bin目录写入PATH，就可以直接使用上面语句安装hexo。

#### Hexo初始化

随便新建一个文件夹，这里假设该文件夹名字为test，键入以下的代码。

```bash
$ cd test
$ hexo init
```

初始化过程由于要下载一些东西，会比较慢，要耐心等待。

clone之前创建的远程仓库的项目至本地，复制之前test文件夹下全部文件到本地仓库，注意不要漏掉.gitigonre文件，可以看出Hexo初始化过程就有产生该文件，说明原意就是要我们使用git来进行管理。

#### Hexo的初次运行

Hexo提供本地的预览服务，键入以下代码进行生成和启动服务。

```bash
$ hexo g && hexo s
```

注意：该命令要在经过hexo初始化后的文件夹下进行。

之后就可以通过本地地址的4000端口进行访问。

```html
http://localhost:4000/
```

## 主题安装

### 主题推荐

个人比较喜欢的是next主题，可以从hexo的官网上查看并选择自己喜欢的主题进行下载。

### 安装方法

主题一般会提供文档指导用户进行安装，下面提供next主题的安装方法。

进入博客的本地仓库，键入以下代码。

```bash
$ cd hexo
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

 修改配置文件_config.yml，找到theme项，修改为next，键入以下命令重新生成网页。

```bash
$ hexo clean && hexo g && hexo s --debug
```

完成安装，重新访问网页即可看到效果。

### 新增分类和标签页面

```bash
$ cd your-hexo-site
$ hexo new page tags
$ hexo new page categories
```

键入以上部分后，会在your-hexo-site/source文件夹中生成categories和tags文件夹，里面各有一个index.md，打开该文件，在categories文件夹中的index.md文件最后加入一行

```markdown
type: "categories"
```

同理，在tags文件夹中的index.md文件最后加入一行

```markdown
type: "tags"
```

如果不修改以上文件，则在分类和标签页面看不到任何内容。

### 常用插件推荐

1. 文章字数和时间的统计：https://github.com/theme-next/hexo-symbols-count-time
2. 本地文章搜索：https://github.com/theme-next/hexo-generator-searchdb
3. 首页隐藏不想展示的文章：https://github.com/Jamling/hexo-generator-index2

## 利用分支便携管理Hexo

我这里参考了 [CrazyMilk](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more) 的博客管理方法，在仓库完成后建立了另一条hexo的分支，该分支用于管理hexo的源文件，master则用于管理hexo生成的静态网页。这样在更换电脑的情况下也可以clone远程仓库的内容进行博客文章的书写，并且起到了一个备份的作用。

之后管理和写作只要在将hexo的源文件上传到hexo分支即可，因当你配置完成hexo后，使用命令即可上传生成的静态网页到master分支，具体配置方法见下文。

### 建立一个新的分支用来管理

```bash
// git初始化
git init
// 与远程仓库建立连接
git remote add origin git@github.com:/用户名/仓库名.git
// 新建hexo分支用来管理
git checkout -b hexo
// 上传到远程仓库
git add .
git commit -m ""
git push origin hexo
```

### Hexo 的 Github 上传

1. 打开配置文件_config.yml，拉至最后，修改deploy项为如下，uesrname为你自己的github名

   ```
   deploy:
     type: git
     repository: git@github.com:username/username.github.io.git 
     branch: master
   ```

2. 安装上传的插件

   ```bash
   $ npm install hexo-deployer-git --save
   ```

3. 之后只要键入以下代码即可将生成的静态网页传至github

   ```bash
   $ hexo d
   ```

   注意：当你上传完成后会发现github上面master分支内容都被清空了，如果你需要保留如README之类的文件，将其放到source文件夹下，因为生成静态页面的时候，source文件夹下的东西都会复制一份到上传内容的public文件夹。

### 新环境下配置

1. 安装node.js和git。
2. 克隆远程仓库到本地，并进入本地仓库文件夹。
3. 安传hexo并安装依赖库

```bash
$ npm install hexo
$ npm install
// 拉取最新的文件到本地
$ git pull
```

4. 重新安装主题
5. 写完文章后

```bash
$ git add .
$ git commit -m "descrption"
$ git push origin hexo
$ hexo clean && hexo g && hexo d
```




## Hexo的常用命令

```bash
npm install hexo -g #安装
npm update hexo -g  #升级
hexo init           #初始化
hexo n "title"      #新建名字为title的文章
hexo g              #生成静态网页
hexo s              #启动预览服务
hexo d              #部署
hexo clean          #清除缓存
```

## 建站布局参考

博客建立的时候，参考了许多人的页面布局，感谢以下博客给我提供的借鉴和参考

- [Cherry's Blog](http://cherryblog.site/)
- [next 主题美化](https://blog.csdn.net/qq_33699981/article/details/72716951)
