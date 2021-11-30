---
title: "HTTPS讲解"
date: 2016-12-13
tags: ["HTTPS"]
draft: false
---

## HTTP缺点
1. 通信使用的是明文，内容可能会被窃听
2. 不验证通信方的身份，有可能遭遇伪装
3. 无法证明报文的完整性，有可能已遭篡改

## 通信使用的是明文
**问题：**

    内容被窃听

**应对方法：**

1.通信加密

    HTTP通过和SSL（Secure Socket Layer,安全套接层）或TLS（Transport Layer Security,安全传输协议层）组合使用，加密通信内容,又被称为HTTPS（HTTP Secure,超文本传输安全协议）

2.内容加密

    将参与通信的内容进行加密，前提要求的是客户端和服务器同时具备加密和解密的机制，这个仍然有很大风险

## 不验证通信方的身份
**问题：**

    1. 无法确定请求发送至目标的服务器是否是真实返回响应的那台服务器。可能是已伪装的服务器。
    2. 无法确定响应返回的客户端是否是意图接受响应的哪个客户端。可能是已伪装的客户端
    3. 无法确定正在通信的对方是否具备访问权限。
    4. 无法判断请求来自哪儿，是谁发送的。
    5. 无法拦截下无意义的请求，即DoS攻击（Denial of Service, 拒绝服务攻击）

**应对方法：**

1.查明对手证书
![image_1b3f3rbrv79p9p7116b1ojcfau9.png-66.6kB][1]
如图：客户端查明服务端的证书合法性

## 无法证明报文完整性

这里说的完整性，主要指的是信息的准确度，无法证明其完整性，通常也就无法判断信息是否准确。
**问题：**

    接受到的内容可能有误

**应对方法：**

    MD5和SHA-1等散列值校验的方法或PDG（Pertty Good Privacy，完美隐私）数字签名

## HTTPS=HTTP+加密+认证+完整性保护
HTTPS并非是一种新的协议，只是披了SSL外壳的HTTP。
![image_1b3f7n2b31d4jairhm79cn112q9.png-20.6kB][2]

## 加密技术
### 共享密钥加密
加密和解密同用一个密钥，也叫做对称密钥加密。
![image_1b3f862eird8jjuj2fuca2iam.png-88.2kB][3]
所以发送密钥就有被窃听的风险，但不发送，对方就不能解密。再说，密钥若能安全发送，那数据也能安全送达了。
优点：处理速度快，效率高
缺点：不安全

### 公开密钥加密
公开密钥加密使用一对非对称的密钥。一把叫做私有密钥，另一把叫做公开密钥。私有密钥不能让任何人知道，公开密钥可以随意发布公开。 
![image_1b3re557m15uhrd61sc512dtpgl9.png-92.9kB][4]
优点：安全
缺点：处理速度慢，效率低


## HTTPS加密机制
HTTPS采用的是共享密钥加密和公开密钥加密两者并用的混合加密机制。
![image_1b3resb6eb0s1i6p1tedae4526m.png-128.4kB][5]

## 如何证明公开密钥是正确的
为了解决公开密钥的正确性，使用数字证书认证机构（CA，Certificate Authority）和其相关机关颁发的公开密钥证书。
![image_1b3rguvvepc21bl21dvn1hhd19vi13.png-155.3kB][6]

什么是数字签名？


    数字签名，是由先对内容进行单向散列函数（MD5,SHA1），再使用私钥进行加密，只能使用公钥解密。

那数字认证机构如何安全的把公开密钥给到客户端？

    浏览器开发商发布时，会事先再内部植入常用认证机关的公开密钥

## 客户端证书
以客户端证书进行客户端证书，证明服务器正在通信的对方是预料之中的客户端。
存在问题：证书的获取和发布
    
    客户端证书需要付费购买。另外让用户自行安装证书各种麻烦。

应用：银行的网上银行类似于U盾的东西


​    


[1]: http://static.zybuluo.com/huis/ph395jvu85ibv5zhczzuuhy1/image_1b3f3rbrv79p9p7116b1ojcfau9.png
[2]: http://static.zybuluo.com/huis/xw022r7nlcvp6p5sq26azezb/image_1b3f7n2b31d4jairhm79cn112q9.png
[3]: http://static.zybuluo.com/huis/2969xw1wtemfegps7gl7stl4/image_1b3f862eird8jjuj2fuca2iam.png
[4]: http://static.zybuluo.com/huis/y0linryc747c8261brthgrls/image_1b3re557m15uhrd61sc512dtpgl9.png
[5]: http://static.zybuluo.com/huis/zw18yrdqmuiz8mz8pazd6tne/image_1b3resb6eb0s1i6p1tedae4526m.png
[6]: http://static.zybuluo.com/huis/dnipae34rj1flomw8nq07ers/image_1b3rguvvepc21bl21dvn1hhd19vi13.png