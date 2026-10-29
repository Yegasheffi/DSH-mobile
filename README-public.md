# DSH Mobile

在 Android 手机上查看、监控和控制 **DeepSeek Harness（DSH）** 的任务。

- **手机端**：原生 Android App，DSH 风格的深色任务面板、任务详情、指令输入、停止任务、后台通知。
- **电脑端**：一个零第三方依赖的 Node.js 桥接服务，把 DSH 的本机 API 安全地映射到局域网或私有虚拟网络。
- **跨平台**：同一份桥接代码支持 **macOS / Windows / Linux**；仓库内提供三个平台各自的启动与开机自启脚本。

> 桥接服务只监听局域网 / VPN 私有网络；DSH 本身仍然只监听电脑的 `127.0.0.1`。手机和电脑可以连接同一个 Wi-Fi，也可以都接入 Tailscale、ZeroTier、WireGuard 等虚拟局域网。
>
> **Public 版**：不包含任何配对密钥、`bridge/config.json`、`android/keystore.jks`、个人截图或视频。首次运行 Bridge 会自动生成随机密钥；APK 首次打开需要手动填写桥接地址和密钥。

![DSH Mobile 工作方式](docs/architecture.png)

---

## 1. 功能

- 任务总览：全部 / 运行中 / 等待处理 / 已完成
- 任务状态、轮次、Token、上下文占用、目标、待办、子任务
- 任务详情：对话、工具调用折叠、运行状态
- 任务控制：发指令（排队 / 打断）、停止任务、新建任务
- 任务搜索：按标题、目录、目标、提示内容搜索
- 后台监控：任务等待或完成时发送 Android 通知，点通知直接打开对应任务
- 连接设置：配对地址 + 配对密钥，密钥只保存在手机本地
- 局域网与 VPN 远程访问都支持

---

## 2. 架构

```text
Android App  ──局域网 / Tailscale──▶  电脑上的 DSH Mobile Bridge
                                             │
                                             ▼
                                  127.0.0.1:<DSH 端口>
                                  DeepSeek Harness
```

- 手机不直接访问 DSH 端口，只访问 Bridge。
- Bridge 在电脑本地读取 DSH 的浏览器会话签名密钥，签发仅用于 `127.0.0.1` 的本地 Cookie。
- 手机和 Bridge 之间使用配对密钥鉴权。
- 所有操作都在你自己的电脑和你自己的网络中完成，不经过第三方云服务（除非你主动使用 VPN/隧道）。

---

## 3. 环境要求

### 手机

- Android 8.0（API 26）及以上
- 与电脑在同一个局域网，或已接入同一个 VPN / 虚拟局域网

### 电脑

- 已安装并启动 DeepSeek Harness 桌面端，或正在运行 `dsh --profile web`
- Node.js 22+  
  也可以直接使用 DSH 自带的 Node 运行时（脚本会自动查找）
- 可用的 TCP 端口：默认 `8790`（Bridge）；DSH 默认 `19387`，Web 版常见 `3080`

> DSH 需要先启动一次，生成 `$DSH_HOME/.credentials.yaml`。Bridge 依赖这份本地凭据。

---

## 4. 快速开始

以下命令默认在项目根目录执行。把 `<项目目录>` 替换为你实际解压/克隆的位置。

先下载或克隆本项目，解压到任意目录（路径不要包含特殊字符），然后按下面顺序操作。

### 4.1 启动 DSH

正常启动 DeepSeek Harness 桌面端；或运行 Web 版：

```bash
dsh --profile web --no-open
```

### 4.2 启动 Bridge

#### macOS / Linux

```bash
cd <项目目录>
chmod +x scripts/*.sh scripts/*.command
./scripts/start-bridge.sh
```

macOS 也可以双击 `scripts/start-bridge.command`。

#### Windows

双击：

```text
scripts\start-bridge.bat
```

或在 CMD / PowerShell 中：

```bat
cd /d <项目目录>
scripts\start-bridge.bat
```

启动后窗口会显示类似：

```text
DSH Mobile Bridge 1.0.0
本机:  http://127.0.0.1:8790/m/
手机:  http://192.168.x.x:8790/m/
密钥:  XXXX-XXXX-XXXX
DSH:   127.0.0.1:19387
```

同时会生成 `bridge/connection.txt`。

> 手机连接时使用“手机”那一行的**电脑局域网 IP**，不要使用 `127.0.0.1`。

### 4.3 安装 APK

把 `dist/DSH-Mobile-v1.0.apk` 传到手机安装：

