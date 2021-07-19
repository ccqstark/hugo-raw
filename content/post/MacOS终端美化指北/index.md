---
title: "MacOS终端美化指北"
date: 2021-01-06T02:07:00+08:00
draft: true
image: Mojave-desktop.jpg
slug: "mac_terminal"
tags:
    - MacOS
categories:
    - 其它技术
---

终于从Windows转到心心念念的MacOS上进行开发，虽然是黑苹果但是软件层面上没有太大的区别，程序员还是得用mac啊这终端上真的比windows好用无数倍，那终端到手后还是要折腾美化的，那就开始吧。
先看下我，还可以吧？
![1](https://static01.imgkr.com/temp/c0757b90608b4f89ac43921f28ecedd5.jpg)  

## 准备工作
先保证自己下载`homebrew`和`wget`，安装软件或下载包很多情况下要用到它们，特别homebrew是mac下最好用的包管理器一定要有。[下载方法](https://zhuanlan.zhihu.com/p/90508170)网上也很多的，建议先下homebrew再用它下wget。

## iTerm2
首先是下载第三方终端`iTerm2`，mac自带的终端用的比较少，大家用的最多还是这个。
[官网下载](https://iterm2.com/)

## zsh
`zsh`是shell的一种，mac默认的shell是`bash`，一般来说我们也是用zsh比较多，因为命令更多更好用。  
### 下载zsh
```shell
brew install zsh
```
### 切换shell为zsh
```shell
# 查看当前使用的shell
echo $SHELL
# 切换为zsh
chsh -s /bin/zsh
```
运行完上面命令后重启一下即可

## oh-my-zsh
`oh-my-zsh`用于美化终端，可以让你拥有很多好看的主题。
### 安装
```shell
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
```
```
sh install.sh
```
运行上面的命令来下载安装脚本并运行脚本，成功后会有如下画面  
![2](https://static01.imgkr.com/temp/f72289ad7de9432aa9e9a583ac9fe56c.jpg)

## 更换主题
oh-my-zsh有很多默认的主题，可以在`~/.zshrc`中修改`ZSH_THEME`来切换不同主题。
这里我推荐`powerlever10k`，它集合了很多不同主题风格的样式，支持自定义，如果默认主题中没有你满意的那推荐就用它。下面就讲`powerlevel10k`的安装方法。
### 下载
```
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
### 安装所需字体
```shell
# 安装 nerd-font 字体
brew tap homebrew/cask-fonts

# 其他所需字体
cd ~
git clone https://github.com/powerline/fonts.git --depth=1

# 到目录下执行安装脚本
cd fonts
./install.sh

# 删除刚刚下载的
cd ..
rm -rf fonts
```
### 配置
```shell
vim ~/.zshrc
```
进入zsh配置文件中修改并增加
```shell
ZSH_THEME = "powerlevel10k/powerlevel10k"
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```
之后启动向导
```shell
p10k configure
```
可以用下面命令查看颜色代号
```shell
for i in {0..255}; do print -Pn "%K{$i}  %k%F{$i}${(l:3::0:)i}%f " ${${(M)$((i%6)):#3}:+$'\n'}; done
```
接下来需要下载一些必要的字体或样式，在此之前需要先改host以正常下载
下载软件[SwitchHosts!](https://oldj.github.io/SwitchHosts/)
之后如图加上
```shell
199.232.68.133 raw.githubusercontent.com
199.232.68.133 user-images.githubusercontent.com
199.232.68.133 avatars2.githubusercontent.com
199.232.68.133 avatars1.githubusercontent.com
```
![3](https://static01.imgkr.com/temp/6e6268a51173424385b94999d2ce0d89.jpg)   
开启`My hosts`后重启终端，就会自动提示下载所需字体，耐心等待它下载完（有点慢）后，就可以根据引导一步步自定义属于自己的主题了！从图标到字体颜色风格到显示信息都可以自定义！

## 背景透明+毛玻璃
打开iTerm2的偏好设置按下图即可调节背景透明和毛玻璃，下面还可以设置默认窗口大小。  
![4](https://static01.imgkr.com/temp/4499e7e7658a44b1bbe30990cd612e5b.jpg)

## 快捷键唤醒
如何让切出终端更快捷？可以设置`Hotkey`
按下图操作  
![5](https://static01.imgkr.com/temp/25d34480ffb441d68d227195a335d59f.jpg)
![6](https://static01.imgkr.com/temp/ce75a9da5d4a4c369990af537139598e.jpg)

## 尾声
至此，一个好用又好看的mac终端基本配置完毕啦  
其他有趣的可以看下[这篇博客](https://sspai.com/post/59666)讲的
