# 工作目录导览与维护备注

整理日期：2026-09-13。项目为「砚 · Yan」，使用 Electron + React + TypeScript，pi 提供 RPC 内核。

本导览依据目录、源码入口、package.json 和构建配置整理。测试结果应以对应修改的实际检查输出为准，文件存在不代表已通过验收。

## 根目录分类

| 路径 | 用途 | 维护标注 |
|---|---|---|
| `src/` | 应用源码：主进程、预加载、界面及共享协议 | **核心源码**，修改功能从这里开始 |
| `scripts/` | 启动、构建辅助、测试和诊断 | **维护工具**，详见下表 |
| `docs/` | 设计规范、原型、截图与开发交接 | **参考资料**，入口为 [文档索引](README.md) |
| `build/icon.ico`、`build/icon.png` | Windows 打包图标 | **保留资源**；虽位于 build，仍受版本管理，不应整目录清理 |
| `resources/pi-runtime/` | 随应用分发的 pi 运行时 | **生成物**，Git 忽略；通过 `npm run vendor:pi` 生成，不手改 |
| `resources/pi-extensions/` | 随应用分发的内置 pi 扩展（浏览器工具） | **源码资源**，随打包分发；主进程用 `--extension` 加载 |
| `out/` | Electron/Vite 编译输出及测试编译文件 | **生成物**，Git 忽略；应用和部分测试依赖它，清理后需构建 |
| `release/` | 安装包、便携版、解包目录及打包元数据 | **分发产物**，Git 忽略；保留需要交付的版本后再考虑清理 |
| `node_modules/` | npm 安装的依赖 | **依赖产物**，Git 忽略；可按锁文件重新安装 |
| `.backup/` | 本地手动快照 | **备份**，Git 忽略；不能当作普通缓存删除 |
| `.git/` | 提交历史、分支与仓库元数据 | **仓库数据**，保留 |
| `启动-砚.cmd`、`开发-砚.cmd` | Windows 双击启动入口 | **用户入口**，保留根目录位置 |
| `package.json`、`package-lock.json` | 项目信息、脚本、依赖及版本锁定 | **构建配置**，配套维护 |
| `electron.vite.config.ts` | 主进程、预加载和界面的构建配置 | **构建配置** |
| `electron-builder.yml` | 安装包、便携版、图标和内置运行时复制规则 | **发布配置** |
| `tsconfig.json`、`tsconfig.node.json`、`tsconfig.web.json` | TypeScript 项目与类型检查配置 | **源码配置** |
| `*.tsbuildinfo` | TypeScript 增量编译缓存 | **缓存**，Git 忽略，可重新生成 |
| `.gitignore`、`.gitattributes` | 忽略规则与文本属性 | **仓库配置** |
| `README.md`、`LICENSE` | 产品介绍、使用说明与许可证 | **项目文档** |

## 源码导航

以下路径相对于 `src/`。

| 路径 | 职责 / 查找提示 |
|---|---|
| `main/index.ts` | Electron 主进程入口 |
| `main/agent.ts`、`protocol.ts`、`normalize.ts` | pi 子进程与 RPC 协议、消息归一化 |
| `main/sessions.ts`、`session-reader.ts` | 会话管理与会话文件读取 |
| `main/settings.ts`、`paths.ts` | 设置与数据路径 |
| `main/credentials.ts` | 模型凭证接入及相关辅助逻辑 |
| `main/files.ts` | 文件相关主进程功能 |
| `main/title.ts`、`compaction.ts` | 会话标题与上下文压缩相关逻辑 |
| `main/zoom.ts`、`zoom-math.ts` | 界面缩放及纯计算 |
| `main/browser.ts` | 内置浏览器控制器：loopback bridge、标签页、观察/操作、下载、用户接管 |
| `main/browser/` | 浏览器底层：CDP 桥、观察器、元素注册表、输入控制、风险策略、坐标换算 |
| `preload/index.ts` | 主进程与渲染界面的桥接入口 |
| `shared/ipc.ts`、`shared/turns.ts` | 共享 IPC 定义、对话回合分组 |
| `renderer/index.html`、`renderer/src/main.tsx`、`renderer/src/App.tsx` | HTML、React 挂载和应用组件入口 |
| `renderer/src/state/store.ts` | 界面状态与动作 |
| `renderer/src/components/chat/` | 对话、输入、推理、工具消息与大纲 |
| `renderer/src/components/rail/` | 左侧导航和会话列表 |
| `renderer/src/components/toolbar/` | 右侧面板、文件树、工具区与分隔拖拽 |
| `renderer/src/components/browser/` | 内置浏览器右栏 UI：地址栏、标签页与 viewport 坐标同步 |
| `renderer/src/components/settings/` | 设置、模型接入与首次引导 |
| `renderer/src/components/shell/`、`components/Pickers.tsx` | 应用外壳、标题栏及选择控件 |
| `renderer/src/i18n/` | 国际化入口和中英文文案 |
| `renderer/src/styles/` | 设计令牌、界面样式、主题与动效 |
| `renderer/src/icons/`、`renderer/src/lib/` | 图标组件及通用辅助逻辑 |

## 内置浏览器结构

该功能横跨主进程、原生视图、pi 扩展和 renderer，找东西时按这条链走：

