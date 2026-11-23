# 汐汐工具箱

**一体化直播助手** —— VTS 皮套联动 / CS2 数据 / B站砸礼物 / B站直播姬 / 三角洲，全部跑在一个端口上。

> 🧪 当前版本 **v1.0.0-beta**（测试版）

---

## 一句话说明

以前直播要开五六个软件：皮套联动一个、CS2 数据一个、礼物感谢一个、三角洲查询一个……
**现在只需要一个汐汐工具箱**，所有功能都在这一个窗口里，OBS 也只对接一个地址。

---

## 功能

| 模块 | 说明 |
|---|---|
| **CS 皮套联动** | CS2 游戏事件（击杀 / 爆头 / 死亡 / 残血）实时触发 VTube Studio 表情和音效 |
| **CS2 数据** | 实时战况、段位查询、武器数据；支持 OBS 浏览器源叠加显示 |
| **B站砸礼物** | 直播间礼物 / 醒目留言实时上屏，支持档位音效、自定义特效 |
| **B站直播姬** | 弹幕、礼物、SC、盲盒、上舰全量记录；历史查询、统计报表 |
| **三角洲行动** | 战绩查询、每日密码、段位追踪、出货提醒；扫码登录即可 |

---

## 下载安装

### Windows（推荐普通用户）

