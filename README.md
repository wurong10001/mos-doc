# MOS 系统文档

本文档基于 `wurong10001/MOS`、`wurong10001/mos-repo`、`wurong10001/mos-android` 三个代码仓库的现状整理，梳理 MOS 整体系统的目标、职责边界、技术架构与演进方向，供 `wurong10001/mos-doc` 作为统一文档入口使用。

## 1. 项目定位

MOS 目前是一个“云端桌面 / 移动端桌面 / 原生客户端”三端协同的系统雏形，核心理念是：

- 以“桌面操作系统”的交互体验为UI语言；
- 以“云端服务 + 本地运行时”的方式承载应用；
- 通过 Web、Android、以及后续扩展能力构成统一的操作系统式产品形态。

从现有代码看，MOS 不是单一应用，而是一个有分层设计的系统原型：

- Web 端负责展示桌面 UI 与云端入口；
- Android 端负责本地运行时与原生宿主；
- 代码仓库 `mos-repo` 作为“系统软件源”承载核心软件资产；
- 文档仓库 `mos-doc` 则承担对整体系统的说明与演进记录。

## 2. 仓库职责

### 2.1 `wurong10001/MOS`

这是当前最核心的前端/云端入口仓库，技术栈是：

- Cloudflare Workers
- 静态资源托管
- HTML + CSS + JavaScript

仓库的核心入口定义于：

- `src/index.js`
- `public/desktop.html`
- `public/mobile.html`
- `public/assets/system.css`

它的职责是：

1. 接收 HTTP 请求；
2. 根据设备 UA 判断访问的是桌面端还是移动端；
3. 将 `/` 重定向到对应页面；
4. 通过 Cloudflare Worker + 静态资源提供 MOS 桌面系统界面。

其关键逻辑来自 `src/index.js`：

```js
if (url.pathname === '/') {
  const userAgent = request.headers.get('user-agent') || '';
  const isTouchDevice = /android|iphone|ipod|ipad|mobile|windows phone|iemobile|opera mini/i.test(userAgent);
  url.pathname = isTouchDevice ? '/mobile.html' : '/desktop.html';
}

return env.ASSETS.fetch(new Request(url, request));
```

这说明 MOS 的第一层入口设计是“按设备自动切换界面”，反映了一个跨终端平台的系统思路。

### 2.2 `wurong10001/mos-repo`

`mos-repo` 的说明文字为：

> MOS 系统软件源

它作为“软件源”仓库，表示该仓库承载的是：

- 系统应用包；
- 可安装的应用资产；
- 软件模块的源代码或收集仓库；
- 后续用于统一分发的基础软件集合。

当前仓库内容非常轻量，说明其现阶段更偏向系统架构雏形，而非完整软件生态。

### 2.3 `wurong10001/mos-android`

`mos-android` 是原生 Android 端实现，当前主要特点是：

- Android 应用工程；
- NativeActivity  / C++ 本地层；
- 使用 `wasm3` 运行 WebAssembly；
- 通过 `sample.wasm` 作为示例运行时资产；
- 使用 NDK 构建 native library。

它的关键文件包括：

- `app/build.gradle`
- `app/src/main/cpp/CMakeLists.txt`
- `app/src/main/cpp/main.cpp`
- `app/src/main/cpp/engine.cpp`
- `app/src/main/cpp/wasm_runner.cpp`
- `app/src/main/assets/sample.wasm`

从 `CMakeLists.txt` 看，Android 端使用了：

- `wasm3` 作为 WebAssembly 解释器
- `native_app_glue` 作为 NativeActivity 事件循环
- `main.cpp` / `engine.cpp` / `wasm_runner.cpp` 作为核心基础设施

这是一个很典型的“原生宿主 + WebAssembly 执行引擎”的设计：

- UI 宿主在 Android 原生层；
- 逻辑与应用可运行在 Wasm 容器中；
- 未来可进一步实现“跨平台应用同一内核”的目标。

### 2.4 `wurong10001/mos-doc`

`mos-doc` 负责系统文档，其目标是统一解释并维护 MOS 的技术架构、系统设计和演进路线。当前仓库只有 README ���稿，接下来需要作为文档规范中心持续维护。

## 3. 总体架构

整体上，MOS 可以按以下三层结构理解：

### 3.1 体验层：桌面 UI

- `MOS/public/desktop.html`：桌面端视觉展示
- `MOS/public/mobile.html`：移动端触屏桌面展示
- `public/assets/system.css`：统一视觉语言

当前页面体现出明显的“操作系统桌面”风格：

- 顶部状态栏
- Dock / 应用栏
- Welcome screen
- 桌面布局与移动端布局差异化
- 深蓝色系统主体色 + 轻拟物质化风格

这说明 MOS 在早期原型阶段更注重“交互体验”而不是完整业务逻辑。

### 3.2 接入层：Web Service

- Cloudflare Worker：`src/index.js`
- 带 `ASSETS` 的静态资源路由

This service acts as the system gateway, forwarding requests to `/desktop.html` or `/mobile.html` based on the client. It is essentially a lightweight web runtime and system shell.

### 3.3 执行层：Android Runtime

