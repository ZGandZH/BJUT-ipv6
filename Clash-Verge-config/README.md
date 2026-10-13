# Clash Verge 客户端配置

## 功能简介

通过使用 **IPv6 节点**，实现本机绝大多数网络流量通过 IPv6 传输，以此避免 BJUT 校园网对 IPv4 流量进行计费。

校园网目前明确规定：**IPv6 流量为免费资源**。

> 订阅地址暂不公开。

---

## 前提条件

本机必须已经获取到可用的 **IPv6 地址**。

Windows 用户可以在 `CMD` 中执行以下命令查看本机 IP 信息：

```bash
ipconfig
```

如下图所示：

<img width="1480" height="759" alt="image" src="https://github.com/user-attachments/assets/8665329c-e45f-4a7b-9fe4-db9444d9514b" />

图中以 `2001` 等公网地址段开头的 IPv6 地址为真实可用的 IPv6 地址。

如果本机只有以 `fe80` 开头的 IPv6 地址，则无法满足该配置要求。

> `fe80::/10` 属于 IPv6 链路本地地址，只能在本地链路范围内使用，不能作为公网 IPv6 地址使用。

---

## 客户端选择

建议使用最新版 **Clash Verge Rev**。

官方获取地址：

https://github.com/clash-verge-rev/clash-verge-rev/releases

---

## 订阅导入与配置

<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/3f36554e-b84a-491b-8e4f-32aefe6fcf86" />

### 1. 导入订阅

按照上图所示操作：

1. 将订阅链接填入“链接输入框”；
2. 点击“导入”按钮；
3. 等待订阅导入完成。

---

### 2. 选择代理模式

导入成功后，在“代理”页面右上角可以选择：

- **规则**
- **全局**

目前该节点的配置中，这两种代理模式在实际使用上的区别并不明显。

<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/2215f411-e486-421f-a509-41cad65563c0" />

---

### 3. 开启系统代理

接下来进入“设置”页面，正常开启 **系统代理** 即可。

开启代理后，本机绝大多数网络流量将通过 IPv6 节点传输。

需要注意的是，仍有少部分流量可能直接通过 IPv4 访问，例如：

```text
使用 IPv4 地址进行 SSH 连接
```

这类直接通过 IPv4 产生的流量，仍会被校园网正常计费。

> **注意：**
> 开启 Clash Verge 系统代理并不代表本机所有流量都会强制通过 IPv6。
> 某些未经过系统代理的软件、直连 IPv4 地址的程序或特殊网络连接，仍可能产生 IPv4 流量。

---

## 额外配置建议

### 1. 校内资源代理绕过

为了在使用该节点时仍能正常访问校内资源，例如：

```text
*.bjut.edu.cn
```

可以在 Clash Verge 中进入：

```text
设置
→ 系统代理
→ 点击旁边的齿轮按钮
→ 配置代理绕过
```

具体操作如下图所示：

<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/7d6607dc-194a-410e-884b-8fc934884a64" />

<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/ed6d5202-81cc-4463-a006-8c1475296d3c" />

完成上述配置后，以：

```text
.bjut.edu.cn
```

结尾的网址将绕过 Clash Verge 系统代理，直接使用本机网络访问。

此时可能会使用本机 IPv4，因此理论上存在被正常计费的可能。

不过正常情况下，校内资源访问本身应属于免费流量，暂未进行实际测试。

其他需要绕过系统代理的网络地址，也可以按照相同方式添加。

---

### 2. 其他配置

其他 Clash Verge 相关配置可根据个人需求自行调整和探索。

---

## 目前的问题

### 1. 网络延迟较高

当前节点仍存在网络延迟相对较高的问题，建议使用该节点进行数据集下载或进行其他对网络延迟要求较低的操作。

### 2. 国内网络资源访问困难

当前节点在洛杉矶，所以访问外网资源比较正常，国内网络资源访问会非常慢（bilibili除外）。

