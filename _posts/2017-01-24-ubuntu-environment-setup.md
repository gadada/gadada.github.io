---
title: "我的Ubuntu环境"
tags:
  - Step by step
---

2003年在浙大读书的时候，有一门选修课是关于Linux操作系统使用的，
上课的时候用的是RedHat 9，想来和Linux接触也有13个年头了。

本科4年时间，基本不玩游戏，
除了上课还有课后学习，课余的时间大多用来折腾系统了，
当时主流应该还是RedHat系列，Debian正有赶超RedHat的趋势，
另外，当时像Ubuntu、还有Fedora也都还没有出现。
基本上把当时的几个Linux/Unix系统都折腾了个遍，
包括RedHat，Debian，Arch，Gentoo，FreeBSD，...等等。
从内核编译到桌面配置，从系统美化到精简优化，玩得不亦乐乎。
周末的时候，从早上起来到晚上熄灯，可以忘我的玩各种配置。
那时候许多系统都还不成熟，经常有一堆的bug，虚拟机也没有现在这么好用，
经常配置着配置着，整个系统就挂了。
所以那时候，一天重新装几次系统也是经常有的事情，
现在回想起来都觉得不可思议。

现在Ubuntu依旧是我的工作、生活中使用的主力系统。
但现在的系统远比当时的成熟稳定，软件基本也比以前好用太多，
所以也有很长时间没有去折腾系统了。
现在基本上是需求导向的，
在美观度可接受的前提下，把系统配置成自己习惯的方式就拉倒了。
所以现在使用Ubuntu基本上就是简单美化一下系统，
然后执行一遍积累下来的配置文件，就用着了，
但像什么内核编译啊，系统精简啊之类的都已经好久没有玩过的东西了。

