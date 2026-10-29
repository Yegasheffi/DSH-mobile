# DSH Mobile

在 Android 手机上监控和控制 **DeepSeek Harness（DSH）** 的任务。

- **手机端**：原生 Android APK，内置 DSH 风格的深色任务面板、会话详情、指令输入、停止任务、后台通知。
- **电脑端**：一个零依赖的 Node.js 桥接服务（`bridge/`），把 DSH 本机 Web API 安全地映射到局域网，并提供配对密钥。
- **跨平台**：同一份桥接代码支持 **macOS / Windows / Linux**，分别提供启动脚本和开机自启脚本。不是三套不同的实现。

> 桥接服务只监听局域网 / Tailscale 私有网络，DSH 本身仍然只监听电脑的 `127.0.0.1`。手机和电脑可以在同一个 Wi-Fi，也可以都连接 Tailscale。
>
> 联调状态：macOS + Android 15 模拟器 + 真实 DSH 已完成端到端验证（任务列表、详情、发指令、停止、新建、深链、后台服务）。Windows/Linux 共用同一份 Node 桥接代码，只替换启动/自启脚本，没有平台专有 API。

![DSH Mobile](docs/preview.png)

---

## 教程视频（横屏）

2 分 42 秒 · 1920×1080 · 30fps · 中文配音 + 字幕。

[![教程视频预览](docs/video-contact-sheet.png)](dist/DSH-Mobile-教程-横屏.mp4)

- [DSH-Mobile-教程-横屏-音乐版.mp4](dist/DSH-Mobile-教程-横屏-音乐版.mp4) — 无旁白，使用指定音乐
- [DSH-Mobile-教程-横屏.mp4](dist/DSH-Mobile-教程-横屏.mp4) — 原配音版
- [字幕 SRT](dist/DSH-Mobile-教程-横屏.srt)
- [旁白文案](dist/DSH-Mobile-教程-横屏-文案.md)
- 剪映草稿：`video/jianying_draft/DSH Mobile 教程视频 横屏`

视频里的配对密钥已经打码；演示任务来自真实 DSH 会话。

---

## 1. 它长什么样

- 任务总览：全部 / 运行中 / 等待处理 / 已完成，运行状态、轮次、Token、上下文占用、目标、待办、子任务。
- 任务详情：对话记录、工具调用折叠、目标进度、待办、状态；可直接发指令继续任务、打断当前步骤、停止任务。
- 新建任务：选择工作目录，输入任务内容，立即在 DSH 中创建会话并执行。
- 后台监控：任务进入“等待处理”或从“运行中”变为“已完成”时发系统通知；点通知直接打开对应任务。
- 连接设置：配对地址 + 配对密钥；密钥只保存在手机本地。

上面是 Android 模拟器上的实际截图：任务面板、任务详情、新建任务、连接设置。

---

## 2. 快速开始

### 2.1 电脑上启动 DSH

先正常启动 DeepSeek Harness 桌面端，或运行 Web 版：

```bash
dsh --profile web --no-open
```

桥接服务会自动探测本机 DSH 的端口（常见：`19387`、`3080`、`8080` 等）。

### 2.2 启动桥接服务

在项目根目录执行对应平台的启动脚本。

#### macOS

```bash
./scripts/start-bridge.sh
```

也可以双击 `scripts/start-bridge.command`。

#### Linux

```bash
./scripts/start-bridge.sh
```

#### Windows

双击：

```text
scripts\start-bridge.bat
```

或者在 PowerShell / CMD 中：

```bat
scripts\start-bridge.bat
```

启动成功后窗口会显示：

```text
本机:  http://127.0.0.1:8790/m/
手机:  http://192.168.x.x:8790/m/
密钥:  XXXX-XXXX-XXXX
DSH:   127.0.0.1:19387  ·  darwin
```

同时会写入 `bridge/connection.txt`。**手机连接地址使用“手机”那一行的局域网 IP，不要用 127.0.0.1。**

### 2.3 安装 APK

> 远程访问（不在同一个局域网）请看第 3 节；需要先在手机安装 Tailscale。

把 `dist/DSH-Mobile-v1.0.apk` 传到手机安装（微信/QQ 传输、USB、网盘均可）。

