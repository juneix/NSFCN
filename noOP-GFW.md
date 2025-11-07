# 拜拜软路由👋noOP-GFW

⚠️ 警告：请遵守天朝法律，做一个遵纪守法的好公民。切勿翻牆从事违法行为，否则后果自负！  
> 声明：本人分享和转载的内容，仅限于研究和学习使用，不售卖任何盈利服务。

## 2025.11 更新
我从 v2rayA 换到 UIF 了，整体思路还是一样的。UIF 采用 singbox 内核，支持的协议更全面。
---
## 前言
本方案使用 Linux 系统部署 ~~v2rayA~~`UIF`➕`AdGuard Home`，作为`透明代理`和 `DNS服务器`，替换掉劝退小白、操作繁琐的 `OpenWrt 软/旁路由`方案。 
 
> 你也可以选择 `ShellCrash`，体验基本差不多，看个人喜好。我觉得 Clash 有点繁琐了。

我先说个「**暴论**」—— 70% 的人其实压根不需要 Openwrt 除了🪜`魔法上网`之外的大部分功能，大部分人都是随大流，根据各种过时的 XX 教程安装了 OP，结果就是各种配置繁杂的过程，折腾时还经常遇到各种原因的网络故障。  

使用 OP 的新手，经过千辛万苦终于出国了，却发现宽带的 IPv6 公网访问没了，NAT1 网络环境也没了？岂不是得不偿失。  
> 针对高阶需求的用户，爱快/OP 是好工具，但它们都不太适合普通人，纯属杀鸡用牛刀，可以但没必要（除非你家里开公司、酒店、民宿等企业场景）。对了，有些老旧设备跑软路由，性能其实不如近两年新出的硬路由产品——我是指家庭网络服务，不只是🪜。

## 方案介绍
本方案操作简单，对设备性能要求很低，Linux 系统运行 UIF➕AGH，整体比 Openwrt 的兼容性和易用性强多了。关于 Linux 的选择，x86设备推荐`飞牛私有云 fnOS`，arm 设备推荐`Armbian`或`DietPi`。
- 推荐全屋网络接入 AGH（修改路由器 DNS ），去除部分广告，防止大数据追踪
- 可全屋自动分流出国（修改路由器网关），也可以特定设备按需出国（单独设置网关或 http 代理）
- IPv6 正常使用，搭配 Lucky 可以轻松实现远程访问、串流游戏等
- NAT1 正常使用，XBox、Switch 可正常联机，大部分时候不需要游戏加速器
- XBox 可快速修改下载服务器 IP，基本跑满带宽

![agh](https://github.com/juneix/noOP-AGHv2/assets/81808039/bcd3a018-f1ce-434b-9047-f1907f4e83ee)
![xbox-down-ip](https://github.com/juneix/noOP-AGHv2/assets/81808039/efec34fb-0653-4293-85ac-d266fd04f829)
![xbox-speed](https://github.com/juneix/noOP-AGHv2/assets/81808039/38ffa48c-4201-4593-babe-cb3d1a8eb69b)

## 系统配置
使用本方案，你只需准备以下**闲置设备**，低成本甚至 **0 成本**即可抄作业：

- 一台运行 Linux 系统的低功耗 arm 或 x86 设备，对性能基本没啥要求
  - 最低配置：~~让卖家帮忙~~刷了 Armbian 的 20 块包邮玩客云，自己刷准备双公头 USB 线
  - 常见配置：旧笔记本刷 Deepin、fnOS 系统，仅需一个 0 成本 Ventoy 万能 U 盘
  - 特殊配置：虚拟机创建 Linux 系统
- 设备至少有一个千兆网口（对，单网卡就行……~~百兆也不是不能用~~.jpg）

- 系统安装参考：
  - [deepin 23 安装指南](https://www.deepin.org/zh/installation-guide-for-deepin-23-new-installation/)
  - [如何安装和初始化飞牛私有云 fnOS？](https://help.fnnas.com/articles/fnosV1/start/install-os.md)
  - [拯救玩客云，刷入armbian](https://mymuwu.net/?p=985)

另一台电脑远程操作该 Linux 设备，需安装  SSH 工具（Win、Mac 自带终端就行，个人推荐简单易用的 [NextSSH](https://codemutex.com/) 或功能更多的 [Xterminal](https://www.terminal.icu/))
> - 如果实在没电脑……手机使用 Termius、ServerBox 等 SSH 工具也可以。

## 安装工具
使用 SSH 工具连上你的 Linux 设备，输入以下一键脚本命令安装所需工具（也可以用 Docker 部署，但推荐安装为系统服务，避免不必要的麻烦）。

### 1. 选装 Github520
请确保你的网络可以顺利访问 Github，我提供一个 [Github520](https://github.com/521xueweihan/GitHub520) 项目供参考，如果还不行请自己解决。  
```
sudo sh -c 'sed -i "/# GitHub520 Host Start/Q" /etc/hosts && curl https://raw.hellogithub.com/hosts >> /etc/hosts'
```

### 2. 安装 UIF
#### （1）Docker
推荐采用 Docker 方式，操作简单，对系统本身无侵入。
作者官方的是 Docker，我顺手改成了 Docker Compose 格式，方便抄作业。
```
services:
  uif:
    image: ui4freedom/uif:latest
    container_name: uif
    network_mode: host
    restart: always
    privileged: true
    logging:
      options:
        max-size: 10m
```
#### （2）一键安装脚本
如果你的设备比较老旧，或者不想使用 Docker，可以选择一键脚本安装。
更多详细内容可参考 [UIF 官网安装文档](https://v2raya.org/docs/prologue/installation/) 。
```
curl -L -O "https://fastly.jsdelivr.net/gh/UIforFreedom/UIF@master/uifd/linux_install.sh" && chmod 755 ./linux_install.sh && bash ./linux_install.sh
```

后台管理地址`http://IP:9527`，更多使用教程见[v2rayA官方文档](https://v2raya.org)。

### 4. 安装 AdGuardHome
GitHub 项目地址 ➡️ https://github.com/AdguardTeam/AdGuardHome  

#### （1）Docker
如果你没有特殊需求，建议直接 host 模式，比较简单省事。
```
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    restart: always
    network_mode: host
    volumes:
      - 【修改为存放配置文件的路径】:/opt/adguardhome/
```
AGH 官方的示例，包含完整桥接映射端口号
```
services:
  adguardhome:
    container_name: adguardhome
    restart: unless-stopped
    volumes:
      - /my/own/workdir:/opt/adguardhome/work
      - /my/own/confdir:/opt/adguardhome/conf
    ports:
      - 53:53/tcp
      - 53:53/udp
      - 67:67/udp
      - 68:68/udp
      - 80:80/tcp
      - 443:443/tcp
      - 443:443/udp
      - 3000:3000/tcp
      - 853:853/tcp
      - 784:784/udp
      - 853:853/udp
      - 8853:8853/udp
      - 5443:5443/tcp
      - 5443:5443/udp
    image: adguard/adguardhome
```
#### （2）一键安装脚本
**一键安装脚本**  
```
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```  
后台管理地址`http://IP:3000`，更多使用教程见 P3TERX 大佬的[AGH优化增强设置详解](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-2.html)。

## 感谢支持
如果本文对你有帮助，可以考虑[赞赏](https://5nav.eu.org/wx-zsm.webp)一下哦～
