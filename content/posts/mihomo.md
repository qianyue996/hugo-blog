---
title: "使用 mihomo 搭建透明代理，实现零DNS泄露"
date: 2026-02-08T22:32:03+08:00
description: "使用 mihomo 搭建透明代理，实现零DNS泄露"
draft: false
---

## 下载内核

使用 `wget` 工具下载 [mihomo](https://github.com/MetaCubeX/mihomo/releases/latest). 下载你对应的 Linux platform 版本.

```sh
wget https://github.com/MetaCubeX/mihomo/releases/download/v1.19.20/mihomo-linux-amd64-compatible-v1.19.20.gz
```

使用 `gunzip` 解压, 确保安装了该解压工具.

```sh
gunzip mihomo-linux-amd64-compatible-v1.19.20.gz
mv mihomo-linux-amd64-compatible-v1.19.20 mihomo
```

## 配置文件

```yaml
mixed-port: 7890
ipv6: false
allow-lan: true
tcp-concurrent: true
unified-delay: true
external-controller: :9090
external-ui: ui
external-ui-url: "https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip"

geodata-mode: true
geox-url:
  geoip: "https://j.1win.ggff.net/https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat"
  geosite: "https://j.1win.ggff.net/https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat"
  mmdb: "https://j.1win.ggff.net/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/country-lite.mmdb"
  asn: "https://j.1win.ggff.net/https://github.com/xishang0128/geoip/releases/download/latest/GeoLite2-ASN.mmdb"

find-process-mode: strict

profile:
  store-selected: true
  store-fake-ip: true

sniffer:
  enable: true
  force-dns-mapping: true
  parse-pure-ip: true
  override-destination: false
  sniff:
    HTTP:
      ports: [80, 8080-8880]
    TLS:
      ports: [443, 8443]
    QUIC:
      ports: [443, 8443]
  skip-domain:
    - "Mijia Cloud"
    - "+.push.apple.com"

tun:
  enable: true
  stack: mixed
  dns-hijack:
    - "any:53"
    - "tcp://any:53"
  auto-route: true
  auto-redirect: true
  auto-detect-interface: true
  strict-route: true

dns: # 白名单模式
  enable: true
  ipv6: false
  enhanced-mode: redir-host
  respect-rules: false # 必须, 否则会无法解析 DNS
  default-nameserver:
    - 223.5.5.5
  nameserver-policy:
    "GEOSITE:cn, GEOSITE:china-list, GEOSITE:geolocation-cn": # 必须设置为白名单模式
      - https://dns.alidns.com/dns-query
  nameserver:
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query

proxies:
  - name: "直连"
    type: direct
    udp: true

proxy-groups:
  - name: 默认
    type: select
    proxies: [自动选择, 手动选择]

  - name: 自动选择
    type: url-test
    include-all: true
    exclude-filter: "剩余|重置|到期"
    exclude-type: direct
    tolerance: 10

  - name: 手动选择
    type: select
    include-all: true
    exclude-filter: "剩余|重置|到期"

rules:
  - DOMAIN-KEYWORD,microsoft.com,默认
  - GEOSITE,google,默认

  - GEOSITE,google-cn,直连
  - GEOSITE,china-list,直连
  - GEOSITE,apple-cn,直连
  - GEOSITE,category-games@cn,直连
  - GEOSITE,category-game-platforms-download,直连
  - GEOSITE,cn,直连

  - GEOIP,lan,直连,no-resolve
  - GEOIP,cn,直连,no-resolve
  - GEOSITE,geolocation-!cn,默认
  - MATCH,默认
```

## 疑难解答

### 安装了 docker 怎么路由网络

由于 docker 使用 iptables 防火墙, 我个人倾向于使用 nftables.

我在 arch 上测试, 只要能够正常启动 nftables, 那么使用 tun 模式启动的 mihomo, 就能够自动配置 nft.

```sh
# 正常启动 nftables, 没有报错即可
systemctl enable --now nftables
```
