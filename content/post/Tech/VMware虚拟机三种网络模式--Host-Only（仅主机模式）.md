---
title: "VMware虚拟机三种网络模式--Host-Only（仅主机模式）"
date: 2017-09-05
tags: [" VMware"]
draft: false
---


Host-Only模式其实就是NAT模式去除了虚拟NAT设备，然后使用VMware Network Adapter VMnet1虚拟网卡连接VMnet1虚拟交换机来与虚拟机通信的，Host-Only模式将虚拟机与外网隔开，使得虚拟机成为一个独立的系统，只与主机相互通讯。其网络结构如下图所示：
![image_1bp8c2n2u7l9qu9159rno1gbs9.png-108.6kB](/blog/image/image_1bp8c2n2u7l9qu9159rno1gbs9.png)

通过上图，我们可以发现，如果要使得虚拟机能联网，我们可以将主机网卡共享给VMware Network Adapter VMnet1网卡，从而达到虚拟机联网的目的。接下来，我们就来测试一下。

首先设置“虚拟网络编辑器”，可以设置DHCP的起始范围。

![image_1bp8c3baeq5krpv8uv1i6qefom.png-89.5kB](/blog/image/image_1bp8c3baeq5krpv8uv1i6qefom.png)

设置虚拟机为Host-Only模式。

![image_1bp8c3r8pv0m1h081l00152s8eq13.png-77.3kB](/blog/image/image_1bp8c3r8pv0m1h081l00152s8eq13.png)

设置网络IP：
![image_1bp8c4q4jtc415r6m4mfgrafo1g.png-93.1kB](/blog/image/image_1bp8c4q4jtc415r6m4mfgrafo1g.png)

利用远程工具测试能否与主机通信。
![image_1bp8c5bd6jtfn849d47gf3md1t.png-103.1kB](/blog/image/image_1bp8c5bd6jtfn849d47gf3md1t.png)

主机与虚拟机之间可以通信，现在设置虚拟机联通外网。
![image_1bp8c5nmn15i811fs1qgd12mvb542a.png-195.2kB](/blog/image/image_1bp8c5nmn15i811fs1qgd12mvb542a.png)

我们可以看到上图有一个提示，强制将VMware Network Adapter VMnet1的ip设置成192.168.137.1，那么接下来，我们就要将虚拟机的DHCP的子网和起始地址进行修改，点击“虚拟网络编辑器”
![image_1bp8c690u1lc910091uoavjqbq2n.png-86.2kB](/blog/image/image_1bp8c690u1lc910091uoavjqbq2n.png)

重新配置网卡，将VMware Network Adapter VMnet1虚拟网卡作为虚拟机的路由。
![image_1bp8c6ot9h31oo95lerg27c734.png-89.6kB](/blog/image/image_1bp8c6ot9h31oo95lerg27c734.png)

然后通过 远程工具测试能否联通外网以及与主机通信。
![image_1bp8c7c3pkeq1rsj100s1tfg1dmc3h.png-141.8kB](/blog/image/image_1bp8c7c3pkeq1rsj100s1tfg1dmc3h.png)

测试结果证明可以使得虚拟机连接外网。