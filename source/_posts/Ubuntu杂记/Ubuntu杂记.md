---
title: Ubuntu杂记
date: 2024-01-15 13:31:46
tags: Ubuntu Remmina 命令行Wi-Fi连接 Linux环境下更换启动项 
top: 9
---





# Remmina安装
<!-- more -->
```bash
------------ 在 Debian/Ubuntu 中 ------------ 
$ sudo apt-get install remmina remmina-plugin-*
------------ 在 CentOS/RHEL 中 ------------ 
# yum install remmina remmina-plugin-*
------------ 在 Fedora 22+ 中 ------------ 
$ sudo dnf copr enable hubbitus/remmina-next
$ sudo dnf upgrade --refresh 'remmina*' 'freerdp*'

```

# 命令行模式下进行Wi-Fi连接

## 确认无线网卡的名字
使用命令行 `iwconfig` 来确认无线网卡的名字，例如我的无线网卡为：wlp2s0
## 扫描当前可用Wi-Fi
```bash
sudo iw dev wlp2s0 scan | less  #其中wlp2s0是刚才的无线网卡的名字
```

如果报错iw找不到该命令时，可以使用下面命令安装iw
```bash
sudo apt install iw -y
```


## 设置登录 WIFI 和密码
通过如下命令行。
```bash
$sudo -i
# wpa_passphrase mywireless secretpassphrase > /etc/wpa_supplicant/wpa_supplicant.conf
特殊说明：
1、mywireless 表示你要登录的 wifi 名字
2、secretpassphrase 表示 wifi 登录的密码
```
上述命令创建了一个conf文件，可以使用cat查看；

## 加载配置文件，连接对应Wi-Fi
```bash
wpa_supplicant -i wlp2s0 -c /etc/wpa_supplicant/wpa_supplicant.conf -B
```
如果命令行返回：
```bash
    Successfully initialized wpa_supplicant 则成功连接对应Wi-Fi。
```
如果命令行返回：
```bash
nl80211: kernel reports: Match already configured
```
执行下述命令杀进程，再连接：
```bash
sudo killall wpa_supplicant
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```
大功告成。





# 在Ubuntu Linux上安装Chrome浏览器的最佳方法

```bash 
wget "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb" -O google.deb

sudo dpkg -i google.deb
```

# Linux入门之安装ubuntu的两种选择（双系统篇）

安装双系统后使用grub引导，默认启动项为ubuntu，且选择启动项界面等待时间长达10秒，虽然安装了双系统，但是常用的毕竟还是windows，这里提供一种将默认启动项改为windows并缩短等待时间的方法。在终端输入如下命令对grub进行编辑：
```bash
sudo vim /etc/default/grub
```
找到：
```shell
GRUB_DEFAULT=0

#GRUB_HIDDEN_TIMEOUT=0

GRUB_HIDDEN_TIMEOUT_QUIET=true

GRUB_TIMEOUT=10
```
- GRUB_DEFAULT的值就是默认启动项的位置值，以前面介绍的方式安装双系统后，windows一般是第五个启动项，所以我们把这里改为4（程序员的习惯）;
- GRUB_TIMEOUT就是选择启动项界面的等待时间，默认为10秒，根据喜好改成相应的秒数，更改完成后保存退出;
在终端输入
```bash 
sudo update-grub
sudo reboot
```
使更改生效，下次启动就默认为windows了。

























-------------------
[树莓派 wifi命令行连接没有报错但连接不上的问题
](https://blog.csdn.net/seveneagleline/article/details/120952404)

[Ubuntu使用命令行配置WIFI](https://blog.csdn.net/justidle/article/details/106585520)

[Linux入门之安装ubuntu的两种选择（双系统篇）
](https://zhuanlan.zhihu.com/p/25433899#:~:text=1%20%E5%AE%89%E8%A3%85%E5%8F%8C%E7%B3%BB%E7%BB%9F%E5%90%8E%E4%BD%BF%E7%94%A8grub%E5%BC%95%E5%AF%BC%EF%BC%8C%E9%BB%98%E8%AE%A4%E5%90%AF%E5%8A%A8%E9%A1%B9%E4%B8%BAubuntu%EF%BC%8C%E4%B8%94%E9%80%89%E6%8B%A9%E5%90%AF%E5%8A%A8%E9%A1%B9%E7%95%8C%E9%9D%A2%E7%AD%89%E5%BE%85%E6%97%B6%E9%97%B4%E9%95%BF%E8%BE%BE10%E7%A7%92%EF%BC%8C%E8%99%BD%E7%84%B6%E5%AE%89%E8%A3%85%E4%BA%86%E5%8F%8C%E7%B3%BB%E7%BB%9F%EF%BC%8C%E4%BD%86%E6%98%AF%E5%B8%B8%E7%94%A8%E7%9A%84%E6%AF%95%E7%AB%9F%E8%BF%98%E6%98%AFwindows%EF%BC%8C%E8%BF%99%E9%87%8C%E6%8F%90%E4%BE%9B%E4%B8%80%E7%A7%8D%E5%B0%86%E9%BB%98%E8%AE%A4%E5%90%AF%E5%8A%A8%E9%A1%B9%E6%94%B9%E4%B8%BAwindows%E5%B9%B6%E7%BC%A9%E7%9F%AD%E7%AD%89%E5%BE%85%E6%97%B6%E9%97%B4%E7%9A%84%E6%96%B9%E6%B3%95%E3%80%82%20%E5%9C%A8%E7%BB%88%E7%AB%AF%E8%BE%93%E5%85%A5%E5%A6%82%E4%B8%8B%E5%91%BD%E4%BB%A4%E5%AF%B9grub%E8%BF%9B%E8%A1%8C%E7%BC%96%E8%BE%91%EF%BC%9A%202%20sudo%20vim%20%2Fetc%2Fdefault%2Fgrub%20%E6%89%BE%E5%88%B0%EF%BC%9A,GRUB_DEFAULT%3D0%20%20%23GRUB_HIDDEN_TIMEOUT%3D0%20%20GRUB_HIDDEN_TIMEOUT_QUIET%3Dtrue%20%20GRUB_TIMEOUT%3D10%20)
