---
title: 通过dSYM分析crash Log
tags: [Xcode]
categories: Xcode基础

---

在项目开发过程中，对于已经上线的App有时会莫名其秒的Crash，切这些Crash在Xcode上是无法复现，导致了我们无法通过Xcode进行调试。还好Device能够记录这些crash日志，我们可用通过Xcode导出这些日志，但是这些日志记录的内容却令人头疼了，我们根据日志的内容（全是些地址信息）无法定位到代码的哪一行产生的错误，我们就需要dSYM文件来帮我们定位分析Crash log了。<br/>

### 使用步骤
#### 1. 导出错误日志 
通过Xcode的Organizer查看设备的Logs，并把你要查看的Log导出到文件夹reportAnalysis,命名app.crash <br/>
#### 2. 导出App的可执行文件
找到该App当前版本的ipa文件，解压后得到Payload文件夹，并把xxx.app复制到reportAnalysis<br/>
#### 3. 导出App对应build版本的dSYM文件
dSYM文件可通过Xcode的Organizer找到对应build版本，右键_show in finder_，并显示该版本的b包内容，文件夹dSYMs内就是该App的dSYM文件，把xxx.app.dSYM文件拷贝到reportAnalysis<br/>
#### 4. 确定dSYM，app和Crash文件的关系
每一个xx.app, xxx.app.dSYM文件都拥有相应的uuid，crash文件也有uuid,只有三者uuid一至才表明之三者可以解析出正确的日志文件。
查看xx.app文件的uuid的方法，在terminal中输入命令：

	dwarfdump --uuid xxx.app/xxx (xxx工程名)

查看xx.app.dSYM文件的uuid的方法，在terminal中输入命令：

	dwarfdump --uuid xxx.app.dSYM (xxx工程名) 


而.crash的uuid位于，crash日志中的Binary Images:中的第一行。如：

	armv7 <8bdeaf1a0b233ac199728c2a0ebb4165> 

对应的<a style="color:#F00">xxx.app.dSYM</a>文件以及<a style="color:#F00">xxx.app</a>文件以及<a style="color:#F00">xxx.crash</a>文件拷贝到同一文件夹中，如：

	~/Desktop/reportAnalysis
	
#### 5. 通过symbolicatecrash分析crash文件
Xcode有自带的symbolicatecrash工具,可以通过dSYM文件将crash文件中的16进制地址转换成可读的函数地址。symbolicatecrash工具位于:

	/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKit.framework/Versions/A/Resources/symbolicatecrash(Xcode 4.5)

	/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash(Xcode 5.0,Xcode 6.0)

	/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash(Xcode 7.0)

该文件是隐藏文件，可以通过如下命令查找并拷贝到系统目录下，并建立快捷方式。


###### 5.1 打开终端，进入到symbolicatecrash工具所在的文件夹目录

	cd /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/

<br/>

###### 5.2 查找确认是否存在symbolicatecrash

	ls -al | grep symbolicatecrash

###### 5.3 将symbolicatecrash工具拷贝到/usr/bin目录下或者讲symbolicatecrash拷贝到reportAnalysis文件夹内

	sudo cp symbolicatecrash /usr/bin/symbolicatecrash
or

	sudo cp symbolicatecrash ~/Desktop/reportAnalysis

###### 5.4 设置DEVELOPER_DIR系统变量

	cd ~/
	
	vi .bash_profile

并输入如下内容

	export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"

保存并退出

	source .bash_profile

###### 5.5 重启终端，确认是否已正确设置DEVELOPER_DIR系统变量

	echo $DEVELOPER_DIR

查看输出结果是否为/Applications/Xcode.app/Contents/Developer

###### 5.6 查看PATH系统变量是否存在如下路径/usr/bin

	echo $PATH

###### 5.7 如果PATH不存在如下路径/usr/bin，可在~/.bash_profile中添加如下代码

	export PATH="/usr/bin:$PATH"

保存并退出

	source .bash_profile

###### 5.8 上述准备工作完成后，进入dSYM和crash文件对应的文件夹目录，如

	cd ~/Desktop/DebugLog

###### 5.9 执行如下命令，即可正确解析crash文件

	symbolicatecrash xxx.crash xxx.app.dSYM > test.txt

<a style="color:#F00">注意：symbolicatecrash的参数顺序，否则会报类似如下错误:</a>

	Use of uninitialized value $data in substitution (s///) at /usr/bin/symbolicatecrash line 678.<br/>
	Use of uninitialized value $data in substitution (s///) at /usr/bin/symbolicatecrash line 681.<br/>
	Use of uninitialized value $data in substitution (s///) at /usr/bin/symbolicatecrash line 685.<br/>
	Use of uninitialized value in pattern match (m//) at /usr/bin/symbolicatecrash line 404.<br/>
	Use of uninitialized value in scalar assignment at /usr/bin/symbolicatecrash line 418.<br/>
	No crash report version in XXX.app.dSYM/ at /usr/bin/symbolicatecrash line 954.


> 参考：http://blog.csdn.net/hjy_x/article/details/20929095