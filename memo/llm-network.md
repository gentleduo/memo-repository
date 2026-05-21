# cisco

## 一、从配置文件直接看出**对外连接端口**

所有节点统一格式：

```
https://xxx.ciscovnp.com:10000
```

**明文固定端口：10000**

1. **控制通道端口**

   Cisco AnyConnect 这里用 HTTPS 管理握手、认证、隧道协商 → **TCP 10000**

2. **实际数据传输端口（IPSec 隧道流量）**

   配置里写死：

xml

```
<RestrictTunnelProtocols>IPSec</RestrictTunnelProtocols>
```

IPSec 标准转发端口：

- **UDP 500**（IKE 密钥协商）
- **UDP 4500**（NAT 穿越下的 IPSec 封装流量）

------

## 二、总结你的流量走向 & 端口

1. 你电脑 → VPN 服务器 **TCP:10000** 做登录、认证、配置拉取
2. 建立 IPSec 隧道：
   - 协商：**UDP 500**
   - 实际上网数据包走隧道：**UDP 4500**
3. 服务器再转发公网

------

## 三、Windows 实时查看当前监听 / 连接端口（精准查）

## 1. 查 AnyConnect 进程占用端口

管理员 CMD 执行：

cmd

```
netstat -ano | findstr anyconnect
```

## 2. 查看当前 VPN 建立的外网连接端口

cmd

```
netstat -an | findstr 10000
netstat -an | findstr 500
netstat -an | findstr 4500
```

## 3. 看本机 VPN 虚拟网卡监听端口

cmd

```
tasklist | findstr vpn
```

------

## 四、最简结论

- **管理连接端口**：TCP 10000（配置文件写死）

- **正式上网隧道流量端口**：UDP 4500

- **密钥协商端口**：UDP 500

  全程没有用本机本地自定义监听端口，是

  主动向外连服务器 10000，再走 IPSec 500/4500。

# Windows代理

## 一、核心原理（非常简单）

- Windows：Cisco AnyConnect 已经连上 VPN → **Windows 本身能走 VPN 外网**
- VirtualBox：Ubuntu 网卡设为 **NAT**（默认就是）
- Ubuntu：把 **所有流量 → Windows 主机 IP → Windows 再走 Cisco VPN 出去**
- **Ubuntu 不需要自己装 VPN、不需要 IPSec、不需要 UDP 500/4500**

------

## 二、第一步：Windows 端准备（必须做）

### 1. 查 Windows 的 VirtualBox NAT 网卡 IP

管理员 CMD：

cmd

```
ipconfig
```

找到类似：

plaintext

```
VirtualBox Host-Only Network
IPv4 Address. . . . . . . . . . . : 192.168.56.1
```

**记下来：Windows 代理 IP = 192.168.56.1（你的可能不一样）**

### 2. Windows 防火墙放行

控制面板 → 防火墙 → 高级设置 → 入站规则 → 新建规则：

- 类型：**自定义**
- 程序：所有程序
- 协议：TCP + UDP
- 本地端口：所有
- 远程端口：所有
- 远程 IP：**192.168.56.0/24**（VirtualBox 网段）
- 允许连接
- 配置文件：全勾
- 名称：**Allow VirtualBox Proxy**

> **如果关闭windows的防火墙，上面的步骤可以不做**

------

## 三、第二步：Windows 上开启一个 Socks5 代理

Cisco VPN 本身**不提供代理端口**，所以你要在 Windows 开一个**Socks5 代理**，监听 192.168.56.1，给 Ubuntu 用。

### 推荐工具（任选一个，都免费）

- **Clash for Windows**（最方便）
- **V2RayN**
- **SocksCap64**

### 以 Clash 为例（10 秒搞定）

1. 打开 Clash
2. 开启：**Allow LAN（允许局域网连接）**
3. 查看 Socks5 端口：默认 **7890**
4. 监听地址选：**192.168.56.1**（刚才查到的 VirtualBox IP）

→ 得到 Ubuntu 要用的代理：

plaintext

```
socks5://192.168.56.1:7890
```

------

## 四、第三步：Ubuntu 里设置全局代理（照抄即可）

### 1. 临时生效（当前终端）

```
export ALL_PROXY=socks5://192.168.56.1:7890
export http_proxy=socks5://192.168.56.1:7890
export https_proxy=socks5://192.168.56.1:7890
```

