# 本人Mac系统版本
> Mac OS版本：10.14.4 Mojave

我们都知道MAC在很早之前，VPN连接中已经不能添加PPTP的协议连接了（可能是安全性考虑吧），也就是说我们不能在`设置`-`网络`中直接配置PPTP的VPN！这应该是很多朋友不敢更新MAC的原因（特别是使用第三方客户端都连不上的人，毕竟工作相关）
但是，我说不能配置PPTP是指UI里面的，实际上还是能够通过配置`pppd`的方式连接PPTP的。
并非Shimo等第三方客户端。


# 一、起因
发现这个方法是因为某次我需要连接某猫星的VPN拉代码（SVN仓），但对方只给我提供了PPTP协议，还不支持L2TP，也就是说通过`网络`设置不了VPN。然后我按照别人说的下载了`Shimo`（这个是收费的软件），更惨烈的是Shimo居然连接不上，错误原因提示还少得可怜。收费软件很多，试过几个都不行。

# 二、发现免费的解决方案
后面通过一轮的查找资料发现，Mac自带的VPN配置是通过调用`pppd`组件进行拨号通信的，然后我还是发现了在终端中使用`pppd`是能够建立PPTP的，苹果官方只是删除了UI的才做入口而已。`pppd`连接`PPTP`的功能还是存在的。
前提是你得配置对应的`拨号配置文件`。


# 三、普及下资料
首先，得告诉大家Mac的VPN配置文件都放在：
> /etc/ppp/peers/

这个目录下。


# 四、开始配置
## 1. 建立PPTP拨号配置
打开终端，编辑配置，输入：
```bash
sudo vim /etc/ppp/peers/inner
```
划重点：**里面的`文件名`就是`连接名称`！！！**
所以我上面输入`inner`就是`连接的名称`，后面进行拨号的时候会用到！起名`inner`表示内部的意思。



## 2. 配置文件内容
然后在`vim`中输入如下配置信息：
```bash
plugin PPTP.ppp
noauth
remoteaddress "----host----"
user "----username----"
password "----password----"
redialcount 1
redialtimer 5
idle 1800
# mru 1368
# mtu 1368
receive-all
novj 0:0
ipcp-accept-local
ipcp-accept-remote
refuse-eap
refuse-pap
refuse-chap-md5
hide-password
mppe-stateless
mppe-128
# require-mppe-128
looplocal
nodetach
# ms-dns 8.8.8.8
usepeerdns
# ipparam gwvpn
defaultroute
debug
```

> 其中：
> remoteaddress：双引号内写VPN服务器访问地址
> user：用户名
> password：密码

替换成你的拨号账号吧！

其他的加了`#`表示注释了，不管它，留着吧，可能以后用得上~~~



## 3. 执行PPTP连接和终止连接
**在执行拨号之前，必须先声明一下：**
> pppd拨号不是守护进程方式运行的，会一直抢占终端的线程，并且不能通过`Ctrl + C`的方式终止。
> 需要使用`pkill`来杀掉整个进程！

（1）**在`终端`进行PPTP拨号：**
```bash
sudo pppd call inner
```
上面的就是表示`pppd`使用`inner`配置进行拨号，然后会弹一堆的log出来，这是因为用了debug模式。
连接之后可以使用`ifconfig`（新终端窗口或tmux）查看IP地址是否拨号成功（log里面也会有内容的）

（2）**终止PPTP连接：**
这个时候你当前的终端界面是在pppd进程中的，没有退出，这个终端是不能进行操作的！(Ctrl+C也没用的)
这个时候**新建`终端`窗口**，输入：
```bash
sudo pkill pppd
```

这时候`pppd`进程会被杀掉，拨号的那个终端窗口会退出pppd进程的操作。

***
***

# 五、最后：使用`alias`进行偷懒：
每次输入：
> sudo pppd call ...
> sudo pkill pppd
手都软了。

我们可以使用`alias`来简化这些操作：
打开终端，输入：
```bash
vim ~/.bash_profile
```

在最后面加入：
```bash
alias vpn-on='sudo pppd call inner'
alias vpn-off='sudo pkill pppd'
```
保存后刷新下：
```bash
source ~/.bash_profile
```

这时候就可以通过：
```bash
vpn-on
vpn-off
```
来控制VPN开关。不过还是记住，这个要分别在两个终端窗口中才能操作。推荐使用`tmux`终端复用这个组件。

> emmm....使用终端连接PPTP还是有点geek味道的

***

此文同时在简书发布：[https://www.jianshu.com/p/c69f38f95217](https://www.jianshu.com/p/c69f38f95217)
此文同时在CSDN发布：[https://blog.csdn.net/nthack5730/article/details/89774908](https://blog.csdn.net/nthack5730/article/details/89774908)
转载要加原文链接！谢谢支持！

***
