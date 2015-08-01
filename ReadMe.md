OpenWRT应用于西电北校区实现校内网路由器上网
=======================================

作者：Steven 邮箱：<qin.steven@foxmail.com>
- - -

**关键字：**

+ **OpenWRT**
+ **西电北校区**
+ **校内网路由上网**
+ **OpenWRT拓展及深入**
+ **资料搜集**

---

一、背景说明
-----------

西电宿舍区并没有无线覆盖，而校内网路由器无法直接上网。我手头的路由器**TP-Link WR703n**一直没能发挥作用。于是我就想通过刷路由器来实现无线上网。查了一些资料，尝试了几次，日前（**2015/07/26**）终于将路由器配置更新完毕，目前正常使用校内网。

在“折腾”的过程中，参考了许多资料，对OpenWRT也有了一个基本的认识，所以就有了这个页面。本文内容可以分成两个部分，其一是关于更新、配置OpenWRT以支持校内网的资料，其二是把关于OpenWRT的资料做一个归类，这些资料涉及到OpenWRT的历史、使用与开发，希望我的劳动能给其他人带去方便。

二、资料整理
-----------

###TP-Link WR703n路由器介绍

我的路由器是**TP-Link WR703n**，针对该路由器，在**OpenWRT**官网上有具体的介绍[WR703n介绍](http://wiki.openwrt.org/zh-cn/toh/tp-link/tl-wr703n)。如果你想知道手头的路由器是否支持OpenWRT，可以通过[这个页面](http://wiki.openwrt.org/toh/start)进行查看，该页面中列出的是具体的路由器型号，简单在页面内 **Ctrl+F** 输入路由器型号，进行查找即可判断某一款路由器是否支持OpenWRT。当然，你也可以根据**芯片的型号**来判断某一款路由器是否支持OpenWRT，[该页面](https://downloads.openwrt.org/backfire/10.03/)中是芯片型号信息。

###刷机以及基本配置

知道自己的路由器是否支持OpenWRT后，可以参考[官方文档](http://wiki.openwrt.org/doc/howto/generic.flashing)进行刷机，当然，网络上已经有许多现成的刷机教程了（谢谢相关教程作者），几个不错的有：

+ [OpenWrt智能、自动、透明翻墙路由器教程][link-include-crosswall]
+ [从零开始学习OpenWrt：刷機 + 使用 + 編譯教程](http://upsangel.com/openwrt/openwrt-beginner-guide/)
+ [WR703N OpenWrt 配置流程](https://gist.github.com/ninehills/2627163)
+ [TP-LINK WR703N 刷OpenWrt并设置pppoe联网、安装LUCI、添加新用户、挂载USB设备、配置ftp服务、借由transmission实现脱机下载](http://blog.csdn.net/qinpeng_zbdx/article/details/8570488)

[link-include-crosswall]: http://softwaredownload.gitbooks.io/openwrt-fanqiang/content/

大概浏览上面的资料后，其实从最初的刷机、配置到高阶应用应该就足够了解了。对于上文中涉及到的内容，此处就不再提及了，如果你没有时间都看，就看最后一个链接的内容，该博文作者基本上一步步写明了专门针对于OpenWRT的刷机、配置流程。

**总结一下**，其实整个刷机还是很简单的，

1. 查找适配相应路由器的固件。
2. 下载相应固件，登陆原路由器管理界面使用factory固件进行第一次固件升级。
3. 使用sysupgrade固件进行第二次固件升级，命令：**mtd -r write /tmp/xxx.bin firmware**，当然，需要先将该固件通过WinSCP传到/tmp目录下。
4. 刷完机使用telnet协议初次登陆路由器，通过passwd命令修改root密码，随后即可通过SSH协议访问路由器进行下一步配置。

这样整个刷机流程就结束了。

**备注**，下面这些细节请注意，

1. 刚刷完的路由器默认不开启无线，且LAN口默认IP为192.168.1.1，所以需要设置PC本地连接IP为同一网段，然后借由网线连接路由器。
2. 如果刷机后发现没有*传说中的*LUCI管理界面，这说明你所使用的不是稳定版本的固件。OpenWRT的固件版本中，稳定版都是默认集成LUCI管理界面的，[这是OpenWRT的版本情况](http://wiki.openwrt.org/about/history)。
3. 如果你已经安装了一个没有管理界面的OpenWRT，并且已经联网成功，参考这个链接[安装管理界面](https://www.logcg.com/archives/608.html)。
4. 如果你发现无法正确更新软件源，请参考这个链接[手工修改软件源](http://sharpbai.gitbooks.io/tp-720n-openwrt-usb-boot-and-install-rtl-usb-wifi/content/opkg/README.html)。
5. [这个链接](http://wiki.openwrt.org/doc/packages)介绍了OpenWRT的可安装软件情况，这个链接是OpenWRT的[OPKG软件包管理工具使用说明](http://wiki.openwrt.org/doc/techref/opkg)。

**重要**，一不小心配置出错了，

进入安全模式进行系统恢复即可，[系统恢复方法](http://wiki.openwrt.org/zh-cn/doc/howto/generic.failsafe)。

**重要**，一不小心刷机失败了，路由器成砖了，

我自己在尝试的时候就遇到了这个情况。当时我下载了错误的固件版本，刷机完毕重启动后路由器的指示灯没有常亮，而是呈现出循环狂闪，应该是不断在U-boot里打转。

遇到这种情况就需要通过串口来进行重新刷机了。

1. 引出OpenWRT的串口线；
2. 使用USB-TTL串口小板连接PC USB口与OpenWRT所引出的串口线；
3. 使用串口助手配合tftpd32软件进行串口刷机。

**附加**，

1. 串口线引出示意图如下所示：

![image-ttl](https://raw.githubusercontent.com/KeepSilenceQP/XidianOpenWRTConfig/master/images/TTL.jpg)

2. 串口小板如下所示：

![image-usb2ttl](https://raw.githubusercontent.com/KeepSilenceQP/XidianOpenWRTConfig/master/images/USB2TTL.jpg)

3. 具体命令参考：

[一般救砖](http://blog.csdn.net/u011582412/article/details/19492983)

[编程器救砖](http://yfrobot.com/thread-2225-1-1.html)

![编程器长这个样子](https://raw.githubusercontent.com/KeepSilenceQP/XidianOpenWRTConfig/master/images/bianchengqi.jpg)

什么？你后悔了？要**刷回原厂固件**，好的，

[TP-Link WR703n官网](http://www.tp-link.com.cn/product_225.html)下载相关原厂固件，刷回来即可。

以上内容有些拖沓、冗杂，但是比较全面，实际上，**当你完成了基本配置，可以通过SSH经由putty登陆OpenWRT系统后**，就可以看下面的内容了。

三、针对西电北校区配置说明
-----------------------
**如果你只想简单使用一下OpenWRT在西电登录校内网，看这部分说明就可以了。**

1. 在[百度网盘](http://pan.baidu.com/s/1o667WvS)下载软件包，解压后得到三个ipk文件，将其通过WinSCP拷贝到/tmp目录下；
2. 先安装libpcap库，再安装客户端，最后安装客户端Web界面；
3. 新增Wan口并配置其为DHCP；
4. 使用命令登录校内网。

详细说明：

1. 新增Wan口需要将配置文件/etc/config/network进行如下配置：

    	config interface 'loopback'
    		option ifname 'lo'
    		option proto 'static'
    		option ipaddr '127.0.0.1'
    		option netmask '255.0.0.0'
    
    	config interface 'lan'
    		#option ifname 'eth0' # 注释该行
    		option type 'bridge'
    		option proto 'static'
    		option ipaddr '192.168.1.1'
    		option netmask '255.255.255.0'

    	# 添加以下行
    	config interface 'wan'
    		option proto 'dhcp' # DHCP协议
    		option ifname 'eth0'

<a name="mac" id="mac">

	option macaddr '00:05:5D:5E:46:43' # 绑定的MAC

</a>

2. 开启无线需要将/etc/config/wireless做如下配置：

    	config wifi-device 'radio0'
    		option type 'mac80211'
    		option channel '11'
    		option macaddr 'ec:17:2f:e3:b4:c8'
    		option hwmode '11ng'
    		option htmode 'HT20'
    		list ht_capab 'SHORT-GI-20'
    		list ht_capab 'SHORT-GI-40'
    		list ht_capab 'RX-STBC1'
    		list ht_capab 'DSSS_CCK-40'
    		option txpower '27'
    		option country 'US'
    
    	config wifi-iface
    		option device 'radio0'
    		option network 'lan'
    		option mode 'ap'
    		option ssid 'wifi名'
    		option encryption '加密方式'
    		option key '密码'
    		option hidden '1' # 隐藏wifi名

注意一个地方，[修改MAC地址](#mac)

3. 进行完毕上述配置后，就可以重启路由器，使用无线连接路由。连接路由器成功后，使用putty登录路由器，然后输入如下命令接入校内网：

    	njit-client 账户名 账户密码 网卡(eth0 etc) &(后台登录)

4. 可以配置一个开机自启动来在路由开机后自动联网，内容如下所示：

    	#!/bin/sh /etc/rc.common
    
    	START=50
    
    	start() {
    		njit-client 帐号名 密码 网卡 &
    	}
    
    	stop() {
    		killall njit-client
    		killall udhcpc
    	}

将该文件保存到init.d目录下，

    chmod +x 文件名 // 赋予可执行权限
    ./文件名 enable // 开机自启动

四、拓展
-------

###关于越墙

如果你完成了上述操作，成功使用路由器登陆使用校内网，还想继续配置一下，实现自动翻墙，除了上面提到的[OpenWrt智能、自动、透明翻墙路由器教程][link-include-crosswall]，还可以参考[这篇文章](https://cokebar.info/archives/850)进行越墙配置。

###关于深入OpenWRT

**OpenWRT以Linux为内核，可以进行相关开发工作。以下是一些资料：**

官方的[开发者中心](https://dev.openwrt.org/)。

这个博客讲了不少[在OpenWRT上安装使用各种软件](http://www.haiyun.me/category/openwrt/)的资料。

交叉编译的介绍[Cross Compile](http://wiki.openwrt.org/doc/devel/crosscompile)。

[Using the SDK](http://wiki.openwrt.org/doc/howto/obtain.firmware.sdk#build_your_own_packages)：用来为特定版本编译userspace软件包。

[Creating packages](http://wiki.openwrt.org/doc/devel/packages)：编译自己的软件包。

[OpenWrt Buildroot](http://wiki.openwrt.org/about/toolchain)：OpenWRT提供的交叉编译工具链。

[How to Build a Single Package](http://wiki.openwrt.org/doc/howtobuild/single.package)：文如其名。

[Developing](http://wiki.openwrt.org/doc/devel/start)：总结OpenWRT相关各种开发文档。

[OpenWrt Development Guide](http://www.ccs.neu.edu/home/noubir/Courses/CS6710/S12/material/OpenWrt_Dev_Tutorial.pdf)：OpenWRT开发指导文档。

**什么？**你不想看这么多没有总结性的文章？

**好，**这是教程：

1. [OpenWrt环境下做开发](http://wiki.wrtnode.com/index.php?title=Openwrt_development/zh-cn)
2. [用openwrt编译器交叉编译](http://wiki.wrtnode.com/index.php?title=How_to_compile_with_the_openwrt_toolchain/zh-cn)

**又怎么了？**这些文档不系统？

**好，**我是好人，收好，

1. [编译OpenWrt - 索引](http://blog.csdn.net/openme_openwrt/article/details/7348452)
2. [上文作者的一系列OpenWRT开发文章](http://blog.csdn.net/column/details/openwrt.html?&page=2)
3. [MicroWRT 应用教程](https://www.microduino.cc/wiki/index.php?title=%E5%BA%94%E7%94%A8%E6%95%99%E7%A8%8B)([MicroWRT](https://www.microduino.cc/wiki/index.php?title=MicroWRT_Getting_started/zh)是兼容OpenWRT的一款硬件学习版，该教程**第一部分**讲解OpenWRT的开发。)

**什么？只有文字看起来太累？**

**好，**给你视频，收好：

[其中有部分视频，需要翻墙看](http://study.163.com/plan/planIntroduction.htm?id=1549012#/planMain)

剩下的开发就是Linux C/C++开发了。

五、权利声明
-----------

本文所有权利属于广大网友。
