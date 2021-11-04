---
title: "VMware虚拟机三种网络模式--NAT（地址转换模式）"
date: 2017-09-05
tags: ["VMware"]
draft: false
---


刚刚我们说到，如果你的网络ip资源紧缺，但是你又希望你的虚拟机能够联网，这时候NAT模式是最好的选择。NAT模式借助虚拟NAT设备和虚拟DHCP服务器，使得虚拟机可以联网。其网络结构如下图所示：
http://www.linuxidc.com/upload/2016_09/160926204664451.png![image_1bp8bod7519j79jp1iu51fd81r5c9.png-117.4kB][1]

在NAT模式中，主机网卡直接与虚拟NAT设备相连，然后虚拟NAT设备与虚拟DHCP服务器一起连接在虚拟交换机VMnet8上，这样就实现了虚拟机联网。那么我们会觉得很奇怪，为什么需要虚拟网卡VMware Network Adapter VMnet8呢？原来我们的VMware Network Adapter VMnet8虚拟网卡主要是为了实现主机与虚拟机之间的通信。在之后的设置步骤中，我们可以加以验证。

首先，设置虚拟机中NAT模式的选项，打开vmware，点击“编辑”下的“虚拟网络编辑器”，设置NAT参数及DHCP参数。

http://www.linuxidc.com/upload/2016_09/160926204664452.png![image_1bp8bp68n157310gckgu185a1oggm.png-67.6kB][2]
http://www.linuxidc.com/upload/2016_09/160926204664453.png![image_1bp8bpgv11ju4lp63rt1rdfhf213.png-47.9kB][3]
http://www.linuxidc.com/upload/2016_09/160926204664454.png![image_1bp8bppt8ipg133f36fgcl1rsb1g.png-37.8kB][4]

将虚拟机的网络连接模式修改成NAT模式，点击“编辑虚拟机设置”。
http://www.linuxidc.com/upload/2016_09/160926204664455.png![image_1bp8bqoc4jb240mlu9ghgf1t.png-116.8kB][5]

点击“网络适配器”，选择“NAT模式”
http://www.linuxidc.com/upload/2016_09/160926204664456.png![image_1bp8br8ml1ed41q5o1i4f1jf515012a.png-73.2kB][6]

然后，编辑网络IP地址：
http://www.linuxidc.com/upload/2016_09/160926204664458.png![image_1bp8brtgo1jv03s71jpi1vheiph2n.png-93.6kB][7]

之前，我们说过VMware Network Adapter VMnet8虚拟网卡的作用，那我们现在就来测试一下。
http://www.linuxidc.com/upload/2016_09/1609262046644510.png![image_1bp8bt4r61vl51kmodab1q5910tk34.png-121.8kB][8]

http://www.linuxidc.com/upload/2016_09/1609262046644511.png![image_1bp8btetku769gk1teq144j156d3h.png-106kB][9]
如此看来，虚拟机能联通外网，确实不是通过VMware Network Adapter VMnet8虚拟网卡，那么为什么要有这块虚拟网卡呢？

之前我们就说VMware Network Adapter VMnet8的作用是主机与虚拟机之间的通信，接下来，我们就用远程连接工具来测试一下。

http://www.linuxidc.com/upload/2016_09/1609262046644512.png![image_1bp8bu2mu1sv7ol815qam7g5v3u.png-124.7kB][10]

然后，将VMware Network Adapter VMnet8启用之后，发现远程工具可以连接上虚拟机了。

那么，这就是NAT模式，利用虚拟的NAT设备以及虚拟DHCP服务器来使虚拟机连接外网，而VMware Network Adapter VMnet8虚拟网卡是用来与虚拟机通信的。


[1]: http://static.zybuluo.com/huis/bkx07y96a0bfekq22t7pkqfb/image_1bp8bod7519j79jp1iu51fd81r5c9.png
[2]: http://static.zybuluo.com/huis/od7srtq5i44nbr844naqdtzh/image_1bp8bp68n157310gckgu185a1oggm.png
[3]: http://static.zybuluo.com/huis/75z2m87rlamiccxzo17c7qq3/image_1bp8bpgv11ju4lp63rt1rdfhf213.png
[4]: http://static.zybuluo.com/huis/rhu2raft9jwn30cuazlrebad/image_1bp8bppt8ipg133f36fgcl1rsb1g.png
[5]: http://static.zybuluo.com/huis/61d4visggha4kjurqwlrhd37/image_1bp8bqoc4jb240mlu9ghgf1t.png
[6]: http://static.zybuluo.com/huis/rlfibzkp15o6xvgj87gnkbn8/image_1bp8br8ml1ed41q5o1i4f1jf515012a.png
[7]: http://static.zybuluo.com/huis/5h4mq63cncfflv1cme37nwng/image_1bp8brtgo1jv03s71jpi1vheiph2n.png
[8]: http://static.zybuluo.com/huis/sdxu3v2mg7sov8pf6rq63krt/image_1bp8bt4r61vl51kmodab1q5910tk34.png
[9]: http://static.zybuluo.com/huis/e8n8wsshybgbwahrw2jvzci2/image_1bp8btetku769gk1teq144j156d3h.png
[10]: http://static.zybuluo.com/huis/dv4h5b5sjixrxzpwkj73qns5/image_1bp8bu2mu1sv7ol815qam7g5v3u.png