从 **[Releases](https://github.com/GuiXiJiangSuki/xixi-toolbox-release/releases/latest)** 下载：

```
XixiToolbox-1.0.0-beta-Setup.exe     （约 95 MB）
```

双击安装，桌面会出现「汐汐工具箱」快捷方式。

> 如果 SmartScreen 拦住了，点「更多信息」→「仍要运行」。
> 软件未做数字签名，这是正常现象。

### Linux 服务器 / NAS（Docker）

```bash
docker run -d \
  --name xixitools \
  --restart unless-stopped \
  -p 8800:8800 \
  -v /你的数据路径:/data \
  -e XIXI_PUBLIC_HOST=你的服务器IP \
  -e TZ=Asia/Shanghai \
  guixijiang/xixi-toolbox:latest
```

访问 `http://你的服务器IP:8800/`

**两个必改参数：**

| 参数 | 说明 |
|---|---|
| `/你的数据路径` | 账号、配置、缓存存哪。**必须挂载，否则重建容器数据全丢** |
| `你的服务器IP` | 服务器局域网 IP。生成的 OBS 地址、CS2 GSI 配置都用它 |

详细文档见 [Docker 部署说明](#docker-部署详解)。

### Linux 桌面版

从 Releases 下载：

```
xixi-toolbox-desktop-1.0.0-beta-linux-x64.tar.gz    （约 100 MB）
```

```bash
tar xzf xixi-toolbox-desktop-1.0.0-beta-linux-x64.tar.gz
cd xixi-toolbox
./start.sh
```

需要图形桌面环境（X11 / Wayland）。

### Linux 服务端版（不用 Docker）

```
xixi-toolbox-server-1.0.0-beta-linux-x64.tar.gz    （仅 7.5 MB）
```

```bash
tar xzf xixi-toolbox-server-1.0.0-beta-linux-x64.tar.gz
cd xixi-toolbox
chmod +x start.sh
XIXI_PUBLIC_HOST=192.168.1.100 ./start.sh
```

需要目标机器装 **Node.js ≥ 22**。

---

## 快速上手

1. **启动软件** —— 双击桌面快捷方式，浏览器会自动打开控制台
2. **连接 VTS** —— 「连接设置」填 VTube Studio 的地址（默认 `127.0.0.1:8001`）
3. **配置 OBS** —— 把需要的内容拖进 OBS 浏览器源
4. **开始直播** —— 游戏事件、礼物、弹幕都会自动上屏

### OBS 浏览器源地址

启动后控制台会打印完整地址，格式是：

| 用途 | 地址 |
|---|---|
| CS2 段位 | `http://127.0.0.1:8800/cs2/overlay` |
| 三角洲战绩 | `http://127.0.0.1:8800/df/overlay` |
| B站砸礼物 | `http://127.0.0.1:8800/live/bonker` |
| 事件音效 | `http://127.0.0.1:8800/vts/sound` |

> 服务器部署时把 `127.0.0.1` 换成服务器 IP。
> 浏览器源建议宽 800、高 600，背景透明。

---

## 系统要求

| | Windows | Linux |
|---|---|---|
| 系统 | Windows 10 / 11 x64 | 主流发行版 x64 |
| 内存 | ≥ 1 GB | ≥ 512 MB（服务端）/ 1 GB（桌面）|
| 磁盘 | 500 MB | 300 MB |
| 其它 | 无需装 Node.js | 服务端版需 Node ≥ 22 |

---

## 更新

软件内置更新检测：

- 启动后自动检查，顶栏会提示「🎉 检测到新版本」
- 设置页 → 「版本与更新」→ 检查更新 → 应用内下载 → 重启并安装

更新包来源是多个源并发竞速（GitHub API / ghfast / gh-proxy / jsDelivr），国内访问不会卡。

---

## Docker 部署详解

### docker-compose（推荐）

新建目录，放 `docker-compose.yml`：

```yaml
services:
  xixitools:
    image: guixijiang/xixi-toolbox:latest
    container_name: xixitools
    restart: unless-stopped

    ports:
      - "8800:8800"

    volumes:
      - ./data:/data

    extra_hosts:
      - "host.docker.internal:host-gateway"

    environment:
      TZ: Asia/Shanghai
      XIXI_HOME: /data
      XIXI_NO_OPEN: "1"
      XIXI_PORT: "8800"
      XIXI_BIND: 0.0.0.0
      # 【必须改】服务器局域网 IP
      XIXI_PUBLIC_HOST: "192.168.1.135"

    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

启动：

```bash
docker compose up -d
docker compose logs -f
```

### 环境变量

| 变量 | 必填 | 说明 |
|---|---|---|
| `XIXI_HOME` | ✅ | 数据目录，容器内固定 `/data` |
| `XIXI_PUBLIC_HOST` | ✅ | **服务器局域网 IP**。容器内看到的是 `172.x.x.x`，对 OBS 不可达 |
| `XIXI_BIND` | ✅ | 固定 `0.0.0.0`，否则宿主机访问不到 |
| `XIXI_NO_OPEN` | ✅ | 固定 `1`，容器里没有浏览器 |
| `XIXI_PORT` | | 默认 8800 |
| `TZ` | | 时区，建议 `Asia/Shanghai` |

### 数据目录

```
/data/
├── config.json          # 全局配置
├── logs/                # 日志
├── vts/                 # VTS 配置 + 音效
├── cs2/                 # CS2 配置和缓存
├── df/                  # 三角洲登录态
├── live/                # B站砸礼物配置
└── bililive/            # 直播姬：history.jsonl + history.sqlite
```

**迁移**：整个 `/data` 拷走，新机器挂上去就能用。

### 更新镜像

```bash
docker compose pull
docker compose up -d
```

### 常见问题

**页面打不开**

```bash
docker ps | grep xixi                    # 容器在跑吗
docker compose logs --tail 50            # 有报错吗
curl http://服务器IP:8800/api/core/info   # 端口通吗
```

**OBS 地址是 `127.0.0.1` 或 `172.x.x.x`**

`XIXI_PUBLIC_HOST` 没设或设错了。必须是宿主机在局域网里能被访问到的 IP。改完 `docker compose down && docker compose up -d`。

**VTS 连不上**

VTube Studio 跑在 Windows 上，容器里看不到宿主机的 `127.0.0.1`。在「连接设置」里填**运行 VTS 那台电脑的局域网 IP**（如 `192.168.1.100`），并在 VTS 里开「允许局域网连接」。

---

## Linux 上功能支持

| 功能 | Windows | Linux |
|---|---|---|
| Web 界面 | ✅ | ✅ |
| VTS 皮套联动 | ✅ | ✅ |
| CS2 数据 | ✅ | ✅ |
| B站砸礼物 | ✅ | ✅ |
| B站直播姬 | ✅ | ✅ |
| 三角洲 | ✅ | ✅（浏览器扫码登录）|
| 开机自启 | ✅ | systemd |
| 最小化到托盘 | ✅ | 仅桌面版 |
| 自动更新 | ✅ | ❌ |

> 三角洲是**浏览器扫码登录 + 纯 HTTP 接口**，不依赖 Windows 组件，Linux 上完全可用。
> 唯一区别是 Overlay 的自动更新功能只对 Windows 客户端生效，服务端部署不需要。

---

## B站直播姬 · 数据统计

B站直播姬模块会把直播间所有事件落库（双写：JSONL 原始记录 + SQLite 索引），支持：

- 按日期 / 时间段查询弹幕、礼物、SC、盲盒、上舰
- 观众贡献榜、礼物排行、盲盒盈亏
- 主播日/周/月报表

数据库文件：`<数据目录>/bililive/history.sqlite`，可以用任何 SQLite 工具直接打开分析。

---

## 常见问题

### Windows 版打开后白屏 / 无响应

1. 检查 `C:\Users\<你的用户名>\Documents\汐汐工具箱\logs\` 里的日志
2. 确认 8800 端口没被占用：`netstat -ano | findstr :8800`

### 更新下载很慢

更新走国内加速源，通常几秒完成。如果一直卡住，在设置页点「检测更新」重新触发。

### 数据会丢吗

不会。所有数据在数据目录（默认 `C:\Users\<用户名>\Documents\汐汐工具箱`），卸载重装不删除。

**但换电脑记得手动拷走。**

### 支持哪些直播平台

目前只支持 B站。

---

## 反馈

- 🐛 Bug 或建议：[提 Issue](https://github.com/GuiXiJiangSuki/xixi-toolbox-release/issues)
- 📢 更新公告：软件内「公告」或看本文档的更新日志

---

## 相关项目

- **AstrBot 插件 · 哔哩哔哩查直播**
  [astrbot_plugin_bili_live_info](https://github.com/GuiXiJiangSuki/astrbot_plugin_bili_live_info)
  在 QQ 群里查直播间数据看板、把直播事件实时转发到群。

---

## 说明

- 本项目为**个人自用工具**，按现状提供，不保证稳定性
- 测试版期间功能和接口可能调整
- 三角洲模块的数据来源为官方公开接口，仅做本地统计展示
- 请遵守哔哩哔哩及各平台的用户协议

---

<sub>汐汐工具箱 · Made with ❤️ by 鬼汐酱</sub>
