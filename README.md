# ipv6代理上网设置教程

##### 本教程参照 [知乎](https://www.zhihu.com/question/29535965)   对搭建ipv6代理的方法进行了总结   如有疑问可以参考 [原文档1](https://link.zhihu.com/?target=http%3A//blog.kyangcis.me/2015/01/01/my-first-vps/)、[原文档2](https://link.zhihu.com/?target=http%3A//teddysun.com/342.html)

大多数校园网用户每月有一定外网流量限制(如THU免费流量仅有20G)，超过部分即要收费。与此同时，大多数校园网使用ipv6访问时不计入外网流量。因此只要能够使用ipv6代理，就能够实现科学的上网，再也不用担心每月流量超限了~

**声明：此教程仅针对THU校园网进行测试，不保证在其他ipv6网络均能正常使用，使用前请自行测试。**

原理：租一个带IPv6的VPS，搭建VPN，通过IPv6转发所有的IPv4流量实现校园网免流量。此方法同时支持Win/Mac/Android/iOS/Linux/OpenWrt平台


以下将具体介绍科学上网的过程：

### iPv6环境检测

一般校园网支持iPv6，且会自动分配iPv6地址。在win下可以用cmd命令`ipconfig`查看是否分配到了iPv6地址，Linux用户则可使用命令`ifconfig`。另一种更直接的方式则是利用浏览器访问iPv6网站(如 [iPv6 test](http://www.test-ipv6.ro/))，如果能够成功打开网站，则说明网络环境支持iPv6。另外对于THU用户，如果网络本身支持iPv6但是访问不了iPv6网站，可以参照http://ipv6.tsinghua.edu.cn 提供的教程搭建iPv6隧道，即可正常访问。


## 租建vps

现在很多vps都提供iPv4/iPv6访问地址，如搬瓦工、Digital Ocean等，具体可根据自身需求选择购买，操作过程大都类似。本文将以Digital Ocean为例，其他vps搭建方式请自行测试。

之所以选用Digital Ocean的主机，主要是因为便宜！！！目前最廉价的主机价格5$/mon，注册即送10刀。DO的主机访问速度还可以，我这边测试旧金山节点平均延迟大概160ms左右(指望打游戏就不用想了)，速度大概有校园网直连的1/4~1/2左右，相当不错。另外 注册GitHub Education还免费送50刀(最早是送100刀的T`T)。因此注册DO只需要缴费5刀即可使用11mon，还是相当划算的。至于之后嘛，完全可以再注册个GitHub Education账号，又送50刀(你的室友已下线)

注册GitHub Education账号需要在Github网站上通过高校邮箱申请，大陆这边即为edu.cn后缀的邮箱。快的话一般一个工作日之内就能收到回复，点击https://www.digitalocean.com 即可进入DO官网注册，另外通过其他DO用户分享的链接注册双方均可获得10刀奖励，这里分享一下我的邀请链接[Digital Ocean](https://www.digitalocean.com/?refcode=c5305e731c7f&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=CopyPaste)。如果你已经注册了GitHub Education账号，就可以用申请到的优惠码在DO账户上换取50刀

完成注册后需要激活你的账号，根据提示填写相关信息。然后就是绑定信用卡或者使用PayPal进行一次充值。我没有信用卡并且PayPal可以使用银联卡付款，进行了一次付款5刀。至此你已经完成了账号的注册和激活。此时账号上应该有15刀的余额。


然后就是购买和配置vps了，在DO中选择新建一个Droplets(DigitalOcean对VPS的别称)，选择最便宜的5刀/mo  地区目前推荐SF  服务器推荐Ubuntu或者Centos  一定记得勾选iPv6  然后添加你的SSH key 如下所示:

![img](https://github.com/Moohi/ipv6config/blob/master/digital%20ocean.jpg)


关于SSH部分请参照[DO官方帮助文档](https://www.digitalocean.com/community/tutorials/how-to-connect-to-your-droplet-with-ssh?utm_source=Customerio&utm_medium=Email_Internal&utm_campaign=Email_UbuntuDistroApacheWelcome)进行设置

这里推荐大家在创建Droplets时就直接添加自己的SSH key，这样你就可以用SSH key对应的PC通过ssh命令访问服务器完成配置，而不需要每次输入账户和密码。当然如果你不添加SSH key，那么DO会给你发一封包含服务器账号密码的邮件，稍后可以利用账号密码访问服务器进行相应配置。


## 配置服务器

到这里你应该已经通过ssh连接到了服务器，接下来的代码会直接在服务器上运行，完成服务器的配置

首先是在服务器上安装shadowsocks软件，以完成iPv4到iPv6的转发。将以下代码复制到Terminal  回车  等待代码执行完毕：

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh  
chmod +x shadowsocks.sh  
./shadowsocks.sh 2>&1 | tee shadowsocks.log  
```

然后显示如下，就可以进行下一步了：

```
Congratulations, shadowsocks install completed!  
Your Server IP:your_server_ip  
Your Server Port:8989  
Your Password:your_password  
Your Local IP:127.0.0.1  
Your Local Port:1080  
Your Encryption Method:aes-256-cfb  
  
Welcome to visit:http://teddysun.com/342.html  
Enjoy it!  
```

编辑服务器上的shadowsocks配置文件，将以下代码复制到Terminal  回车运行：

```
vi /etc/shadowsocks.json
```

运行上述代码会用vi编辑器打开一个文本文件，按i进入编辑模式，在编辑模式中，可以用方向键控制箭头位置，对文本进行编辑。编辑结果如下：

```
{  
  "server":"::",  
  "server_port":8989,  
  "local_address":"127.0.0.1",  
  "local_port":1080,  
  "password":"yourpassword",  
  "timeout":300,  
  "method":"aes-256-cfb",  
  "fast_open": false  
}  
```

记得将yourpassword换成你想设定的密码，所有字符均是英文字符

按ESC键退出编辑模式，然后输入 :wq 回车  保存以上编辑的内容并退出；最后键入以下命令重启ShadowSock服务端让配置生效：

```
/etc/init.d/shadowsocks restart  
```

此时可以关闭ssh连接了并退出Terminal了


## 客户端连接

此时，只需要用你的PC/手机/iPad等设备，通过shadow socks客户端连接服务器代理，即可实现iPv6代理上网

以win为例，首先在网上下载shadowsocks客户端。貌似直接用百度检索是找不到这个软件的(被墙了)，你可以在Github网站上搜索，也可以先下载一个lantern代理软件翻墙，然后下载shadowsocks客户端；运行下载的shadowsocks客户端，点击 服务器-》编辑服务器-》添加 (服务器IP:iPv6 Address 服务器端口:8989 密码:yourpassword 加密:aes-256-cfb 代理端口:1080)

iPv6 Address可以登陆你的DO账号查看，在Droplets里的信息会显示你的服务器的iPv6地址和iPv4地址，填写服务器的iPv6地址即可(这里如果填iPv4地址也能访问网站 但是需要登陆校园网账号 会计入校园网流量; 而如果填iPv6，应该即使不登陆校园网账号 浏览器也能访问网络)

shadowsocks系统代理模式分为全局代理和PAC代理，PAC代理一般访问国内网站直连，访问国外网站会通过代理；如果想要免校园网流量，需要使用全局代理。配置好本地客户端之后建议把DNS服务器添加2001:4860:4860::8888和2001:4860:4860::8844，此时，不登陆校园网应该也能访问各个网站了。

Android/Mac/Linux下的shadowsocks配置几乎完全一样。Linux使用图形化界面的shadowsocks Qt5版本，当然也可以使用命令行版本的shadowsocks客户端。iPad/iPhone等iOS设备商店里貌似没有可用的的shadowsocks 推荐使用shadowrocket(收费软件6元) 还挺好用的，不过这个软件目前更新之后有bug，用不了iPv6代理(之前是可以的)，作者应该会很快修复，可以留意评论区状态决定后是否使用。

以上设置对于PC而言只是实现了浏览器的代理，其他APP需要另外配置(推荐使用Proxifier完成其他软件的代理 Win/Mac网上有破解版)，对于手机而言则实现了全局代理。另外通过代理访问网络使用的是服务器的IP地址(比如旧金山)，部分视频网站版权保护，海外用户无法正常观看视频，需要先关闭代理。
