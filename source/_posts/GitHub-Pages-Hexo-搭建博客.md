---
title: GitHub Pages + Hexo 搭建博客
date: 2016-10-18 20:13:19
comments: true
tags: 
- GitHub Pages
- GitHub
- Hexo
- Blog


---

## 前言

一直想自己搭建一个Blog，刚好前几天接触到hexo，对其方便及强大的功能感到惊叹，简单几步就能搭建出一个简洁漂亮的空间，而且文档非常完善，主题也很多，完全满足要求。即便如此，在搭建过程中也遇到了一些问题，故写一个教程。
**注意下面命令里的”$”都不用自己输入**
**下文提到的运行cmd，意思都是在blog文件夹里右键git base here，或者cmd之后cd到blog文件夹内**

<!-- more -->

## 安装

1. 首先需要安装[Git](https://git-for-windows.github.io/)
2. 然后安装[Node.js](https://nodejs.org/en/)，当前安装的是6.8.1版
3. 验证Node.js是否安装成功，运行cmd，输入”node -v”，如果输出版本号，则安装成功
4. 安装hexo，运行cmd输入npm install -g hexo-cli，至此hexo就已经安装完毕了

## 搭建Blog

**由于hexo在GitHub Pages上保存的是html文件，如果丢失源文件(.md)的话后期编辑会非常麻烦，因此在搭建初始就应该考虑到将源文件也保存到GitHub上。**
在适当的位置创建空文件夹，进入文件夹内，鼠标在空白处右键，选择Git Bash，使用以下命令安装hexo到该文件夹内

```base
$ hexo init
$ npm install
```

这样hexo的框架就已经搭建好了，但为了之后维护方便，需要进行这几步操作。

1. 在GitHub上创建新的Repositories，注意repo的名字，一定要以.github.io结尾，创建完毕应该是一个空项目，啥文件都没有的

   {% asset_img new-repo.png 创建新Repo %}

2. 在blog文件夹执行git init命令，由于我这里使用的是tortoisegit，直接右键文件夹选择”创建版本库”即可
   {% asset_img git-init.png 创建版本库 %}

3. 将文件夹与项目连接起来，同样以tortoisegit的操作为准，右键文件夹->设置->git->远端，填入项目的地址即可(提示是否获取origin的分支，随意了，反正项目上暂时也没有分支)
   {% asset_img git-ori.png 连接git %}

4. 重点在这里，在本地创建分支master(主分支名字建议不要改)和hexo(名字随意)，将master分支的文件夹全部删除，操作完毕后push到github上

5. 然后切换到hexo分支，编辑_config.yml文件
   repo后面的是github上的项目地址
   branch则是主分支名字
   注意”:”(冒号)后面有一空格，至于有没空格影响不影响(懒得试了，网上搜集的资料说会导致失败，懒得试，按道理应该没问题的)

   ```base
   deploy:
     type: git
     repo: git@github.com:pye52/pye52.github.io.git
     branch: master
   ```

6. 运行cmd，执行以下命令

   ```base
   $ hexo g
   $ hexo d
   ```

这样blog就搭建完毕了

## 安装next主题

运行cmd，然后输入以下命令(也可以下载next的压缩包直接复制到blog的themes文件夹里)

```base
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

打开_config.yml文件，找到”themes”，将后面的替换为next，如下

```base
theme: next
```

保存，运行cmd，执行以下命令

```base
hexo clean
hexo d
```

然后登录你的github pages网站就可以看到主题用上去了
至于如何配置next主题，可参考[next](http://theme-next.iissnan.com/getting-started.html)

## 附

[Hexo文档](https://hexo.io/zh-cn/docs/index.html)