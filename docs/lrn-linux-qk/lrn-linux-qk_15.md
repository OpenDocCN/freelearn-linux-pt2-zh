网络出了什么问题？

当网络出现问题时，我们都会感到愤怒。没有连接到互联网，这个世界就没有乐趣。在本章中，你将学习 Linux 网络的基础知识。你还将学习如何检查两个主机之间的网络连接，并获得对 DNS 工作原理的实际理解，以及更多！

# 第十六章：测试网络连接

在 Linux 机器上检查是否有互联网访问的简单方法是尝试连接互联网上的任何远程主机（服务器）。这可以通过使用`ping`命令来完成。一般来说，`ping`命令的语法如下：

```
ping [options] host
```

例如，要测试你是否能够到达`google.com`，你可以运行以下命令：

```
root@ubuntu-linux:~# ping google.com
PING google.com (172.217.1.14) 56(84) bytes of data.
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=1 ttl=55 time=38.7 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=2 ttl=55 time=38.7 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=3 ttl=55 time=40.4 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=4 ttl=55 time=36.6 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=5 ttl=55 time=40.8 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=6 ttl=55 time=38.6 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=7 ttl=55 time=38.9 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=8 ttl=55 time=37.1 ms
^C
--- google.com ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 66ms 
rtt min/avg/max/mdev = 36.555/38.724/40.821/1.344 ms
```

`ping`命令发送一个叫做**ICMP 回显请求**的数据包（数据单位）到指定的主机，并等待主机发送一个叫做**ICMP 回显回复**的数据包来确认它确实收到了初始数据包。如果主机像我们在例子中看到的那样回复，那么就证明我们能够到达主机。这就像你给朋友家寄一个包裹，等待朋友发短信确认收到一样。

请注意，没有任何选项，`ping`命令会持续发送数据包，直到你按下*Ctrl* + *C*。

你可以使用`-c`选项来指定你想发送到主机的数据包数量。例如，只向`google.com`发送三个数据包，你可以运行以下命令：

```
root@ubuntu-linux:~# ping -c 3 google.com
PING google.com (172.217.1.14) 56(84) bytes of data.

64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=1 ttl=55 time=39.3 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=2 ttl=55 time=49.7 ms 
64 bytes from iad23s25-in-f14.1e100.net (172.217.1.14): icmp_seq=3 ttl=55 time=40.8 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 59ms rtt min/avg/max/mdev = 39.323/43.267/49.708/4.595 ms
```

如果你没有连接到互联网，你将从`ping`命令得到以下输出：

```
root@ubuntu-linux:~# ping google.com
ping: google.com: Name or service not known
```

# 列出你的网络接口

你可以通过查看`/sys/class/net`目录的内容来列出系统上可用的网络接口：

```
root@ubuntu-linux:~# ls /sys/class/net 
eth0 lo wlan0
```

我的系统上有三个网络接口：

1.  `eth0`：以太网接口

1.  `lo`：回环接口

1.  `wlan0`：Wi-Fi 接口

请注意，根据你的计算机硬件，你可能会得到不同的网络接口名称。

## ip 命令

你也可以使用`ip link show`命令查看系统上可用的网络接口：

```
root@ubuntu-linux:~# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
 link/ether f0:de:f1:d3:e1:e1 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
 link/ether 10:0b:a9:6c:89:a0 brd ff:ff:ff:ff:ff:ff
```

## nmcli 命令

我更喜欢的另一种方法是使用`nmcli`设备状态命令：

```
root@ubuntu-linux:~# nmcli device status 
DEVICE TYPE STATE CONNECTION
wlan0 wifi      connected   SASKTEL0206-5G 
eth0  ethernet  unavailable --
lo    loopback  unmanaged   --
```

你可以从输出中看到每个网络接口的连接状态。我目前是通过我的 Wi-Fi 接口连接到互联网的。

# 检查你的 IP 地址

没有手机号码，你就不能给朋友打电话；同样，你的计算机需要一个 IP 地址才能连接到互联网。你可以使用许多不同的方法来检查你的机器的 IP 地址。你可以使用老式（但仍然流行的）`ifconfig`命令，后面跟着连接到互联网的网络接口的名称：

