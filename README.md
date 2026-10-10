<div align="center">

# ESP32 Web Flasher

### 一个 HTML 文件搞定 ESP32 的烧录、调试与 MicroPython 开发

纯浏览器实现 · 无需安装驱动软件 / Python / IDE · 打开网页即用

[![Chrome](https://img.shields.io/badge/Chrome-%E2%9C%93-success)](https://www.google.com/chrome/)
[![Edge](https://img.shields.io/badge/Edge-%E2%9C%93-success)](https://www.microsoft.com/edge)
[![Platform](https://img.shields.io/badge/ESP32-%2F%20S3%20%2F%20C3-blue)](#支持的芯片)
[![License](https://img.shields.io/badge/license-MIT-green)](#许可证)

[在线使用 (建议自行部署)](#快速开始) · [功能特性](#功能特性) · [使用说明](#使用说明) · [常见问题](#常见问题)

</div>

---

## 这是什么

`flasher.html` 是一个**单文件** ESP32 网页工具，基于浏览器的 **Web Serial API** 与芯片直接通信。它把日常开发中四个独立工具的工作流整合到一个页面里：

| 传统方式 | 本工具 |
|---|---|
| esptool.py / Flash Download Tool 烧录固件 | 网页选文件 → 一键烧录 |
| 串口助手看日志 | 串口调试页实时监视 |
| Thonny 管理 MicroPython 文件 | 文件页上传 / 下载 / 编辑 / 运行 |
| 万用表 + 示波器排查引脚 | 全引脚状态网格 + 直接输出 / PWM 控制 |

**零安装、零配置、跨平台**（Windows / macOS / Linux / ChromeOS），也可以部署到内网给团队共用。

## 功能特性

### 1. 固件烧录
- 基于官方 [esptool-js](https://github.com/espressif/esptool-js)，协议与 `esptool.py` 一致
- 自动进入下载模式（DTR/RTS 自动复位电路），**无需手动按 BOOT 键**
- 支持 115200 ~ 921600 波特率，实时进度条与速度显示
- 合并固件烧 `0x0`，app 固件烧 `0x10000`；支持整片擦除
- 烧录完成自动复位启动

### 2. 串口调试
- 实时串口监视（默认 115200），硬件复位（EN）
- 发送任意数据，可选追加换行；一键清屏、自动滚动

### 3. MicroPython 文件管理（类 Thonny）
- **列文件 / 下载打开 / 上传 / 删除 / 保存**，板端文件一览
- 编辑器代码一键**直接运行**，MicroPython 运行时自动编译，无需 mpy-cross
- 输出**实时流式显示**，`while True` 主循环也适用，Ctrl-C 随时停止
- raw REPL 协议 + 多项抗干扰设计（见下）

### 4. 全引脚监测
- 支持 **ESP32 / ESP32-S3 / ESP32-C3** 三种芯片配置
- **自动探测可用引脚**：开始时逐脚尝试构造，Flash / PSRAM 占用脚自动标记禁用——不同容量模组（N8 / N16R8、Quad / Octal PSRAM）均自适应，不再出现 `ValueError: invalid pin`
- 每个引脚可独立配置 **数字 / ADC / 跳过**，支持批量模式
- 数字高电平绿色高亮、电平变化黄色描边；ADC 显示 0~4095，亮度随电压变化
- 可选上拉 / 下拉 / 悬空，300ms / 500ms / 1s 轮询；板上**无常驻程序**，不污染串口

### 5. 直接控制引脚
- 任意引脚**数字输出**（置高 / 置低）
- **PWM 输出**：1kHz，占空比 0~1023 滑块调节
- PWM 对象板端按引脚缓存复用，拖动滑块不重启外设，60ms 防抖

### 针对真实硬件踩坑的鲁棒设计
- **软重启 (Ctrl-D) 代替硬复位**：S3 原生 USB 口硬复位会重新枚举导致 "device has been lost"，软重启不重枚举，同时干净销毁全部定时器与线程
- 启动窗口连续 Ctrl-C，抢在 `main.py` 自启前拦截，防止后台定时器打印污染协议
- **文本标记定界传输**：对 USB 包边界错位 / 数据中误插入 EOT 免疫
- 操作前静默校验、执行超时自动 Ctrl-C、设备掉线自动复位 UI
- 串口在「烧录 / 监视 / IDE / 引脚」各模块间自动加解锁切换，不会互相抢占

## 快速开始

### 方式一：直接打开（最简单）

下载或克隆本仓库，双击 `flasher.html` 用 **Chrome / Edge** 打开即可。

> 个别浏览器版本对 `file://` 页面限制 Web Serial，此时用方式二。

### 方式二：本地服务

```bash
# 任选其一
python -m http.server 8000
npx serve .
```

访问 <http://localhost:8000/flasher.html>

### 方式三：部署到服务器

把单个 `flasher.html` 放到任意静态托管（GitHub Pages / Nginx / 内网服务器），浏览器访问即可，非常适合教学和团队批量部署。

## 使用说明

1. USB 连接开发板，点 **连接串口**，选择对应串口（CH340 / CP210x / 原生 USB）
2. 按需要切换标签页：
   - **固件烧录**：选 `.bin`、填地址（合并固件 `0x0`）→ 开始烧录
   - **串口调试**：开始监视，查看日志 / 发送命令
   - **文件 / MicroPython**：刷新文件、编辑、运行、保存（需已刷入 MicroPython）
   - **引脚监测**：配置引脚 → 开始监测；底部可直接控制输出 / PWM

烧 MicroPython 固件：官网 <https://micropython.org/download/>，ESP32-S3 选 [ESP32_GENERIC_S3](https://micropython.org/download/ESP32_GENERIC_S3/)，官网所有 `.bin` 均为含 bootloader + 分区表的合并固件，地址填 `0x0`。

## 支持的芯片

| 芯片 | 烧录 | 串口 | MicroPython | 引脚监测 |
|---|:---:|:---:|:---:|:---:|
| ESP32 | ✅ | ✅ | ✅ | ✅ |
| ESP32-S3 | ✅ | ✅ | ✅ | ✅ |
| ESP32-C3 | ✅ | ✅ | ✅ | ✅ |
| 其他 ESP 系列 | 未测试 | — | — | — |

## 常见问题

<details>
<summary><b>提示“当前浏览器不支持 Web Serial”</b></summary>

请使用桌面版 Chrome / Edge（或其他 Chromium 内核浏览器）。手机端不支持 Web Serial。
</details>

<details>
<summary><b>报 “The device has been lost”</b></summary>

S3 原生 USB 口在硬复位（machine.reset / EN）时会重新枚举 USB，浏览器中的串口句柄失效。工具内部已改用 Ctrl-D 软重启规避；若因拔线等原因仍出现，重新点「连接串口」即可。
</details>

<details>
<summary><b>下载文件报 base64 异常 / 输出里混入“气体:raw=”之类内容</b></summary>

板内 <code>main.py</code> 的定时器 / 线程在后台持续打印，raw REPL 的 Ctrl-C 无法停止已创建的定时器。点「软重启进 raw」拦截自启程序后重试即可。
</details>

<details>
<summary><b>高波特率烧录失败</b></summary>

劣质 USB 线 / 集线器下可能不稳定，波特率降到 230400 或 115200 即可。
</details>

<details>
<summary><b>ADC2 引脚读不到值</b></summary>

WiFi 工作时 ADC2 不可用（ESP32 / S3 均如此），请改用 ADC1 引脚。
</details>

<details>
<summary><b>需要联网吗？</b></summary>

烧录功能首次需从 CDN 加载 esptool-js；串口、文件、引脚功能在页面加载完成后不依赖网络。完全离线使用可将 esptool-js 下载到本地并修改 import 路径。
</details>

## 硬件注意事项

- Strapping / USB 引脚（S3 GPIO0 / 3 / 19 / 20 / 45 / 46）谨慎接线
- ESP32 的 GPIO34–39 仅输入，不能输出 / PWM
- ADC 使用 11dB 衰减，量程约 0 ~ 3.3V
- 需带自动复位电路的开发板才能全自动烧录（主流开发板均具备）

## 技术栈

- [Web Serial API](https://developer.mozilla.org/docs/Web/API/Web_Serial_API)
- [esptool-js](https://github.com/espressif/esptool-js)（ESP 烧录协议）
- MicroPython raw REPL 协议（文件管理与代码执行）
- 零依赖：除 esptool-js CDN 外无任何第三方库，无构建步骤

## 开发 & 测试

协议层与 DOM 解耦，可用 Node.js 对模拟设备跑无硬件测试，覆盖 raw REPL 握手、文件分片上传 / 下载往返、stderr、超时重同步、引脚探测与轮询变化检测等：

```bash
node --version  # 需要 Node 18+
# 测试脚本可由 flasher.html 中的 @@ 标记区块生成（详见仓库脚本）
```

## 许可证

[MIT](LICENSE)

---

<div align="center">

如果这个工具对你有帮助，欢迎 ⭐ Star / Issue / PR

</div>
