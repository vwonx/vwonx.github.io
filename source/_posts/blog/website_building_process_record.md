---
title: Hexo + Github Actions + Github图床建站过程记录
subtitle: 利用仓库分支和Github Actions实现Hexo博客的方便管理，同时使用Github仓库作为图床以供文章图片使用。
catalog: true
tags:
  - Hexo
  - 教程
categories:
  - Hexo
abbrlink: 9f560b87
date: 2022-05-05 13:45:20
---

该类建站教程，在许多博客上均有记录，按照自己的需求的不同，可能会产生一些不同的地方，但大致的流程一致。

对于我个人来说，所建的站需要满足以下三点需求：
1. 源码和生成的静态网页以不同分支/仓库进行分别管理
2. 能够实现CI/CD
3. 能够有稳定的图床，供博客文章图片使用

对于需求1，由于分库管理个人不是很喜欢，所以仅采用了不同分支进行管理。（见Hexo + Github Pages 建站过程）
对于需求2，目前能够和Github共同使用的第三方平台有许多，例如Hexo的官方中文文档里使用了 **Travis CI**。但Github官方就有提供有一个CI/CD的方式，即Github Actions，因此可直接使用其实现功能。（见Github Actions 配置）
对于需求3，Github仓库本身就可以作为一个图床使用，但是用这种方式直接作为图床会带来容量和速度的问题。好在这些问题目前还不算大问题，因此先这样使用。

# Hexo + Github Pages 建站过程
1. 基本的本机Git和Github绑定，如Git安装、密钥生成配置等，这类教程很多，使用任意检索网站进行搜索即可。主要就是保证仓库能够进行正常的 **pull/push** 操作，而配置密钥则可以免密码方便地进行操作。
2. Github上建库，有两个注意点。其一仓库名需要取 **Github用户名.github.io**，这样之后可以直接使用对应的仓库名访问博客，否则需要加上二级域名 **Github用户名.github.io/仓库名** 才能访问。其二不能设置为私库，私库需要 **Github Pro** 才能设置为Github Pages。
![1](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/1.png)
![2](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/2.png)
3. 本地创建文件夹，以blog命名为例。
```bash
mkdir blog
hexo init blog # hexo初始化文件夹
git init blog  # git初始化文件夹
```
4. 安装主题（可选），以butterfly主题为例。
```bash
cd blog
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
vim _config.yml # 修改对应主题配置为butterfly
npm install hexo-renderer-pug hexo-renderer-stylus --save # 安装主题必要插件
```
5. 将本地仓库和Github仓库相关联，当你创建好一个空的Github仓库时，仓库主页有对应代码，按顺序执行就行。
6. 由于butterfly主题是另一个仓库，所以直接上传会发现Github上面对应的themes文件夹是空的。因此进行子仓库绑定。（可选）
```bash
git submodule add -b master git@github.com:jerryc127/hexo-theme-butterfly.git themes/butterfly
```
7. 提交commit，推送远程仓库。
注意此时虽然推送到了仓库，但是并没有生成静态网页文件，所以博客是无法访问的。需要通过接下来的Github Actions的配置执行完工作流，才会在仓库的另一个分支里生成静态网页。