- NativeActivity 开启应用主循环
- C++ 内核负责系统事件、引擎调度与运行时日志
- wasm3 解释器运行 WebAssembly 代码
- `sample.wasm` 充当测试/示例应用资产

该设计具有明显的“未来可扩展性”：

- 前端可以以 Web 形式展示；
- 原生层可以承载更高性能的能力；
- Wasm 可以作为跨平台业务逻辑执行容器。

## 4. 现状功能与设计特征

从代码中可归纳出 MOS 当前的设计特征：

### 4.1 设备自适应

`MOS` 的首页根据 UA 自动选择：

- `desktop.html`：桌面端
- `mobile.html`：触屏端

这体现了“统一桌面系统，不同终端具备特定视觉与交互习惯”的思路。

### 4.2 空间与桌面感

界面中有明显的“OS 桌面”要素：

- 状态栏
- 应用坞
- 主页区域
- 桌面图标位预留
- 任务页指示器

这说明 MOS 当前更偏向系统原型，而不是传统网页应用。

### 4.3 原生与 WebAssembly 的混合架构

`mos-android` 中最关键的特征是：

- Android 端界面和系统环境由原生代码承载；
- 业务逻辑与应用代码通过 Wasm 执行；
- 解释器为 `wasm3`，具备小巧、低依赖、适合嵌入式/移动端场景的特点。

这种架构非常符合“系统化、跨平台和轻内核”的发展愿景。

## 5. 技术栈总结

| 维度 | 当前实现 | 说明 |
| --- | --- | --- |
| Web 前端 | HTML + CSS + JS | MOS 云桌面界面 |
| Web 服务 | Cloudflare Worker | 统一路由入口 |
| Android 宿主 | Android NativeActivity + C++ | 原生系统层 |
| 逻辑执行 | WebAssembly + wasm3 | 可扩展应用执行环境 |
| 资源管理 | 静态资产 + wasm 资源 | UI 和应用数据分离 |
| 文档层 | `mos-doc` | 架构说明与演进记录 |

## 6. 架构推演：MOS 的可能演化方向

结合三套仓库的现状，可以将 MOS 理解为一个“OS-like product prototype”，其后续演进方向大致包括：

### 6.1 Web 端演进

- 提供更完整的桌面应用启动器；
- 增加文件管理器、系统设置、通知中心等基础模块；
- 使页面从静态展示逐步转成真正的应用容器。

### 6.2 Android 端演进

- 增加系统级能力：窗口管理、权限管理、日志监控、崩溃收集；
- 以 Wasm 作为应用执行层，支持跨平台模块复用；
- 继续保留 Native 层的稳定服务能力。

### 6.3 软件源演进

- `mos-repo` 用作“系统应用源”；
- 不同应用发布为独立模块；
- 通过统一软件源来管理应用安装、版本、依赖与更新。

### 6.4 文档系统演进

- 持续记录架构演进与接口约定；
- 定义桌面应用规范、应用包规范、系统事件规范；
- 成为开发者协作和后续模块扩展的基础文档。

## 7. 运行与构建建议

### 7.1 Web 端

在 `MOS` 仓库中，可使用：

```bash
npm install
npm run dev
```

其运行方式依赖 `wrangler`，适合 Cloudflare Workers + 静态站点模型。

### 7.2 Android 端

Android 项目配置显示，使用的是：

- Gradle 工程
- Android 30 编译目标兼容性处理
- CMake 构建 native 代码
- `jniLibs` 预编译库方案

典型构建思路为：

```bash
./gradlew assembleDebug
```

但当前代码中明确说明，Android 原生库采用“预编译后放入 jniLibs”的方式，不直接依赖宿主 NDK 生成能力，适用于特定环境（如 Termux、低配或非标准工具链环境）。

## 8. 当前阶段判断

从现有代码判断，MOS 目前更像是：

- 一个概念验证型操作系统界面原型；
- 一个面向桌面/移动端跨平台体验的系统雏形；
- 一个具备 Web + Android + Wasm 混合架构雏形的研究型项目。

其核心价值不在于“成熟功能完整”，而在于：

- 把桌面系统体验前端化；
- 把系统运行时和原生能力分层表达；
- 为后续更完整的 OS/应用生态铺路。

## 9. 结论

MOS 不是传统意义上的单体站点，而是一套“桌面系统式产品”的原型工程。它将：

- Web 端作为系统入口与桌面展示层；
- Android 端作为原生宿主与运行时层；
- `mos-repo` 作为软件源层；
- `mos-doc` 作为系统设计与演进文档层。

未来如果继续迭代，MOS 很可能逐步发展成：

- 一个轻量云桌面系统；
- 一个跨平台应用容器；
- 一个支持 WebAssembly / 原生混合的移动端操作系统雏形。

该文档会持续更新，随着各仓库代码的完善而逐步细化系统设计和接口规范。

---

如果需要下一步，我可以继续为 `mos-doc` 补充以下内容中的任意一个：

1. “系统架构图（Mermaid）”
2. “模块接口说明”
3. “Android / Web / Wasm 交互流程图”
4. “MOS 功能清单与里程碑计划”
5. “开发者手册（环境搭建 + 目录说明）”

