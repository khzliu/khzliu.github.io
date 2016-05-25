---
title: 使用cocoapods管理第三方库
tags: [iOS]
categories: iOS

---

# CocoaPods简介
[CocoaPods](https://cocoapods.org/)是一个负责管理iOS项目中第三方开源库的工具。CocoaPods的项目源码在[Github](https://github.com/CocoaPods/CocoaPods)上管理。该项目开始于2011年8月12日，在这两年多的时间里，它持续保持活跃更新。开发iOS项目不可避免地要使用第三方开源库，CocoaPods的出现使得我们可以节省设置和更新第三方开源库的时间。
# CocoaPods安装
Cocoapod的安装网上的例子一大堆，这里及时简单列举几个步骤以便日后使用。
## 安装Ruby环境 
Mac OS 自带Ruby环境，所以只需要升级一下Ruby版本就可以了<br/>
终端输入如下命令（把Ruby镜像指向taobao，避免被墙，你懂得）<br/>
淘宝已经关闭HTTP协议的景象服务，改为HTTPS协议。<br/>
淘宝ruby地址：`https://ruby.taobao.org/` <br/>


	gem sources --remove https://rubygems.org/ 
	gem sources -a https://ruby.taobao.org/ 
	gem sources -l  （用来检查使用替换镜像位置成功）
	
## 安装cocoapods
下载安装CocoaPods,终端输入：
	
	sudo gem install cocoapods 
	
如果网速可以的话（没有被墙），几分钟就下载安装完成了。
# CocoaPods使用
我们在桌面新建一个项目，名字叫`PodTest`, Xcode会生成 `PodTest`,`PodTest.xcodeproj`和`PodTestTests`三个文件夹。<br/>
终端中，cd到项目总目录（注意：包含`PodTest`文件夹、`PodTest.xcodeproj`、`PodTestTest`的那个总目录）

	cd /Users/xxx/Desktop/PodTest

<font color="red">注意：`xxx` 代表的是你的账户名。</font>

建立`Podfile`（配置文件）
接着上一步，终端输入 
	
	vim Podfile
	
开始编辑`Podfile`文件内容如下：

	platform :ios, '7.0'

	target "PodTest" do
	pod "Masonry", "~> 1.0.0"
	pod "GPUImage", "~> 0.1.7"
	pod "MBProgressHUD", "~> 0.9.2" 
	end

激动人心的时刻到了：确定终端cd到项目总目录，然后输入 pod install，等待一会，大约3分钟。
	
	pod install
	
完成之后你代码cocoapods给你创建好的WorkSpace就可以了。<br/>
<br/>
<font color="red">注意：</font>现在打开项目不是点击 `PodTest.xodeproj` 了，而是点击 `PodTest.xcworkspace`

如果你想要下载其它库，但是你又不知道这个库的版本好 可以输入：
	
	pod search xxxx
	
# cocoapods常见错误

OS X 10.11 安装Cocoapods 出现问题的解决方法:

今天尝试用 Cocoapods安装个第三方库.. 输入`pod install`， 发现 `command not find`。 WTF！

估计是升级10.11后Cocoapods被干掉了。

我输入 `sudo gem install cocoa pods` 之后，出现如下问题：

	ERROR:  While executing gem ... (Gem::DependencyError)
	    Unable to resolve dependencies: cocoapods requires cocoapods-core (= 0.33.1), claide (~> 0.6.1), cocoapods-downloader (~> 0.6.1), cocoapods-plugins (~> 0.2.0), cocoapods-try (~> 0.3.0), cocoapods-trunk (~> 0.1.1), nap (~> 0.7)

解决方法：

	sudo gem update --system
  
但是出现了另一个错误：

	ERROR:  While executing gem ... (Errno::EPERM)
	    Operation not permitted - /usr/bin/xcodeproj

在Stackoverflow上找到了解决方法：

在终端中输入：

	sudo nvram boot-args="rootless=0"; sudo reboot
然后你的电脑会重启
之后再输入 

	sudo gem install cocoapods -V 
就可以了
不放心的话输入

	pod --version
	0.37.2 //显示出版本就说明成功了

但是之后我`pod install`的时候又花式出错

	[!] Unable to add a source with url `https://github.com/CocoaPods/Specs.git` named `master`.
	You can try adding it manually in `~/.cocoapods/repos` or via `pod repo add`.

我尝试按提示的方法

	pod repo add master https://github.com/CocoaPods/Specs.git

然而还是有错..

	[!] /usr/bin/git clone http://git.oschina.net/akuandev/Specs.git master
	
	xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun

最后的解决方法：

	sudo xcode-select -switch /Applications/Xcode-beta.app/Contents/Developers

后面的地址你可以打开Xcode显示包内容，找到那个文件夹拖到终端里面比较不容易错。

	CocoaPods 0.38.0.beta.2 is available.
	To update use: `gem install cocoapods --pre`
	[!] This is a test version we'd love you to try.
	
	For more information see http://blog.cocoapods.org
	and the CHANGELOG for this version http://git.io/BaH8pQ.
	
	
	CocoaPods 0.38.0.beta.2 is available.
	To update use: `gem install cocoapods --pre`
	[!] This is a test version we'd love you to try.
	
	For more information see http://blog.cocoapods.org
	and the CHANGELOG for this version http://git.io/BaH8pQ.

最后终于修成正果..

参考：
> http://blog.csdn.net/showhilllee/article/details/38398119/
> http://www.myexception.cn/operating-system/1962318.html