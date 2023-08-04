---
title: 来给校园网多拨提速吧
date: 2023-03-30 16:00:00
tags: ["奇淫异巧"]
---
![](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/c39fc3f47804.png&$format=octetStream&api-version=5.0)
这学期校园网砍成了 10M 带宽，还不能自己拉宽带，谁能受得了？现在的游戏动辄 50G 往上，10M 带宽就是小马拉大车。

*最近喜欢草东，一起来欣赏这张被我反复循环的专辑吧（笑*
{% meting "34674226" "netease" "album" %}
<!-- more -->
## 准备

先给你的路由器刷上[高恪固件](http://www.gocloud.cn/bbs/thread-19614-1-1.html)，使用它的原因，首先是不需要你去理解一些概念，非常轮椅。
还有关键的一点，它的分流很方便，否则使用 OpenWrt 还得维护一个分流表，去分流一些应用的流量去向，比如游戏、BT 下载、流媒体等等，有点麻烦。
博主使用的 N 年前矿渣歌华链路由器，MT7621 + 512 MB 内存，刷高恪体验很不错。再加上一个最便宜的 WIFI6 ty300 做有线 AP，勉强堪用。

## 判断鉴权方式

这里做个大胆的猜想，大部分的学校都是根据学生 ID + 密码，再加上 MAC 地址判断鉴权，就能上网了。
我这里只需要鉴权一次，这个 MAC 就可以一直使用，但有的学校可能每次连接都需要鉴权。
对于后者，你可能需要手动抓包实现一个登录的脚本。

## 通过 Clone 多个 MAC 地址通过验证

将 PC 通过（无线 / 有线）连接校园网，然后修改网卡配置，填入一个随机生成的 MAC 地址，然后登录你校提供的验证器，此时这个 MAC 就可以上网了，**把 MAC 地址记到小本本上**。
![Clone MAC](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/372a0a306519.png&$format=octetStream&api-version=5.0)

接着执行上一步，重复 N 遍，理论网速就是 N 倍。要是有连接设备限制，那就找同学借个号吧（笑。

## 配置高恪

### 硬件连接

将刷了高恪的路由器连接到学校 AP 的 Lan 口。

### 流控多线 -> 单线多拨

勾选启动时不检查多拨配置，添加多拨配置，数量填上面得到的 MAC 地址数量，PPPoE 账号密码随便填，其他按照实际情况填。
![单线多拨](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/4bac32f7e2c8.png&$format=octetStream&api-version=5.0)

### 网络设置 -> 外网设置 -> 批量设置

**MAC 地址是前面记在小本本上的地址。**
修改原来的配置，修改为如下形式：

```
wan_mX dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
```

最后应该得到的是形如如下的格式：

```
wan dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m2 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m3 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m1 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m4 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m5 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
wan_m6 dhcp 0 * * [MAC地址] 1500 telecom fiberx10m 937 937 *
```

保存提交。

### 系统状态 -> 概览

回到概览，正常状态下，虚拟 WAN 应该正确通过 DHCP 得到了地址，并现在已经可以通过这个网口上网。
![概览](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/28ac49a80fbe.png&$format=octetStream&api-version=5.0)

### 流控多线

#### 多线策略

推荐这么设置，也可按需选择。
![多线策略](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/dc59d060d85a.png&$format=octetStream&api-version=5.0)

#### 应用分流

把游戏流量分流到单个虚拟 WAN 里，否则可能会出现断连的现象。
![应用分流](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/4/4596eef51a80.png&$format=octetStream&api-version=5.0)

## 完成

至此，你的网速应该已经叠上了。
在天朝想合理不受监管的上网，想要愉快地网上冲浪，是得做一些特殊的工作。从最开始的 ISP 选择，再到 DNS 分流，最后做广告屏蔽等上层应用，都要花点小心思去做。如果你叠了网速还意犹未尽，弄一台软路由玩玩吧！
完。
