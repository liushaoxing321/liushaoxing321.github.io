---
layout: post
title: IOS企业开发者帐号如何把ipa包和plist文件托管在oschina
category:  技术
tags: 
description: IOS企业开发者帐号如何把ipa包和plist文件托管在oschina
keywords: 
---

# oschina企业分发iOS步骤


> 存放ipa包

	本地搭建服务器地址为http的，把ipa上传到本地服务器 例如：http://test.qwzt.net:8084/WholeNet.ipa
	or
	使用oschina 来做，把包传到oschina 然后使用http的oschina地址登陆；然后得到ipa包的http地址 如：
	http://git.oschina.net/2xxxyz/U/raw/master/WholeNet.ipa

> 存放plist文件

	使用https://git.oschina.net登陆oschina  必须后面是https
	oschina中新建一个项目 然后点击项目中的+号
	新建一个文件manifest.plist(文件格式是苹果的规范)  是可以通过xcode编译的时候导出来的 这个文件 可以使用xcode得到规范的文件
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
		<plist version="1.0">
		<dict>
		    <key>items</key>
		    <array>
		        <dict>
		            <key>assets</key>
		            <array>
		                <dict>
		                    <key>kind</key>
		                    <string>software-package</string>
		                    <key>url</key>
		                    <string>http://test.qwzt.net:8084/WholeNet.ipa</string>
		                </dict>
		            </array>
		            <key>metadata</key>
		            <dict>
		                <key>bundle-identifier</key>
		                <string>com.company.wholeNet2</string>
		                <key>bundle-version</key>
		                <string>2.2</string>
		                <key>kind</key>
		                <string>software</string>
		                <key>title</key>
		                <string>WholeNet</string>
		            </dict>
		        </dict>
		    </array>
		</dict>
		</plist>

	修改plist中文件
		（<string>http://test.qwzt.net:8084/WholeNet.ipa</string>
	这个改成前面得到的ipa包的http地址
	http://git.oschina.net/2xxxyz/U/raw/master/WholeNet.ipa

> 获取plist地址

	点击oschina右上角的原始数据；获取一个https的地址(注意是https的)；例如：
	https://git.oschina.net/2xxxyz/U/raw/master/manifest.plist
	--
	##对plist地址封装上itms协议使得苹果手机浏览器能够识别
	在前面plist的地址拼接 加入前缀
	实际的下载地址为：
	itms-services://?action=download-manifest&url=https://git.oschina.net/2xxxyz/U/raw/master/manifest.plist

	>直接打开 会提示appstore打开，所以需要嵌入到网页中，就可以不提示该东西，直接提示安装软件

> 在网页中嵌入该下载地址

	<a href=“itms-services://?action=download-manifest&url=https://git.oschina.net/2xxxyz/U/raw/master/manifest.plist">Install App</a>


# 开始你的oschina托管之旅吧
