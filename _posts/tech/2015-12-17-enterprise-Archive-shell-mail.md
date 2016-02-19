---
layout: post
title: IOS企业开发者帐号自动化打包脚本
category:  技术
tags: 
description: IOS企业开发者帐号自动化打包脚本
keywords: 
---


# xcodebuild 和 xcrun 简单介绍
	xctool 是FaceBook开源的一个命令行工具，用来替代苹果的xcodebuild工具。
	xcodebuild —help   ——有用
	xcodebuild 是一款用来打包 Xcode projects 或者 workspaces 的命令行工具。用 xcodebuild 把工程打成 app 格式后再用 xcrun 来签名和打成 ipa 格式的包。关于 xcrun 请参看我的另一篇博文 xcrun 命令详解。

	-exportArchive
	指定一个可以被导出的 archive 文件。需要 -exportFormat，-archivePath和-exportPath` 配合使用，不能在编译时单独使用。
	-exportFormat format
	指定需要被导出的 archive 文件的格式。可行的格式是 IPA（iOS 包文件），PKG（Mac 包文件）和 APP。如果未指定，则 xcodebuild 则会自动检测使用IPA 或 PKG 格式。
	-archivePath xcarchivepath
	指定 archive 路径。
	-exportPath destinationpath
	指定导出的目标文件路径。
	-exportProvisioningProfile profilename
	指定导出 archive 文件时所使用的 provisioning pofile。

	-worksace workspacename

	指定 workspace 的名称。

	-scheme schemename
	指定 scheme 的名称，编译 workspace 时是必须的。
	localhost:WholeNet master$ xcodebuild -workspace WholeNet.xcworkspace  -scheme WholeNet   ——可以编译
	-arch architecture
	当编译每个 target 时使用 architecture 指定的架构类型。
	---
	-exportProvisioningProfile profilename
	指定导出 archive 文件时所使用的 provisioning pofile。
	-exportSigningIdentity identityname
	指定导出 archive 文件时所使用的应用签名 id。在可能的情况下，这个可以被 -exportProvisioningProfile自动推导出来。
	-exportInstallerIdentity identityname
	指定导出 archive 文件时所使用的安装签名 id。如果可能，这个可以被 -exportSigningIdentity 或 -exportProvisioningProfile 自动推导出来。
	-exportWithOriginalSigningIdentity
	指定创建可被导出的 archive 文件时所使用的签名文件。
	---

	使用示例
	xcodebuild clean install

	xcodebuild -workspace MyWorkspace.xcworkspace -scheme MyScheme archive

	xcodebuild -exportArchive -exportFormat IPA -archivePath MyMobileApp.xcarchive -exportPath MyMobileApp.ipa -exportProvisioningProfile 'MyMobileApp Distribution Profile'


# 实际使用介绍

## 清理项目
	localhost:WholeNet master$ xcodebuild clean -workspace WholeNet.xcworkspace  -scheme WholeNet
	** CLEAN SUCCEEDED **
	 Archive
	xcodebuild archive -workspace WholeNet.xcworkspace  -scheme WholeNet -archivePath WholeNet.xcarchive
	xcodebuild archive -project ${PROJECT_NAME}.xcodeproj \
	                  -scheme ${SCHEME_NAME} \
	                  -destination generic/platform=iOS \
	                   -archivePath bin/${PROJECT_NAME}.xcarchive
	xcodebuild archive -project WholeNet\ WholeNet.xcworkspace \
	                   -scheme WholeNet \
	                   -archivePath bin/WholeNet.xcarchive \
	                   || failed "xcodebuild archive"
	** ARCHIVE SUCCEEDED **

## Export ipa
	xcodebuild -exportArchive -archivePath WholeNet.xcarchive -exportPath WholeNet -exportFormat ipa -exportProvisioningProfile 11111111111  uploadBitcode NO
	WholeNet22222222   这个是WholeNet 导出的
	11111111111  这个是 TodayViewController 导出的    这个是可以成功的

	xcodebuild -exportArchive -archivePath ${PROJECT_NAME}.xcarchive \
	                          -exportPath ${PROJECT_NAME} \
	                          -exportFormat ipa \
	                          -exportProvisioningProfile ${PROFILE_NAME}

## 上传ipa包到occhina来托管
	#! /bin/sh
	PAM=$(date)

	cd /Users/master/Desktop/git/U
	sleep 1
	rm -rf WholeNet.ipa
	sleep 3
	mv /Users/master/Desktop/WholeNet.ipa .
	sleep 3
	git add WholeNet.ipa
	sleep 3
	git commit -m "$PAM.WholeNet"
	sleep 3
	git push
	sleep 10
	echo "chenggong"

## 发送邮件给相关人员
	echo 'hi,\n\nThe app is updated recently. Use the safari browser on iOS device to download the app. Here is the URL: http://www.udianfang.cn/downloadPage/ .\n\nThanks!' | mail -s 'iOS客户端更新' jia85860161@163.com
	##使用root来执行发送邮件的操作
		#! /bin/bash
		expect -c "
		spawn su - root
		expect \"Password:\"
		send \"123456\r\"
		interact
		"

# 具体的shell脚本集合iOSipabuildArchiveMailMustRoot.sh
	#!/bin/sh

	#用来标示成功还是失败的
	function failed() {
	    echo "Failed: $@" >&2
	    exit 1
	}

	# unlock login keygen  打开系统的 密码的
	LOGIN_KEYCHAIN=~/Library/Keychains/login.keychain
	security unlock-keychain -p 123456 ${LOGIN_KEYCHAIN} || failed "unlock-keygen"
	sleep 3


	# clean 清理
	xcodebuild clean -workspace WholeNet.xcworkspace \
	                 -scheme WholeNet \
	                 || failed "xcodebuild clean"
	sleep 15
	# archive 打包
	xcodebuild archive -workspace WholeNet.xcworkspace \
			   -scheme WholeNet \
			   -archivePath WholeNet.xcarchive \
		          || failed "xcodebuild archive"
	sleep 15	           
	#sleep 310
	# export ipa 导出包
	xcodebuild -exportArchive -archivePath WholeNet.xcarchive \						  -exportPath WholeNet \
				  -exportFormat ipa \								  -exportProvisioningProfile WholeNet2 \
				  uploadBitcode NO \
			          || failed "xcodebuild export archive" 
	sleep 15
	#sleep 25
	# move ipa to dest directory
	PAM=$(date)
	cd ~/Desktop/git/U || failed "cd ~/Desktop/git/U" 
	sleep 2
	rm -rf WholeNet.ipa || failed "rm ipa" 
	sleep 3
	mv ~/Desktop/U/CRM/code/ios/WholeNet/WholeNet.ipa . || failed "mv ipa"
	sleep 3
	git add WholeNet.ipa || failed "git add WholeNet.ipa"
	sleep 3
	git commit -m "$PAM.WholeNet" || failed "git commit -m"
	sleep 3
	git push || failed "git push"
	sleep 10
	echo "git push Success"
	 
	# clean bin files
	echo "clean xcarchive files ..."
	rm -rf ~/Desktop/U/CRM/code/ios/WholeNet/WholeNet.xcarchive
	sleep 3
	echo "clean bin Success."

	#echo "su -root"
	#expect -c "
	#spawn su - root
	#expect \"Password:\"
	#send \”123456\r\"
	#interact
	#"
	expect -c "
	spawn su - root
	expect \"Password:\"
	send \"123456\r\"
	interact
	"
	sleep 3

	su root

	sleep 3
	echo "发送邮件给suw@usuretech.com"
	echo 'hi,\n\nThe app is updated recently. Use the safari browser on iOS device to download the app. Here is the URL: http://www.udianfang.cn/downloadPage/ .\n\nThanks!' | mail -s 'iOS客户端更新' jia85860161@163.com
	#suw@usuretech.com >>build.log

# 开始你的一键打包发送吧


