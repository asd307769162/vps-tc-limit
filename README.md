# VPS TC Limit

一个小型、透明的 Linux VPS 出站带宽限制工具。它自动识别默认出口网卡，使用内核 `tc` 的 TBF 队列限速，并通过 `systemd` 在重启后自动恢复规则。

## 功能

- 自动识别 IPv4 或 IPv6 默认出口网卡
- 首次安装自动进入设置向导，只需输入出站限速值
- 交互式菜单：启动、关闭、重启、设置速度、状态、安装、卸载
- 同时提供适合自动化的命令行子命令
- 使用 `tc qdisc replace`，可重复执行
- `systemd` 开机自动启动
- 停止时只删除本工具创建的 TBF 根队列
- 配置值严格校验，避免把任意 shell 内容当作配置执行

> 当前版本限制的是 VPS **出站流量**，即用户从 VPS 下载、服务器向外发送数据的方向。它不会限制服务器接收数据的入站方向。

## 系统要求

- Linux
- root 权限
- Bash 4+
- `iproute2`（提供 `ip` 和 `tc`）
- `systemd`（仅持久化安装需要）

Debian/Ubuntu：

```bash
apt update && apt install -y iproute2 git
```

RHEL/AlmaLinux/Rocky Linux：

```bash
dnf install -y iproute-tc git
```

## 安装

一键安装：

```bash
curl -fsSL https://raw.githubusercontent.com/asd307769162/vps-tc-limit/main/vps-tc-limit -o /tmp/vps-tc-limit && chmod +x /tmp/vps-tc-limit && sudo /tmp/vps-tc-limit install
```

或者克隆仓库安装：

```bash
git clone https://github.com/asd307769162/vps-tc-limit.git
cd vps-tc-limit
sudo bash install.sh
```

一键安装完成后会自动显示识别到的出口网卡，并提示输入限速值，例如 `15mbit`。网卡自动识别失败时才会要求手动输入。

以后可随时打开中文交互菜单：

```bash
sudo vps-tc-limit
```

直接按回车会采用默认配置：自动网卡、`15mbit`、`32kb` burst、`400ms` 最大排队延迟。一般只需设置限速值，不需要修改 burst 和 latency 高级参数。

## 命令行用法

```bash
vps-tc-limit start
vps-tc-limit stop
vps-tc-limit restart
vps-tc-limit configure
vps-tc-limit status
vps-tc-limit install
vps-tc-limit uninstall
```

配置文件位置：

```text
/etc/vps-tc-limit.conf
```

示例配置：

```bash
INTERFACE="auto"
RATE="15mbit"
BURST="32kb"
LATENCY="400ms"
```

## 验证限速

```bash
tc -s qdisc show dev eth0
```

实际网卡不一定叫 `eth0`，可以运行：

```bash
ip route get 1.1.1.1
```

正常情况下，状态中会出现：

```text
qdisc tbf ... root ... rate 15Mbit ...
```

注意：`15 Mbit/s` 约等于 `1.875 MB/s`。测速工具显示的瞬时速度可能因统计窗口和 TBF burst 略高，应以较长时间的平均速度为准。

## 卸载

```bash
sudo vps-tc-limit uninstall
```

卸载会关闭限速并删除程序和服务，但保留 `/etc/vps-tc-limit.conf`，方便以后重新安装。若不再需要，可以手动删除该配置文件。

## 安全提示

- 应用规则前建议保持现有 SSH 会话，并另开一个终端验证连接。
- 如果服务商控制台提供带宽限制，优先使用服务商的网络层限速。
- 本工具不修改防火墙、SSH、BBR 或其他内核参数。

## License

[MIT](LICENSE)