- USB 传输
- 微信 / QQ / 网盘
- 或手机浏览器访问 Bridge 的下载地址：
  `http://<电脑局域网IP>:8790/download/dsh-mobile.apk?key=<配对密钥>`

首次安装需要允许“安装未知来源应用”。

### 4.4 手机连接

打开 App：

- 如果 APK 已预填地址和密钥，直接点“连接”。
- 如果为空或换了电脑/网络，填写：
  - 桥接地址：`http://<电脑局域网IP>:8790`
  - 配对密钥：Bridge 窗口显示的密钥
- 连接成功后进入任务面板。

---

## 5. 三平台启动与开机自启

| 平台 | 手动启动 | 开机自启 |
|---|---|---|
| macOS | `scripts/start-bridge.command` / `scripts/start-bridge.sh` | `scripts/install-autostart-macos.sh` |
| Linux | `scripts/start-bridge.sh` | `scripts/install-autostart-linux.sh` |
| Windows | `scripts\start-bridge.bat` | `scripts\install-autostart-windows.bat` |

### macOS

```bash
./scripts/install-autostart-macos.sh
```

安装为当前用户的 LaunchAgent：登录后自动启动，崩溃自动重启。日志在 `bridge/logs/`。

如果脚本在受限终端或 SSH 中报 `launchctl bootstrap ... Input/output error`，请从 **Terminal.app** 再执行一次；plist 已经写好，也可以手动加载：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.dsh.mobile.bridge.plist
```

### Linux

```bash
./scripts/install-autostart-linux.sh
```

安装为 systemd 用户服务：

```bash
systemctl --user status dsh-mobile-bridge
journalctl --user -u dsh-mobile-bridge -f
```

### Windows

双击 `scripts\install-autostart-windows.bat`，会创建当前用户的计划任务 `DSH Mobile Bridge`，登录时启动。

也可以在 PowerShell 中手动放行防火墙端口（仅专用网络）：

```powershell
New-NetFirewallRule -DisplayName "DSH Mobile Bridge" `
  -Direction Inbound -LocalPort 8790 -Protocol TCP -Action Allow -Profile Private
```

### 停止

- macOS：`launchctl bootout gui/$(id -u)/com.dsh.mobile.bridge`
- Linux：`systemctl --user stop dsh-mobile-bridge`
- Windows：`Stop-ScheduledTask -TaskName "DSH Mobile Bridge"`
- 手动脚本启动的：按 `Ctrl+C`；脱离终端运行的：`./scripts/stop-bridge.sh`

---

## 6. 远程访问（不在同一个局域网）

推荐使用 **Tailscale、ZeroTier 或 WireGuard** 组一个私有虚拟局域网。手机用流量也能连，不需要公网 IP，也不需要把端口映射到公网。

### 6.1 Tailscale（通用步骤）

1. 电脑和手机安装 Tailscale 官方客户端（macOS、Windows、Linux、Android 均有）。
2. 两端登录**同一个 Tailscale 账号**，手机开启 VPN。
3. 在电脑上查看 Tailscale IP（通常是 `100.x.y.z`）或 MagicDNS 名称。
4. App 中填写：

```text
http://100.x.y.z:8790
```

或：

```text
http://<电脑设备名>.<你的 tailnet>.ts.net:8790
```

5. 如果希望使用 HTTPS，可以在电脑上启用 Tailscale Serve：

```bash
tailscale serve --bg 8790
```

然后 App 中填写：

```text
https://<电脑设备名>.<你的 tailnet>.ts.net
```

Tailscale Serve 需要先在 tailnet 管理后台启用；它是可选的。直接使用 `http://100.x.y.z:8790` 也可以，Tailscale 链路本身是加密的。

### 6.2 ZeroTier / WireGuard

同理：电脑和手机加入同一个虚拟网络，App 填电脑的虚拟网络 IP 加端口 `8790`。

### 6.3 不建议直接端口映射

不要把 `8790` 直接映射到公网：

- Bridge 使用明文 HTTP（除非你额外加了 HTTPS 反向代理）
- 配对密钥可能在链路上被截获
- 一旦密钥泄露，对方可以控制你的 DSH

必须公网访问时，至少使用 HTTPS 反向代理、强密钥、IP 白名单，并优先考虑 VPN。

---

## 7. 配置

配置文件：`bridge/config.json`（首次启动自动生成）。

```json
{
  "host": "0.0.0.0",
  "port": 8790,
  "dshPort": 19387,
  "pairingKey": "请改成你自己的随机密钥",
  "defaultCwd": "/path/to/your/projects",
  "defaultPreset": "cordis",
  "refreshSeconds": 4
}
```

### 修改 Bridge 端口

```bash
node bridge/src/server.mjs --port 8791
```

