# 拜拜软路由👋noOP-GFW

⚠️ 警告：请遵守天朝法律，做一个遵纪守法的好公民。切勿翻牆从事违法行为，否则后果自负！  
> 声明：本人分享和转载的内容，仅限于研究和学习使用，不售卖任何盈利服务。

## 0、更新
2025 年 11 月，我从 v2rayA 换到 UIforFreedom 了，整体思路还是一样的。UIforFreedom 采用 singbox 内核，支持的协议更全面。

---
## 1、方案介绍
我先说个「**暴论**」—— 除了🪜`魔法上网`，80% 的家庭压根不需要 Openwrt 。很多人只是盲目跟风，照着教程把简单问题复杂化，结果 **从“用户”被迫变成了“网管”**，初期陷入各种修网络故障的泥潭（老司机觉得是小菜一碟，但对小白简直是折磨）。  

**⚔️ 个人观点**：家庭网络应「各司其职」。硬路由负责核心连接（稳），NAS 负责应用扩展（玩）。与其「加钱」上高配软路由当轻 NAS，不如直接「硬路由 + NAS」组合。
> 硬路由服务于人，OpenWrt 则需要人服务它。至于稳定性，不是靠“刷固件”能解决的——厂商通过软硬件深度调试得来的优势，软件难以弥补物理层的劣势。

| 核心体验 | 🛡️ 品牌硬路由 | 🐌 低配 OpenWrt 盒子 |
| :--- | :--- | :--- |
| **易用性** | ✅ **傻瓜式**<br>即插即用，魔法上网按需开启 | ⚠️ **需折腾**<br>iStoreOS 界面有改善，依然**被迫当网管** |
| **稳定性** | ✅ **稳如老狗**<br>软硬件高度耦合，经久耐用 | ⚠️ **参差不齐**<br>社区固件百家饭，驱动拼凑 |
| **游戏/小包** | ✅ **硬件加速**<br>专用 NPU 通道，延迟低，抖动小 | ❌ **CPU 转发**<br>小包依赖 CPU，玩游戏时易跳 Ping、卡顿 |
| **网络吞吐** | ✅ **硬件 NAT**<br>跑满千兆时几乎不占 CPU，稳定持续 | ⚠️ **消耗算力**<br>千兆能跑，但 CPU 负载高**拖累系统响应** |
| **并发连接** | ✅ **海量连接**<br>针对 IoT 优化，百台设备轻松扛 | ⚠️ **内存瓶颈**<br>512M/1G 小内存，连接数一多**爆内存死机** |
| **WiFi 覆盖** | ✅ **满血性能**<br>原厂调校 FEM 功放，信号强，漫游丝滑 | ❌ **残废或无**<br>通常无 WiFi 或驱动极烂，**必须外挂 AP** |
| **IPv6 公网** | ✅ **默认支持**<br> IPv6 公网、NAT1 全支持（甚至开放 80/443） | ⚠️ **部分支持**<br>配置繁琐，常缺失 IPv6 或 NAT 类型受限 |
| **功能扩展** | ⚠️ **几乎没有**<br>部分设备可开启 SSH (如装 Lucky) | ✅ **高度定制**<br>万物皆可装，**但这正是系统不稳的根源** |

本方案直接在 NAS 或闲置设备上部署`UIforFreedom`➕`AdGuard Home`，作为高性能`透明代理`和 `DNS 服务器`，替换掉劝退小白、操作繁琐的 `OpenWrt`，不用虚拟机，也不需要额外购买软路由。
> 关于透明代理部分，你可以选择`UIforFreedom`、`v2rayA`、`ShellCrash`，甚至是无图形界面的`dae`，体验基本差不多，看个人喜好。

- 推荐全屋网络接入 AGH（修改路由器 DNS ），去除部分广告，防止大数据追踪
- 可全屋自动分流出国（修改路由器网关），也可以特定设备按需出国（单独设置网关或 http 代理）
- IPv6 正常使用，搭配 Lucky 可以轻松实现远程访问、串流游戏等
- NAT1 正常使用，XBox、Switch 可正常联机，大部分时候不需要游戏加速器
- XBox 可快速修改下载服务器 IP，基本跑满带宽