首次安装需要在 Android 上允许“安装未知来源应用”。APK 使用项目内 `android/keystore.jks` 自签名，后续升级需要同一个 keystore。

### 2.4 手机连接

打开 App：

- 如果 APK 是在这台电脑上构建的，默认地址和密钥已经填好，直接点“连接”（当前构建默认指向 `192.168.31.236:8790`）。
- 如果换了电脑/网络，把启动窗口里的 **手机地址** 和 **密钥** 填进去。

连接成功后进入任务面板。手机和电脑必须在同一个局域网；如果连不上，先看第 5 节防火墙。

---

## 3. 远程访问（不在同一个局域网）

推荐 **Tailscale**：手机用流量也能连，电脑不需要公网 IP，DSH 仍然只监听 `127.0.0.1`。

### 3.1 已配置好的 macOS

> 首次签发的 HTTPS 证书可能需要 1–2 分钟；如果 App 报 `SSL routines` / `TLSV1_ALERT_INTERNAL_ERROR`，等 1 分钟再点一次连接即可。

本机已经安装并登录 Tailscale，桥接通过 **Tailscale Serve** 暴露为 tailnet 内 HTTPS：

```text
https://dsh-mobile-mac.tailc44e94.ts.net
```

手机端：

1. 安装 Tailscale Android APK（见 `dist/tailscale-android-1.104.4.apk`，或官方 GitHub Release）。
2. 登录**同一个 Tailscale 账号**（本机是 `Yegasheffi@github`）。
3. 打开 Tailscale 的 VPN 开关。
4. 手机浏览器先访问 `https://dsh-mobile-mac.tailc44e94.ts.net/health`，应返回 `{"ok":true...}`。
5. 在 DSH Mobile 设置里填：
   - 桥接地址：`https://dsh-mobile-mac.tailc44e94.ts.net`（不要加端口）
   - 配对密钥：见 `bridge/connection.txt`

Tailscale Serve 是 **tailnet-only**，不会把地址暴露到公网；手机必须开着 Tailscale。

### 3.2 macOS 重启后

当前 Tailscale 以用户态进程运行。重启后执行：

```bash
cd /Users/tmy1209/DSH-Mobile
./scripts/start-remote.command
```

它会启动 Tailscale userspace daemon、确保 `tailscale serve` 指向 `127.0.0.1:8790`，并启动桥接。

查看状态：

```bash
/opt/homebrew/opt/tailscale/bin/tailscale --socket=$HOME/.tailscale-dsh/tailscaled.sock status
/opt/homebrew/opt/tailscale/bin/tailscale --socket=$HOME/.tailscale-dsh/tailscaled.sock serve status
```

停止：

```bash
./scripts/stop-tailscale.sh
./scripts/stop-bridge.sh
```

### 3.3 Windows / Linux

在电脑和手机上安装官方 Tailscale 客户端，登录同一个账号并开启 VPN。桥接仍然监听 `0.0.0.0:8790`，手机端直接用 Tailscale 分配的 `100.x.y.z` 地址：

```text
http://100.x.y.z:8790
```

如果该平台支持 `tailscale serve`，也可以像 macOS 一样用 HTTPS 域名；否则 HTTP + 配对密钥也能工作（Tailscale 链路本身是加密的）。

> 不要把 `8790` 直接端口映射到公网。Tailscale 已经提供加密组网；公网明文 HTTP 会暴露配对密钥。

## 4. 开机自启 / 后台常驻

### macOS

```bash
./scripts/install-autostart-macos.sh
```

安装为当前用户的 LaunchAgent：登录后自动启动，崩溃自动重启。日志在 `bridge/logs/`。

如果脚本在受限终端/SSH 里报 `launchctl bootstrap ... Input/output error`，请从 **Terminal.app** 手动执行一次脚本；plist 已经写好，也可以直接执行：

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

双击：

```text
scripts\install-autostart-windows.bat
```

会创建当前用户的计划任务 `DSH Mobile Bridge`，登录时启动；可在“任务计划程序”里查看或删除。

如果系统弹出防火墙提示，请允许 **专用网络** 访问 Node.js。也可以手动放行端口：

```powershell
New-NetFirewallRule -DisplayName "DSH Mobile Bridge" -Direction Inbound -LocalPort 8790 -Protocol TCP -Action Allow -Profile Private
```

### 手动停止