```
root@ubuntu-linux:~# ifconfig wlan0
wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
 inet 172.16.1.73 netmask 255.255.255.0 broadcast 172.16.1.255 
       inet6 fe80::3101:321b:5ec3:cf9 prefixlen 64 scopeid 0x20<link> 
       ether 10:0b:a9:6c:89:a0 txqueuelen 1000 (Ethernet)
 RX packets 265 bytes 27284 (26.6 KiB)
 RX errors 0 dropped 0 overruns 0 frame 0
 TX packets 165 bytes 28916 (28.2 KiB)
 TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

你也可以使用`-a`选项列出所有网络接口：

```
root@ubuntu-linux:~# ifconfig -a
eth0: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500
 ether f0:de:f1:d3:e1:e1 txqueuelen 1000 (Ethernet) 
      RX packets 0 bytes 0 (0.0 B)
 RX errors 0 dropped 0 overruns 0 frame 0
 TX packets 0 bytes 0 (0.0 B)
 TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0 
      device interrupt 20 memory 0xf2500000-f2520000

lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536
 inet 127.0.0.1 netmask 255.0.0.0
 inet6 ::1 prefixlen 128 scopeid 0x10<host>
 loop txqueuelen 1000 (Local Loopback) 
     RX packets 4 bytes 156 (156.0 B)
 RX errors 0 dropped 0 overruns 0 frame 0
 TX packets 4 bytes 156 (156.0 B)
 TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
 inet 172.16.1.73 netmask 255.255.255.0 broadcast 172.16.1.255 
     inet6 fe80::3101:321b:5ec3:cf9 prefixlen 64 scopeid 0x20<link> 
     ether 10:0b:a9:6c:89:a0 txqueuelen 1000 (Ethernet)
 RX packets 482 bytes 45500 (44.4 KiB)
 RX errors 0 dropped 0 overruns 0 frame 0
 TX packets 299 bytes 57788 (56.4 KiB)
 TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

你可以从输出中看到，我只通过我的 Wi-Fi 接口（`wlan0`）连接到互联网，我的 IP 地址是`172.16.1.73`。

**回环是什么？**

回环（或`lo`）是计算机用来与自身通信的虚拟接口；它主要用于故障排除。回环接口的 IP 地址是`127.0.0.1`，如果你想 ping 自己！尽管 ping `127.0.0.1`。

你也可以使用更新的`ip`命令来检查你的机器的 IP 地址。例如，你可以运行`ip address show`命令来列出并显示所有的网络接口的状态：

```
root@ubuntu-linux:~# ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
 inet 127.0.0.1/8 scope host lo
 valid_lft forever preferred_lft forever 
    inet6 ::1/128 scope host
 valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state 
        DOWN link/ether f0:de:f1:d3:e1:e1 brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state 
    UP link/ether 10:0b:a9:6c:89:a0 brd ff:ff:ff:ff:ff:ff
 inet 172.16.1.73/24 brd 172.16.1.255 scope global dynamic 
      noprefixroute wlan0 valid_lft 85684sec preferred_lft 85684sec
 inet6 fe80::3101:321b:5ec3:cf9/64 scope link noprefixroute 
      valid_lft forever preferred_lft forever
```

# 检查你的网关地址

你的计算机从路由器那里获取了一个 IP 地址；这个路由器也被称为默认网关，因为它将你连接到外部世界（互联网）。这些路由器随处可见；它们在你家、咖啡店、学校、医院等等。

你可以通过运行以下任一命令来检查你的默认网关的 IP 地址：

+   `route -n`

+   `netstat -rn`

+   `ip route`

让我们从第一个命令`route -n`开始：

```
root@ubuntu-linux:~# route -n Kernel IP routing table
Destination Gateway       Genmask       Flags  Metric Ref Use Iface
0.0.0.0     172.16.1.254  0.0.0.0       UG     600     0   0 wlan0
172.16.1.0  0.0.0.0       255.255.255.0 U      600     0   0 wlan0
```

