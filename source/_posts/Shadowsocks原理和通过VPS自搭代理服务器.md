---
title: Shadowsocks原理和通过VPS自搭代理服务器
date: 2017-11-11 19:33:21
tags: [Shadowsocks,VPS,服务器]
categories:  Shadowsocks
---
[Shadowsocks](https://github.com/clowwindy/shadowsocks/)是一个轻量级[socks5](https://en.wikipedia.org/wiki/SOCKS_%28protocol%29#SOCKS5)代理插件，最初用 Python 编写,主要用于翻越GFW\(中国国家防火墙\),ShadowSocks支持远程DNS解析，可以防止DNS污染。所有数据流量全部经过加密，加密算法可选并支持自定义算法，隐蔽性很强， 相比于传统的VPN方式，ShadowSocks支持PAC列表，根据PAC中的规则进行智能切换，兼顾了访问速度与访问效率。

<!--more-->
#### 1.1Shadowsocks的翻墙原理：

###### 天朝局域网通过GFW隔离了我们与外界的交流，当然，这个隔离并非完全隔离，而是选择性的，天朝不希望你上的网站就直接阻断。每一个网络请求都是有数据特征的，不同的协议具备不同的特征，比如 HTTP/HTTPS 这类请求，会很明确地告诉 GFW 它们要请求哪个域名；再比如 TCP 请求，它只会告诉 GFW 它们要请求哪个 IP。 GFW 封锁包含多种方式，最容易操作也是最基础的方式便是域名黑白名单，在黑名单内的域名不让通过，IP 黑白名单也是这个道理。如果你有一台国外服务器不在 GFW 的黑名单内，天朝局域网的机器就可以跟这一台机器通讯。那么一个翻墙的方案就出来了：境内设备与境外机器通讯，境内想看什么网页，就告诉境外的机器，让境外机器代理抓取，然后送回来，我们要做的就是保证境内设备与境外设备通讯时不被 GFW 怀疑和窃听

#### 原理图：

![原理](http://yp.guohaonan.cn/Shadowsocks%E5%8E%9F%E7%90%86tu.jpg)
**在我们和目标地址之间有一个代理服务器，这个VPS一定是要能访问到，并且可以访问目标网站的，所以一般VPS选择国外**的。

1. 我们首先通过SS Local和VPS进行通信，通过Socks5协议进行通信。
2. SS Local连接到VPS， 并对Socks5中传输的数据进行对称加密传输，传输的数据格式是SS的协议。
3. SS Server收到请求后，对数据解密，按照SS协议来解析数据。
4. SS Server根据协议内容转发请求。
5. SS Server获取请求结果后回传给SS Local
6. SS Local获取结果回传给应用程序

#### VPN 与 Shadowsocks 的区别：

**VPN**顾名思义，虚拟专网，你接入VPN就是接入了一个专有网络，那么你访问网络都是从这个专有网络的出口出去，好比你在家，你家路由器后面的网络设备是在同一个网络，而VPN则是让你的设备进入了另一个网络。同时你的IP地址也变成了由VPN分配的一个IP地址。通常是一个私网地址。你和VPN服务器之间的通信是否加密取决于连接VPN的具体方式/协议。

**Shadowsocks**的Sock5代理服务器则是把你的网络数据请求通过一条连接你和代理服务器之间的通道，由服务器转发到目的地。你没有加入任何新的网络，只是http/socks数据经过代理服务器的转发送出，并从代理服务器接收回应。你与代理服务器通信过程不会被额外处理，如果你用https，那本身就是加密的。

#### Shadowsocks中全局模式与PAC模式之间的区别：

**PAC模式**就是会在你连接网站的时候读取PAC文件里的规则，来确定你访问的网站有没有被墙，如果符合，那就会使用代理服务器连接网站，而PAC列表一般都是从GFWList更新的。GFWList定期会更新被墙的网站（不过一般挺慢的）。

**全局模式**下，所有网站默认走代理。而PAC模式是只有被墙的才会走代理，推荐PAC模式，如果PAC模式无法访问一些网站，就换全局模式试试，一般是因为PAC更新不及时（也可能是GFWList更新不及时）导致的。

---

根据以上的原理，我们需要一个VPS,在这里我选择tultr的VPS,由于只有tultr，搬瓦工，香港阿里云的支付方式支持支付宝，无奈学生党只有支付宝，而tultr又较为优惠。

依次点击

> Servers
>
> Deploy New Servers \(那个加号\).
>
> Server Location: 就选你要选的机房（日本，或者美国旧金山的网络较好）
>
> Server Type: CentOS或者Ubuntu（国内这两个的教程最多出现了错误方便查找）
>
> Server Size: 最低配就可以
>
> 其它的不用填写也可以
>
> Deploy Now

#### 注意：

创建好服务器之后，先打开电脑控制台ping一下ip地址,当ping成功之后在进行其他的操作（我就由于日本的机房没ping就配置服务器导致配置好之后连不上，检查了半天发现那边的机房是连不了的，浪费了许多时间。
![ping](http://yp.guohaonan.cn/ping.png)

##### CentOS系统建立完成之后进入到server Inormation界面，单击右上角一个像显示器的图标“View Console”即可进入控制台
![面板](http://yp.guohaonan.cn/86D4C942-9CD4-472B-89FD-3EAD49837CB0.png)

进入后，输入“root“，并输入服务器状态页面显示的密码即登陆成功

如果嫌密码不好记可以在这里输入"Passwd"，即可修改密码

但是以上通过view Console的方式配置服务器比较麻烦，多次配置出现错误就换了了安全终端模拟软件

---

#### 安全终端模拟软件：可以远程浏览服务器系统的终端，通过SHH协议链接

> 常用的有两种一是putty一个是Xshell5，而在使用Xshell5的时候不知为什么就是连不上服务器。

在软件只填主机ip地址，端口号不用动，因为经过测试刚开的服务器只开通了22端口连接进入后在以下窗口输入账号密码（在server information界面都有\)
![Xshell5](http://yp.guohaonan.cn/Xshell.png)

而使用putty就连上了：
![putty](http://yp.guohaonan.cn/putty.png)

#### 配置环景：

> pip是 python 的包管理工具。在本文中将使用 python 版本的 shadowsocks，此版本的 shadowsocks 已发布到 pip 上，因此我们需要通过 pip 命令来安装.

![搭建环境](http://yp.guohaonan.cn/%E6%90%AD%E5%BB%BA%E7%8E%AF%E5%A2%83.png)

##### 输入一下命令

```
yum install python-setuptools && easy_install pip
```

##### 接着会询问你是否安装，输入：

```
y
```

#### 安装shadowsocks

```
pip install shadowsocks
```

##### 接着，创建一个Shadowsocks配置文件,输入以下命令：

```
vi /etc/shadowsocks.json
```

##### 然后进入文件编辑界面，按i开始编辑，输入：

```
{
"server":"your_server_ip", //你的ip地址
"server_port":5656, //你的登录端口
"password":"your_password", //你自己设置的一个密码，用于终端登录
"method":"aes-256-cfb", //加密方式
}
```

编辑完成之后按esc然后打 ：wq 退出并保存文件

#### 开通你的登录端口：

```
firewall-cmd --zone=public --add-port=5656/tcp --permanent
```

着这里可以通过站长工具网站检查站点是否开通，开启前：
![开启端口](http://yp.guohaonan.cn/%E5%BC%80%E5%90%AF%E7%AB%AF%E5%8F%A3.png)
开启后：
![开启站点](http://yp.guohaonan.cn/%E6%A3%80%E6%9F%A5%E7%AB%AF%E5%8F%A3.png)

#### 启动shadowsocs

```
ssserver -c /etc/shadowsocks.json -d start
```

##### 如果需要关闭就输入：

```
ssserver -c /etc/shadowsocks.json -d stop
```

### **至此就安装完成了，就可以通过github上下载的shadowsocks终端进行登录翻墙了**

---

#### 涉及的知识：

##### Shadowsocks原理和搭建 :     [http://blog.021xt.cc/archives/98](http://blog.021xt.cc/archives/98)

##### Vim入门基础:      [ www.jianshu.com/p/bcbe916f97e1](http://www.jianshu.com/p/bcbe916f97e1)

##### CentOS7使用firewalld打开关闭防火墙与端口:     [http://www.cnblogs.com/moxiaoan/p/5683743.html](http://www.cnblogs.com/moxiaoan/p/5683743.html)