或修改 `config.json` 的 `port`。手机地址同步改端口。

### 指定 DSH 端口

Bridge 会自动探测常见 DSH 端口；如果 DSH 使用非常规端口：

```bash
node bridge/src/server.mjs --dsh-port 3080
```

或修改 `config.json` 的 `dshPort`。

### 修改配对密钥

```bash
node bridge/src/server.mjs --key "你的新密钥"
```

或修改 `config.json` 的 `pairingKey`。至少 6 个字符，建议 12 个以上随机字符。

### 默认工作目录 / Agent 预设

```json
{
  "defaultCwd": "/path/to/your/projects",
  "defaultPreset": "cordis"
}
```

Windows 示例（JSON 中反斜杠要转义）：

```json
{
  "defaultCwd": "C:\\\\Users\\\\you\\\\projects"
}
```

### 修改 APK 默认地址/密钥

编辑 `android/src/com/dsh/mobile/Config.java`：

```java
public static final String DEFAULT_SERVER = "http://192.168.1.10:8790";
public static final String DEFAULT_KEY = "";
```

然后重新构建。

> **公开分发 APK 前必须清空 `DEFAULT_KEY`。** 否则每个拿到 APK 的人都拥有你的配对密钥。

---

## 8. 防火墙与网络

手机连不上时按顺序检查：

1. 手机和电脑是否在同一 Wi-Fi / VPN。
2. 电脑本机能否打开 `http://127.0.0.1:8790/health`。
3. 手机浏览器能否打开 `http://<电脑局域网IP>:8790/health`。
4. 防火墙是否放行 TCP `8790`：
   - **macOS**：系统设置 → 网络 → 防火墙 → 选项，允许 Node 接收传入连接；或临时关闭防火墙测试。
   - **Windows**：允许 Node.js 通过“专用网络”；或用第 5 节的 `New-NetFirewallRule`。
   - **Linux**：`sudo ufw allow 8790/tcp`（只在可信局域网）。
5. 公司、酒店、学校 Wi-Fi 可能启用了 AP 隔离，设备之间不能互访；换手机热点测试。

Bridge 默认监听 `0.0.0.0:8790`，不会把 DSH 端口暴露出去。

---

## 9. 安全说明

- 手机端所有控制请求都必须带配对密钥；密钥错误会被拒绝，连续失败会限流。
- DSH 的浏览器会话 Cookie 由 Bridge 在电脑本地生成，只用于访问 `127.0.0.1`，不会下发给手机。
- Bridge 不保存 DSH 的模型 API Key/Token；它只读取 `$DSH_HOME/.credentials.yaml` 中的浏览器会话签名密钥，在内存中签发短期本地 Cookie。
- 局域网 HTTP 没有 TLS；同网段抓包理论上能看到配对密钥。**不要在不可信网络使用**；需要远程访问时优先用 VPN。
- `bridge/config.json`、`bridge/connection.txt`、`android/keystore.jks` 都含有敏感信息，不要提交到公开仓库或发给别人。
- 公开分发时：
  1. 清空 APK 的 `DEFAULT_KEY`；
  2. 不要把 `config.json`、`connection.txt`、`keystore.jks` 放进公开包；
  3. 让每个用户生成自己的密钥和签名证书。

---

## 10. 跨平台说明

Bridge 是纯 Node.js，使用 `node:http`、`node:fs`、`node:crypto`、`node:os` 等标准库，没有平台专有 API。

平台差异只在启动方式：

| 平台 | 启动 | 开机自启 |
|---|---|---|
| macOS | `start-bridge.command` / `.sh` | LaunchAgent |
| Linux | `start-bridge.sh` | systemd 用户服务 |
| Windows | `start-bridge.bat` | 计划任务 |

Node 查找顺序：

1. `DSH_NODE` 环境变量（可选）
2. `PATH` 中的 `node`
3. `~/.dsh/dsh-runtimes/*/dependencies/node/bin/node`（macOS/Linux）
4. `%USERPROFILE%\.dsh\dsh-runtimes\*\dependencies\node\node.exe`（Windows）

如果 DSH 安装目录不同，安装 Node.js 22+ 并把 `node` 放进 `PATH` 即可。

### 支持哪些 DSH

- DSH Desktop（Electron）——自动探测内嵌 Web 服务端口
- `dsh --profile web`——默认 3080，自动探测
- 自定义端口——用 `--dsh-port` 指定

DSH 的凭据目录可以用 `DSH_HOME` 覆盖。

---

## 11. 从源码构建 APK

需要：

- JDK 17
- Android SDK Platform 35 + Build Tools 35.0.0
- 约 1 GB 磁盘

