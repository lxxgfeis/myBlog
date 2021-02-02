---
title: "My macOS Setup"
date: 2021-02-02T22:48:52+08:00
draft: false
tags: ["macOS"]
categories: ["macOS相关就搁这儿吧"]
featured_image: ""
description: ""
---

# my macOS setup

📝本文记录我的mac的设置，以及我常用的软件、工作环境等信息。

## 调整new mac的系统设置

### 三指拖移

系统偏好设置 -> 辅助功能 -> 指针控制 -> 触控板选项 -> 启用拖移（三指拖移）

### launchpad图标大小调整

```bash
defaults write com.apple.dock springboard-columns -int 10
defaults write com.apple.dock springboard-rows -int 8
defaults write com.apple.dock ResetLaunchPad -bool TRUE
killall Dock
```

## 科学上网

macOS科学上网工具：[clashX](https://github.com/yichengchen/clashX)

## 包管理软件

### [homebrew](https://brew.sh/index_zh-cn)

```bash
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 安装常见软件

| 软件               | 说明                     |
| ------------------ | ------------------------ |
| google-chrome      | 谷歌浏览器               |
| typora             | 所见即所得markdown编辑器 |
| appcleaner         | 软件卸载                 |
| iterm2             | terminal替代品           |
| tmux               | terminal分屏             |
| Alfred4            | finder替代品             |
| visual-studio-code | 编辑器，maybe IDE        |
| wechat             | 微信                     |
| 企业微信           | 工作使用                 |
| WPS                | 办公套件                 |
| OneNote            | 微软出品的笔记软件       |
| Skim               | PDF阅读器                |
| Snipaste           | 截图工具                 |
| MacTex             | Latex编译器              |
| texstudio          | Latex编辑器              |
| Zotero             | 论文管理                 |
| Itsycal            | 接管macOS日历            |
| RunCat             | 一个奔跑的小猫           |
| 网易云音乐         | 听歌                     |

## chrome插件

1. Google 翻译
2. SwitchyOmega 一个代理设置工具
3. OneNote Web Clipper 

## 配置开发环境

### oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### tmux

### 生成ssh key

主要是用于下载git项目。

```bash
ssh-keygen -t ed25519 -C "mymbp@home"
```

### hugo

### GO

## 其他

### 密码管理

1. bitwarden
2. chrome浏览器自动填充

## 总结

```bash
brew install \
	google-chrome \
	typora \
	appcleaner \
	iterm2 \
	alfred \
	visual-studio-code \
	snipaste \
	tmux \
	go \
	hugo 
```


<br>
<center>  ·End·  </center>
