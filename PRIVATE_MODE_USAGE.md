# Syncthing 私有模式使用说明

本版本的 Syncthing 已经过定制修改，**默认不加入公共 P2P 网络**，确保您的隐私和网络安全。

## 主要特性

### 1. 默认禁用自动升级
- ✅ 不会自动连接升级服务器
- ✅ 不会自动下载和安装新版本
- ✅ 完全由您控制何时升级

### 2. 默认不加入 P2P 网络
- ✅ 不连接任何公共发现服务器
- ✅ 不使用任何公共中继服务器
- ✅ 不进行局域网广播发现
- ✅ 完全隔离，保护您的隐私

### 3. 服务器启动提示
- ✅ 发现服务器和中继服务器启动时会显示警告信息
- ✅ 明确提示不会加入公共网络

## 使用方法

### 基本启动（私有模式）

```bash
# 直接启动，默认启用私有模式
syncthing

# 或者显式指定
syncthing serve
```

默认配置下：
- **不会**连接到 Syncthing 公共发现服务器
- **不会**使用公共中继服务器
- **不会**进行自动升级
- **只能**通过手动配置的直连地址与其他设备同步

### 启用 P2P 网络（可选）

如果您信任公共 P2P 网络并希望使用它：

```bash
# 临时启用 P2P 网络
syncthing serve --no-p2p=false

# 同时启用自动升级
syncthing serve --no-p2p=false --no-upgrade=false
```

### 命令行参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--no-upgrade` | `true` | 禁用自动升级 |
| `--no-p2p` | `true` | 禁用 P2P 网络（发现和中继） |

## 设备连接配置

在私有模式下，您需要手动配置设备连接：

### 方法一：直连 IP 地址（推荐）

1. 打开 Web GUI（默认 http://127.0.0.1:8384）
2. 点击"添加远程设备"
3. 输入设备 ID
4. 在"地址"栏填写：`tcp://192.168.1.100:22000`（替换为实际 IP）
5. 取消勾选"动态"

### 方法二：私有发现/中继服务器

如果您搭建了自己的发现和中继服务器：

#### 配置私有发现服务器

编辑配置文件（通常在 `~/.config/syncthing/config.xml`）：

```xml
<options>
    <globalAnnounceServer>https://your-discovery-server.com</globalAnnounceServer>
    <globalAnnounceEnabled>true</globalAnnounceEnabled>
</options>
```

#### 配置私有中继服务器

```xml
<options>
    <listenAddress>default</listenAddress>
    <listenAddress>relay://your-relay-server.com:22067</listenAddress>
    <relaysEnabled>true</relaysEnabled>
</options>
```

## 启动私有发现服务器

```bash
cd cmd/stdiscosrv
go run . -listen :8443 -db-dir ./discovery.db
```

启动时会显示：
```
=======================================================
Discovery Server is starting
WARNING: This server is NOT joining the public P2P network
It will only serve local/private discovery requests
=======================================================
```

## 启动私有中继服务器

```bash
cd cmd/strelaysrv
go run . -listen :22067
```

启动时会显示：
```
=======================================================
Relay Server is starting
WARNING: This server is NOT joining the public P2P network
It will only serve local/private relay requests
=======================================================
```

## 网络隔离验证

启动 Syncthing 后，检查日志确认私有模式已生效：

```
[INFO] P2P network is disabled (discovery and relay servers will not be used)
```

如果看到此消息，说明：
- ✅ 已成功隔离公共网络
- ✅ 不会占用他人网络资源
- ✅ 他人不会占用您的网络资源

## 安全建议

### 1. 局域网内同步
- 推荐：使用直连 IP 地址
- 优点：速度快、无需中继、完全私密
- 配置：`tcp://192.168.x.x:22000`

### 2. 跨网络同步
- 方案 A：配置 VPN，然后使用直连
- 方案 B：搭建私有中继服务器
- 方案 C：配置端口转发 + 直连公网 IP

### 3. 定期检查配置
```bash
# 查看当前配置
syncthing --paths
```

检查配置文件确保：
- `globalAnnounceEnabled` 为 `false`（或使用私有服务器）
- `localAnnounceEnabled` 为 `false`
- `relaysEnabled` 为 `false`（或使用私有服务器）

## 常见问题

### Q: 设备无法自动发现怎么办？
A: 这是正常的。私有模式下需要手动配置设备地址。

### Q: 如何临时启用公共网络？
A: 使用命令行参数：`syncthing serve --no-p2p=false`

### Q: 配置文件在哪里？
A:
- Linux: `~/.config/syncthing/config.xml`
- Windows: `%LOCALAPPDATA%\Syncthing\config.xml`
- macOS: `~/Library/Application Support/Syncthing/config.xml`

### Q: 如何完全重置为公共模式？
A:
1. 删除配置文件
2. 使用 `syncthing serve --no-p2p=false --no-upgrade=false` 启动
3. 重新配置所有设备

## 技术细节

### 默认配置修改

本版本修改了以下默认配置：

```protobuf
// proto/lib/config/optionsconfiguration.proto
global_discovery_enabled = false      // 禁用全局发现
local_discovery_enabled = false       // 禁用本地发现
relays_enabled = false                // 禁用中继
auto_upgrade_interval_h = 0           // 禁用自动升级（0=禁用）
```

### 监听地址变更

默认监听地址已移除中继服务器：

```go
// 原始默认配置
DefaultListenAddresses = []string{
    "tcp://0.0.0.0:22000",
    "dynamic+https://relays.syncthing.net/endpoint",  // 已移除
    "quic://0.0.0.0:22000",
}

// 当前默认配置
DefaultListenAddresses = []string{
    "tcp://0.0.0.0:22000",
    "quic://0.0.0.0:22000",
}
```

## 编译说明

如果需要重新编译：

```bash
# 重新生成 protobuf 文件
go generate ./...

# 编译主程序
go build ./cmd/syncthing

# 编译发现服务器
go build ./cmd/stdiscosrv

# 编译中继服务器
go build ./cmd/strelaysrv
```

## 版本信息

- 基于：Syncthing 官方源码
- 修改分支：`private`
- 主要修改：隔离 P2P 网络，禁用自动升级

## 联系与反馈

如有问题或建议，请检查：
1. 配置文件是否正确
2. 防火墙是否允许连接
3. 设备 ID 是否正确
4. 网络连接是否正常

---

**重要提醒：**
- ⚠️ 默认模式下，设备之间需要手动配置连接地址
- ⚠️ 不会自动发现局域网内的其他设备
- ⚠️ 不会通过公共中继服务器转发数据
- ✅ 这是为了保护您的隐私和网络资源

**享受安全、私密的文件同步！** 🔒
