---
title: ssr
date: 2019-10-16 16:38:27
tags: 运维
categories: ssr
---
ubuntu16.04环境下安装服务端Shadowsocks
## 安装pip
安装pip3
```bash
sudo apt install python3-pip
```
## 安装Shadowsocks
```
pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip
sudo ssserver --version
```
## 创建配置文件
```
{
    "server":"::",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

## 测试是否配置成功
```
ssserver -c /mnt/shadowsocks.json
```

## 配置Systemd管理Shadowsocks
```bash
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/home/ubuntu/.local/bin/ssserver -c /mnt/shadowsocks.json
Restart=on-abort

[Install]
WantedBy=multi-user.target

```
## 启动命令
```bash
sudo systemctl start shadowsocks-server
sudo systemctl restart shadowsocks-server
sudo systemctl daemon-reload
systemctl status shadowsocks-server.service
```