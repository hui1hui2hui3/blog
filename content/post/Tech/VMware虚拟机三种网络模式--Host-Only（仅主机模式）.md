---
title: "VMware虚拟机三种网络模式--Host-Only（仅主机模式）"
date: 2017-09-05
tags: [" VMware"]
draft: false
---


Host-Only模式其实就是NAT模式去除了虚拟NAT设备，然后使用VMware Network Adapter VMnet1虚拟网卡连接VMnet1虚拟交换机来与虚拟机通信的，Host-Only模式将虚拟机与外网隔开，使得虚拟机成为一个独立的系统，只与主机相互通讯。其网络结构如下图所示：
http://www.linuxidc.com/upload/2016_09/160926204874121.png![image_1bp8c2n2u7l9qu9159rno1gbs9.png-108.6kB][1]

通过上图，我们可以发现，如果要使得虚拟机能联网，我们可以将主机网卡共享给VMware Network Adapter VMnet1网卡，从而达到虚拟机联网的目的。接下来，我们就来测试一下。

首先设置“虚拟网络编辑器”，可以设置DHCP的起始范围。

http://www.linuxidc.com/upload/2016_09/160926204874122.png![image_1bp8c3baeq5krpv8uv1i6qefom.png-89.5kB][2]

设置虚拟机为Host-Only模式。

http://www.linuxidc.com/upload/2016_09/160926204874123.png![image_1bp8c3r8pv0m1h081l00152s8eq13.png-77.3kB][3]

设置网络IP：
http://www.linuxidc.com/upload/2016_09/160926204874124.png![image_1bp8c4q4jtc415r6m4mfgrafo1g.png-93.1kB][4]

利用远程工具测试能否与主机通信。
http://www.linuxidc.com/upload/2016_09/160926204874125.png![image_1bp8c5bd6jtfn849d47gf3md1t.png-103.1kB][5]

主机与虚拟机之间可以通信，现在设置虚拟机联通外网。
http://www.linuxidc.com/upload/2016_09/160926204874126.png![image_1bp8c5nmn15i811fs1qgd12mvb542a.png-195.2kB][6]

我们可以看到上图有一个提示，强制将VMware Network Adapter VMnet1的ip设置成192.168.137.1，那么接下来，我们就要将虚拟机的DHCP的子网和起始地址进行修改，点击“虚拟网络编辑器”
http://www.linuxidc.com/upload/2016_09/160926204874127.png![image_1bp8c690u1lc910091uoavjqbq2n.png-86.2kB][7]

重新配置网卡，将VMware Network Adapter VMnet1虚拟网卡作为虚拟机的路由。
http://www.linuxidc.com/upload/2016_09/160926204874128.png![image_1bp8c6ot9h31oo95lerg27c734.png-89.6kB][8]

然后通过 远程工具测试能否联通外网以及与主机通信。
http://www.linuxidc.com/upload/2016_09/160926204874129.png![image_1bp8c7c3pkeq1rsj100s1tfg1dmc3h.png-141.8kB][9]

测试结果证明可以使得虚拟机连接外网。


[1]: http://static.zybuluo.com/huis/wfw8w3r2q92j1ribl779uxrn/image_1bp8c2n2u7l9qu9159rno1gbs9.png
[2]: http://static.zybuluo.com/huis/0go4icwq4jdc77yahz62ilws/image_1bp8c3baeq5krpv8uv1i6qefom.png
[3]: http://static.zybuluo.com/huis/zxo098ombzllksnuxy5ekn09/image_1bp8c3r8pv0m1h081l00152s8eq13.png
[4]: http://static.zybuluo.com/huis/0bemhm3b2euvl7ai703or7e7/image_1bp8c4q4jtc415r6m4mfgrafo1g.png
[5]: http://static.zybuluo.com/huis/49iveqjrwgzczuqjlww5a2td/image_1bp8c5bd6jtfn849d47gf3md1t.png
[6]: http://static.zybuluo.com/huis/s08nfdeemfd5eimymrjz7rp3/image_1bp8c5nmn15i811fs1qgd12mvb542a.png
[7]: http://static.zybuluo.com/huis/i3kblulpmfakwosqxqjvv8mt/image_1bp8c690u1lc910091uoavjqbq2n.png
[8]: http://static.zybuluo.com/huis/47a7bvhgh3ckpauxhxwckkxm/image_1bp8c6ot9h31oo95lerg27c734.png
[9]: http://static.zybuluo.com/huis/15lvpmmkmng065vyw0bl2m6f/image_1bp8c7c3pkeq1rsj100s1tfg1dmc3h.png