您可以从输出中看到我的默认网关 IP 地址为`172.16.1.254`。现在让我们尝试第二个命令`netstat -rn`：

```
root@ubuntu-linux:~# netstat -rn 
Kernel IP routing table
Destination   Gateway      Genmask       Flags  MSS Window irtt Iface
0.0.0.0       172.16.1.254 0.0.0.0       UG     0   0      0    wlan0
172.16.1.0    0.0.0.0      255.255.255.0 U      0   0      0    wlan0
```

输出几乎看起来相同。现在输出与第三个命令`ip route`有一点不同：

```
root@ubuntu-linux:~# ip route
default via 172.16.1.254 dev wlan0 proto dhcp metric 600
172.16.1.0/24 dev wlan0 proto kernel scope link src 172.16.1.73 metric 600
```

默认网关 IP 地址显示在第一行：默认通过`172.16.1.254`。您还应该能够 ping 默认网关：

```
root@ubuntu-linux:~# ping -c 2 172.16.1.254
PING 172.16.1.254 (172.16.1.254) 56(84) bytes of data.
64 bytes from 172.16.1.254: icmp_seq=1 ttl=64 time=1.38 ms
64 bytes from 172.16.1.254: icmp_seq=2 ttl=64 time=1.62 ms

--- 172.16.1.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms rtt min/avg/max/mdev = 1.379/1.501/1.624/0.128 ms
```

# 使用 traceroute 飞行

您现在已经准备好离开家去上班了。您必须通过不同的街道最终到达目的地，对吧？嗯，这与您尝试在互联网上到达主机（网站）时非常相似；您采取的路线从默认网关开始，以目的地结束。

您可以使用`traceroute`命令跟踪到任何目的地的路由。`traceroute`命令的一般语法如下：

```
traceroute destination
```

例如，您可以通过运行以下命令跟踪从您的计算机到`google.com`的路由：

```
root@ubuntu-linux:~# traceroute google.com
traceroute to google.com (172.217.1.14), 30 hops max, 60 byte packets
 1 172.16.1.254 (172.16.1.254) 15.180 ms 15.187 ms 15.169 ms
 2 207-47-195-169.ngai.static.sasknet.sk.ca (207.47.195.169) 24.059 ms
 3 142.165.0.110 (142.165.0.110) 50.060 ms 54.305 ms 54.903 ms
 4 72.14.203.189 (72.14.203.189) 53.720 ms 53.997 ms 53.948 ms
 5 108.170.250.241 (108.170.250.241) 54.185 ms 35.506 ms 108.170.250.225
 6 216.239.35.233 (216.239.35.233) 37.005 ms 35.729 ms 38.655 ms
 7 yyz10s14-in-f14.1e100.net (172.217.1.14) 41.739 ms 41.667 ms 41.581 ms
```

正如您所看到的，我的机器花了七次旅行（跳跃）才到达我的最终目的地`google.com`。请注意，第一跳是我的默认网关，最后一跳是目的地。

当您在解决连接问题时，`traceroute`命令非常有用。例如，要到达特定目的地可能需要很长时间；在这种情况下，`traceroute`可以帮助您检测到达目的地路径上的任何故障点。

# 破坏您的 DNS

互联网上的每个网站（目的地）都必须有一个 IP 地址。然而，我们人类对数字不太擅长，所以我们发明了**域名系统**（**DNS**）。DNS 的主要功能是将名称（域名）与 IP 地址关联起来；这样，我们在浏览互联网时就不需要记住 IP 地址了...感谢上帝的 DNS！

每次您在浏览器上输入域名时，DNS 都会将（解析）域名转换为其相应的 IP 地址。您的 DNS 服务器的 IP 地址存储在文件`/etc/resolv.conf`中：

```
root@ubuntu-linux:~# cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 142.165.200.5
```

我正在使用由我的**互联网服务提供商**（**ISP**）提供的 DNS 服务器`142.165.200.5`。您可以使用`nslookup`命令来查看 DNS 的工作情况。`nslookup`命令的一般语法如下：

```
nslookup domain_name
```

