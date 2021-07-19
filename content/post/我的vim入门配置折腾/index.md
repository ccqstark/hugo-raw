---
title: "我的vim入门配置折腾"
date: 2021-01-31T21:26:00+08:00
draft: true
image: vim.png
slug: "vim"
tags:
    - Vim
categories:
    - 其它技术
--- 

由于这几天开始看《CS:APP》，我就开始寻求一款Mac上的轻量的C语言编辑器。找来找去，无非是`VSCode`、`CLion`和大名鼎鼎的`Vim`。

为了减少磁盘占用同时让自己更接近于底层，我还是硬着头皮折腾起了Vim，这个上古神器之前就一直让我望而却步，我对它的掌握程度也差不多是会退出的程度，这一次就打算好好来折腾下。

### 安装NeoVim

Vim其实到目前为止，不同的分支版本还是很多的，比较流行的现代版本就要属NeoVim了，所以我在终端安装了它，用`iTerm2`运行着。

```bash
brew install neovim
```

安装完成后用nvim命令就可以打开

```bash
nvim
```

**配置文件路径**

传统的vim的配置配置文件为`~/.vimrc` 

而nvim的配置文件为`/.config/nvim/init.vim` ，之后修改nvim配置文件就用这个，以下简称为`init.vim`

### 安装SpaceVim

作为小白，快速搭建一个好看实用的Vim开发环境那最好的选择就是`SpaceVim` 了，下面是官方的介绍：

> SpaceVim 是一个社区驱动的模块化的 Vim IDE，以模块的方式组织管理插件以及相关配置， 为不同的语言开发量身定制了相关的开发模块，该模块提供代码自动补全， 语法检查、格式化、调试、REPL 等特性。用户仅需载入相关语言的模块即可得到一个开箱即用的 Vim IDE。

官网地址：

