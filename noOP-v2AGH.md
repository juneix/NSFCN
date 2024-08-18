# 拜拜软路由👋noOP-v2AGH

⚠️ 警告：请遵守天朝法律，做一个遵纪守法的好公民。切勿翻牆从事违法行为，否则后果自负！  
> 声明：本人分享和转载的内容，仅限于研究和学习使用，不售卖任何盈利服务。
---
本方案使用 Linux 系统一键脚本安装 `v2rayA`➕`AdGuard Home`，作为`透明代理`和 `DNS服务器`，替换掉劝退小白、操作繁琐的 `OpenWrt 软/旁路由`方案。 
> 你也可以选择 `ShellCrash` ➕ AdGuard Home，体验基本差不多，看个人喜好。我觉得 Clash 有点繁琐了，就用 v2rayA 了。

我先说个「**暴论**」—— 70% 的人其实压根不需要 Openwrt 除了魔法上网的大部分功能，大部分人都是随大流，根据各种 XX 教程安装了 OP，结果就是各种配置繁杂的过程，还有各种易冲突的插件。  

使用 OP 的新手，经过千辛万苦终于出国了，却发现宽带的 IPv6 公网访问没了，NAT1 网络环境也没了？岂不是得不偿失。  
> 针对高阶需求的用户，OP、爱快是好工具，但它们都不太适合普通人，基本用不上（除非你家里开公司、开酒店民宿）。  

本方案操作简单，对设备性能要求很低，你的 Linux 设备上除了 v2A➕AGH，依然可以安装 CasaOS、1Panel、OMV 等随便折腾，比 Openwrt 的扩展性和易用性强多了。
- 推荐全屋网络接入 AGH（修改路由器 DNS ），去除部分广告，防止大数据追踪
- v2A 可全屋自动分流出国（修改路由器网关），也可以特定设备按需出国（单独设置网关或 socks5 代理）
- IPv6 正常使用，搭配 Lucky 可以轻松实现远程访问、串流游戏等
- NAT1 正常使用，XBox、Switch 可正常联机，大部分时候不需要游戏加速器
- XBox 可快速修改下载服务器 IP，基本跑满带宽

![agh](https://github.com/juneix/noOP-AGHv2/assets/81808039/bcd3a018-f1ce-434b-9047-f1907f4e83ee)
![xbox-down-ip](https://github.com/juneix/noOP-AGHv2/assets/81808039/efec34fb-0653-4293-85ac-d266fd04f829)
![xbox-speed](https://github.com/juneix/noOP-AGHv2/assets/81808039/38ffa48c-4201-4593-babe-cb3d1a8eb69b)

使用本方案，你只需准备以下设备，低成本甚至 0 成本即可抄作业：

- 一台运行 Linux 系统的低功耗 arm 或 x86 设备
  - 最低配置：~~让卖家帮忙~~刷了 Armbian 的 20 块包邮玩客云
  - 常见配置：旧笔记本刷 Deepin 系统，仅需一个 0 成本 Ventoy 万能 U 盘
  - 特殊配置：虚拟机创建 Debian 等 Linux 系统
- 设备至少有一个千兆网口（对，单网卡就行……~~百兆也不是不能用~~.jpg）

> 另一台电脑远程操作该 Linux 设备，需安装  SSH 工具（Win、Mac 自带终端就行，个人推荐简单易用的 [NextSSH](https://codemutex.com/) 或功能更多的 [Xterminal](https://www.terminal.icu/))
> - 如果实在没电脑……手机使用 Termius、ServerBox 等 SSH 工具也可以。

## 一、安装工具
使用 SSH 工具连上你的 Linux 设备，输入以下一键脚本命令安装所需工具（也可以用 Docker 部署，但更推荐脚本安装，设备独立 IP 没有 NAT）。

### 1. 选装 Github520
请确保你的网络可以顺利访问 Github，我提供一个 [Github520](https://github.com/521xueweihan/GitHub520) 项目供参考，如果还不行请自己解决。  
```
sudo sh -c 'sed -i "/# GitHub520 Host Start/Q" /etc/hosts && curl https://raw.hellogithub.com/hosts >> /etc/hosts'
```

### 2. 安装 v2rayA
1. v2rayA 推荐 ➡️ https://github.com/v2rayA/v2rayA  
2. ShellCrash 备选 ➡️ https://github.com/juewuy/ShellCrash

我使用的是基于 Debian 的系统（比如 OMV、Armbian），其他 Linux 发行版参考 [v2rayA 官网安装文档](https://v2raya.org/docs/prologue/installation/)。

**一键安装脚本**  
安装来源是 v2rayA 作者的官网，部分地区可能速度较慢，请耐心等待。  
```
sudo sh -c "$(wget -qO- https://hubmirror.v2raya.org/v2rayA/v2rayA-installer/raw/main/installer.sh)" @ --with-v2ray
```  

**安装 iptables 模块**（已安装可跳过）  
```
apt add iptables ip6tables
```

**启动 v2rayA 服务**  
```
sudo systemctl start v2raya.service
```

**设置 v2rayA 开机自启动**  
```
sudo systemctl enable v2raya.service
```

后台管理地址`http://IP:2017`，更多使用教程见[v2rayA官方文档](https://v2raya.org)。

### 3. 安装 AdGuardHome
GitHub 项目地址 ➡️ https://github.com/AdguardTeam/AdGuardHome  

**一键安装脚本**  
```
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```  

后台管理地址`http://IP:3000`，更多使用教程见 P3TERX 大佬的[AGH优化增强设置详解](https://p3terx.com/archives/use-adguard-home-to-build-dns-to-prevent-pollution-and-remove-ads-2.html)。

## 二、搭配使用 v2A➕AGH
### 1. AGH 初始化
设置时，网页端口默认 `3000`，**DNS 端口建议默认 53，减少不必要的麻烦**。  
如果提示 53 端口已绑定`bind: address already in use`，请参考 [AGH官方文档](https://adguard-dns.io/kb/zh-CN/adguard-home/faq/#bindinuse) 解除占用。  
> AGH 文档暂时没中文版本，请打开浏览器翻译功能，推荐[沉浸式翻译扩展](https://immersivetranslate.com/)。

### 2. v2A 设置参考
点击右上角【⚙️设置】  
- 透明代理：大陆白名单或者 GFWList，☑️开启 IP 转发，☑️开启端口分享
- 实现方式：tproxy
- 分流模式：同上（支持 http 和 socks5 代理）
- 防止 DNS 污染：关闭（搭配 AGH 默认 53 端口使用）
- 其他默认

![v2rayA](https://github.com/juneix/noOP-v2AGH/assets/81808039/497f4eb9-9dc1-426d-9e73-81d427e8d477)

按需【创建】单个节点，或【导入】订阅链接。  
建议选中 3-6 个节点后，左上角启动，更多详细教程可以参考油管或谷歌搜索。
