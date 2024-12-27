---
title: 折腾日记--搭建自己的个人服务器
date: 2023-05-26 16:32:37
tags: 服务器
---
最近腾讯云一年的2核4G服务器快要无了，当时花了100多买了一年，上面挂了一些脚本和服务，很方便。但是看到腾讯云1000多的续费还是有点肉疼的......

![image-20230526152203369.png](http://server.killuayz.top:8089/images/2023/05/26/image-20230526152203369.png)

# mini PC


我琢磨了琢磨，又琢磨了琢磨，去京东上看了看，发现1000多的价位，可以在京东上买一台配置很不错的mini PC了

![_20230526153451.jpeg](http://server.killuayz.top:8089/images/2023/05/26/_20230526153451.jpeg)

买个PC可以用很多年，配置还可以随心所欲的改，多好！说干就干！

# ipv6

前面装m2固态和内存条的过程就忽略掉，我们直奔问题的核心——怎么访问到这台PC呢？

之前有考虑过使用花生壳内网穿透，但是内网穿透限流限速，想要比较好的网速，还得氪金，于是开始想别的办法，这段时间在上计算机网络的课程，课上学到了ipv6，看到ipv6如此庞大的ip空间，不由得让我想，是不是说，只要我的设备支持ipv6，那么这个设备的ip就是唯一的，外网的主机也就可以通过这个ip访问到我了呢？于是我在自己的电脑上做了一些小实验，随便用flask架了一个博客，然后用ipv6访问，发现真的能够访问到。



# 域名

访问到主机分配给pc的ipv6的地址是会变的，所以必须要搭配域名解析。这里我购买了腾讯云的域名，一年20多块，十分划算

![Screenshot-2023-05-26-154824.png](http://server.killuayz.top:8089/images/2023/05/26/Screenshot-2023-05-26-154824.png)

经过漫长的域名备案之后，我终于拿到了自己的域名。





# 系统安装

系统我使用了arch，arch的安装方式网上到处都是，我就不赘述。因为学校里面每天23点会断电，所会我想要达到的效果是，每天来电了都会自动打开，自动联网，自动启动各种服务。来电启动不难，在bios中设置就好。难的是自动联网。学校宿舍里的wifi有两个

![Screenshot-2023-05-26-160000.png](http://server.killuayz.top:8089/images/2023/05/26/Screenshot-2023-05-26-160000.png)

RUC-Web需要进入一个网页进行登录操作，RUC-Mobile可以自动联网，不用说，肯定选择RUC-Mobile。RUC-Mobile是Peap协议的，而简单好用的NetworkManager似乎不支持这个协议，找了一圈只找到了wpa_supplicant，wpa配置如下

```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=wheel
network={
        key_mgmt=WPA-EAP
        ssid="RUC-Mobile"
        eap=PEAP
        identity="*****你的用户名*****"
        password="*****你的密码******"
        phase2="autheap=MSCHAPV2"
}
```

下一步是在开机时运行以下几个命令，以连接wifi

```
killall wpa_supplicant
wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
dhclient wlp3s0
```

网上看了很多说把这个脚本注册到systemd中，这样开机就可以启动了，理论感觉还没错，但是实践时出了大问题

```
May 22 13:27:36 550W systemd[1]: /etc/systemd/system/rc-local.service:10: Support for option SysVStartPriority= has been removed and it is ignored
-- Boot 1d875d0d745e43608a927de9ed7d809b --
May 22 13:27:58 550W systemd[1]: Starting "/etc/rc.local Compatibility"...
May 22 13:27:58 550W dhclient[299]: Failed to get interface index: No such device
May 22 13:27:58 550W dhclient[299]: 
May 22 13:27:58 550W dhclient[299]: If you think you have received this message due to a bug rather
May 22 13:27:58 550W dhclient[299]: than a configuration issue please read the section on submitting
May 22 13:27:58 550W dhclient[299]: bugs on either our web page at www.isc.org or in the README file
May 22 13:27:58 550W dhclient[299]: before submitting a bug.  These pages explain the proper
May 22 13:27:58 550W dhclient[299]: process and the information we find helpful for debugging.
May 22 13:27:58 550W dhclient[299]: 
May 22 13:27:58 550W dhclient[299]: exiting.
May 22 13:27:58 550W systemd[1]: Started "/etc/rc.local Compatibility".
May 22 13:29:18 550W systemd[1]: rc-local.service: Deactivated successfully.
May 22 13:29:18 550W systemd[1]: Stopped "/etc/rc.local Compatibility".
-- Boot 3a2dd056e78d4106b8ae0c25a59d766f --
May 22 13:29:39 550W systemd[1]: Starting "/etc/rc.local Compatibility"...
May 22 13:29:40 550W dhclient[294]: Failed to get interface index: No such device
May 22 13:29:40 550W dhclient[294]: 
May 22 13:29:40 550W dhclient[294]: If you think you have received this message due to a bug rather
May 22 13:29:40 550W dhclient[294]: than a configuration issue please read the section on submitting
May 22 13:29:40 550W dhclient[294]: bugs on either our web page at www.isc.org or in the README file
May 22 13:29:40 550W dhclient[294]: before submitting a bug.  These pages explain the proper
May 22 13:29:40 550W dhclient[294]: process and the information we find helpful for debugging.
May 22 13:29:40 550W dhclient[294]: 
May 22 13:29:40 550W dhclient[294]: exiting.
May 22 13:29:40 550W dhclient[320]: Failed to get interface index: No such device
May 22 13:29:40 550W dhclient[320]: 
May 22 13:29:40 550W dhclient[320]: If you think you have received this message due to a bug rather
May 22 13:29:40 550W dhclient[320]: than a configuration issue please read the section on submitting
May 22 13:29:40 550W dhclient[320]: bugs on either our web page at www.isc.org or in the README file
May 22 13:29:40 550W dhclient[320]: before submitting a bug.  These pages explain the proper
May 22 13:29:40 550W dhclient[320]: process and the information we find helpful for debugging.
May 22 13:29:40 550W dhclient[320]: 
May 22 13:29:40 550W dhclient[320]: exiting.
May 22 13:29:40 550W systemd[1]: Started "/etc/rc.local Compatibility".
May 22 13:31:28 550W systemd[1]: /etc/systemd/system/rc-local.service:10: Support for option SysVStartPriority= has been removed and it is ignored
May 22 13:31:46 550W systemd[1]: rc-local.service: Deactivated successfully.
May 22 13:31:46 550W systemd[1]: Stopped "/etc/rc.local Compatibility".
........
```

所以我改变思路，想要在系统完全启动之后再运行这几行命令，这里就不得不提到pm2了，我个人觉得他安装简单，易用。所以我写了个pm2的任务：

```json
{
        apps:[{
                name: "network-pm2-startup",
                script: "/bin/bash /etc/network-pm2-startup/run.sh",
                autorestart: true,
                max_restart:1,
                error_file: "/etc/network-pm2-startup/error.log",
                out_file: "/etc/network-pm2-startup/out.log"
        }]
}
```

让pm2运行脚本

```bash
#!/bin/bash
# /etc/network-pm2-startup/run.sh

sleep 30s
killall wpa_supplicant
wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
dhclient wlp3s0
```

这样一来，每次开机后，先等30s，等网络设备都启动后，再使用wpa_supplicant连接RUC-Mobile就好了。但问题接种而至，当脚本运行结束后，pm2会自动重新启动执行，重新执行wpa_supplicant会出错（我也不知道为什么），但想要关闭自动重新启动，每次开机后pm2也不会自动运行这个脚本，所以我为了方便，用了一个不太优雅的方法，修改脚本如下：

```
#!/bin/bash
# /etc/network-pm2-startup/run.sh

sleep 30s
killall wpa_supplicant
wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
dhclient wlp3s0
sleep 114514d
```

sleep了114514天，那么对于我这种应用场景来说，相当于是每次开机只会运行这个脚本一次，这样就避免了重复运行的问题，再运行

```
pm2 save
sudo systemctl enable pm2-root
```

就可以让pm2每次开机都运行这个脚本，连接RUC-Mobile了！



# DDNS

最后便是做DDNS，这里我使用了[jeessy2/ddns-go: 简单好用的DDNS。自动更新域名解析到公网IP(支持阿里云、腾讯云dnspod、Cloudflare、Callback、华为云、百度云、Porkbun、GoDaddy、Google Domain) (github.com)](https://github.com/jeessy2/ddns-go) ，他支持docker部署，还提供了web界面，十分nice，从DNSPod中获取到id和token填入，简单配置后便可进行动态DNS解析。

![Screenshot-2023-05-26-155342.png](http://server.killuayz.top:8089/images/2023/05/26/Screenshot-2023-05-26-155342.png)

在docker里面启动后，不要忘记设置--restart=always，让其开机自动启动。



# 大功告成 

到此为止，我们就成功配置好了自己的mini PC，让其每次来电，就会自动开机，自动联网，自动ddns，让我们可以使用远程ssh访问。

![Screenshot-2023-05-26-163056.png](http://server.killuayz.top:8089/images/2023/05/26/Screenshot-2023-05-26-163056.png)