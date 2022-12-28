---
layout: post
title: "OpenFlow 初學之路(四) Meter Table 基本設定與操作"
date: 2022-03-05 15:10:41 +0800
category: network
img: cover/openflow.jpg
description: "本篇簡單介紹基本的 Meter Table 建立，以及透過 Mininet 需要的設定，並且也會進行流量的測試"
lang: zh-TW
tags: [network, openflow, sdn, opendaylight]
published: true
---

{{page.description}}

沒看過前幾篇的可以點這邊:

+ [OpenFlow 初學之路(一) SDN、OpenFlow 簡介](https://bingdoal.github.io/network/2022/02/sdn-openflow-intro/)
+ [OpenFlow 初學之路(二) 實作環境架設 OpenDaylight、Mininet](https://bingdoal.github.io/network/2022/02/sdn-openflow-controller-opendaylight-mininet/)
+ [OpenFlow 初學之路(三) Flow Table 基本設定與操作](https://bingdoal.github.io/network/2022/03/sdn-openflow-flow-table/)

## 拓樸環境建立

```bash
sudo mn --topo single,2 --mac --switch ovsk --controller remote,ip=192.168.xx.xxx,port=6633 -x --ar
```

先建一個跟上次一樣的拓樸來用，一樣基本上 host 間一開始是沒辦法互聯

不過除此之外 mininet 還需要特別改個模式才能正常使用 meter 的功能，輸入下面指令:

```bash
sh ovs-vsctl set bridge s1 datapath_type=netdev
```

## Meter

Openflow 提供一個 Meter 的功能，用途在於限制該 Flow 的流量上限，設計上需要先針對 Meter 的規則建立一張表，在建立 Flow Entry 時只需要指定想要套用的 Meter 表，就能夠讓通過這個 Flow Entry 的流量被限制

### 對照組

既然要進行流量限制的測試，那首先要有正常情況的對照組來才可以，這邊對照組的 Flow Table 設定可以參考[前篇文章](https://bingdoal.github.io/network/2022/03/sdn-openflow-flow-table/)，基本兩個 host 互通就可以了

不過測速這邊我們需要用到一個工具 `iperf`，這是在 CLI 下用的測速工具，首先在 h2 的 xterm 裡啟動一個 `iperf` 的 server

```shell
iperf -s
```

然後再從 mininet 的 CLI 下從 h1 到 h2 進行測速:

```shell
h1 iperf -c h2 -i 1 -t 15
```

參數的意義是，每 1 秒測一次，測速時間為 15 秒，測速的結果會 output 出來到 terminal 上

不過這邊筆者在進行的時候有遇到 `iperf` 沒辦法連通的情況，這邊請先確認 `h1 ping h2` 是暢通的，然後再進行以下指令設置:

```bash
s1 ethtool -K enp0s3 tx off
h1 ethtool -K h1-eth0 tx off
h2 ethtool -K h2-eth0 tx off
```

設定好之後應該可以成功使用 `iperf` 測速，並且看到以下輸出:

```shell
mininet> h1 iperf -c h2 -i 1 -t 15
------------------------------------------------------------
Client connecting to 10.0.0.2, TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.1 port 56366 connected with 10.0.0.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec  22.5 MBytes   189 Mbits/sec
[  3]  1.0- 2.0 sec  20.9 MBytes   175 Mbits/sec
[  3]  2.0- 3.0 sec  20.0 MBytes   168 Mbits/sec
[  3]  3.0- 4.0 sec  20.8 MBytes   174 Mbits/sec
[  3]  4.0- 5.0 sec  18.5 MBytes   155 Mbits/sec
[  3]  5.0- 6.0 sec  16.0 MBytes   134 Mbits/sec
[  3]  6.0- 7.0 sec  18.4 MBytes   154 Mbits/sec
[  3]  7.0- 8.0 sec  19.5 MBytes   164 Mbits/sec
[  3]  8.0- 9.0 sec  21.0 MBytes   176 Mbits/sec
[  3]  9.0-10.0 sec  19.0 MBytes   159 Mbits/sec
[  3] 10.0-11.0 sec  16.2 MBytes   136 Mbits/sec
[  3] 11.0-12.0 sec  18.2 MBytes   153 Mbits/sec
[  3] 12.0-13.0 sec  21.0 MBytes   176 Mbits/sec
[  3] 13.0-14.0 sec  19.6 MBytes   165 Mbits/sec
[  3] 14.0-15.0 sec  19.1 MBytes   160 Mbits/sec
[  3]  0.0-15.0 sec   291 MBytes   162 Mbits/sec
```

可以看到平均大概在 `160 Mbits/sec` 左右，那下面我們就來實際測試套用到 meter 之後的流量差異

### Meter Table 建立

那首先就來建一個 Meter Table，一樣我們透過 OpenDaylight 的 API:

```json
/*
PUT http://localhost:8181/restconf/config/opendaylight-inventory:nodes/node/openflow:1/flow-node-inventory:meter/1
Basic Auth: admin/admin
*/

// Request Body:
{
    "flow-node-inventory:meter": {
        "meter-id": "1",
        "meter-name": "meter test",
        "flags": "meter-kbps",
        "meter-band-headers": {
            "meter-band-header": [
                {
                    "band-id": 0,
                    "drop-rate": 500,
                    "drop-burst-size": 0,
                    "meter-band-types": {
                        "flags": "ofpmbt-drop"
                    }
                }
            ]
        }
    }
}
```

參數相較於 flow entry 簡單很多，主要控制流量大小的就是 `drop-rate` 這個欄位，單位是 Kbps，這邊我們設定的是 500 Kbps，也就是 500 Kbps 的流量就會被 drop 掉

API 發送成功後可以用以下指令檢閱 switch 的 meter table 有沒有正確新增

```bash
sh ovs-ofctl -O openflow13 dump-meters s1
```

然後 flow entry 的設定也要小改一下，讓流量經過 meter table 之後才會被限速:

```json
/*
PUT http://localhost:8181/restconf/config/opendaylight-inventory:nodes/node/openflow:1/flow-node-inventory:table/0/flow/1
Basic Auth: admin/admin
*/

// Request Body:
{
    "flow-node-inventory:flow": [
        {
            "id": "1",
            "table_id": 0,
            "priority": 2,
            "hard-timeout": 1200,
            "installHw": false,
            "flow-name": "h1 to h2",
            "strict": false,
            "match": {
                "in-port": 1,
                "ethernet-match": {
                    "ethernet-destination": {
                        "address": "00:00:00:00:00:02"
                    }
                }
            },
            "instructions": {
                "instruction": [
                     {
                        "order": 0,
                        "meter": {
                            "meter-id": 1
                        }
                    },
                    {
                        "order": 1,
                        "apply-actions": {
                            "action": [
                                {
                                    "order": 0,
                                    "output-action": {
                                        "output-node-connector": 2,
                                        "max-length": 60
                                    }
                                }
                            ]
                        }
                    }
                ]
            },
            "idle-timeout": 3400
        }
    ]
}
```

記得 h2 -> h1 的 flow entry 也要改下，然後同樣輸入上面的 `iperf` 指令，得到結果:

```bash
mininet> h1 iperf -c h2 -i 1 -t 15
------------------------------------------------------------
Client connecting to 10.0.0.2, TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.1 port 56374 connected with 10.0.0.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec   419 KBytes  3.43 Mbits/sec
[  3]  1.0- 2.0 sec  94.7 KBytes   776 Kbits/sec
[  3]  2.0- 3.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  3.0- 4.0 sec  0.00 Bytes  0.00 bits/sec
[  3]  4.0- 5.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  5.0- 6.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  6.0- 7.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  7.0- 8.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  8.0- 9.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  9.0-10.0 sec  63.6 KBytes   521 Kbits/sec
[  3] 10.0-11.0 sec  63.6 KBytes   521 Kbits/sec
[  3] 11.0-12.0 sec  63.6 KBytes   521 Kbits/sec
[  3] 12.0-13.0 sec  0.00 Bytes  0.00 bits/sec
[  3] 13.0-14.0 sec  63.6 KBytes   521 Kbits/sec
[  3] 14.0-15.0 sec  63.6 KBytes   521 Kbits/sec
[  3]  0.0-15.4 sec  1.18 MBytes   647 Kbits/sec
```

雖然有一些輸出怪怪的，但大致的平均速度也是落在 `500 kbits/sec` 左右，可以知道 meter 是有在正常運作，至此算是完成了這一次的 demo

---

## 結論

meter 在 demo 的時候有遇到比較多的障礙是在環境建置上，包含 mininet 的模式設定跟 iperf 不通的問題，主要也想把這些流程筆記下來，不過寫到這邊好像有點越寫越心虛，似乎真的要多補一點背景知識才對，不過還是想先快點記錄下來，生怕會遺忘這些流程，下篇預計撰寫 group 的部分，應該也是照這個模式，搭配實做跟 demo 驗證
