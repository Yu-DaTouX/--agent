# 开发交接 · 砚

更新：2026-09-13（pi 管理/升级收尾）。本文维护当前约定；旧会话流水账和已失效的方案已清理，历史可从 Git 查看。目录见 [工作目录导览](../WORKSPACE.md)。

## 当前产品与已确认边界

- 砚是独立 agent 桌面端，Electron + React + TypeScript；pi 作为 RPC 子进程提供模型循环与工具执行。
- **任务面板保持现状**（用户本次明确确认）：继续读取会话中的扩展任务清单，不安排重构，不修改用户扩展来改变任务引导。
- 记忆存储、remember/recall/forget、记忆扩展和提示词注入已移除，不恢复这些模块，也不删除用户遗留数据。
- 推理内容沿用模型输出，不注入思考语言要求；推理窗口在整个回合结束后折叠。
- 会话分支按当前左栏与对话消息入口实现。旧的 get_tree 浏览链路曾主动移除，不把它重新列作缺失功能。
- 深浅主题、设置面板、模型接入 UI、字体 npm 分片依赖及 Windows 打包已经实现，不再作为未开发事项。
- 登录目前只预留，本地档案不能显示虚假的已登录或同步状态。

## 尚未完成

| 顺序 | 工作 | 当前边界 |
|---|---|---|
| 后续 | 发布与账号能力 | 代码签名、macOS/Linux 打包、登录；不属于本次清理范围 |

### 已收尾（内置 pi 管理，2026-09-13 本会话）

- `PiInfo` / `PiProbe` 增加 `source`（内置/系统安装/自定义/环境变量/PATH/shell）、`home`、
  `bundledAvailable`、`error`。设置「关于」页显示来源、版本、包目录、入口路径，并提供**重新检测**；
  pi 之前没找到时，重新检测会顺手把 agent 拉起来。缺件时给出「内置缺失 → `npm run vendor:pi`；
  安装版重装」的修复说明。
- `vendor:pi:check` 支持 `--if-present`，已接入 `npm run check`（新克隆缺内置运行时跳过而非失败）。
- 新增 `npm run upgrade:pi`（版本对比 → 必要时提取 → 自检失败给修复清单）；`--check` 只报版本，
  `--force` 强制重跑。升级后需**重启应用**（版本在进程内缓存）。
- **Ctrl+Shift+P（上一个模型）**：pi 的 RPC 只有向前的 `cycle_model`，故在 `AgentController.cycleModelBack`
  里用 `get_available_models` + `set_model` 反向走一项；主进程 `before-input-event` 先拦 Ctrl+Shift+P
  （必须排在 Ctrl+P 前），统一发 `cycleModelBack` 动作。对齐 pi 默认 `app.model.cycleBackward`。
  探针 `hotkeys` 已覆盖：真实按键得到 `cycleModelBack → cycleModel → cycleThinking`，双向切换可逆。

### 外部浏览器（本机 Chrome）—— 已接入

- 方案：**独立 `--user-data-dir` + 调试端口 + 原生 CDP**。Chrome 136 起默认配置目录会被拒绝开
  `--remote-debugging-port`；独立 profile 需要用户在其中**登录一次**目标站点（如 ChatGPT）。
- 分层：
  - `src/main/browser/CdpChannel.ts`：只定义 `attach/send/screenshot/detach`。Observer / InputController /
    geometry 改吃这个接口 —— 同一套「观察→ref→点击/输入」逻辑不再绑死 Electron。
  - `CDPBridge.ts`（Electron `webContents.debugger`）与 `RawCdp.ts`（WebSocket；Node 22 全局 `WebSocket`，
    不引 puppeteer）是同一接口的两个实现。启动参数 `--remote-allow-origins=*` 不可漏，否则 DevTools 连接被拒。
  - `src/main/chrome.ts`：探测 Chrome/Chromium/Edge、选空闲端口、拼参数、启动/停止。
  - `src/main/browser.ts`：内嵌标签页与外部 Chrome **互斥**；`parts()` 统一路由 observe/click/type/press/
    scroll/screenshot，navigate/back/forward/reload 对两种模式分叉。状态新增 `mode` 与 `external`。
- 入口：工具栏「Chrome」按钮（`[data-testid="browser-external-chrome"]`）；pi 工具
  `browser_connect_local_chrome` / `browser_disconnect_local_chrome`（走 loopback `/external/open|close`）。
- 登录交接：复用已有的 `browser_request_user_control` —— 用户在 Chrome 窗口里登录后恢复 Agent 控制。
- 验证：`npm run probe:chrome`（无头 Chrome 通道 7 项）、`npm run test:live -- externalchrome`
  （面板按钮 → 接入 → observe → 断开，全程无头、隔离 profile）。
