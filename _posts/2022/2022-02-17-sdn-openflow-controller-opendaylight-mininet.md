---
layout: post
title: "OpenFlow 之路(二) 實作環境架設 OpenDaylight、Mininet"
date: 2022-02-17 15:46:14 +0800
category: network
img: cover/openflow.jpg
description: "本篇會介紹開發用的工具，本篇使用 OpenDaylight 來作為 Controller 並以 Mininet 模擬 Switch 網路拓樸環境，這篇先簡介一下工具的環境架設以及基本用法"
lang: zh-TW
tags: [network, openflow, sdn]
published: true
---

{{page.description}}

### 實作環境
+ OpenDaylight Aluminium SR4 [官方連結](https://docs.opendaylight.org/en/stable-aluminium/downloads.html)
+ Mininet 2.3.0 [官方連結](http://mininet.org/download/)

ODL 跟 Mininet 兩者本身都有許多可以研究的地方，不過這個大坑就等到之後慢慢補吧😂

## Mininet
Mininet 可以用來模擬網路拓樸的建構，包含 Switch、終端機器，而且還提供各個節點的終端操作，可以簡單的在個人環境下建構 SDN 的環境，並且實際模擬節點間的封包傳送

### 建立拓樸
透過上面的官方連結安裝好 Mininet 之後，直接輸入 `mn` 就可以建立一個簡單的拓樸了

```bash
sudo mn
```

拓樸建立成功會看到下面訊息，可以看到現在有兩個 host 被接到同一個 Switch 上

```bash
*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2
*** Adding switches:
s1
*** Adding links:
(h1, s1) (h2, s1)
*** Configuring hosts
h1 h2
*** Starting controller
c0
*** Starting 1 switches
s1 ...
*** Starting CLI:
mininet>
```

透過 `net` 指令可以再次確認網路結構

```bash
mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
c0
mininet>
```

使用 `xterm` 則可以開啟指定節點的終端

![]({{site.baseurl}}/assets/img/mininet-xterm.png)


加上 `-x` 可以在拓樸建立後順便開啟各節點的終端

```bash
sudo mn -x
```

可以簡單在 host 間互相 ping 確認線路順暢

```bash
mininet> h1 ping h2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=1.19 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.187 ms
^C
--- 10.0.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.187/0.689/1.191/0.502 ms
```

後續會利用 Mininet 來建立較複雜的網路架構，並且模擬 OpenFlow Switch 連接到我們準備好的 Controller 來做操作

## OpenDaylight

### 簡介
OpenDaylight 本身是一個 Open Source 的專案，主要用途是支援各種 SDN 的工具，包含這次要講的 OpenFlow，其他還包含多種協定用以管理各種網路設備或是更深入到 Network Function Virtualization(NFV) 相關，這也是個更深入的領域了，主要開發語言為 Java

### 準備環境
先從上面的官方連結下載後解壓縮，由於 ODL 是以 Java 開發的，需要先設定一下 `JAVA_HOME` 的路徑，Aluminium SR4 需要到 Java 11 的支援，這部分就不詳述了

設定好之後執行解壓縮後的 `bin/karaf`，應該會看到下面畫面

```bash
karaf.bat: KARAF_LOG doesn't exist: "D:\Forwork\Project\opendaylight\aluminium-sr4\opendaylight-0.13.4\bin\..\data\log"
karaf.bat: Creating "D:\Forwork\Project\opendaylight\aluminium-sr4\opendaylight-0.13.4\bin\..\data\log"
Apache Karaf starting up. Press Enter to open the shell now...
100% [========================================================================]

Karaf started in 0s. Bundle stats: 14 active, 14 total

    ________                       ________                .__  .__       .__     __
    \_____  \ ______   ____   ____ \______ \ _____  ___.__.|  | |__| ____ |  |___/  |_
     /   |   \\____ \_/ __ \ /    \ |    |  \\__  \<   |  ||  | |  |/ ___\|  |  \   __\
    /    |    \  |_> >  ___/|   |  \|    `   \/ __ \\___  ||  |_|  / /_/  >   Y  \  |
    \_______  /   __/ \___  >___|  /_______  (____  / ____||____/__\___  /|___|  /__|
            \/|__|        \/     \/        \/     \/\/            /_____/      \/


Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown OpenDaylight.

opendaylight-user@root>
```

`karaf` 提供一個互動式介面可以下指令來操作，首先為了操作 OpenFlow 的協定，要先安裝我們需要的 plugin

```bash
opendaylight-user@root>feature:install odl-openflowplugin-flow-services-rest odl-openflowplugin-app-table-miss-enforcer odl-openflowplugin-nxm-extensions odl-mdsal-apidocs odl-openflowplugin-app-topology-manager
```

安裝完成後不會有任何提示，不過建議是先重啟 ODL 比較保險，輸入 `Ctrl+D` 或是 `logout` 可以關閉 `karaf`

重啟之後前往 [http://localhost:8181/apidoc/explorer/index.html](http://localhost:8181/apidoc/explorer/index.html) 預設帳密是 admin/admin，可以看到現有的 API，OpenFlow 的 API 很多可能要等久一點

之後的操作則會透過 RestConf 提供的北向接口來進行操作

---

## 結語
OpenDaylight 實在是一個強大的工具，由於工作上不只是使用還要改 code，當初真的是花了很多的心力在研究這套軟體，而且網路上的資料不是不齊全就是年代久遠，一路走來也是碰了很多牆，才想分享出來希望後人可以不用繼續碰壁，而除了 OpenFlow 之外工作上也有用到 ODL 的其他功能，希望有天能把這個坑給補起來