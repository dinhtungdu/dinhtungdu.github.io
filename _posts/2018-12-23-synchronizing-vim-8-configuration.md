---
layout: post
title: Synchronizing vim 8 config and plugins with git submodule
---
A vim user may change the config or plugins every day, and I was no exception. My current configuration is completely different from the first day I use vim. Because my current config takes me so much time and effort to achieve, it's wise to keep it safe.

In this post, I'm using git and git submodule to store my vim config.
Assume that we have an empty `.vim` folder and a `.vimrc` at our home directory.

## Prepare for synchronizing

First, move your `.vimrc` file to `.vim` folder:
```
mv ~/.vimrc ~/.vim/vimrc
```

Create a symbolic link which points to `~/.vim/vimrc`:
```
ln -s ~/.vim/vimrc ~/.vimrc
```

Initialize `.vim` folder as a git repository:
```
cd ~/.vim
git init
git add .
git commit -m "Initial commit"
```
## Installing plugins
Vim 8 introduced a native way to load third-party packages. We will use this new method and git submodule to install our plugins.

For example, we're going to install deoplete, an asynchronous completion plugin. Instead of downloading or cloning the repository, we use git submodule.
```
cd ~/.vim
git submodule add https://github.com/Shougo/deoplete.nvim pack/username/start/deoplete.nvim
```

You can replace `username` to your username of choice.

After installing all the plugins we need, we will commit it to our remote repository to keep everything safe.

## Install our Vim 8 config on a new machine

Follow these step to synchronize our vim config to another machine:
```
cd ~
git clone http://github.com/username/dotvim.git ~/.vim
ln -s ~/.vim/vimrc ~/.vimrc
cd ~/.vim
git submodule update --init
```

## Update all plugins
It's boring to go to every plugin folder and pull the newest changes. We can update all plugins by a single command:
```
git submodule foreach git pull origin master
git add .
git commit -m "update submodules"
```
