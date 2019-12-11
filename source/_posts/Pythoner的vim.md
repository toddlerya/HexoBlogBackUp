---
title: Pythoner的vim
date: 2017-09-08 23:08:23
lastmod:
tags: [vim, 工具]
categories:
 - 工具
typora-root-url: ..
typora-copy-images-to: ../images
---

去年申请的免费aws上个月底到期了（ss梯子没有了T_T），这个月搞了个半年免费的阿里云VPS。
国内的VPS网速果然好快，决定好好利用起来~

正好最近在学Scrapy，感觉用vim写代码，不能自动补全好难受，在公司的内网环境用vim不好折腾插件也就罢了，自己的服务器还是要搞的顺手点，磨刀不误砍柴工嘛。

为了一劳永逸，决定开个坑，维护自己的vim配置，以后换个环境就能开箱使用啦！

先上个项目地址：https://github.com/toddlerya/awesome-vim/

自动补全主要用了[`jedi-vim`](https://github.com/davidhalter/jedi-vim)插件，这插件太给力了，自动补全方法，还能提示参数，查看文档。  
还有一部分配置参考了[`k-vim-server`](https://github.com/wklken/vim-for-server)，这位同学的vim配置很给力，他还有一个完全版的vim插件配置[`k-vim`](https://github.com/wklken/k-vim)，大家可以去看看~

### 使用效果如下：
![](/images/snipaste_20170908_223050.png) 


### 部署步骤：

#### 1. 备份你的vimrc配置(如果有的话)
```
cp ~/.vimrc ~/.vimrc_bak  
```

#### 2. 安装Vundle
```
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

#### 3. 安装jedi-vim
```
pip install jedi-vim
```

#### 4. 下载并设置vimrc
```
git clone https://github.com/toddlerya/awesome-vim.git && ln -s awesome-vim/vimrc ~/.vimrc
```

#### 5. 安装Vundle插件
打开vim，运行命令
```
:PluginInstall
```
等显示`Done`后，退出vim就好啦，时间长短看网速，耐心等待。
此过程会安装这几个插件：
```
davidhalter/jedi-vim
tmhedberg/SimpylFold
vim-scripts/indentpython.vim
jnurmine/Zenburn
Lokaltog/powerline
```

#### 6. 到此就完成啦，享受生活吧！

-----

2017年09月8日 于 南京  
[Email](toddlerya@qq.com)  
[GitHub](https://github.com/toddlerya)
