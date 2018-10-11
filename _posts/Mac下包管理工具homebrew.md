---
title: Mac下包管理工具homebrew
date: 2016-09-20 23:30:00
ctime: 2016-09-20 23:30:00
utime: 2016-09-20 23:30:00
modif_times: 0
tags:
- Mac
- homebrew
categories:
- Mac
---

## Homebrew

homebrew,一个在Mac OS上的软件包管理工具。是一款有Ruby开发的智能包管理系统.

[官网地址](http://brew.sh)

- 安装：
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
- 功能：
> 软件包管理。homebrew会将套件安装到独立目录，并将文件软连接链接到/usr/local

- 命令格式
```
$ brew -h
Example usage:
  brew search [TEXT|/REGEX/]
  brew (info|home|options) [FORMULA...]
  brew install FORMULA...
  brew update
  brew upgrade [FORMULA...]
  brew uninstall FORMULA...
  brew list [FORMULA...]
Troubleshooting:
  brew config
  brew doctor
  brew install -vd FORMULA
Brewing:
  brew create [URL [--no-fetch]]
  brew edit [FORMULA...]
  https://github.com/Homebrew/brew/blob/master/share/doc/homebrew/Formula-Cookbook.md
Further help:
  man brew
  brew help [COMMAND]
  brew home
```

- 示例
```
$ brew install wget
```


## brew-cask

brew-cask,是一套建立在homebrew之上的Mac软件安装命令行工具。其与brew的区别是，后者侧重与软件套件和软件环境的配置安装。

[官网](http://caskrom.github.io)

- 安装：
```
$ brew install brew-cask
```

- 命令格式
```
$ brew cask -h
brew-cask provides a friendly homebrew-style CLI workflow for the
administration of macOS applications distributed as binaries.
Commands:
    audit                  verifies installability of Casks
    cat                    dump raw source of the given Cask to the standard output
    cleanup                cleans up cached downloads and tracker symlinks
    create                 creates the given Cask and opens it in an editor
    doctor                 checks for configuration issues
    edit                   edits the given Cask
    fetch                  downloads remote application files to local cache
    home                   opens the homepage of the given Cask
    info                   displays information about the given Cask
    install                installs the given Cask
    list                   with no args, lists installed Casks; given installed Casks, lists staged files
    search                 searches all known Casks
    style                  checks Cask style using RuboCop
    uninstall              uninstalls the given Cask
    update                 a synonym for 'brew update'
    zap                    zaps all files associated with the given Cask
See also "man brew-cask"
```

- 示例
```
$ brew cask install iterm2
```
