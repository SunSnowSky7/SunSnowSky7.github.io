---
title: ESP32学习笔记
date: 2023-10-07 17:46:33
tags: ESP32 MACOS

---

# 1. 环境搭建
平台：MacBook Pro   
软件：vscode + ESP-IDF  
硬件：ESP32 WROOM-32、macbook 拓展坞、USB数据线（连接esp32和笔记本）    
<!-- more -->	

## 1.1 下载安装vscode软件

[Mac版本的vscode](https://code.visualstudio.com/sha/download?build=stable&os=cli-darwin-arm64)

下载安装的过程省略，因为我会，所以默认都会了。  
下载完成之后，记得安装中文插件，当然英语好的老6可以忽略。	
![image12](image-12.png)

安装完中文插件重启之后，就会显示中文字体。

## 1.2 安装esp-idf 插件
重新打开vscode，然后在扩展中搜索espress,见下图：
![Alt text](image-13.png)


安装完成后,在左侧点击刚安装好的扩展程序（乐鑫的图标），初始界面如下图
![Alt text](esp1.png)

选择`EXPRESS`,然后按照下图选择并安装esp-idf的安装包，选择好之后，点击左下角的`install`
![esp](esp13.png)

安装中。。。视网络环境看，大概需要20分钟左右下载安装完毕；  
![esp2](esp14.png)

等出现下图中的日志时，说明安装成功。
![esp3](esp15.png)

到此，**环境搭建好了～撒欢儿～哏儿～**

# 2. 第一次例程下载
## 2.1 hello_world 横空出世
作为程序猿（媛），学习新知识的一个案例必须是`hello world`.

先点击左右乐鑫的图标，然后点击F1，出来一个搜索框，然后输入`idf:show` ,看下图：  
![Alt text](image-15.png)

如果出现了下面选择路径的情况的话，直接选择自己安装的路径即可。
![Alt text](image-16.png)

111111111111111
![esp20](eps20.png)

选择保存的路径：
![](esp21.png)
即可看到下图所示的工程，箭头所指即为主程序，
![](esp231.png)
**重点！！！重点！！！这张图一定要记住**
![](eps24.png)
按照上图的配置，我这里修改的地方如下：
- 串口选择的是`/dev/cu.usbserial-0001`
- 芯片类型选择的 `esp32`
- flash method 选择的是 `UART`
配置好之后，点击编译，如下图所示：
![](eps25.png)

点击⚡️按钮将程序下载到开发板里面去，下图为下载完成之后的亚子～
![](esp26.png)

如上图下载完成之后，电机那个小电视图标，就可以看到开发板打印的东西了。      
**完美～～～    perfect ！！！**
![](esp27.png)
