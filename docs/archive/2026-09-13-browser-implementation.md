# 内置浏览器阶段工作封存记录

日期：2026-09-13
项目：砚（pi-desktop）
状态：功能已接入；当时未闭环的视觉定位问题已由 [2026-09-13-browser-fixes.md](2026-09-13-browser-fixes.md) 修复

> 本记录保留当时的实现事实，**不重写结论**。修正的是后来核对的用词错误（`BrowserView` → `WebContentsView`、
> 以及被中途改名的工具名），并给「已知未闭环问题」补上后续说明。

## 用户目标

新增类似 Codex 的内置浏览器，优先采用 pi 插件实现；浏览器入口固定在右侧工具栏标题旁，以滑动开关控制，不改变原有工具栏位置。

## 已完成工作

1. 接入主进程内置浏览器控制器，使用 Electron `WebContentsView` 加载 http/https 页面。
2. 增加本机 loopback bridge 和随机 Token，供 pi 扩展调用，不修改用户 pi 配置。
3. 增加浏览器导航、后退、前进、刷新、截图、页面快照、点击、输入和 JavaScript 执行能力。
4. 增加内置 pi extension，并注册 `browser_*` 工具（导航 / 观察 / 点击 / 输入 / 按键 / 滚动 / 标签页 / 截图 / 下载 / 用户接管）；后续工具名以 [HANDOFF.md](../dev/HANDOFF.md) 与 `resources/pi-extensions/browser.js` 为准。
5. 在右侧工具栏标题旁增加“浏览器”滑动开关；开启后浏览器显示在右栏，中间聊天区域保持原布局。
6. 增加 `WebContentsView` 与 renderer 可见区域的坐标同步，并尝试隔离网页缩放，固定页面缩放为 100%。
7. 针对旧版本遗留视图追加清理：每次打开浏览器前移除窗口内已有视图，再挂载当前视图。
8. 更新构建打包配置、中文/英文文案、测试探针和使用说明。

## 本阶段涉及的浏览器功能文件

### 新增

- `src/main/browser.ts`：主进程浏览器控制器、bridge、`WebContentsView` 生命周期和页面操作。
- `src/renderer/src/components/browser/BrowserSurface.tsx`：浏览器工具栏、地址栏和 viewport 坐标同步。
- `resources/pi-extensions/browser.js`：pi 浏览器扩展及浏览器工具注册。
- `scripts/probe/browser.js`：真实 Electron 浏览器场景探针。

### 修改

- `src/shared/ipc.ts`：增加浏览器状态、IPC 方法和推送事件类型。
- `src/main/agent.ts`：向 pi agent 注入浏览器能力。
- `src/main/protocol.ts`：给 pi 子进程注入附加环境变量（浏览器 bridge 地址与 token）。
- `src/main/index.ts`：初始化和销毁浏览器控制器。
- `src/preload/index.ts`：向 renderer 暴露浏览器 API。
- `src/renderer/src/App.tsx`：保持中栏为聊天区域，移除中栏浏览器模式。
- `src/renderer/src/components/toolbar/RightPanel.tsx`：右侧工具栏标题旁滑动开关和浏览器面板。
- `src/renderer/src/components/shell/TitleBar.tsx`：调整入口归属，避免浏览器移动到标题栏其他位置。
- `src/renderer/src/styles/redesign.css`：右栏浏览器布局、开关和 BrowserSurface 样式。
- `src/renderer/src/i18n/zh-CN.json`、`src/renderer/src/i18n/en-US.json`：浏览器文案。
- `electron-builder.yml`：打包内置 pi extension 资源。
- `scripts/test-live.mjs`：浏览器 live 测试场景。
- `README.md`、`docs/README.md`：使用说明和功能记录。

## 已写入/生成的文件

- 构建产物写入 `out/`（由 `npm run build` 生成，通常不纳入源码提交）。
- 本封存记录：`docs/archive/2026-09-13-browser-implementation.md`。

## 验证结果

- `npm run typecheck`：通过。
- `npm run build`：通过。
- `npm run test:live -- browser`：通过。
- `启动-砚.cmd`：已实际运行，确认构建产物为最新并成功启动。

自动化探针确认了右栏开关、中栏未切换为浏览器模式和浏览器状态闭环；但当时没有取得用户实际窗口的整屏截图，因此不能把自动化通过等同于视觉截图验证。

## 已知未闭环问题（后续已处理）

用户实际截图曾显示网页内容从中栏位置开始、右栏出现白色空区。
**后续结论见 [2026-09-13-browser-fixes.md](2026-09-13-browser-fixes.md)**：根因是
`getBoundingClientRect()` 的 CSS 像素坐标被直接当成 DIP 传给了 `WebContentsView.setBounds()`，
界面缩放 ≠ 100% 时整体偏移；已在主进程按 `getZoomFactor()` 换算并回归通过。

当时的排查建议（保留作方法记录）：

1. 取得真实 Electron 窗口整屏截图或等价的原生窗口坐标证据。
2. 对比 renderer viewport 的 `getBoundingClientRect()`、主进程视图 bounds、窗口 content bounds 和系统缩放比例。
3. 确认用户启动的是完全退出旧进程后由 `启动-砚.cmd` 启动的最新实例。

## 工作区说明

封存时 `git status` 还显示若干其他修改和未跟踪文件；这些可能来自本任务前已有的工作区变更，未在本记录中强行归因，也没有执行 reset、checkout 或清理操作。
