---
title: ChatGPT无法访问的解决
date: 2023-04-12 15:55:39
author: repoog
excerpt: 虽然ChatGPT很好用，但是由于访问来源的限制，对于非原生IP的要求，许多时候即便翻墙也不能正常访问。本文介绍如果通过利用Cloudflare的Warp解决ChatGPT无法访问的问题。
tags:
  - ChatGPT
  - file descriptor
  - OpenAI
  - Ubuntu
  - Warp
categories:
  - 人工智能
---

ChatGPT是当下最火的AI聊天机器人，由于访问量大等原因，官方从今年开始逐渐收紧了对于平台访问的策略，不久之前笔者的账号也突然无法访问，访问页面显示：

![](images/2023/04/chatgpt-access-denied.jpg)

拒绝访问错误是由于笔者的VPS没有使用原生IP（即机房地址和IP归属地是同一个地方），通过https://bgp.he.net/ 可以查询IP地址的真正归属地，笔者的VPS机房在美国洛杉矶，但IP地址其实属于加拿大。

ChatGPT使用Cloudflare的服务来判断访问者是否使用原生IP，从而拒绝非真实用户的访问。

解决这个问题的办法有两种思路，一种是通过Tor网络，一种是使用Cloudflare Warp。前者通过Tor节点的出口节点可以保证访问IP是原生的，后者是Cloudflare推出的流量加密服务，通过WireGuard隧道以及Cloudflare部署在全球的数据中心为设备和访问目标之间提供安全的、快速的访问。类似于VPN服务，使用Cloudflare Warp后的出口IP地址是Cloudflare自有的数据中心，因此可以保证最终的访问IP是原生的。

使用Tor网络访问不会出现拒绝访问，但会持续不断的要求做真人校验或不断刷新，加上访问速度慢，所以不具备正常使用的条件。

使用Cloudflare Warp解决的网络拓扑如下：

![](images/2023/04/vps-warp-dragam.png)

需要部署和修改的部分如上图中橙色的内容，即部署Cloudflare Warp，修改V2ray配置的Route和OutBound部分。

以Ubuntu为例，部署Cloudflare Warp步骤如下：

``` Shell
curl https://pkg.cloudflareclient.com/pubkey.gpg  sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main"  sudo tee /etc/apt/sources.list.d/cloudflare-client.list
apt install cloudflare-warp

warp-cli register
# 切记要设置为代理模式，全局模式会让VPS成为Cloudflare节点的一部分，导致远程连接断开
warp-cli set-mode proxy
warp-cli connect
```

代理模式默认的端口是40000，在设置模式后便可以通过warp-cli status查看服务状态，如果显示有错误，则无法连接成功。

笔者在这一步遇到以下错误：

> Status update: Unable to connect. Reason: Insufficient system resource: file descriptor

文件描述符资源不足的错误，以Ubuntu 18.04.6 LTS系统为例，修改文件描述符的操作如下：

修改/etc/systemd/system.conf，去掉DefaultLimitNOFILE注释，数值改为65535

修改/etc/systemd/user.conf，去掉DefaultLimitNOFILE注释，数值改为65535

修改/etc/security/limits.conf，在文件末尾添加：

``` Shell
ubuntu soft     nproc          65535
ubuntu hard     nproc          65535
ubuntu soft     nofile         65535
ubuntu hard     nofile         65535
```

systemd目录下的system.conf和user.conf文件分别是Ubuntu系统下系统实例和用户实例运行时的系统配置文件，DefaultLimitNOFILE可以同时修改软限制（soft limit）和硬限制（hard limit）。修改完成后可以通过执行systemctl daemon-reexec重新加载资源管理器令配置生效。

security目录下的limits.conf文件是为了在多用户操作系统下限制不同用户的资源上限，第一列是用户名，第二列是限制类型，第三列是资源类型，第四列是限制的数值。

配置修改完成后使用ulimit -n命令查看open file的数值从1024改为了65535。

Warp连接成功后可执行

``` Shell
curl chat.openai.com
curl chat.openai.com --proxy socks5://127.0.0.1:40000
```

对比以上两条命令执行结果，后者未返回1020错误，即表示连接正常。而后需要修改V2ray的配置文件，增加routing和outbound配置信息并重启V2ray服务：

```
"routing":{
"domainStrategy":"IPIfNonMatch",
"rules":[
        {
            "type":"field",
            "outboundTag":"cloudflare-warp",
            "domain":[
                "openai.com"
            ]
        }
        ]
    }
"outbounds":[
{
    "tag":"cloudflare-warp",
    "protocol": "socks",
    "settings": {
            "servers": [
            {
                "address": "127.0.0.1",
                "port": 40000
            }
            ]
        }
}
    ]
```

这时访问ChatGPT就一切正常了，但有可能依然会遇到页面打转的问题，查看网络连接会发现session请求返回429错误（Too Many Request），这是由于Warp连接出口的IP有太多人共用的原因，所以需要通过重新连接Warp来切换其他IP进行访问：

``` Shell
warp-cli disconnect
warp-cli connect
```

或者

``` Shell
warp-cli disconnect && sleep 5 && warp-cli connect
```

随着使用这种方式的人越来越多，共用IP的情况也会越来越多，因此需要多尝试几次，直到发现一个稳定的、可用的出口IP地址。