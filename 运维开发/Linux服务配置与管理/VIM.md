---
title: VIM
tags:
  - vim
  - vi
---

![](https://gitee.com/fnaichu/mypicbed/raw/master/img/202205111548935.png)

**VIM基本配置**

```vimrc
" 设定默认解码 
set fenc=utf-8
set fencs=utf-8,usc-bom,euc-jp,gb18030,gbk,gb2312,cp936

" 侦测文件类型
filetype on 
" 载入文件类型插件 
filetype plugin on  

" 语法高亮 
syntax enable
syntax on 


" 不要备份文件（根据自己需要取舍） 
set nobackup 

" 不要生成swap文件，当buffer被丢弃的时候隐藏它 
setlocal noswapfile 
set bufhidden=hide 

" 使回格键（backspace）正常处理indent, eol, start等 
set backspace=2 


" 继承前一行的缩进方式，特别适用于多行注释 
set autoindent 
" 为C程序提供自动缩进 
set smartindent 
"使用C样式的缩进 
set cindent 

" 制表符为4 
set tabstop=4 
" 统一缩进为4 
set softtabstop=4 
set shiftwidth=4 

set cindent "c/c++自动缩进"
set smartindent
set autoindent"参考上一行的缩进方式进行自动缩进"
filetype indent on "根据文件类型进行缩进"
set softtabstop=4 "4 character as a tab"
set shiftwidth=4
set smarttab 

```

**VIM默认主题推荐**

* darkblue
* delek


**以十六进制查看文件**

	%!xxd
