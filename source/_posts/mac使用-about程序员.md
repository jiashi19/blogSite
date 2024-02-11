---
title: about-macOS
date: 2023-09-01 12:29:54
tags:
categories:
- others
---
## 安装软件

1. .app文件

   macOS 应用程序或 .app 文件是一个 bundle 或从操作系统的角度来看相关资源的集合。例如 Safari.app 包不仅包含可执行文件，还包含信息属性列表、图标、插件等资源。

   与 Windows 不同，在 macOS 上应用程序 不仅仅是一个可执行文件。应该视为一个独立的实体，具有必要的框架和库来独立运行。（可以简单地拖放 .app 以在 macOS 上“安装”程序）

2. .pkg文件

3. .dmg文件

   

## 部分功能

要在Mac上获得剪切粘贴功能，首先，使用常规的Command + C复制文件/文件夹 ， 但是在粘贴时，使用 Command + Option + V 而不是 Command +V。 



## 使用homebrew

1. 包管理软件；
2. 四个部分组成：brew、homebrew-core 、homebrew-cask、homebrew-bottles
3. brew：Homebrew 源码仓库
4. homebrew-core：Homebrew 核心源
5. homebrew-cask：提供macOS应用和大型二进制文件的安装
6. homebrew-bottles：预编译二进制软件包

| brew --version 或者 brew -v | 显示 brew 版本信息                           |
| --------------------------- | -------------------------------------------- |
| brew install <软件名>       | 安装指定软件                                 |
| brew uninstall <软件名>     | 卸载指定软件                                 |
| brew list                   | 显示所有的已安装的软件                       |
| brew search <软件名>        | 搜索本地远程仓库的软件，已安装会显示绿色的勾 |
| brew search /<软件名>/      | 使用正则表达式搜软件                         |

```bash
$ brew install <package>

$ brew install --cask <package>
```