这篇博文主要是想分享一下，我从重新安装系统完毕后到开始正常使用，
都会干些啥、都是怎么干的，我自己用着感觉还是挺方便的。
得益于Github，我现在基本上是吧自己的配置文件都放在了[Github](https://github.com/huajianmao/config)上了。
如果有需要，欢迎[fork](https://github.com/huajianmao/config)
或[提交issue](https://github.com/huajianmao/config/issues/new)。

{% include toc %}

## 系统美化
相对以前的Linux系统来说，现在的Ubuntu也好，Fedora也好，其实已经相当好看了。
但就色调搭配来说，个人不是特别的喜欢Ubuntu默认的紫色系配色。
都说，Mac是面向程序员最好的平台，
虽然平时Mac、Linux都用，
但个人认为，Mac主要吸引我的地方是它的UI、图标以及色彩搭配，
而软件使用的便捷性、系统可定制性、程序员友好性等方面，
我觉得Linux其实并不输Mac。

MacBuntu](http://www.noobslab.com/p/themes-icons.html)主题就是面向Linux系统，高仿Mac界面的一个主题，
它通过替换系统的图标、桌面主题、菜单栏样式等方法，
把Ubuntu在界面上改造成一个类Mac的系统。
个人非常喜欢MacBuntu在Ubuntu上的美化。所以一直用着它也有好长一段时间了。
[noobslab.com](http://www.noobslab.com/2016/04/macbuntu-1604-transformation-pack-for.html)上有详细的配置过程，以下是精简后的脚本：

``` shell
mkdir -p ~/tmp/macbuntu
cd ~/tmp/macbuntu

# 1. Download MacBuntu OS Wallpapers and extract to pictures directory.
wget http://drive.noobslab.com/data/Mac/MacBuntu-Wallpapers.zip
pushd .; mkdir ~/Pictures/macbuntu; cd ~/Pictures/macbuntu; unzip ~/tmp/macbuntu/MacBuntu-Wallpapers.zip; popd
echo "You should set the background in Appearance."

# 2. Add macbuntu package repo to apt
sudo add-apt-repository ppa:noobslab/macbuntu
sudo apt-get update

# 3. Install MacBuntu OS Y Theme, Icons and cursors
sudo apt-get install macbuntu-os-icons-lts-v8 macbuntu-os-ithemes-lts-v8
echo "After installation choose theme, icons and mac cursor from tweak tool"

# 4. Install Plank dock with following command then install Mac themes for Plank
sudo apt-get install plank macbuntu-os-plank-theme-lts-v8
echo 'Start plank & Press "Ctrl + Right Click" on the plank to access context menu.'

# 5. Replace 'Ubuntu Desktop' text with 'Mac' on the Panel
cd && wget -O Mac.po http://drive.noobslab.com/data/Mac/change-name-on-panel/mac.po
cd /usr/share/locale/en/LC_MESSAGES; sudo msgfmt -o unity.mo ~/Mac.po;rm ~/Mac.po;cd

# 6. Apple Logo in Launcher
wget -O launcher_bfb.png http://drive.noobslab.com/data/Mac/launcher-logo/apple/launcher_bfb.png
sudo mv launcher_bfb.png /usr/share/unity/icons/

# 7. Tweak Tools to change Themes & Icons
sudo apt-get install unity-tweak-tool
sudo apt-get install gnome-tweak-tool

# 8. (Optional) Mac fonts
wget -O mac-fonts.zip http://drive.noobslab.com/data/Mac/macfonts.zip
sudo unzip mac-fonts.zip -d /usr/share/fonts; rm mac-fonts.zip
sudo fc-cache -f -v
echo "You can change fonts from Unity-Tweak-Tool, Gnome-Tweak-Tool."
```


## 软件与配置

得益于平时完全不完游戏，我的电脑主要用来开发和平时上网以及看视频，
完全没有游戏配置的需求，所以我的常用软件主要集中在开发上。

### 软件安装原则

为了方便在不同电脑，不同系统下使用相同的配置，我自己主要遵从一下几个软件原则：

 1. 手动安装不比`apt-get`安装复杂的话，则选择手动安装；
 2. 手动安装的软件，全部安装在`/opt`下；
 3. 涉及到多版本时，以软件名作为`/opt`下的子目录，并且以版本号作为子目录下的子目录，同时建立`current`软链接[^1]；

    ``` shell
    /opt/example
    ├── current -> version-1
    ├── version-1
    ├── version-2
    └── version-3
    ```

[^1]: 使用current软链接的原因是，在配置文件里直接写为current，当要切换软件版本时，只需要重新将`current`指向相应版本软件而不需要去更改配置文件。在使用版本控制保存配置文件时，这点特别有用。

[//]: # ### 系统快捷键设置
[//]: # [FIXME] TBD...

### 常用软件

装完系统后，高频使用的软件其实也就那么几个。

 - zsh
 - vim
 - tmux
 - Xmind
 - Shadowsocks
 - Jetbrains IDEA
 - Java, Grails, Gradle, Latex, ...

#### vim

 1. 通过Vundle来管理vim插件；
 2. 不同功能的相应设置放在不同的文件；
 3. 使用`runtime! config/**/*.vim`来自动加载所有的配置信息。

vim的配置内容以及安装脚本可在[huajianmao/config的vim目录](https://github.com/huajianmao/config/tree/master/vim)中找到。
我的vim配置目录内容如下：

```
config/vim
├── dotvim
│   ├── after
│   │   └── ftplugin
│   │       └── markdown
│   │           └── instant-markdown.vim
│   ├── compiler
│   │   └── gradle.vim
│   ├── config
│   │   ├── colorschemes.vim
│   │   ├── ctrlp.vim
│   │   ├── gradle.vim
│   │   ├── grep.vim
│   │   ├── indentLine.vim
│   │   ├── indent.vim
│   │   ├── javacomplete2.vim
│   │   ├── jsx.vim
│   │   ├── misc.vim
│   │   ├── nerdtree.vim
│   │   ├── syntastic.vim
│   │   ├── taglist.vim
│   │   ├── ultisnips.vim
│   │   ├── vim-airline.vim
│   │   ├── vim-markdown.vim
│   │   └── YouCompleteMe.vim
│   └── ftdetect
│       └── thrift.vim
├── dotvimrc
└── setup.sh
```

[dotvimrc](https://github.com/huajianmao/config/blob/master/vim/dotvimrc)的内容：

``` vim
let mapleader="," 

set nocompatible              " be improved, required
filetype off                  " required

set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Bundle 'gmarik/Vundle.vim'
" Bundle 'plasticboy/vim-markdown'
Bundle 'kien/ctrlp.vim'
Bundle 'flazz/vim-colorschemes'
Bundle 'tpope/vim-surround'
" Bundle 'Yggdroot/indentLine'
Bundle 'Lokaltog/vim-easymotion'

Plugin 'vim-airline/vim-airline'
Bundle 'vim-airline/vim-airline-themes'

Bundle 'edkolev/tmuxline.vim'
Bundle 'bling/vim-bufferline'
Bundle 'scrooloose/syntastic'
Bundle 'scrooloose/nerdtree'
Bundle 'taglist.vim'
Bundle 'ervandew/supertab'

Bundle 'pangloss/vim-javascript'
Bundle 'mxw/vim-jsx'
Bundle 'justinj/vim-react-snippets'

" Bundle 'artur-shaik/vim-javacomplete2'
" Bundle 'Valloric/YouCompleteMe'
Bundle 'SirVer/ultisnips'
Bundle 'honza/vim-snippets'
" Bundle 'LaTeX-Suite-aka-Vim-LaTeX'
Bundle 'Align'
" Bundle 'Vim-R-plugin'
" Bundle 'tpope/vim-rails'
" Bundle 'airblade/vim-rooter'
call vundle#end()            " required

filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
" filetype plugin on

runtime! config/**/*.vim
```

其中，`config`目录放置了所有相关的配置内容。可根据实际需要从github下载或自行编辑添加。

vim的配置脚本 [setup.sh](https://github.com/huajianmao/config/blob/master/vim/setup.sh)：

``` shell
#!/bin/sh

CONFIG=`pwd`/dotvimrc
DOTVIM=`pwd`/dotvim
RCFILE=~/.vimrc

if [ -f $RCFILE ];
then
  mv $RCFILE $RCFILE.`date +%Y-%m-%d_%H:%M:%S`
fi
ln -s $CONFIG $RCFILE

git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
vim +BundleInstall +qall

ln -s $DOTVIM/* ~/.vim/

echo "Done config vim, but you need to install the bundles in vim!"
echo "We need to install instant-markdown-d with node.js. Please refer to [vim-instant-markdown](https://github.com/suan/vim-instant-markdown)"
```

其中，常用到的Java, Gradle, Grails等等的HOME变量，都是以current软链接哦形式指定。

#### zsh

用zsh的主要原因是它强大的功能和加上[`oh-my-zsh`](https://github.com/robbyrussell/oh-my-zsh)后的高颜值。:)
因为是git的重度使用者，用了`oh-my-zsh`后，git的分支信息、库的修改信息等等一目了然。
另外zsh还有许多其他挺有用的功能，比如，自动补全等等。
所以，一直以来对zsh特别钟爱。

直接上我的zsh配置以及安装脚本吧，详细内容详见[Github](https://github.com/huajianmao/config/tree/master/zsh)

我的[.zshrc](https://github.com/huajianmao/config/blob/master/zsh/dotzshrc)

``` shell
# Path to your oh-my-zsh installation.
export ZSH=$HOME/.oh-my-zsh

## ZSH_THEME="robbyrussell"
## ZSH_THEME="agnoster"
## ZSH_THEME="fino-time"
## ZSH_THEME="fino"
ZSH_THEME="hjmao"
## ZSH_THEME="xiong-chiamiov-plus"

CASE_SENSITIVE="true"
DISABLE_AUTO_UPDATE="true"

# Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
plugins=(git)

source $ZSH/oh-my-zsh.sh

# User configuration

export PATH=$HOME/bin:/usr/local/bin:$PATH
# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
export LANG=en_US.UTF-8

alias tmux="TERM=xterm-256color tmux"

export JAVA_HOME=/opt/java/current
export GRADLE_HOME=/opt/gradle/current
export GRAILS_HOME=/opt/grails/current

export ANDROID_SDK=/opt/google/android-sdk/current
export ANDROID_NDK_HOME=/opt/google/android/ndk/current
export IDEA_HOME=/opt/IntelliJ/idea/current
export WEBSTORM_HOME=/opt/IntelliJ/WebStorm/current
export VSCODE_HOME=/opt/microsoft/VSCode

export PATH=$GRADLE_HOME/bin:$JAVA_HOME/bin:$IDEA_HOME/bin:$GRAILS_HOME/bin:$ANDROID_SDK/tools:$WEBSTORM_HOME/bin:$VSCODE_HOME:$ANDROID_NDK_HOME:$PATH

export EDITOR=vim

# autoload predict-on
# predict-on

export DISPLAY=:0.0
```

我的zsh相关的[安装脚本](https://github.com/huajianmao/config/blob/master/zsh/setup.sh)

``` shell
#!/bin/sh

RCFILE=~/.zshrc
CONFIG=`pwd`/dotzshrc

OMZRC=~/.oh-my-zsh
OHMYZSH=`pwd`/oh-my-zsh

FONTFILE=`pwd`/PowerlineSymbols.otf
FONTDIR=/usr/share/fonts/extra

if [ -f $RCFILE ];
then
  mv $RCFILE $RCFILE.`date +%Y-%m-%d_%H:%M:%S`
fi
ln -s $CONFIG $RCFILE

if [ -d $OMZRC ];
then
  mv $OMZRC $OMZRC.`date +%Y-%m-%d_%H:%M:%S`
fi
# ln -s $OHMYZSH $OMZRC

echo "To install patch font into $FONTDIR, root privilliage needed!"
sudo mkdir -p $FONTDIR
sudo cp $FONTFILE $FONTDIR
sudo fc-cache -fv

git clone https://github.com/robbyrussell/oh-my-zsh.git $OMZRC
cp hjmao.zsh-theme $OMZRC/themes/
```

#### tmux

tmux脚本及配置文件如下，详见[Github](https://github.com/huajianmao/config/tree/master/tmux)

[.tmux](https://github.com/huajianmao/config/blob/master/tmux/dottmux.conf)

``` shell
#remap prefix to Control + l
set -g prefix C-l
unbind C-b
bind C-l send-prefix
bind r source-file ~/.tmux.conf\; display "Reloaded!"

# split window
bind s split-window -h
bind v split-window -v

bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
bind H resize-pane -L 5
bind J resize-pane -D 5
bind K resize-pane -U 5
bind L resize-pane -R 5

set -g default-terminal "xterm"
```

[setup.sh]()

``` shell
#!/bin/sh

CONFIG=`pwd`/dottmux.conf
RCFILE=~/.tmux.conf

if [ -f $RCFILE ];
then
  mv $RCFILE $RCFILE.`date +%Y-%m-%d_%H:%M:%S`
fi
ln -s $CONFIG $RCFILE
```

#### shadowsocks

``` shell
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

通过PPA源安装Shadowsocks-qt5应该是最简单的方法了。
安装后，启动shadowsocks-qt5，然后添加你的Server就好了。
更详细的Shadowsocks-qt5相关的内容可以参考[官方wiki](https://github.com/shadowsocks/shadowsocks-qt5/wiki)。