- macOS LaunchAgent：`launchctl bootout gui/$(id -u)/com.dsh.mobile.bridge`
- Linux：`systemctl --user stop dsh-mobile-bridge`
- Windows：`Stop-ScheduledTask -TaskName "DSH Mobile Bridge"`
- 直接运行脚本启动的：在窗口按 `Ctrl+C`。
- 脱离终端运行的：`./scripts/stop-bridge.sh`（Windows 用任务管理器结束 node.exe）。

---

## 5. 自定义

### 4.1 改桥接端口

```bash
node bridge/src/server.mjs --port 8791
```

或修改 `bridge/config.json` 的 `port`。手机端地址同步改成新端口。

### 4.2 指定 DSH 端口

通常不需要。自动探测失败时：

```bash
node bridge/src/server.mjs --dsh-port 3080
```

或修改 `bridge/config.json` 的 `dshPort`。

### 4.3 改配对密钥

修改 `bridge/config.json`：

```json
{ "pairingKey": "你的新密钥" }
```

重启桥接服务，在手机 App 设置里重新连接。密钥至少 6 个字符。

### 4.4 默认工作目录 / Agent 预设

```json
{
  "defaultCwd": "/Users/you/projects",
  "defaultPreset": "cordis"
}
```

Windows 示例：`"defaultCwd": "C:\\\\Users\\\\you\\\\projects"`（JSON 中反斜杠要转义）。

### 4.5 修改 APK 内置的默认地址/密钥

编辑 `android/src/com/dsh/mobile/Config.java`：

```java
public static final String DEFAULT_SERVER = "http://192.168.1.10:8790";
public static final String DEFAULT_KEY = "XXXX-XXXX-XXXX";
```

然后重新构建。**如果把 APK 发给别人，建议清空 `DEFAULT_KEY`，让对方手动输入密钥。**

---

## 6. 防火墙与网络

手机连不上时按顺序检查：

1. 手机和电脑是否在同一个 Wi-Fi / 局域网（不要一个连 5G、一个连访客网络）。
2. 电脑是否能打开 `http://127.0.0.1:8790/health`。
3. 手机浏览器是否能打开 `http://<电脑局域网IP>:8790/health`。
4. 防火墙是否放行 TCP `8790`：
   - **macOS**：系统设置 → 网络 → 防火墙 → 选项，允许 Node 接收传入连接；或临时关闭防火墙测试。
   - **Windows**：允许 Node.js 通过“专用网络”；或用上面的 `New-NetFirewallRule`。
   - **Linux**：`sudo ufw allow 8790/tcp`（只建议在可信局域网）。
5. 公司/酒店 Wi-Fi 可能开启了 AP 隔离，设备之间不能互访；换手机热点测试。

> 桥接服务默认监听 `0.0.0.0:8790`，只在局域网可达；它不会把 DSH 端口暴露出去。

---

## 7. 安全说明

- 手机端所有控制请求都必须带配对密钥；密钥错误会被拒绝，连续失败会限流。
- DSH 的浏览器会话 Cookie 由桥接服务在电脑本地生成，只用于访问 `127.0.0.1`，不会下发给手机。
- 桥接服务不保存 DSH 的密钥/Token；它读取 `$DSH_HOME/.credentials.yaml` 里的浏览器会话签名密钥，在内存中签发短期本地 Cookie。
- 局域网 HTTP 没有 TLS，同网段抓包理论上能看到配对密钥。**不要在不可信网络使用**；需要远程访问时建议用 Tailscale / WireGuard 等 VPN，而不是把端口映射到公网。
- `bridge/config.json`、`bridge/connection.txt`、`android/keystore.jks` 都含有敏感信息，不要提交到公开仓库或发给别人。`dist/DSH-Mobile-v1.0.zip` 为了方便本机使用也包含了这些文件；如果要分享，请先删掉密钥和 keystore。

---

## 8. 跨平台说明

桥接服务是纯 Node.js（无第三方依赖），使用 `node:http`、`node:fs`、`node:crypto`、`node:os` 等标准库。平台差异只在启动方式：

| 平台 | 启动 | 开机自启 |
|---|---|---|
| macOS | `scripts/start-bridge.command` / `.sh` | `install-autostart-macos.sh` |
| Linux | `scripts/start-bridge.sh` | `install-autostart-linux.sh` |
| Windows | `scripts/start-bridge.bat` | `install-autostart-windows.bat` |