[主页 | SpaceVim](https://spacevim.org/cn/)

官网的文档还是很全的，按官方文档就可以快速搭建出来了。以MacOS为例：

**安装spacevim**

```bash
curl -sLf https://spacevim.org/cn/install.sh | bash
```

完成后重新打开nvim就会自动下载相关插件。

然后就是主界面：

![%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled.png](%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled.png)

**主题**的修改可以参考官方文档

[SpaceVim colorscheme 模块 | SpaceVim](https://spacevim.org/cn/layers/colorscheme/)

**快捷键符号说明**

`SPC` 代表空格

`<Leader>` 默认为`\`

以下一些功能需要对mac进行一定的设置才能正常使用。

打开`系统偏好设置` → `键盘` 

勾选`将F1、F2等键用作标准功能键` 

这同时也解决了Chrome的`F12` 不能打开控制台的问题

![%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%201.png](%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%201.png)

**文件目录树**

按`F3` 可以打开或关闭

**语法函数树**

按`F2`可以打开或关闭

mac下可能会出现错误，解决方案：

```bash
brew install ctags-exuberant
```

然后`init.vim`里添加下面这行即可

```bash
let g:Tlist_Ctags_Cmd='/usr/local/Cellar/ctags/5.8_1/bin/ctags'
```

shell**终端** 

`SPC '` 即可打开系统shell，如果用了oh-my-zsh主题什么的也会保留的

### 配置C/C++环境

spacevim对大部分语言都有相关支持，文档也齐全，比如我需要的C语言环境：

[使用 Vim 搭建 C/C++ 开发环境 | SpaceVim](https://spacevim.org/cn/use-vim-as-a-c-cpp-ide/)

按照官方文档修改spacevim的配置文件`~/.SpaceVim.d/init.toml` 即可，以下简称`init.toml`

对于自带的模块基本只需要添加`[[layers]]`

```bash
# 语法高亮
[[layers]]
    name = 'lang#c'
    enable_clang_syntax_highlight = true

# 代码格式化
[[layers]]
  name = "format"

# 语法检查
[[layers]]
  name = "checkers"
```

spacevim内置的模块还是很多的，但是很多功能相对简陋，为了实现更好的效果，我们还需要安装其他的插件。

### 安装插件管理器vim-plug

虽然SpaceVim已经自带插件管理工具，在`init.toml` 里添加

```bash
[[custom_plugins]]
	repo = "插件的github地址"
```

重启后即可自动下载

用spacevim管理的这些从github上下载的插件存储在`~/.cache/vimfiles/repos/github.com/`

这个自带的插件管理器大部分情况下还可以，但是有时不太好使，所以我选择多下载一个vim的主流插件管理器`vim-plug` 

使用curl安装，下面一条命令就够

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

如果遇到了`Connection refused`的错误，使用`SwitchHosts!` ，添加以下的host：

```bash
199.232.68.133 raw.githubusercontent.com
199.232.68.133 user-images.githubusercontent.com
199.232.68.133 avatars2.githubusercontent.com
199.232.68.133 avatars1.githubusercontent.com
```

使用插件管理器安装新的插件的方法是，在`init.vim` 配置文件里添加：

```bash
call plug#begin()

call plug#end()
```

然后把要安装的插件添加到这两行代码中间，以`Plug 'xxx/xxx'` 的格式，如：

```bash
call plug#begin()
Plug 'junegunn/vim-easy-align'
call plug#end()
```

之后重启nvim，使用命令`:PlugInstall` 就可以安装了。

若要卸载插件，就从配置文件中去除相应的那一行，然后使用命令`:PlugClean` 就可以卸载了。

### 安装Coc.nvim

SpaceVim自带的补全还行，但是为了达到效果更好的补全和语法检查，我选择安装插件`coc.nvim`              

[neoclide/coc.nvim](https://github.com/neoclide/coc.nvim)

这可以说是神器了，除了基本的补全之外，还提供了众多扩展来支持不同的语言和不同的特性。

用之前下的`vim-plug` 来安装

```bash
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

安装完后可以先直接把官方的示例配置文件粘贴进`init.vim`，示例配置文件在github的readme里有。

为了C语言更好的补全支持，我下载相关的coc扩展——`coc-clangd`

[neoclide/coc.nvim](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions)

首先保证有`node`环境，然后运行以下命令安装coc-clangd

```bash
:CocInstall coc-clangd
```

完成后打开一个c语言文件，若提示找不到clangd，则自己手动下载。

进入github release页面下载`clangd-mac-11.0.0.zip`

[Release 11.0.0 · clangd/clangd](https://github.com/clangd/clangd/releases/tag/11.0.0)

解压后目录下有一个`bin`文件夹和一个`lib`文件夹

将bin下的clangd文件移动至`/usr/local/bin/`目录下

将lib目录下的clang文件夹移动至`~/Library/`目录下

再次打开即可正常使用，注意把spacevim自带的补全给禁用

```bash
[[layers]]
	name = 'autocomplete'
	enable = false
```

效果如下：

![%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%202.png](%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%202.png)

可以看到代码提示效果是非常好的

当然，里面还有很多很实用的扩展也可以根据需要下

### 括号引号自动补全

`coc.nvim`插件对于括号和引号还是没有自动补成一对的功能，可以在`init.vim`里添加

```bash
inoremap ( ()<Esc>i
inoremap [ []<Esc>i
inoremap { {<CR>}<Esc>O
autocmd Syntax html,vim inoremap < <lt>><Esc>i| inoremap > <c-r>=ClosePair('>')<CR>
inoremap ) <c-r>=ClosePair(')')<CR>
inoremap ] <c-r>=ClosePair(']')<CR>
inoremap } <c-r>=CloseBracket()<CR>
inoremap " <c-r>=QuoteDelim('"')<CR>
inoremap ' <c-r>=QuoteDelim("'")<CR>

function ClosePair(char)
 if getline('.')[col('.') - 1] == a:char
 return "\<Right>"
 else
 return a:char
 endif
endf

function CloseBracket()
 if match(getline(line('.') + 1), '\s*}') < 0
 return "\<CR>}"
 else
 return "\<Esc>j0f}a"
 endif
endf

function QuoteDelim(char)
 let line = getline('.')
 let col = col('.')
 if line[col - 2] == "\\"
 return a:char
 elseif line[col - 1] == a:char
 return "\<Right>"
 else
 return a:char.a:char."\<Esc>i"
 endif
endf
```

### 彩虹括号

vim里也有类似vscode或IDEA那样的彩虹括号插件

```bash
Plug 'luochen1990/rainbow'
#或
repo = 'luochen1990/rainbow'
```

`init.vim` 添加

```bash
let g:rainbow_active = 1
let g:rainbow_conf = {
\   'guifgs': ['darkorange3', 'seagreen3', 'royalblue3', 'firebrick'],
\   'ctermfgs': ['lightyellow', 'lightcyan','lightblue', 'lightmagenta'],
\   'operators': '_,_',
\   'parentheses': ['start=/(/ end=/)/ fold', 'start=/\[/ end=/\]/ fold', 'start=/{/ end=/}/ fold'],
\   'separately': {
\       '*': {},
\       'tex': {
\           'parentheses': ['start=/(/ end=/)/', 'start=/\[/ end=/\]/'],
\       },
\       'lisp': {
\           'guifgs': ['darkorange3', 'seagreen3', 'royalblue3', 'firebrick'],
\       },
\       'vim': {
\           'parentheses': ['start=/(/ end=/)/', 'start=/\[/ end=/\]/', 'start=/{/ end=/}/ fold', 'start=/(/ end=/)/ containedin=vimFuncBody', 'start=/\[/ end=/\]/ containedin=vimFuncBody', 'start=/{/ end=/}/ fold containedin=vimFuncBody'],
\       },
\       'html': {
\           'parentheses': ['start=/\v\<((area|base|br|col|embed|hr|img|input|keygen|link|menuitem|meta|param|source|track|wbr)[ >])@!\z([-_:a-zA-Z0-9]+)(\s+[-_:a-zA-Z0-9]+(\=("[^"]*"|'."'".'[^'."'".']*'."'".'|[^ '."'".'"><=`]*))?)*\>/ end=#</\z1># fold'],
\       },
\       'css': 0,
\   }
\}
```

### 安装CtrlP

CtrlP是vim下一款很好的文件模糊搜索跳转插件

[kien/ctrlp.vim](https://github.com/kien/ctrlp.vim)

安装

```bash
repo = 'kien/ctrlp.vim'
```

使用

```bash
:CtrlP [要搜索的目录]
```

如果目录下的文件过多，比如系统根目录，就会花比较多的时间去索引。所以建议尽量缩小范围。

之后便可以输入关键字进行搜索了

![%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%203.png](%E6%88%91%E7%9A%84vim%E5%85%A5%E9%97%A8%E9%85%8D%E7%BD%AE%E6%8A%98%E8%85%BE%20a647efa0f247430983e0241115e52dca/Untitled%203.png)

## 其他插件

比如更好的代码格式化`vim-clang-format`和代码时间记录`wakatime`

[rhysd/vim-clang-format](https://github.com/rhysd/vim-clang-format)

[wakatime/vim-wakatime](https://github.com/wakatime/vim-wakatime)

基本上想要的插件都可以在github上找到，根据官方文档使用即可

### 常用快捷键

`SPC 1/2/3` 切换不同窗口，数字为窗口编号

`SPC l r` 运行代码

`SPC b f` 代码格式化

`g d` 函数跳转

`<c-o>` 回调到上次的位置（这个写法表示`ctrl+o`）

更多功能可以`SPC`空格键唤出菜单查看

### 尾声

到此一个基本的vim编程环境已经搭好了，用来写点小东西还是够用的。

这一套折腾下来最大的感受是曾经觉得vim好难学好难用，但是通过这几天捣鼓之后发现只要熟练了其实效率还是很高的。难怪至今还有很多vim的使用者和爱好者。同时自己安装各种插件，修改配置文件，通过自定义来获得一款完全属于自己的编辑器的过程也是充满乐趣的，我很享受这个过程。