`nslookup`命令使用 DNS 获取域名的 IP 地址。例如，要获取`facebook.com`的 IP 地址，您可以运行以下命令：

```
root@ubuntu-linux:~# nslookup facebook.com 
Server: 142.165.200.5
Address: 142.165.200.5#53

Non-authoritative answer:
Name: facebook.com 
Address: 157.240.3.35 
Name: facebook.com
Address: 2a03:2880:f101:83:face:b00c:0:25de
```

请注意，它在输出的第一行显示了我的 DNS 服务器的 IP 地址。您还可以看到`facebook.com`的 IP 地址`157.240.3.35`。

您还可以 ping`facebook.com`：

```
root@ubuntu-linux:~# ping -c 2 facebook.com
PING facebook.com (157.240.3.35) 56(84) bytes of data.
64 bytes from edge-star-mini-shv-01-sea1.facebook.com (157.240.3.35): 
icmp_seq=1 ttl=55 time=34.6 ms
64 bytes from edge-star-mini-shv-01-sea1.facebook.com (157.240.3.35): 
icmp_seq=2 ttl=55 time=33.3 ms

--- facebook.com ping statistics ---

2 packets transmitted, 2 received, 0% packet loss, time 2ms 
rtt min/avg/max/mdev = 33.316/33.963/34.611/0.673 ms
```

现在让我们破坏一切！我妈妈曾经告诉我，我必须破坏一切，这样我才能理解它们是如何工作的。让我们看看没有 DNS 的生活是什么样子，通过清空文件`/etc/resolv.conf`：

```
root@ubuntu-linux:~# echo > /etc/resolv.conf 
root@ubuntu-linux:~# cat /etc/resolv.conf

root@ubuntu-linux:~#
```

现在让我们对`facebook.com`进行`nslookup`：

```
root@ubuntu-linux:~# nslookup facebook.com
```

您会看到它挂起，因为它无法再解析域名。现在让我们尝试 ping`facebook.com`：

```
root@ubuntu-linux:~# ping facebook.com
ping: facebook.com: Temporary failure in name resolution
```

您会收到错误消息`名称解析临时失败`，这是说您的 DNS 出了问题的一种花哨方式！但是，您仍然可以通过使用其 IP 地址来 ping`facebook.com`：

```
root@ubuntu-linux:~# ping -c 2 157.240.3.35
PING 157.240.3.35 (157.240.3.35) 56(84) bytes of data.
64 bytes from 157.240.3.35: icmp_seq=1 ttl=55 time=134 ms
64 bytes from 157.240.3.35: icmp_seq=2 ttl=55 time=34.4 ms

--- 157.240.3.35 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms 
rtt min/avg/max/mdev = 34.429/84.150/133.872/49.722 ms
```

让我们修复我们的 DNS，但这次我们将不使用我们的 ISP 的 DNS 服务器；相反，我们将使用 Google 的公共 DNS 服务器`8.8.8.8`：

```
root@ubuntu-linux:~# echo "nameserver 8.8.8.8" > /etc/resolv.conf 
root@ubuntu-linux:~# cat /etc/resolv.conf
nameserver 8.8.8.8
```

现在让我们再次对`facebook.com`进行`nslookup`：

```
root@ubuntu-linux:~# nslookup facebook.com Server: 8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name: facebook.com 
Address: 31.13.80.36 
Name: facebook.com
Address: 2a03:2880:f10e:83:face:b00c:0:25de
```

请注意，我的活动 DNS 现在已更改为`8.8.8.8`。我还得到了`facebook.com`的不同 IP 地址，这是因为 Facebook 在世界各地的许多不同服务器上运行。

# 更改您的主机名

每个网站都有一个在互联网上唯一标识它的域名；同样，计算机有一个在网络上唯一标识它的主机名。

您计算机的主机名存储在文件`/etc/hostname`中：

```
root@ubuntu-linux:~# cat /etc/hostname 
ubuntu-linux
```

您可以使用主机名来访问同一网络（子网）中的其他计算机。例如，我有另一台计算机，主机名为`backdoor`，目前正在运行，我可以 ping 它：

