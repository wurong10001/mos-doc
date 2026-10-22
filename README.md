# MOS 文档

MOS 的官方文档站，同时也是这个仓库的内容来源。纯静态站点，托管在 GitHub Pages。

在线阅读：https://wurong10001.github.io/mos-doc/

## 文档讲什么

MOS 是一套把桌面操作方式带到浏览器和手机上的系统。这份文档面向两类人，分成两条线：

**使用 MOS** — 不需要写代码

| 章节 | 内容 |
| --- | --- |
| 01 开始 | 文档的组织方式和读法 |
| 02 访问与界面 | 怎么打开，为什么手机和电脑上界面不一样 |
| 03 桌面与应用 | 启动、切换、窗口操作，应用从哪来、怎么装 |

**开发 MOS 应用** — 需要一个能编译到 WebAssembly 的语言

| 章节 | 内容 |
| --- | --- |
| 04 预备概念 | WebAssembly、沙箱与导入导出、wasm3、NativeActivity |
| 05 系统构成 | 四层结构，应用与系统之间的接口 |
| 06 写第一个应用 | 环境、模块骨架、编译、调试 |
| 07 打包与发布 | 包结构、清单字段、版本规则、提交到软件源 |

## 仓库结构

```
mos-doc/
├── index.html          01 开始
├── access.html         02 访问与界面
├── using.html          03 桌面与应用
├── concepts.html       04 预备概念
├── architecture.html   05 系统构成
├── appdev.html         06 写第一个应用
├── packaging.html      07 打包与发布
└── assets/style.css    全站样式
```

每个页面自带侧边栏导航和底部翻页。改一个页面只影响它自己，但导航块是各页复制的，
增删章节时要同步改所有页面。

## 本地预览

没有构建步骤，起个静态服务器就行：

```bash
python3 -m http.server 8000
```

然后打开 http://localhost:8000

## 发布

改完推到 `main` 分支，GitHub Pages 自动重新部署，通常一两分钟内生效。

## 相关仓库

| 仓库 | 内容 |
| --- | --- |
| `wurong10001/MOS` | Web 入口、桌面与触屏界面 |
| `wurong10001/mos-android` | 原生宿主、wasm3、C++ 主循环 |
| `wurong10001/mos-repo` | 软件源，放应用包 |
| `wurong10001/mos-doc` | 这个仓库 |

四个仓库之间没有代码依赖，只是职责上分层。

## 改文档

发现写错了、或者哪段话没说清楚，直接提 Pull Request。
改文案不需要动 CSS，正文用的是 `p` / `h2` / `h3`，列表用 `ul`，
需要强调的地方有 `.callout` 提示框和 `.term-list` 术语卡片可以用。
