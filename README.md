# BJUT-ipv6-vps-config

## 1. 检查 VPS 双栈是否正常

首先检查 VPS 的 IPv4 和 IPv6 网络是否均可正常访问：

```bash
curl -4 https://api.ipify.org
curl -6 https://api64.ipify.org
```

以上两条命令正常时，应分别返回 VPS 的 IPv4 地址和 IPv6 地址。

---

## 2. 准备一个域名

准备一个能够解析到 VPS IPv6 地址的域名，例如：

```text
node.example.com -> 260x:xxxx:xxxx:xxxx::xxxx
```

如果没有自己的域名，可以使用：

```text
https://dynv6.com/
```

申请一个免费域名，并为该域名添加 AAAA 记录，将其解析到 VPS 的 IPv6 地址。

---

## 3. 安装基础软件

更新系统软件包：

```bash
apt update
apt upgrade -y
```

安装所需软件：

```bash
apt install -y nginx certbot python3-certbot-nginx curl openssl ufw
```

### 配置防火墙

开放 SSH、HTTP、HTTPS 和 Hysteria2 所需端口：

```bash
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 443/udp
ufw enable
```

查看防火墙状态：

```bash
ufw status
```

正常情况下应看到类似输出：

```text
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
```

---

## 4. 搭建 HTTPS 网站

创建订阅文件存放目录：

```bash
mkdir -p /var/www/sub
```

创建 Nginx 配置文件：

```bash
vim /etc/nginx/sites-available/node
```

写入以下内容：

```nginx
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
```

其中：

```text
node.example.com
```

需要替换为自己的域名。

### 启用 Nginx 配置

```bash
ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node
```

检查 Nginx 配置：

```bash
nginx -t
```

如果没有报错，重启 Nginx：

```bash
systemctl restart nginx
```

---

## 5. 申请 HTTPS 证书

执行：

```bash
certbot --nginx -d node.example.com
```

其中：

```text
node.example.com
```

替换为自己的域名。

按照 Certbot 的提示输入邮箱并同意相关协议即可。

---

## 6. 安装 Hysteria2

切换到 root 用户：

```bash
sudo -i
```

执行 Hysteria2 官方安装脚本：

```bash
HYSTERIA_USER=root bash <(curl -fsSL https://get.hy2.sh/)
```

---

## 7. 生成 Hysteria2 密码

执行：

```bash
openssl rand -hex 32
```

命令会生成一串随机密码，例如：

```text
6f384fd386ac04e41fb31e245a5a5d71xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

请保存好该密码，后续服务端和客户端配置都需要使用。

---

## 8. 配置 Hysteria2 只监听 IPv6

编辑 Hysteria2 配置文件：

```bash
vim /etc/hysteria/config.yaml
```

写入：

```yaml
listen: "[::]:443"

tls:
  cert: /etc/letsencrypt/live/node.example.com/fullchain.pem
  key: /etc/letsencrypt/live/node.example.com/privkey.pem

auth:
  type: password
  password: "这里填写刚才生成的密码"
```

需要将：

```text
node.example.com
```

替换为自己的域名，并将：

```text
这里填写刚才生成的密码
```

替换为前面通过 `openssl` 生成的 Hysteria2 密码。

---

## 9. 启动 Hysteria2

执行：

```bash
systemctl enable --now hysteria-server.service
```

查看服务状态：

```bash
systemctl status hysteria-server.service
```

如果看到：

```text
active (running)
```

说明 Hysteria2 已正常运行。

---

## 10. 配置证书自动续期后重启 Hysteria2

创建 Certbot 证书续期后的执行脚本：

```bash
vim /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh
```

写入：

```bash
#!/bin/sh
systemctl restart hysteria-server.service
```

保存后添加执行权限：

```bash
chmod +x /etc/letsencrypt/renewal-hooks/deploy/restart-hysteria.sh
```

这样，当 Let's Encrypt 证书自动续期后，会自动重启 Hysteria2，使新的证书立即生效。

---

## 11. 创建 Clash / Mihomo 订阅

在 Nginx 的订阅目录中创建一个 YAML 文件，例如：

```bash
vim /var/www/sub/6f384fd386ac04e41fb31e245a5a5d71.yaml
```

其中 YAML 文件名可以自行修改，建议使用较长的随机字符串。

在文件中写入：

```yaml
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

    # TLS 证书仍然使用域名进行验证
    sni: node.example.com
    skip-cert-verify: false

proxy-groups:
  - name: "PROXY"
    type: select
    proxies:
      - "CloudCone-V6"

rules:
  # 本机和局域网保留直连
  - IP-CIDR,127.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR6,::1/128,DIRECT,no-resolve
  - IP-CIDR,10.0.0.0/8,DIRECT,no-resolve
  - IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
  - IP-CIDR,192.168.0.0/16,DIRECT,no-resolve

  # 其他互联网流量全部进入 VPS
  - MATCH,PROXY
```

需要根据自己的实际情况修改以下内容：

- `server`：填写 VPS 的 IPv6 地址；
- `password`：填写前面生成的 Hysteria2 密码；
- `sni`：填写申请 HTTPS 证书时使用的域名。

配置完成后，即可得到类似下面的订阅地址：

```text
https://node.example.com/sub/6f384fd386ac04e41fb31e245a5a5d71.yaml
```

其中域名和 YAML 文件名需要根据自己的实际配置进行修改。

将该地址添加到 Clash Verge / Mihomo 客户端，即可加载对应配置。