### 11.1 安装 Android SDK（macOS / Linux）

```bash
./scripts/setup-android-sdk.sh
```

Windows 建议安装 Android Studio，或使用 WSL / Git Bash。

### 11.2 构建

```bash
cd android
./build.sh
```

产物：

```text
dist/DSH-Mobile-v1.0.apk
```

> Public 压缩包里附带的 APK 使用临时签名，方便直接体验；如果你要长期升级，请自己构建并固定 `keystore.jks`。从预编译 APK 切换到自签名 APK 需要先卸载旧版。

首次构建会生成 `android/keystore.jks`：

- alias：`dshmobile`
- store/key password：`dshmobile`

升级 APK 必须使用同一个 keystore；请备份 `keystore.jks`。如果只是自己使用，可以保留默认；如果要公开发布，请换成你自己的签名。

### 11.3 安装

```bash
adb install -r dist/DSH-Mobile-v1.0.apk
```

或在手机上直接打开 APK。

---

## 12. 项目结构

```text
DSH-Mobile/
├── bridge/                 # 电脑端桥接服务（Node，零第三方依赖）
│   ├── src/config.mjs
│   ├── src/dsh.mjs         # DSH 本地 API 客户端、Cookie、会话映射
│   ├── src/server.mjs      # HTTP 服务、鉴权、移动 API
│   ├── public/             # 手机端 Web UI（任务面板）
│   ├── config.json         # 端口、密钥、默认目录
│   └── connection.txt      # 启动后生成的连接信息
├── android/                # Android 壳 + 构建脚本
│   ├── src/com/dsh/mobile/
│   ├── res/
│   ├── AndroidManifest.xml
│   └── build.sh
├── scripts/                # 三平台启动 / 自启 / SDK 安装脚本
├── docs/                   # 架构图与文档图片
├── dist/                   # APK 和可选的教程视频
└── README.md
```

---

## 13. 已知限制

- **阻塞式提问 / 权限确认**：DSH 的阻塞式 `ask_user_question` 或部分权限确认依赖桌面端的前台回答通道。如果任务卡在“运行中”但桌面端有弹窗，请先在桌面端处理，或在 App 里停止任务。
- **工具调用富交互**：App 展示工具调用名称、摘要和截断后的输出；没有 DSH 桌面端完整的 diff、文件预览、终端等面板。需要完整操作时回桌面端。
- **局域网 HTTP**：Bridge 本身没有 TLS；远程访问请用 VPN 或 HTTPS 反向代理。
- **DSH 端口**：自动探测覆盖常见端口；非常规端口请用 `--dsh-port`。
- **Android 版本**：minSdk 26，targetSdk 35。
- **通知**：Android 13+ 需要授予通知权限；部分厂商的后台省电策略可能限制轮询。

---

## 14. 常见问题

**Q: 手机显示“无法连接桥接服务”？**  
A: 确认手机地址是电脑局域网 IP，不是 `127.0.0.1`；确认在同一 Wi-Fi / VPN；检查防火墙是否放行 `8790`。

**Q: Bridge 启动时报“无法读取 DSH 凭据”？**  
A: 先启动一次 DSH，让 `$DSH_HOME/.credentials.yaml` 生成。如果使用自定义 `DSH_HOME`，启动 Bridge 前设置同名环境变量。

**Q: DSH 换了端口？**  
A: Bridge 会自动探测常见端口；也可以 `--dsh-port <端口>` 固定。

**Q: 能不能在外网用？**  
A: 可以，推荐 Tailscale / ZeroTier / WireGuard。不要把 `8790` 直接映射到公网。

**Q: 为什么只有一个 Bridge 实现，不是三份？**  
A: 业务代码完全跨平台，差异只在启动脚本；一份实现避免三套逻辑漂移。macOS、Windows、Linux 的启动与自启脚本都已提供。

**Q: APK 里的默认地址/密钥怎么改？**  
A: 编辑 `android/src/com/dsh/mobile/Config.java`，然后重新构建。公开分发前请清空 `DEFAULT_KEY`。

---

## 15. 分发建议

如果你要把这个项目发给别人使用：

1. 删除 `bridge/config.json`、`bridge/connection.txt`、`bridge/logs/`；
2. 删除 `android/keystore.jks`，让对方自己生成；
3. 清空 `Config.java` 的 `DEFAULT_SERVER` / `DEFAULT_KEY`；
4. 不要打包自己的教程视频、任务截图或会话数据；
5. 在 README 里保留“需要用户自己生成配对密钥”的说明。

这样每个人拿到的都是干净、独立、跨平台的版本。