```
root@ubuntu-linux:~# ping backdoor
PING backdoor (172.16.1.67) 56(84) bytes of data.
64 bytes from 172.16.1.67 (172.16.1.67): icmp_seq=1 ttl=64 time=3.27 ms
64 bytes from 172.16.1.67 (172.16.1.67): icmp_seq=2 ttl=64 time=29.3 ms
64 bytes from 172.16.1.67 (172.16.1.67): icmp_seq=3 ttl=64 time=51.4 ms
^C
--- backdoor ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 20ms 
rtt min/avg/max/mdev = 3.272/27.992/51.378/19.662 ms
```

请注意，`backdoor`位于相同的网络（子网）并且具有 IP 地址`172.16.1.67`。我也可以 ping 自己：

```
root@ubuntu-linux:~# ping ubuntu-linux
PING ubuntu-linux (172.16.1.73) 56(84) bytes of data.
64 bytes from 172.16.1.73 (172.16.1.73): icmp_seq=1 ttl=64 time=0.025 ms
64 bytes from 172.16.1.73 (172.16.1.73): icmp_seq=2 ttl=64 time=0.063 ms
^C
--- ubuntu-linux ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 14ms 
rtt min/avg/max/mdev = 0.025/0.044/0.063/0.019 ms
```

这是一种聪明的方法来显示您计算机的 IP 地址-简单地 ping 自己！

您可以使用`hostnamectl`命令来查看和设置计算机的主机名：

```
root@ubuntu-linux:~# hostnamectl 
    Static hostname: ubuntu-linux
 Icon name: computer-vm 
            Chassis: vm
 Machine ID: 106fd80252e541faafa4e54a250d1216 
            Boot ID: c5508514af114b4b80c55d4267c25dd4
 Virtualization: oracle
 Operating System: Ubuntu 18.04.3 LTS 
             Kernel: Linux 4.15.0-66-generic
 Architecture: x86-64
```

要更改计算机的主机名，您可以使用`hostnamectl set-hostname`命令，然后跟上新的主机名：

```
hostnamectl set-hostname new_hostname
```

例如，您可以通过运行以下命令将计算机的主机名更改为`myserver`：

```
root@ubuntu-linux:~# hostnamectl set-hostname myserver 
root@ubuntu-linux:~# su -
root@myserver:~#
```

请记住，您需要打开一个新的 shell 会话，以便您的 shell 提示显示新的主机名。您还可以看到文件`/etc/hostname`已更新，因为它包含新的主机名：

```
root@ubuntu-linux:~# cat /etc/hostname 
myserver
```

# 重新启动您的网络接口

这可能是一种被滥用的方法，但有时重新启动是许多与计算机相关的问题的最快解决方法！我自己也常常滥用重新启动解决大部分计算机问题。

您可以使用`ifconfig`命令将网络接口关闭；您必须在网络接口名称后面跟随`down`标志，如下所示：

```
ifconfig interface_name down
```

例如，我可以通过运行以下命令关闭我的 Wi-Fi 接口`wlan0`：

```
root@myserver:~# ifconfig wlan0 down
```

您可以使用`up`标志来启用网络接口：

```
ifconfig interface_name up
```

例如，我可以通过运行以下命令重新启动我的 Wi-Fi 接口：

```
root@myserver:~# ifconfig wlan0 up
```

您可能还希望同时重新启动所有网络接口。这可以通过以下方式重新启动`NetworkManager`服务来完成：

```
root@myserver:~# systemctl restart NetworkManager
```

现在是时候通过一个可爱的知识检查练习来测试您对 Linux 网络的理解了。

# 知识检查

对于以下练习，打开您的终端并尝试解决以下任务：

1.  将您的主机名更改为`darkarmy`。

1.  显示您的默认网关的 IP 地址。

1.  从您的计算机到`www.ubuntu.com`的路由跟踪。

1.  显示您的 DNS 的 IP 地址。

1.  显示`www.distrowatch.com`的 IP 地址。

1.  关闭您的以太网接口。

1.  重新启动您的以太网接口。
