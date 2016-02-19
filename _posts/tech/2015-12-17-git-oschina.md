---
layout: post
title: 如何简单使用oschina的git上传删除操作
category:  技术
tags: 
keywords: 
---


>很多时候大家都懒的使用命令来进行操作；而是使用简单易用的客户端；其实呢！使用命令更方便更快捷

# oschina
	https://git.oschina.net
	git 切换里面的文件呢  的使用方法
	1 . 桌面创建一个文件夹 git
	2. git init
	git config --global user.name "XXX"
	git config --global user.email XXX@usuretech.com
	git config -l 查看是否配置对
	3.  clone https://git.oschina.net/2xxxyz/U.git  下载源
	4. git config --global push.default matching    设置推送的源是matching 主枝干
	5. mv  WholeNet.ipa 到当前目录
	6. git add WholeNet.ipa
	7. git commit -m "切换WholeNet.ipa”  提交
	8. git push   推送到主目录

	删除呢
	git rm  "LICENSE" "README.md" "udianfang.plist"
	git commit -m "删除不需要的"
	git push

	git pull origin master  下载 服务器的和本地合并


# 开始你的oschina托管之旅吧