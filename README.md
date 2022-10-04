Transparent Proxies
xtoys edited this page on 24 Feb · 3 revisions
 Pages 3
Getting Started
Http Proxies
Transparent Proxies
Clone this wiki locally
https://github.com/xtoys/clash.wiki.git
修改 clash 配置
port: 7890
socks-port: 7891
redir-port: 7892
tproxy-port: 7893 #必须
allow-lan: true
mode: rule
log-level: warning # info / warning / error / debug / silent
ipv6: false
external-controller: 0.0.0.0:9090

dns: # DNS server settings
  enable: true
  listen: 0.0.0.0:53
  ipv6: false
  default-nameserver:
    - 119.29.29.29
  enhanced-mode: redir-host
  nameserver: # 国内域名使用 nameserver 请求
    - https://doh.pub/dns-query #腾讯DNS
    - https://dns.alidns.com/dns-query #阿里DNS
    # - 119.29.29.29
  fallback: # 国外域名使用 fallback 请求 (没有被污染的DNS)
    - https://cloudflare-dns.com/dns-query #Cloudflare DNS
    - https://doh.dns.sb/dns-query #DNS.SB
  fallback-filter: # fallback请求过滤
    geoip: true
......
完整配置示例

Dashboard http://[ip]:9090/ui

宿主机以及 docker 相关设置
1. 配置宿主机网络环境
临时开启宿主机网卡混杂模式
ip link set eth0 promisc on
永久开启宿主机网卡混杂模式
vim /etc/rc.local
在 exit 0 之上添加 ifconfig eth0 promisc 保存即可

2. 配置 docker 网络环境
docker network create -d macvlan \
  --subnet=10.10.10.0/24 \
  --gateway=10.10.10.1 \
  -o parent=eth0 \
  macnet
虚拟网络名称为 macnet，驱动为 macvlan 模式

将 subnet 10.10.10.0 修改为你自己主路由的网段

将 geteway 10.10.10.1 修改为你自己的主路由网关

3. 创建 clash 容器
docker run -d \
 --name clash \
 --network macnet \
 --hostname clash \
 --ip 10.10.10.11 \
 --privileged \
 --restart unless-stopped \
 -e ENHANCED_MODE=enable \
 -v /clash/config.yaml:/clash/config.yaml \
 xtoys/clash:latest
将 10.10.10.11 修改为你主路由网段的 IP

将 /clash/config.yaml 修改为宿主机映射配置所在路径

旁路网关的相关设置
个体设备
设置 旁路由 为网关，个别设备独自通过 旁路由 上网

优点：折腾旁路由时，不影响其他设备网络

缺点：重复设置每个设备

所有设备
路由器 设置 旁路由 为该路由器网关，该路由器下所有内网设备都通过 旁路由 上网

优点：该路由器下所有内网设备都能以旁路由模式上网，无需每台单独设置

缺点：折腾時可能会影响其他设备网络