- **已知边界**：外部模式下 `switchTab`（列/切已有标签）未做，`newTab` 可用；外部 Chrome 的下载不进
  `lastDownload`（那是内嵌 session 的事件）；暂为单页面目标。主进程的 back/forward 对外部已实现
  （`Page.getNavigationHistory`），但状态里 `canGoBack/canGoForward` 对外部保守报 false，所以**工具栏
  前进/后退按钮是灰的**（Agent 工具仍可调）；用 Chrome 自己的按钮或补上历史状态即可。

### 其他修复：工具调用栏展开规则（2026-09-13）

- 用户报：模型一调工具，**整个**工具调用栏（`.tgroup`）就展开。
- 根因：`ToolGroup` 曾用 `open = running && streaming`，一条在跑就弹开整组。
- 修法：正在跑 / 排队的工具由 `TurnView` 单独渲染并自动展开详情；已结束的才进
  `ToolGroup`，默认收起，用户点了才开（手动展开不被自动规则推翻）。
- 回归：`npm run test:live -- toolgroup`（注入合成回合，不烧 token）。

### 已知上游限制：扩展快捷键（registerShortcut）

pi 0.85.1 的 `registerShortcut` **只在交互式 TUI 里生效**，RPC 模式拿不到也触发不了：

- `dist/modes/rpc/rpc-mode.js` 的命令 switch 里没有任何 shortcut 相关命令（只有 `get_commands`）；
- `extensionRunner.getShortcuts()` / `getShortcutDiagnostics()` 只在 `dist/modes/interactive/interactive-mode.js` 被调用；
- 官方 `docs/rpc.md` 的命令清单里没有列举/触发快捷键的接口。

因此桌面端无法通用地枚举或调用第三方扩展注册的快捷键。可行路径：

1. **上游扩展 RPC**（推荐）：pi 增加 `get_shortcuts` + 按命令名调用的接口，桌面端本地映射按键、
   调用 `executeCommand`。契约干净、可重映射；需改 pi。
2. **通用 `dispatchKeybinding` RPC**：桌面端把按键发给 pi 自己解析。改动小，但更绑定 TUI 语义。
3. **桌面端静态映射表**（现状兜底）：只能覆盖已知命令，无法应对动态/第三方扩展。

注意：扩展用 `registerCommand` 注册的**命令**已经能通过 `get_commands` 看到，并可用 `/命令` 触发 ——
受限的只是 `registerShortcut` 注册的快捷键处理器，不要把两者混为一谈。
上游支持前，不再将其列为可实现待办。

## 内置浏览器当前状态

- 浏览器运行时在 `src/main/browser.ts`，页面使用共享 `persist:yan-browser` Session 的 `WebContentsView`。
- `src/main/browser/` 分别负责 CDP、观察、generation-scoped element ref、输入控制和风险策略。
- Pi 扩展工具为 `browser_open`、`browser_observe`、`browser_click`、`browser_type`、`browser_press`、`browser_scroll`、标签页、截图、下载和用户接管；旧 `browser_navigate` 保留为兼容别名。
- 任意页面 JavaScript 默认关闭；密码、验证码、Passkey、支付及高风险操作应由 `browser_request_user_control` 交给用户处理。
- 元素 ref 只在**整篇文档被替换**（`Page.frameNavigated` / `DOM.documentUpdated`）时作废，不再因任意节点增删清空注册表；节点被移除时由 `DOM` 的 detached 报错翻译成 `STALE_ELEMENT`。
- `browser_click` 会先 `scrollIntoViewIfNeeded` 再**重新测量**包围盒（`src/main/browser/geometry.ts`）；复用观察时的坐标会让离屏元素点到空处。
- 网页是原生 `WebContentsView`，永远盖在渲染层之上：加载/错误提示只能放在工具栏，且 `.browser-surface` 单列必须固定为 `minmax(0, 1fr)`，否则长文案会把区域撑变形。
- **原生层坐标换算**：渲染端 `getBoundingClientRect()` 是主窗口的 CSS 像素，而 `WebContentsView.setBounds` 要 DIP，主进程按 `win.webContents.getZoomFactor()` 相乘。界面缩放 ≠ 100% 时漏掉这步，页面会整体偏左上、且比面板窄一圈（缩放越大越明显）。
- 启动时的自动缩放要在 `ready-to-show` 之后再补一次（`loadURL/loadFile` 会把加载前的 zoom 重置掉）；**不能**在 `did-finish-load` 里同步调 `setZoomFactor`，实测会让渲染进程 `render-process-gone: crashed`。
- 主进程未捕获异常 / 未处理 Promise 通过 `ch: 'log'` 进右栏日志抽屉，不再弹 Electron 的“A JavaScript error occurred in the main process”。
- 工具栏的「↗ 在外部浏览器打开」走 `shell.openExternal`（只放行 http/https），内嵌视图无登录态时交给用户自己的浏览器。
- 右栏 5px 宽度把手会被原生视图遮住，`.rightpanel.browser-mode .browser-viewport` 让开 5px，否则浏览器打开时从页面区域拖不了宽。
- 内嵌浏览器与用户本机 Chrome 完全隔离（独立 `persist:yan-browser`）；登录态不共享。
- 回归在 `scripts/probe/browser.js`（含错误提示可见性与面板宽度不变形两项断言）。
- 实现与后续修复的来龙去脉见归档：[实现](../archive/2026-09-13-browser-implementation.md) · [修复](../archive/2026-09-13-browser-fixes.md)。

