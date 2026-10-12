# BJUT-ipv6

## 检查先检查 VPS 双栈是否正常
curl -4 https://api.ipify.org
curl -6 https://api64.ipify.org

以上两条命令正常时分别返回ipv4和ipv6地址

## 准备一个域名

例如：node.example.com -> 260::::::(ipv6地址)
可使用https://dynv6.com/平台申请一个域名来实现这个功能。

## 安装基础软件

apt update
apt upgrade -y
apt install -y nginx certbot python3-certbot-nginx curl openssl ufw

配置防火墙
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 443/udp
ufw enable

运行ufw status应该有：

Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
443/udp                    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
443/udp (v6)               ALLOW       Anywhere (v6)

## 搭建一个HTTPS网站

mkdir -p /var/www/sub
vim /etc/nginx/sites-available/node
填写如下内容：
'''
    server {
        listen 80;
        listen [::]:80;
    
        server_name node.example.com;
    
        location /sub/ {
            alias /var/www/sub/;
            default_type text/yaml;
        }
    
        location / {
            return 404;
        }
    }
'''
把其中的node.example.com换成自己的域名

启用：
ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node

检查并重启服务：
nginx -t
systemctl restart nginx

## 申请HTTPS证书

执行如下命令，其中的域名换成自己的
certbot --nginx -d node.example.com
按照要求输入邮箱，然后同意协议

##  安装 Hysteria2

sudo -i
HYSTERIA_USER=root bash <(curl -fsSL https://get.hy2.sh/)

## 生成 Hysteria2 密码
执行如下命令，来生成密码（记住这个密码）
openssl rand -hex 32


## 配置 Hysteria2 只监听 IPv6

vim /etc/hysteria/config.yaml
在该文件中填入如下内容：
'''
    listen: "[::]:443"
    
    tls:
      cert: /etc/letsencrypt/live/node.example.com/fullchain.pem
      key: /etc/letsencrypt/live/node.example.com/privkey.pem
    
    auth:
      type: password
      password: "这里填写刚才生成的密码"
'''

## 启动 Hysteria2

systemctl enable --now hysteria-server.service
查看状态：
systemctl status hysteria-server.service
出现active (running)为正常

## 证书自动续期后重启 HY2
创建：
vim /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh
写入：
'''
    #!/bin/sh
    systemctl restart hysteria-server.service
'''

添加权限：
chmod +x /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh

## 创建 Clash/Mihomo 订阅
vim /var/www/sub/6f384fd386ac04e41fb31e245a5a5d71.yaml(其中的yaml文件名可以自己更改)
yaml文件中填入如下内容,把 IPv6、域名和密码全部换成你自己的：
'''
    mixed-port: 7890
    allow-lan: false
    mode: rule
    log-level: info
    ipv6: true
    
    dns:
      enable: true
      ipv6: true
      enhanced-mode: fake-ip
      fake-ip-range: 198.18.0.1/16
    
      default-nameserver:
        - '2400:3200::1'
        - '2001:4860:4860::8888'
    
      nameserver:
        - '2400:3200::1'
        - '2001:4860:4860::8888'
    
    proxies:
      - name: "CloudCone-V6"
        type: hysteria2
    
        # 直接填写 VPS IPv6，完全绕过节点 DNS
        server: "2607:xxxx:xxxx:xxxx::1234"
        port: 443
    
        password: "你的HY2密码"
    
        # TLS 证书仍然使用域名验证
        sni: node.example.com
        skip-cert-verify: false
    
    proxy-groups:
      - name: "PROXY"
        type: select
        proxies:
          - "CloudCone-V6"
    
    rules:
      # 本机和局域网保留
      - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
      - IP-CIDR6,::1/128,DIRECT,no-resolve
      - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
      - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
      - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve
    
      # 其他互联网流量全部进入 VPS
      - MATCH,PROXY
'''

到此就形成了https://node.example.com/sub/6f384fd386ac04e41fb31e245a5a5d71.yaml订阅地址，地址中的域名和yaml文件名需要根据自己的实际情况填写。