Node.js 查找顺序：

1. `DSH_NODE` 环境变量（可选）
2. `PATH` 里的 `node`
3. `~/.dsh/dsh-runtimes/*/dependencies/node/bin/node`（macOS/Linux）
4. `%USERPROFILE%\.dsh\dsh-runtimes\*\dependencies\node\node.exe`（Windows）

如果 DSH 安装目录不同，安装 Node.js 22+ 后把 `node` 放进 `PATH` 即可。

### 支持哪些 DSH

- DSH Desktop（Electron）——自动探测它内嵌的 Web 服务端口。
- `dsh --profile web`——默认 3080，自动探测。
- 自定义端口——用 `--dsh-port` 指定。

不依赖 macOS 专有 API；Windows/Linux 上同样通过 DSH 的本地 HTTP API 和凭据文件工作。

---

## 9. 从源码构建 APK

需要：

- JDK 17
- Android SDK Platform 35 + Build Tools 35.0.0
- 约 1 GB 磁盘

### 8.1 安装 Android SDK（macOS/Linux）

```bash
./scripts/setup-android-sdk.sh
```

Windows 建议装 Android Studio，或使用 WSL。

### 8.2 构建

```bash
cd android
./build.sh
```

产物：

```text
dist/DSH-Mobile-v1.0.apk
```

首次构建会生成 `android/keystore.jks`：

- alias：`dshmobile`
- store/key password：`dshmobile`

升级 APK 必须使用同一个 keystore；请备份 `keystore.jks`。

### 8.3 安装到手机

```bash
adb install -r dist/DSH-Mobile-v1.0.apk
```

或在手机上直接打开 APK。

---

## 10. 项目结构

```text
DSH-Mobile/
├── bridge/                 # 电脑端桥接服务（Node，零依赖）
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
├── video/                  # 横屏教程视频工程（分镜、TTS、渲染、剪映草稿）
├── dist/
│   ├── DSH-Mobile-v1.0.apk
│   ├── tailscale-android-1.104.4.apk
│   └── DSH-Mobile-教程-横屏.mp4
└── README.md
```

---

## 11. 已知限制

- **阻塞式提问 / 权限确认**：DSH 的阻塞式 `ask_user_question` 或部分权限确认依赖桌面端的前台回答通道；如果任务卡在“运行中”但桌面端有弹窗，请先在桌面端处理，或在 App 里停止任务。DSH 投影到 `userQuestions` 的 timed/continued 问题会显示在“等待处理”卡片里，并可通过 App 回答。
- **工具调用的富交互**：App 展示工具调用名称、摘要和截断后的输出；没有 DSH 桌面端完整的 diff、文件预览、终端等面板。需要完整操作时回桌面端。
- **局域网 HTTP**：没有 TLS，不建议在不可信网络使用。
- **DSH 端口**：自动探测覆盖常见端口；如果 DSH 使用非常规随机端口，请用 `--dsh-port`。
- **Android 8+**：minSdk 26，targetSdk 35。
- **通知**：Android 13+ 需要授予通知权限；不同厂商的后台省电策略可能限制长连接/轮询。

---

## 12. 常见问题

**Q: 手机显示“无法连接桥接服务”？**  
A: 确认手机地址是电脑局域网 IP，不是 127.0.0.1；确认在同一 Wi-Fi；检查防火墙放行 8790。

**Q: 桥接服务启动时报“无法读取 DSH 凭据”？**  
A: 先启动一次 DSH 桌面端/网页版，让 `~/.dsh/.credentials.yaml` 生成。若使用自定义 `DSH_HOME`，在启动桥接服务前设置同名环境变量。

**Q: DSH 换了端口？**  
A: 桥接会自动探测常见端口；也可以 `--dsh-port <端口>` 固定。

**Q: 能不能在外网用？**  
A: 不建议直接端口映射。用 Tailscale/WireGuard 组网后，手机访问电脑的 VPN 内网 IP 即可，桥接服务不需要改。

**Q: 为什么只有一个桥接实现，不是三份？**  
A: 业务代码完全跨平台，差异只在启动脚本；一份实现避免三套逻辑漂移。macOS/Windows/Linux 的启动与自启脚本都已提供。
