---
title: "My macOS Setup"
date: 2021-02-02T22:48:52+08:00
draft: false
tags: ["macOS"]
categories: ["macOS相关就搁这儿吧?"]
featured_image: ""
description: ""
---


📝本文记录我的`mac`的设置，以及我常用的软件、工作环境等信息。

## 🆕调整新`mac`的系统设置

### 访达设置

将个人的home目录和dev目录加入左侧边栏

### 三指拖移

系统偏好设置 -> 辅助功能 -> 指针控制 -> 触控板选项 -> 启用拖移（三指拖移）

### `launchpad`图标大小调整

```bash
defaults write com.apple.dock springboard-columns -int 16
defaults write com.apple.dock springboard-rows -int 10
defaults write com.apple.dock ResetLaunchPad -bool TRUE
killall Dock
```

### 🔧安装基础工具包

```bash
xcode-select --install
```

## 科学上网

macOS科学上网工具：[clashX](https://github.com/yichengchen/clashX)

## 包管理软件

### [homebrew](https://brew.sh/index_zh-cn)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 安装常见软件

| 软件               | 说明                     | brew安装                         |
| ------------------ | ------------------------ | :------------------------------- |
| google-chrome      | 谷歌浏览器               | *                                |
| typora             | 所见即所得markdown编辑器 | *                                |
| appcleaner         | 软件卸载                 | *                                |
| iterm2             | terminal替代品           | *                                |
| tmux               | terminal分屏             | *                                |
| Alfred4            | finder替代品             | *                                |
| visual-studio-code | 编辑器，maybe IDE        | *                                |
| wechat             | 微信                     | https://weixin.qq.com/           |
| 企业微信           | 工作使用                 | https://work.weixin.qq.com/      |
| WPS                | 办公套件                 | https://mac.wps.cn/              |
| OneNote            | 微软出品的笔记软件       | App Store                        |
| Skim               | PDF阅读器                | https://skim-app.sourceforge.io/ |
| Snipaste           | 截图工具                 | *                                |
| MacTex             | Latex编译器              | http://www.tug.org/mactex/       |
| texstudio          | Latex编辑器              | https://www.texstudio.org/       |
| Zotero             | 论文管理                 | https://www.zotero.org/          |
| Itsycal            | 接管macOS日历            | https://www.mowglii.com/itsycal/ |
| RunCat             | 一个奔跑的小猫           | App Store                        |
| 网易云音乐         | 听歌                     | https://music.163.com/#/download |
| Notion             | 笔记软件                 | https://www.notion.so/           |
| PicGo              | 图床管理                 | https://molunerfinn.com/PicGo/   |
| XMind              | 思维导图工具             | https://www.xmind.cn/            |

## chrome插件

1. Google翻译：网页翻译工具
2. SwitchyOmega：一个代理设置工具
3. Bitwarden：密码管理插件，主要用来生成随机密码
4. OneNote Web Clipper：OneNote的网页剪藏插件
5. Zotero Connector：Zotero剪藏插件

## 配置开发环境

### oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 生成ssh key

主要是用于下载git项目。

```bash
ssh-keygen -t ed25519 -C "mymbp@home"
```

## 其他

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