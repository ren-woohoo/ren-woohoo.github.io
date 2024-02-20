---
title: Linux Wi-Fi 测试实例
date: 2019-04-10 14:15:27
tags:
    - Linux
    - Wi-Fi
---

在 Linux 下面，我们经常会用到 Wi-Fi 模块，当我们在系统配置好 Wi-Fi 之后，需要通过命令行来测试 Wi-Fi 模块工作是否正常。

## STA 工作模式

### 打开 Wi-Fi

```shell
$ ifconfig wlan0 up

$ ifconfig wlan0
wlan0 Link encap:Ethernet HWaddr 58:2D:34:00:2E:6C
UP BROADCAST MULTICAST MTU:1500 Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:0 (0.0 B) TX bytes:0 (0.0 B)
```

如上图此时的模块加载正常。

### 运行 wpa_supplicant 服务

初始化配置 wpa_suppclicant.conf 文件，这一步的主要目的是为了配置 sockets 接口路径 ctrl_interface，wpa_cli 需要与 wpa_supplicant 进行 socket 通信。

```shell
$ echo ctrl_interface=/var/run/wpa_supplicant > /etc/wpa_supplicant.conf
```

运行 wpa_supplicant 如下（有些平台有现成的 systemd 服务，也可通过 systemctl start wpa_supplicant 启动）

```shell
$ wpa_supplicant -B -i wlan0 -Dnl80211 -c /etc/wpa_supplicant.conf
Successfully initialized wpa_supplicant
```

此时 wpa_supplicant 已成功运行。

### 运行 wpa_cli

默认路径 sockets 接口路径就是 /var/run/wpa_supplicant，如果需要修改路径，可以通过参数 -p <path> 修改：

```shell
$ wpa_cli -p /var/run/wpa_supplicant
wpa_cli v2.5
Copyright (c) 2004-2015, Jouni Malinen <j@w1.fi> and contributors
This software may be distributed under the terms of the BSD license.
See README for more details.
Interactive mode

\>
```

***<u>注意：wpa_cli 如果无法成功运行，请确定 sockets 接口文件是否异常</u>***

### 连接 Wi-Fi

这一步完全是在 wpa_cli 命令窗口执行的。

1. 扫描当前环境 Wi-Fi 列表

    ```shell
    > scan
    OK

    \> scan_results
    bssid / frequency / signal level / flags / ssid
    40:31:3c:16:c6:b8 2472 -43 [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS] AAC@NET_Test
    18:31:bf:47:02:80 2437 -41 [WPA2-PSK-CCMP][ESS] Cleargrass_SZ
    a4:c6:4f:42:fd:cc 2442 -50 [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS] CU_udXA
    ```

2. 选择一个测试所用 Wi-Fi

    我们选用 "Cleargrass_SZ" 为我们所需要的 Wi-Fi，此时 Wi-Fi 信息如下：

    | 参数名   | 数值          |
    | :------- | ------------- |
    | ssid     | Cleargrass_SZ |
    | psk      | 12345678      |
    | key_mgmt | WPA-PSK       |

    ```shell
    > add_network
    0

    \> set_network 0 ssid "Cleargrass_SZ"
    OK

    \> set_network 0 key_mgmt WPA-PSK
    OK

    \> set_network 0 psk "12345678"
    OK

    \> select_network 0
    OK
    <3>WPS-AP-AVAILABLE
    <3>Trying to associate with 18:31:bf:47:02:80 (SSID='Cleargrass_SZ' freq=2437 MHz)
    <3>Associated with 18:31:bf:47:02:80
    <3>WPA: Key negotiation completed with 18:31:bf:47:02:80 [PTK=CCMP GTK=CCMP]
    <3>CTRL-EVENT-CONNECTED - Connection to 18:31:bf:47:02:80 completed [id=0 id_str=]

    \> quit
```

### udhcpc 获取动态 IP

上一步已经成功连接 Wi-Fi，但是此时网络仍然是无法连接的，主要是因为没有获取到有效的动态 IP 地址，在路由器配置为 DHCP 的情况下，设备端可以通过 udhcpc（或者 dhclient）获取动态 IP，如下：

```shell
$ udhcpc -i wlan0 -x hostname:ClearGrass-AirMonitor -R -b
udhcpc: started, v1.25.0
Failed to kill daemon: No such file or directory
udhcpc: sending select for 192.168.1.200
udhcpc: lease of 192.168.1.200 obtained, lease time 86400
Failed to kill daemon: No such file or directory
deleting routers
adding dns 192.168.1.1

$ ifconfig wlan0
wlan0 Link encap:Ethernet HWaddr 58:2D:34:00:2E:6C
inet addr:192.168.1.200 Bcast:192.168.1.255 Mask:255.255.255.0
inet6 addr: fe80::5a2d:34ff:fe00:2e6c/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:1252010 errors:0 dropped:4294775096 overruns:0 frame:0
TX packets:165937 errors:0 dropped:11 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:254033222 (242.2 MiB) TX bytes:45160618 (43.0 MiB)

$ ping -c 4 www.baidu.com

PING www.baidu.com (180.97.33.108): 56 data bytes
64 bytes from 180.97.33.108: seq=0 ttl=56 time=10.074 ms
64 bytes from 180.97.33.108: seq=1 ttl=56 time=8.514 ms
64 bytes from 180.97.33.108: seq=2 ttl=56 time=46.697 ms
64 bytes from 180.97.33.108: seq=3 ttl=56 time=24.836 ms

--- www.baidu.com ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 8.514/22.530/46.697 ms
```

 Wi-Fi STA 模式联网测试结束。

done.