![agh](https://github.com/juneix/noOP-AGHv2/assets/81808039/bcd3a018-f1ce-434b-9047-f1907f4e83ee)
![xbox-down-ip](https://github.com/juneix/noOP-AGHv2/assets/81808039/efec34fb-0653-4293-85ac-d266fd04f829)
![xbox-speed](https://github.com/juneix/noOP-AGHv2/assets/81808039/38ffa48c-4201-4593-babe-cb3d1a8eb69b)

## 2、硬件配置
使用本方案，你可以选择 NAS 或找台闲置设备，*0 成本*即可抄作业：

- NAS 设备：系统不限，群晖、飞牛、绿联云、极空间等都可以（用 unraid、TrueNAS 老司机应该不会看这个XP）
- 网络要求：设备至少有一个千兆网口（对，单网卡就行~~百兆也不是不能用~~.jpg）
- 闲置设备：旧笔记本或电视盒子都行，对性能基本没啥要求
  - x86：旧笔记本刷 fnOS 飞牛系统，仅需一个 Ventoy 万能 U 盘，成本 0 元
  - arm：电视盒子刷 Armbian、DietPi、海纳思等系统，有闲置就刷机，不要特意去买

## 3、Docker 安装（二选一）

以前我习惯用原生安装的方式，但最近和 AI 交流后发现，在配置较高的设备上，Docker 的 `host` 模式跟原生安装几乎没有差别，而且 Docker 对系统侵入性更小，方便调试。

下面是我在使用的 docker-compose 配置文件，方便你一键抄作业。  
> 配置文件已采用`毫秒镜像`加速，无需出国

```
services:
  adguardhome:
    image: docker.1ms.run/adguard/adguardhome:latest
    container_name: adguardhome
    restart: unless-stopped
    network_mode: host
    volumes:
      - /vol1/1000/docker/agh/work:/opt/adguardhome/work #飞牛直接套用，其他系统修改/vol1/1000/docker/agh/work为自定义目录
      - /vol1/1000/docker/agh/conf:/opt/adguardhome/conf #飞牛直接套用，其他系统修改/vol1/1000/docker/agh/conf为自定义目录
  uif:
    image: docker.1ms.run/ui4freedom/uif:latest
    container_name: uif
    restart: unless-stopped
    network_mode: host
    privileged: true
    devices:
      - /dev/net/tun
    logging:
      options:
        max-size: 10m
networks: {}
```

UIF 后台管理地址`http://IP:9527`，更多使用教程见[[UIF 官方文档](https://ui4freedom.org/UIF_help/docs/quic/intro)。  
AGH 后台管理地址`http://IP:3000`，更多使用教程见 P3TERX 大佬的[AGH 优化增强设置详解](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-2.html)。

## 4、原生安装（二选一）
原生安装需使用 SSH 工具连上你的 Linux 设备，使用官方一键脚本命令（已配置加速，无需出国）安装为系统服务，开机自启动。  
> Win、Mac 自带的终端就行，或者使用功能强大的 [Xterminal](https://www.terminal.icu/)

### 1. 安装 UIforFreedom
[UIforFreedom 项目地址](https://github.com/UIforFreedom/UIF)  

```
curl -L -O "https://fastly.jsdelivr.net/gh/UIforFreedom/UIF@master/uifd/linux_install.sh" && chmod 755 ./linux_install.sh && bash ./linux_install.sh
```

### 2. 安装 AdGuardHome
[AdGuardHome 项目地址](https://github.com/AdguardTeam/AdGuardHome)  

```
curl -s -S -L https://gh-proxy.org/https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```  

## 5、感谢支持
如果本文对你有帮助，可以考虑[赞赏](https://5nav.eu.org/wx-zsm.webp)一下哦～