## 源码与协议边界

- 主进程入口：src/main/index.ts；pi 协议相关逻辑集中于 protocol.ts、agent.ts、normalize.ts。
- 渲染端通过 preload 和 shared/ipc.ts 定义的接口调用主进程、消费 MainPush；不要直接 import pi 内部模块。
- pi 会话沿用 JSONL 格式；快速加载由 session-reader.ts 读取，随后后台切换 pi 会话。
- 分支 entryId 来自 get_fork_messages，不从 DOM 或归一化消息 id 猜测。
- 任务面板读取扩展写入的 custom entry；应用不向模型注入任务指令。
- 主进程快捷键的热路径避免异步重读设置；缩放使用内存中的当前状态。

## 运行与验证

命令以根目录 package.json 为准，不在交接文档固定测试数量或重复宣称历史结果是当前通过状态。

| 命令 | 用途 |
|---|---|
| npm run launch / npm run launch:dev | 普通启动 / 开发启动 |
| npm run typecheck | 主进程与界面类型检查、CSS 约定检查 |
| npm run build | 构建 out/ |
| npm run test:unit | 单元测试，先构建 |
| npm run check | 类型、构建、单测、设计测量和配置中的真实应用场景 |
| npm run test:live -- 场景名 | 按需验收真实应用 |
| npm run vendor:pi / npm run vendor:pi:check | 提取 / 校验内置运行时（check 支持 `--if-present`） |
| npm run upgrade:pi | 对比版本并在需要时重提取 + 自检（`--check` 只报版本，`--force` 强制） |
| npm run probe:chrome | 外部 Chrome 通道冒烟（无头启动 + CDP 观察/点击；无 Chrome 时跳过） |
| npm run test:live -- externalchrome | 应用内接入本机 Chrome 的端到端（无头 + 隔离 profile） |
| npm run dist / npm run dist:check | Windows 打包 / 解包产物验收 |

真实模型场景 e2e、image、queue 可能消耗额度，按需执行。旧的 memory 测试场景已删除。

## 维护中应保留的经验

- 测试用 YAN_USER_DATA、YAN_SESSIONS_DIR、YAN_DATA_DIR、YAN_PI_DIR 隔离用户数据和凭证；不要在真实目录造测试数据。pi 的会话目录还需通过启动参数正确传入。
- 测试按 fixture 路径标识定位会话，不依赖会被模型重写的标题。面板状态、缩放、引导层需明确初始化；用条件轮询代替固定等待。
- 布局测量等待几何稳定；滚动定位避免 smooth 被重渲染取消。跨组件拖拽监听 window，结束后清理监听。
- 设计令牌先改 [DESIGN.md](../design/DESIGN.md)，再同步 tokens.css；grid 的弹性列用 minmax(0,1fr)，避免长内容撑破布局。
- styles/ 中的 stage1、stage2、redesign 等仍按顺序导入；名字旧不等于样式无用，清理需核对覆盖和动态类名。
- pi 内置运行时搬运其自带 bundle 与必要依赖，不自行改造成单文件：外部包、WASM、worker、运行时读取的素材及动态加载的 jiti 都可能需要独立文件。
- resources/pi-runtime/ 是生成物，不手改；pi 升版后跑 `npm run upgrade:pi`（等价于重跑 vendor:pi 并自检）。版本信息在进程内缓存，更换运行时后重启应用。自检失败时脚本会打印修复清单；内置运行时不可用时不得打包。
- electron-builder 的 extraResources 必须把 pi-runtime/node_modules 单独列出，避免打包器静默遗漏依赖。
- Windows 打包仅附带当前 electron-builder.yml 指定的运行时资源；已不包含 resources/pi/yan-memory.ts。
- build/icon.ico 与 build/icon.png 是保留资源；.backup 是手工备份，不当缓存清理。

## 文档维护规则

只记录当前决定、可操作待办和可复用经验。历史性能数字、套餐价格、临时工具路径、旧用户数据数量及被后续实现推翻的决策不作为当前事实保留。