| 层 | 位置 | 说明 |
|---|---|---|
| 主进程控制器 | `src/main/browser.ts` | 持有视图/标签页，开一个带 token 的 127.0.0.1 loopback bridge |
| 底层算法 | `src/main/browser/` | `CDPBridge`（Electron 调试器实现）/ `RawCdp`（外部 Chrome 的 WebSocket 实现，两者实现同一个 `CdpChannel`）/ `Observer`（可交互元素）/ `ElementRegistry`（generation-scoped ref）/ `InputController`（点击输入）/ `BrowserPolicy`（高风险拦截）/ `geometry.ts`（视口包围盒） |
| 外部 Chrome | `src/main/chrome.ts` | 探测/启动/停止本机 Chrome（独立 `--user-data-dir` + 调试端口）；已接入 BrowserController（`mode: 'external'`），工具栏可接入/断开，pi 有 connect/disconnect 工具 |
| UI | `src/renderer/src/components/browser/BrowserSurface.tsx` | 只画工具栏并把可见区域坐标同步给主进程；网页本身是原生视图 |
| pi 工具 | `resources/pi-extensions/browser.js` | 通过 `--extension` 临时加载，只访问 bridge，不碰 Electron 对象 |
| 验收 | `scripts/probe/browser.js` | 真实应用里的浏览器场景探针 |
| 记录 | `docs/archive/2026-09-13-browser-*.md` | 实现与后续修复的封存记录 |

## 脚本导航

以下路径相对于 `scripts/`；命令在项目根目录运行。

| 文件 / 目录 | 用途 | 常用入口 |
|---|---|---|
| `launch.mjs` | 检查依赖及构建后启动应用 | `npm run launch`；开发用 `npm run launch:dev` |
| `vendor-pi.mjs` | 从本地已安装的 pi 提取分发运行时 | `npm run vendor:pi`；校验用 `npm run vendor:pi:check`（`--if-present` 允许缺件跳过） |
| `upgrade-pi.mjs` | 对比内置/源版本并按需重提取、自检 | `npm run upgrade:pi`（`--check` 只报版本，`--force` 强制） |
| `build-icon.mjs` | 生成打包图标 | `npm run icon` |
| `lint-css.mjs` | CSS 约定检查 | `npm run lint:css` |
| `test-unit.mjs` | 单元测试入口，使用构建产物 | 先 `npm run build`，再 `npm run test:unit` |
| `test-turns.mjs`、`test-zoom.mjs` | 回合与缩放测试模块 | 由单元测试入口组织 |
| `test-live.mjs`、`probe/` | 真实应用中的场景测试及探针 | `npm run test:live -- 场景名` |
| `probe/browser.js`、`probe/browser-shot.js` | 内置浏览器场景探针 / 位置诊断（`browser-shot` 未注册为场景，手动用 `YAN_PROBE` 跑） | `npm run test:live -- browser` |
| `probe/chrome-cdp.mjs` | 外部 Chrome 通道冒烟（无头 Chrome + 临时 profile + 原生 CDP） | `npm run probe:chrome` |
| `probe/toolgroup.js` | 工具调用栏展开规则（注入合成回合，验「只展开运行中的那条」） | `npm run test:live -- toolgroup` |
| `probe/external-chrome.js` | 应用内接入本机 Chrome 的端到端（无头 + 隔离 profile） | `npm run test:live -- externalchrome` |
| `test-packaged.mjs` | 验收已有解包产物 | `npm run test:packaged` |
| `probe-pi.mjs` | pi 查找及启动诊断 | `npm run probe-pi` |
| `shot.mjs` | Electron 截图辅助 | `npm run shot` |

完整检查入口为 `npm run check`，实际执行列表以 [package.json](../package.json) 为准。真实模型场景 `e2e`、`image`、`queue` 可能消耗模型额度，按需运行。

## 设计与交接资料

| 路径 | 标注 |
|---|---|
| [design/DESIGN.md](design/DESIGN.md) | 设计令牌的规范来源；修改令牌时与 `src/renderer/src/styles/tokens.css` 同步 |
| [design/prototype.html](design/prototype.html) | 交互设计稿 |
| `design/icons/` | 图标素材、索引与预览 |
| `design/preview/` | 界面截图和阶段对比；部分由 README 引用，移动前需检查链接 |
| [design/font-test/README.md](design/font-test/README.md) | 字体实验资料；本地字体文件可能被 Git 忽略 |
| `design/archive/` | 已归档原型和一次性修复脚本，日常开发无需执行 |
| [dev/HANDOFF.md](dev/HANDOFF.md) | 当前决定、有效待办与维护经验；旧流水账从 Git 历史查看 |
| [dev/NEXT-SESSION.md](dev/NEXT-SESSION.md) | 精简的新会话入口，引用交接文档，避免重复维护状态 |
| [archive/README.md](archive/README.md) | 已完结阶段的工作记录索引；用于追溯某项改动的来龙去脉 |

## 本次盘点备注

- 首次目录盘点时 Git 工作区干净；随后按用户要求清理无用代码和过期记录。任务面板保持现状。
- 2026-09-13 追加：内置浏览器功能接入（主进程 + 原生视图 + pi 扩展 + renderer），
  并在同一天修复定位/引用生命周期问题，记录见 [`archive/`](archive/README.md)。
  浏览器相关新增目录：`src/main/browser/`、`src/renderer/src/components/browser/`、`resources/pi-extensions/`。
- `.backup/` 当前含 `20260911-020928/`、`ui-before-redesign/` 两份快照；未对比内容或判断哪份可删除。
- `release/` 当前有 `win-unpacked/`、`砚-0.1.0-setup.exe`、`砚-0.1.0-portable.exe` 及配套元数据；文件存在不等于与当前源码一致，也不代表已通过本次验收。
- `resources/pi-runtime/` 本地已存在，但不入库。新克隆目录若要内置运行时或打包，需要先准备本地 pi，再运行提取命令。
- 样式目录中的 `stage1.css`、`stage2.css` 等名称不能作为过期依据，删除或合并前需核对导入关系与覆盖顺序。
- 现有分层已经清楚，本次保留文件位置，通过分类说明完成整理，避免移动文件后破坏脚本路径和文档引用。