# Github Actions 配置
1. Github的账户设置中新建一个token，这个token能够使得Github Actions 获取控制仓库的权限。
![3](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/3.png)
2. token的权限选择，只需要第一项“repo”，因为只需要仓库的控制权。之后会跳回原先token的总页面，这时能看到刚才新建的token，点击复制按钮/或进行复制（这个token离开页面后就不可见了，所以务必确认已经复制，否则要重新按步骤1进行操作）。
![4](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/4.png)
![5](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/5.png)
3. 在博客所在的仓库设置里，将复制的token配置为Actions Secret。配置过程中所取的名字后面会用到，所以也需要记住，这里以 **ACCESS_TOKEN** 为例。
![6](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/6.png)
![7](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/7.png)
4. 点击仓库里的Actions标签，选择Simple workflow，在生成的yml文件里，配置如下代码。代码都很简单，很容易看懂。基本上主要的行也均有注释，按照注释结合自己的实际情况进行修改。
```yml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  # 类似数据库的触发器，当下述行为发生时候，执行下述脚本。这里下面的代码是指当master分支有push操作时候触发
  push:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      - name: Checkout # 步骤名字
        uses: actions/checkout@v2 # 使用的插件，其实就是对应Github上的仓库地址，可以直接取访问，查看具体配置。如对于该插件，地址为github.com/actions/checkout
        with:
          ref: master # 切换到master分支
          submodules: true
          fetch-depth: 0

      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: '16.x' # 使用16版本的nodejs

      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install # 缓存和安装依赖包

      - name: Build
        run: npm run build # 执行目录下package.json里的build脚本命令
        
      - name: Deploy
        # You may pin to the exact commit or the version.
        uses: peaceiris/actions-gh-pages@v3
        with:
          # deploy_key: ${{ secrets.DEPLOY_KEY }}
          github_token: ${{ secrets.ACCESS_TOKEN }} # 这里就是上述配置Actions Secret时候设置的名字
          publish_branch: gh-pages # 生成的静态网页文件放在哪个分支里
          publish_dir: public
          disable_nojekyll: true

      - name: Get the output
        run: |
          echo "${{ steps.deploy.outputs.notify }}"
```
5. 配置好了，提交。会生成一个commit，之后会发现脚本开始执行了。但是博客还是不能访问，这是由于没有配置Github Pages使用哪个分支，默认是master，所以无法部署。在仓库的设置里进行更改后，就能正常访问博客了。至此，博客建站完成。**后续只需要在本地仓库写博客，然后推送master分支，即能自动在gh-pages分支生成静态文件，进行博客部署操作。**
![8](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/8.png) 

# Github图床
写博客的过程中，不可避免的就是需要在文章里插入图片。图片虽然可以直接放在博客的仓库里，但是随着图片数量的增加，会明显增大博客仓库的体积，也不易于管理。

一种简单的解决方案就是找一个第三方图床平台，保存需要上传的图片。目前市面上存在有免费/收费两种图床。免费的虽然可以用，但是之前发生过免费图床平台图片失效，而我本地又没有保存副本，导致直接丢失了图片。因此，找寻一个免费稳定的图床，也是一个建站过程中比较重要的步骤。

总的来说，我寻求的图床需要具备有以下特点：
1. 稳定，图片不丢失，易于管理
2. 访问速度快
3. 有较大的容量

鉴于上述分析，最后选用了Github仓库作为图床。很明显，使用自己的Github仓库作为图床，可以完美的对应上述的第一个特点。但是对于后两个特点，Github仓库又难以较好地匹配。好在对于特点2，可以使用免费的CDN进行加速，实际测试后，速度完全可以接受。而对于特点3，Github仓库有1G的容量限制，好在博客使用的图片尺寸均不大，而且经过压缩后基本都在100k以内，暂时可以满足个人需求。

确定使用Github仓库作为图床后，如何方便地上传图片？参考其他博客后，也决定采用PicGo。

## 配置PicGo
1. Github上建立一个仓库，本地不用建库。注意库必须是public，否则不能访问。
2. 建立一个token（见Github Actions 配置中的第1步至第3步）。
3. 在PicGo中进行配置，按照选项名字进行配置就行。这里的指定存储路径就是就是指定图片上传后放在哪个子文件夹里，如：配置写“a/”，则上传图片后会保存在仓库的a文件夹下。
![9](https://cdn.jsdelivr.net/gh/vwonx/blog-imgs/website-building-process-record/9.png)

## 配置CDN加速
其实就是上图选项中的自定义域名，配置为 https://cdn.jsdelivr.net/gh/Github用户名/图片仓库名 。

## 图片压缩
可以直接使用butterfly主题博客推荐的[在线网站](https://tinypng.com)，免费版有单文件大小限制，但是一般都够用了。