### 2. 永久生效（所有终端、重启不丢）

```
nano ~/.bashrc
```

```
export ALL_PROXY=socks5://192.168.56.1:7890
export http_proxy=socks5://192.168.56.1:7890
export https_proxy=socks5://192.168.56.1:7890
```

保存退出（Ctrl+O → 回车 → Ctrl+X）

```
source ~/.bashrc
```

### 3. APT 也走代理（必须！否则 apt update 不走 VPN）

```
sudo nano /etc/apt/apt.conf.d/99proxy
```

```
Acquire::http::Proxy "socks5://192.168.56.1:7890";
Acquire::https::Proxy "socks5://192.168.56.1:7890";
```

------

## 五、验证是否成功（Ubuntu 里执行）

```
curl ifconfig.me
```

显示的 IP **和 Windows 浏览器访问 ifconfig.me 结果一样** → 成功！

------

## 六、为什么不用关心 Cisco 的 UDP 端口？

- Cisco AnyConnect 在 Windows 底层走 **UDP 500/4500（IPSec）+ TCP 10000（控制）**
- 但这是 **Windows ↔ VPN 服务器** 的事
- Ubuntu → Windows 只需要 **Socks5（TCP）**
- Windows 收到 Ubuntu 的流量后，**自动通过已建立的 Cisco VPN 隧道转发**

→ **Ubuntu 完全不用碰任何 UDP/IPSec 配置**

------

## 七、如果你不想装 Clash/V2Ray，我给你一个极简方案

Windows 用 **Python 一键开 Socks5 代理**（不用装软件）：

```
pip install socks5server
socks5server -b 192.168.56.1:7890
```

然后 Ubuntu 照旧用 7890 端口。

# ChatGPT 账号登录模式

------

## 一、先删掉旧 API Key 认证（必须）

Ubuntu 里：

```
# 1. 删掉旧的 auth（API key 存在这里）
rm -rf ~/.codex/auth.json

# 2. 清空 OpenAI 环境变量（避免继续走 key）
unset OPENAI_API_KEY
unset openai_api_key

# 3. 可选：防止以后自动带 key
echo "" >> ~/.bashrc
sed -i '/OPENAI_API_KEY/d' ~/.bashrc
```

------

## 二、执行「账号登录」（两种方式，你选一种）

### ✅ 方式 A：Ubuntu 有浏览器（桌面版，最简单）

```
codex login
```

- 会自动弹出浏览器 → 登录你的 **ChatGPT Plus/Pro 账号**
- 授权后自动回到终端 → 显示 **Login successful**

### ✅ 方式 B：Ubuntu 无浏览器（虚拟机 / 纯命令行，你大概率是这种）

用 **设备码登录（device auth）**，不用在 Ubuntu 开浏览器：

```
codex login --device-auth
```

```
1. 打开链接：https://auth.openai.com/codex/device
2. 输入一次性验证码：XXXX-XXXX
```

你 **在 Windows 主机浏览器打开这个链接**，输入验证码，登录你的 ChatGPT 账号，授权即可；**Ubuntu 这边自动完成登录**。

------

## 三、确认当前是「账号模式」而非 API Key

```
codex login --status
```

```
Authenticated with ChatGPT (plan: plus/pro)
```

就对了，**不再走 API key 计费，走你账号订阅**。

------

## 四、关键：Ubuntu 必须能正常访问 OpenAI（你之前的 Cisco/Clash 代理要通）

你现在环境：

- Windows：Cisco VPN → 能上外网
- VirtualBox Ubuntu：要走 Windows 代理

### ✅ Ubuntu 临时代理（终端生效）

```
# 假设你 Windows 网卡 NAT IP：10.0.2.2（VirtualBox 默认）
export http_proxy=http://10.0.2.2:7890
export https_proxy=http://10.0.2.2:7890
export ALL_PROXY=socks5://10.0.2.2:7890
```

> 端口：**7890** 是 Clash for Windows 默认端口，要确认 CFW 开了 **Allow LAN**。

### ✅ 测试连通性

```
curl -I https://auth.openai.com
```

能返回 `200 OK` 再执行 `codex login`，否则会一直卡在登录。

------

## 五、以后怎么切换回 API key（备用）

```
# 先登出账号
codex logout

# 再用 key 登录（管道输入，不进历史）
echo "sk-xxxxxxxx" | codex login --with-api-key
```