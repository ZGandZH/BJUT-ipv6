# clash verge 客户端配置

## 功能简介
通过使用ipv6节点实现本机绝大多数流量走ipv6,以此避免BJUT校园网对网络流量计费（校园网明确规定ipv6流量为免费资源）。

## 前提条件
本机必须获取到了ipv6地址，windows可在cmd中使用ipconfig命令查看本机ip信息，如下图所示：
<img width="1480" height="759" alt="image" src="https://github.com/user-attachments/assets/8665329c-e45f-4a7b-9fe4-db9444d9514b" />
图中以“2001”为前缀的ipv6地址为真实可用v6地址，若只有以“fe80”为前缀的ipv6地址是不行的（fe80类似ipv4地址中的127. 非公网可用ip）


## 客户端选择
建议使用最新版Clash Verge  
获取链接：https://github.com/clash-verge-rev/clash-verge-rev/releases

## 订阅导入与配置
<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/3f36554e-b84a-491b-8e4f-32aefe6fcf86" />
1.按照上图所示操作，将订阅链接填入“链接输入框”，然后点击“导入”按钮。
2.如下图所示，导入成功之后，在“代理”栏中右上角可以选择“规则”也可选择“全局”，目前该节点的配置中两种代理模式无明显区别。
<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/2215f411-e486-421f-a509-41cad65563c0" />
3.接下来正常在“设置”栏中开启系统代理即可，开启代理之后本机绝大多数流量将走ipv6（有少部分流量走ipv4，例如使用ipv4地址进行ssh连接的操作。走ipv4的流量会被校园网正常计费！！）。

## 额外配置建议
1.为实现在使用该节点时可正常访问校内资源（以bjut.edu.cn结尾的网站），可在Clash Verge 的“设置”->"系统代理"旁边齿轮按钮中添加代理绕过设置，具体操作如下图所示：
<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/7d6607dc-194a-410e-884b-8fc934884a64" />
<img width="1176" height="913" alt="image" src="https://github.com/user-attachments/assets/ed6d5202-81cc-4463-a006-8c1475296d3c" />
完成如上操作之后，以“.bjut.edu.cn”结尾的网址是使用本机的ipv4访问，可能正常计费（正常来说校内资源免费访问，不会被计费，未实际测试）
其他需要绕过系统代理的网络地址也可按照如上方法添加。

2.其他配置大家可自由探索。

## 目前的问题
1.网络延迟较高





