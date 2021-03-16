---
title:  "YouCompleteMe Plugin for vim users"
date:   2019-04-29
categories: Vim
tags: [Vim]
---

# You-Complete-Me
YouCompleteMe (YCM) is an open source-based project, used as one of the most popular vim plugins for semantic analysis such as
1. Code Completion
2. GoTo
3. Code Diagnostics

# Installation
Before installing YCM, there are some pre-requisites
1. cmake
2. build-essentials
3. python3-dev

on Ubuntu 14.04:
```
$ sudo apt-get install cmake3 build-essentials python3-dev
```

on Ubuntu 16.04 and later:
```
$ sudo apt-get install cmake build-essentials python3-dev
```

Install YCM with Vundle: Add the following line to your vimrc and install.
```
Plugin 'Valloric/YouCompleteMe'
```

After installing YCM with Vundle, you will have YouCompleteMe directory created under `$HOME/.vim/bundle/`. Navigating to that directory will help you find the installation script written in python.

Install YCM by executing the below lines.
```
$ cd ~/.vim/bundle/YouCompleteMe
$ ./install.py --clang-completer
```

The reason why I put `--clang-completer` option is because we need to install `clang`. YCM uses libclang or clang in order to perform the semantic analysis for C-
family languages.


If you are working on C/C++ codes, omitting `--clang-completer` will lead you to the following warning message:

```
No semantic completer exists for filetypes: ['cxx']
```
Options other than `--clang-completer` are also availble for various languages:
* C# support: install Mono with Homebrew or by downloading the Mono Mac package and add --cs-completer when calling install.py.
* Go support: install Go and add `--go-completer` when calling install.py.
* JavaScript and TypeScript support: install Node.js and npm and add `--ts-completer` when calling install.py.
* Rust support: install Rust and add `--rust-completer` when calling install.py.
* Java support: install JDK8 (version 8 required) and add `--java-completer` when calling install.py.

# Pitfalls
Sometimes (in my case) vim fail to locate the library path for Python. After some extensive googling I found one solution: Re-install Vim via git clone, re-configure/make/make install with particular python flags off. I will explain in detail.

First you need to re-install vim.

1. **Remove Vim if you already got one.**
```
sudo apt remove vim vim-runtime gvim
```
On Ubuntu 12.04.2 you probably have to remove these packages as well:
```
sudo apt remove vim-tiny vim-common vim-gui-common vim-nox
```
2. **Get the Vim source, Install it using Python!**

* Note: If you are using Python, your config directory might have a machine-specific name (e.g. config-3.5m-x86_64-linux-gnu). Check in /usr/lib/python[2/3/3.5] to find yours, and change the python-config-dir and/or python3-config-dir arguments accordingly.
* Note for Ubuntu users: You can only use Python 2 or Python 3. If you try to compile vim with both `--with-python-config-dir` and `--with-python3-config-dir`, YouCompleteMe will give you an error YouCompleteMe unavailable: requires Vim compiled with Python (2.6+ or 3.3+) support, when you start vim.
* I recommend using python 3. Add `--enable-python3interp=yes` and `--with-python3-config-dir=/usr/lib/python3.7/"Your_Config_Directory"`.
* My config directory was `/usr/lib/python3.7/config-3.7m-x86_64-linux-gnu/` but this can be different depending on the environment.
```
cd ~
git clone https://github.com/vim/vim.git
cd vim
./configure --with-features=huge \
            --enable-multibyte \
	    --enable-rubyinterp=yes \
	    --enable-python3interp=yes \
	    --with-python3-config-dir=/usr/lib/python3.7/config-3.7m-x86_64-linux-gnu \
	    --enable-perlinterp=yes \
	    --enable-luainterp=yes \
            --enable-gui=gtk2 \
            --enable-cscope \
	   --prefix=/usr/local
cd src
sudo make -j
sudo make install -j
```
Then, `cd` to YCM directory and install YCM again.
```
$ cd ~/.vim/bundle/YouCompleteMe
$ python3 install.py --clang-completer
```
Done!
