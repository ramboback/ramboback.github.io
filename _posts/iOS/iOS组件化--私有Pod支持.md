---
layout: post
title:  "私有库Pod支持"
date:   2019-12-17 13:30:21 +0800
categories: iOS
---
### iOS工程Pod私有库支持(组件化依赖)
首先在动手前,将整体的思路抽象整理一下:目前利用github仓库实现,可同时应用在码云和gitlab上,适合将公司内部高度可定制化的模块抽取,按需应用.
demo步骤如下:

	1.创建索引库.
	2.创建工程库.
	3.工程库增加pod支持.
	4.发布工程到索引库支持.
	5.引用工程库
	
### 创建私有索引库
提供私有库的索引管理仓库,我在github上创建一个空仓库如下:

	https://github.com/ramboback/mySpecs.git

### 本地添加私有库

	pod repo add mySpecs https://github.com/ramboback/mySpecs.git
	
成功添加后,~/.cocoapods/repos路径下将多出mySpecs索引库.

### 创建工程库
	https://github.com/ramboback/RamboModulePodTest.git
	
clone对应工程库到本地,本地工程目录为:~/Documents/test/
	
	git clone https://github.com/ramboback/RamboModulePodTest.git	
	
	
### 创建工程库Pod支持
	pod lib create RamboModulePodTest
	
我选择的配置选项为(参考):
	
	1.platform  :    iOS
	2.language  :     ObjC
	3.demo application : Yes
	4.testing frameworks : None
	5.view based testing : Yes
	6.class prefix : Rambo
	

### 配置podspec文件
配置文件属性很好理解,校验一下各个路径和名称是否一致即可,然后验证一下是否可用:
	
	pod lib lint	
若可用则返回:passed validation字样.

### 关联本地和远程工程仓库
	git add .
	git commit -m "first commit"
	git remote add origin https://github.com/ramboback/RamboModulePodTest.git
	git push -u origin master
	
### 发布工程代码到索引库
发布工程版本
	
	git tag "0.1.0"
	git push --tags
	
添加到私有索引库中

	pod repo push mySpecs RamboModulePodTest.podspec
	
若成功关联,远程索引库显示该工程索引:

	mySpecs/RamboModulePodTest/0.1.0/
	
本地索引库也将显示该工程索引:

	~/.cocoapods/repos/mySpecs/RamboModulePodTest
至此,一个完整的pod支持流程走完了.

### 工程引用
在工程podfile文件中,将私有的索引库源关联:

	source 'https://github.com/ramboback/mySpecs.git'
然后将工程集成:

	pod 'RamboModulePodTest', '~> 0.1.0'
然后构建该库:
	
	pod install --repo-update
### 更新版本库
	(工程代码推荐放置路径为:.../RamboModulePodTest/Classes/)
新增一个test.h 和test.m文件

修改podspec文件s.version,推荐和tag保持一致

给版本打上tag
	
	git tag "0.1.1"
	git push --tags
发布当前新版本
	
	pod repo push mySpecs RamboModulePodTest.podspec
	
最后,把工程代码关联到远程仓库

	git add .
	git commit -m "update version"
	git push origin master
注意:发布版本库首先发布索引,其后发布工程代码,否则会报找不到当前的索引分支错误.