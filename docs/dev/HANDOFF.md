# 交接文档 · pi desktop

> **给新会话的第一个指令**（直接复制粘贴的版本在 [`NEXT-SESSION.md`](NEXT-SESSION.md)）：
> `读 %USERPROFILE%\Desktop\pi-desktop\docs\dev\HANDOFF.md，然后继续。`
>
> 最后更新：2026-09-12（**第 14 版：宽度拖拽自动收起 / 工具库拖拽 / 分区高度 / 自动压缩提示 / 隐藏项开关 / 命令自动管理**）
>
> ⚠️ **第 4 版开头的「方向变更」先读** —— 代码按「砚 · 个人 agent」实现（不是编码工具）。

---

## 0. 一句话目标

做一个 **个人 agent 的桌面端**（产品名暂定「**砚**」）：Electron 外壳 + React 渲染层，
中文界面 + 完整 i18n 通道，**类终端调性**（全等宽、无投影、颜色即语义），可开源分发。

**记忆是第一公民**：界面刻意区分「已确认的记忆」（来源：你）与「我的印象」（来源：我，未确认），
「身份」区块只读（agent 不能改自己）。这是本产品区别于普通 agent 客户端的地方。

**当前阶段：阶段 0 已完成 —— UI 骨架已跑通（2026-09-11）。**

| 阶段 | 状态 |
|---|---|
| 设计稿 | ✅ v0.2（已改名「砚」，见下方 ⚠️ 方向变更） |
| 技术栈 | ✅ **Electron + Vite + React 19 + TypeScript**（已拍板） |
| 阶段 0：UI 骨架 | ✅ 完成 |
| 阶段 1：接 RPC | ✅ 完成 —— 应用真的连上了 pi，可用 |
| **阶段 2：功能补完** | ✅ **完成** —— 图片 / 斜杠命令 / !bash / fork·clone·导出·删除 / 模型选择器 / 开关 / 虚拟化 |
| **阶段 2.5：桌面化** | ✅ **完成**（2026-09-12）—— **内置 pi** / 工具栏（文件树·日志）/ 用户档案 / 界面缩放（DPI 取整）/ 启动器 / 面板开关对称 |
| **阶段 2.6：可调与可整理** | ✅ **完成**（2026-09-12）—— 左右栏**宽度拖拽** / 分区**拖拽排序** / **工具库**（收进库·取回）/ 引导检查本地已有 key（含环境变量） |
| **阶段 2.7：细节打磨** | ✅ **完成**（2026-09-12）—— 拖过窄**自动收起** / 收起态改**细缝** / 工具库**拖到工具栏**（带实时预览）/ 分区**高度可调** / **自动压缩触发点**可视 / 隐藏项开关 / 命令**自动管理+填充** |
| 阶段 3：打包分发 | ⏭ 下一步（唯一阻塞「给别人用」的）—— 内置运行时已就位，只差 electron-builder |

---

## ⚠️⚠️ 方向变更：从「编码工具」到「个人 agent」（**需用户确认**）

`docs/design/prototype.html` 在本次会话前已被改写为「**砚**」，
`docs/design/preview/` 里的 `yan-1440x900.png` / `yan-1920x960.png` 也是新版。
它与本文件前半部分（第 1–3 版）描述的**编码客户端**不是同一条路线：

| | 编码客户端（HANDOFF 前 3 版） | **砚（当前 prototype.html + 已实现代码）** |
|---|---|---|
| 右栏 | 状态面板（上下文/工作区/文件树/任务/计费） | **记忆**（身份 / 关于你 / 我的印象 / 人 / 项目 / 状态） |
| 中栏 | 三栏 = 会话列表 / 消息流 / 状态 | 三栏 = 话题+会话 / 对话+连续性 / 记忆 |
| 核心创新 | 工具卡智能展开、改动回滚 | **记忆的认识论可见化**（已确认 vs 我的印象） |
| 无 | diff、文件树、测试状态、工作区 | 这些概念**全部去掉** |

**已实现的应用代码按「砚」做。**
本文件里关于 diff / 文件树 / 工具卡 / 改动回滚的章节（§1.2、§1.3、§3.4、§9 v1 范围）
**当前不适用**，保留作历史参考 —— 需要用户确认是**彻底转向**还是**两条线并行**。

> 已明确不受影响的部分：设计令牌（§1.1 字体 / §2 字号）、图标、i18n 机制、
> 三栏布局骨架、RPC 事实（§11）、上下文窗口修复（§5.5）、成本约定（§6）。

---

## 1. 已确认的需求（用户已拍板，不要再问）

| # | 需求 | 状态 |
|---|---|---|
| 1 | 界面中文，但**保留完整多语言接口** | ✅ 已定 |
| 2 | 视觉：类 Codex 的现代化圆角，但 **UI 贴近终端风格** | ✅ 设计稿 v0.1 |
| 3 | **可能会分发**这款软件 | ✅ 已定 |
| 4 | 要**符合 GitHub 规范**（README / LICENSE / CI / Issue 模板等） | ✅ 已定 |
| 5 | 技术栈**暂缓决定** | ✅ 已定 **Electron**（见下 §2 #1） |
| 6 | **字体：Maple Mono CN**，基准 **12.5px**（代码块 12px） | ✅ 已定并实测，见 §1.1 |
| 7 | **图标：reicon**（MIT） | ✅ 已抓取 92 个 |
| 8 | **改动审阅：只做回滚**，不做接受/拒绝 | ✅ 已定，见 §1.2 |
| 9 | 左栏做**可折叠的任务**列表 | ✅ 已定 |
| 10 | 右栏**中下部作为文件树** | ✅ 已定 |
| 11 | 工具卡片**智能展开** | ✅ 已定，见 §1.3 |
| 12 | 只做**深色**，但保留浅色方案 | ✅ 已定 |
| 13 | 状态面板卡片**可调顺序**（拖拽） | ✅ 已定 |

### 1.1 字体（已实测）

**Maple Mono CN**，拉丁列宽 `0.600em`，汉字 `1.200em`，严格 **1:2**。

必须内嵌，**不能靠系统回退** —— 常见英文字体的 ASCII 宽不是整数倍步进
（Cascadia 1.707 / Consolas 1.819 / 雅黑 1.023），凑不出 2.000。

**基准字号 12.5px**（v0.1 曾写 12px，v0.2 上调）。决定性指标不是观感，
而是**汉字格宽是否为整数像素**：

| 字号 | 拉丁列宽 | 汉字格宽 | 整数像素 | 栅格最大偏差 |
|---|---|---|---|---|
| 12px | 7.200px | 14.400px | ✗ | 0.234px |
| **12.5px** | **7.500px** | **15.000px** | **✓** | **0.000px** |

只有 12.5px 落回整数像素栅格（DPR 1 与 DPR 2 都成立），
汉字笔画不发虚、字与字之间无细微分差。代价是 780px 每行少 4 列（108 → 104）。
代码块 / diff 仍用 12px。

字号阶：`10 / 11 / 12 / 12.5 / 14`，行高 `1.45 / 1.65`。

详见 [`docs/design/DESIGN.md`](docs/design/DESIGN.md) §2.1–2.2，
对比图 [`docs/design/preview/fontsize-compare.png`](docs/design/preview/fontsize-compare.png)。

### 1.2 改动审阅：只做回滚（用户已拍板）

pi 设计上**无权限弹窗、写入立即落盘**（`docs/usage.md:309`）。
因此不做"接受/拒绝"，而是：

1. 编辑照常立即写入
2. 桌面端从 **session JSONL 读回 `old` 内容**，提供「↺ 回滚此改动」
3. **"接受"是隐式的**
4. 加一条常驻汇总条：`3 处改动待确认 · [全部回滚] [查看汇总 diff]`

会话 JSONL 里 edit 工具的参数含 `oldText` / `newText`，足够复原。

### 1.3 工具卡片展开规则

| 情况 | 默认 |
|---|---|
| 失败（非 0 退出 / 报错） | **展开** |
| 含 diff（edit / write） | **展开** |
| 进行中 | **展开** |
| 成功的长输出（read / bash） | **折叠** |
| 成功的短输出 | 折叠 |

用户可手动展开，手动状态优先于默认。

---

## 2. 待用户决策（**开工前必须问清**）

| # | 问题 | 我的建议 | 状态 |
|---|---|---|---|
| 1 | ~~**技术栈**：Electron / Tauri / 纯浏览器~~ | ✅ **已定 Electron**（2026-09-11）。UI 层用 Vite + React 19 + TypeScript + 原生 CSS 变量（**不用 Tailwind，不用组件库** —— 设计稿是反设计系统的，见下方说明） | ✅ 已定 |
| 2 | **项目名** | UI 概念名已是「**砚**」，包名暂用 `yan-desktop` | ⏸ 待确认 |
| 3 | ~~用户说的"Codex"指哪个~~ | 已按网页版"干净卡片"风格起草 | ✅ 已定 |
| 4 | **开源还是私有仓库** | 公开（GitHub Actions 对公开仓库免费无限分钟） | ⏸ 待定 |
| 5 | **是否购买代码签名证书** | 仅小范围分享 → 不签，文档写明绕过方法 | ⏸ 待定 |
| 6 | **目标平台** | 先 Windows，后续再加 | ⏸ 待定 |
| 7 | ~~语言范围~~ | zh-CN + en-US | ✅ 已定 |
| 8 | ~~浅色主题~~ | v1 只做深色，令牌保留 | ✅ 已定 |
| 9 | 注释语言 | 注释中文 / 类型英文 / 对外文档中英双语 | ✅ 已按此实现 |
| 10 | **字体子集方案** | `cn-font-split` 切 woff2（~2–3MB）；**当前是 18.6MB 全量 TTF** | ⏸ 待定 |

> 真正阻塞开工的需求已全部解决。剩下一个**方向**问题（砚 vs 编码客户端，见开头）
> 和一个**产品名**问题，都不阻塞阶段 1（接 RPC 只改 store 层）。

### UI 技术选型（已定，附理由）

| 层 | 选型 | 为什么 |
|---|---|---|
| 渲染框架 | React 19 | 生态厚（markdown / 虚拟列表 / 高亮 / diff）；RPC 事件流用 store 接很顺 |
| 构建 | electron-vite 5 + Vite 7 | Electron 两边一套配置，HMR 快 |
| 样式 | **原生 CSS + CSS 变量** | 见下 |
| i18n | **自写 `t()` + 从 zh-CN.json 推导类型** | DESIGN §6 已这么规定；漏翻译编译报错 |
| 图标 | reicon sprite 内联 | 已生成，`npm run icons` 可重跑 |

**为什么不用 Tailwind / 组件库（Ant Design / MUI / shadcn）：**

1. DESIGN §4 明令禁止投影与渐变 —— 组件库的整个层级语言建立在它们上面，用了等于每写一个组件打一次架
2. 视觉真源是**自己实测出来的令牌**（12.5px / 0.6em 列宽 / `--fact` vs `--warn`），不是组件库的 theme token
3. **最大复用点是 `prototype.html` 里的 CSS** —— 已逐像素调好，抽成 `tokens.css` + `app.css` 后组件只负责加 class
4. `check.mjs` 靠正则扫 CSS/HTML 做禁止项检查，构建期类名生成会削弱这套零成本校验

**阶段 2+ 可能引入的依赖**（用到再加，不提前装）：`zustand`（流式状态）、`virtua`（消息虚拟化）、
`react-markdown` + `shiki`（正文渲染）、`@base-ui-components/react`（模态框无障碍）。

---

## 3. 当前产出

### 3.0 应用代码（**阶段 1 已完成 —— 可用**，2026-09-11）

```
%USERPROFILE%\Desktop\pi-desktop\
├── src\
│   ├── main\
│   │   ├── index.ts         窗口 + IPC + 生命周期（含稳定退出）
│   │   ├── protocol.ts  ⭐ 手写 pi RPC 客户端（JSONL / 请求关联 / 扩展 UI）
│   │   │                       并负责**定位 pi**（内置运行时 → 全局 → PATH）
│   │   ├── agent.ts     ⭐ 协议 → UI 归一化（唯一认识 pi 协议的地方）
│   │   ├── normalize.ts     归一化的纯函数部分（拆出来供 session-reader 复用）
│   │   ├── session-reader.ts 直读 JSONL（大会话打开 486ms，不走 pi RPC）
│   │   ├── memory.ts        记忆存储 + 认识论规则 + soul.md 只读
│   │   ├── sessions.ts      会话列表（只读扫描 sessions/*.jsonl）
│   │   ├── credentials.ts    模型接入（读写 pi 的 auth.json）
│   │   ├── title.ts         用独立 RPC 进程总结会话标题
│   │   └── settings.ts      桌面端设置（不碰 pi 的 settings.json）
│   ├── preload\index.ts     contextBridge 白名单（形态由 YanBridge 约束）
│   ├── shared\ipc.ts        共享类型 + MainPush（已归一化的 UI 补丁）
│   └── renderer\src\
│       ├── state\store.ts   zustand：只负责套用 MainPush
│       ├── components\        TitleBar Rail Continuity Message Composer
│       │                    MemoryPanel UiBridge EmptyStream Onboarding AuthTab …
│       ├── styles\          tokens app stage1 stage2 redesign settings electron highlight
│       └── i18n\            中英双语，类型安全
├── resources\
│   ├── pi\yan-memory.ts     ⭐ 注入给 pi 的记忆扩展
│   └── pi-runtime\          ⭐ 内置 pi 运行时（生成物，不入库；npm run vendor:pi）
├── scripts\                 项目工具（纯应用侧）
│   ├── probe-pi.mjs         单独验证「pi 能否被找到并启动」
│   ├── probe\*.js           在真实应用里跑的 DOM/交互断言（live/memory/sessions/e2e…）
│   ├── test-live.mjs        跑上面这些场景
│   ├── test-unit.mjs        纯逻辑单测（吸附 test-turns.mjs）
│   ├── test-turns.mjs       回合分组 / 段落拆分 / 缓存命中率
│   ├── lint-css.mjs         拦裸 1fr（这个坑出现过 3 次）
│   ├── vendor-pi.mjs        抽取内置 pi 运行时
│   └── shot.mjs             截图
├── docs\
│   ├── dev\                 HANDOFF.md / NEXT-SESSION.md（开发过程文档）
│   └── design\              设计稿与设计工具（prototype.html / check.mjs / icons…）
└── 根目录                    README.md / LICENSE / package.json / tsconfig* / electron.vite.config.ts
```

**验证结果（实测）：**

| 项 | 结果 |
|---|---|
| `npm run typecheck` | ✅ 两个 tsconfig 均无错 |
| `npm run build` | ✅ renderer 1.49MB + 字体 18.6MB |
| `npm run test:live -- live memory sessions` | ✅ 全绿，不烧 token |
| `npm run test:live -- e2e` | ✅ 真发消息、真工具调用全通过 |
| `npm run probe-pi` | ✅ 找到 pi（AppData\Roaming\npm）、contextWindow = 1000000 |

真实能力已验证：流式对话、markdown + 语法高亮、bash/edit 工具卡（含彩色 diff）、
工具结果回执、多会话切换、新建会话、记忆确认流程、审阅条。

截图：`docs/design/preview/yan-live-1440x900.png`（真实对话）、
`yan-live-diff.png`（edit 的 diff）、`yan-live-tools.png`

### 3.1 设计稿

```
%USERPROFILE%\Desktop\pi-desktop\
├── docs\dev\                        ← 开发过程文档（HANDOFF / NEXT-SESSION 已移到这里）
│   ├── HANDOFF.md                    ← 交接文档（本文件）
│   └── NEXT-SESSION.md               ← 新会话开场指令（可直接复制）
└── docs\design\
    ├── prototype.html               ← 可交互设计稿 v0.2（112KB，浏览器直接打开）
    ├── DESIGN.md                    ← 设计规范（令牌真源，已到 v0.2）
    ├── check.mjs                    ← 静态自检（9 类，零成本）
    ├── measure-design.mjs           ← 量设计稿溢出（npm run measure:design）
    ├── embed-icons.mjs               ← 把用到的图标 symbol 子集内联进设计稿
    ├── build-icons.mjs              ← 图标抓取 + 生成（可重跑，从 reicon 拉）
    ├── extract-icons.mjs            ← 设计稿 sprite → renderer 的 TS 模块（npm run icons）
    ├── archive\                     ← 旧设计稿（v0.2 编码版）与一次性修复脚本
    ├── icons\                       ← reicon 图标集（已生成）
    │   ├── reicon.svg               ← 239KB 完整 sprite，166 个 symbol
    │   ├── icons.ts / icons.json    ← 名称 ↔ symbol id 映射
    │   ├── preview.html             ← 92 个图标总览（已视觉验证）
    │   └── preview.png
    ├── font-test\                   ← 字体调研（已定：Maple Mono CN）
    │   ├── README.md                ← 选型结论 + 实测表
    │   ├── fonts.html               ← 三字体逐字测宽对比
    │   ├── maple-size.html          ← Maple 四档字号对比
    │   ├── compare.png / msize.png
    │   └── MapleMono-CN-Regular.ttf ← 18MB，已 gitignore
    └── preview\                     ← 渲染截图（⚠️ 文件名后来随 UI 重做换过，以实际为准）
        ├── ui-live.png / ui-settings.png   ← README 用的两张
        ├── yan-app-*.png                   ← 设计稿渲染
        ├── yan-live-*.png                  ← 真实对话截图
        └── redesign-before/after.png       ← 一次 UI 重做的前后对比
```

**设计稿状态：v0.2 已完成。用户 8 条意见全部落实，`check.mjs` 全绿，已逐张目视校验。**

### v0.2 相对 v0.1 的改动（全部已完成）

| # | 改动 | 落在哪里 |
|---|---|---|
| 1 | 字体换 Maple Mono CN，基准 **12.5px**（代码 12px） | 令牌 §0；DESIGN §2.1–2.2 |
| 2 | 图标全换 reicon sprite（25 个内联子集），删光 `◐ ↺ ✓ ✗` | `embed-icons.mjs` + ICON-SPRITE 标记 |
| 3 | 上下文窗口 `128,000` → `1,000,000`（29%） | 右栏上下文卡 |
| 4 | 左栏：时间分组可折叠 + 会话内嵌可折叠任务 + 整栏可折叠 | 左栏 + 标题栏最左钮 |
| 5 | 右栏中下部：文件树（目录折叠 + 改动 ±N） | 新增 §3.4 区 |
| 6 | 工具卡智能展开（手动优先，折叠留摘要行） | §3.1 规则 + JS |
| 7 | 状态面板分区可拖拽排序（Alt+↑↓ / localStorage / 重置） | §3.3 |
| 8 | 常驻汇总条「3 处改动待确认 · [全部回滚] [查看汇总 diff]」 | 输入区上方 |
| 9 | 每条 edit 卡加「回滚此改动」 | 编辑卡操作栏 |

### v0.2 已实现的内容（设计稿是活文档，不只是静态图）

- 三栏布局：会话列表(216) / 消息流(780 上限) / 状态面板(320)，左右两栏可折叠
- 消息流：用户气泡、助手正文、思考折叠块、工具卡片（read / edit+diff / bash 成功 / bash 失败 / 进行中）、错误条
- **工具卡智能展开**：失败/diff/进行中展开，成功长输出折叠，折叠时留一行摘要；点过一次就转手动（`data-manual`），不再被自动规则推翻
- **状态面板六区**：会话 / 上下文窗口 / 工作区 / 文件树 / 任务 / 计费，可拖拽排序 + `Alt+↑↓`，`localStorage` 持久化，「重置顺序」还原
- 文件树：目录折叠（可见性整体重算，非增量）、选中态、改动文件带 `+N −N`
- 汇总条：`3 处改动待确认` + 全部回滚 + 查看汇总 diff（接受是隐式的）
- i18n：`LOCALES` 含 zh-CN + en-US 各 **55 键**，右上按钮实时切换（含 `title` 属性）
- 深浅主题令牌（浅色未打磨）
- 峰谷指示器：**实时计算**，用 CommandCode 真实时段

---

## 4. 关键事实（已核实，勿重复调查）

### 环境

| 项 | 值 |
|---|---|
| Node | v24.19.0 |
| npm | 11.17.0 |
| Windows | 10.0.26200（Win11 24H2+） |
| git | 2.55.0 ✅，但 **`user.name` / `user.email` 未配置** |
| gh CLI | ❌ 未安装（分发需 `winget install GitHub.cli`） |
| Rust | 1.96.0（Tauri 可行） |
| WebView2 | 149.0.4022.80（Tauri 可行） |
| Edge | 151.0.4129.93（可用于 headless 截图） |
| 磁盘 | C: 剩 60G |

### pi 包

| 项 | 值 |
|---|---|
| 版本 | 0.85.1 |
| 路径 | `%USERPROFILE%\AppData\Roaming\npm\node_modules\@earendil-works\pi-coding-agent` |
| 许可证 | **MIT** → 可以打包分发，保留版权声明 |
| 体积 | 424 MB（`node_modules` 401MB，其中 `@esbuild` **284MB**） |
| `dist/bundle` | **7.7 MB**，但实测**仍需外部依赖**（报缺 `@earendil-works/chord`） |
| 运行时依赖 | `@earendil-works/{chord,pi-agent-core,pi-ai,pi-telemetry,pi-tui}` 共 15MB + openai/google 等 |
| 无独立二进制 | 只有 `dist/bundle/cli.js`（需 Node） |

**打包 pi 是待解问题**：`dist/bundle` 不自包含，`@esbuild` 占 284MB。
估算裁剪后可到 ~60MB。分发前必须先解决。

### 分发路径（三选一，未定）

| 方案 | 用户门槛 | 体积 | 复杂度 |
|---|---|---|---|
| A. 要求用户自己 `npm i -g pi` | 高 | ~0 | 低 |
| B. 打包进应用 | 低 | +60~100MB | 中 |
| C. 混合：检测到就用，没有用内置 | 低 | +60~100MB | 中高 |

建议：开发期用 A，分发时做 C。

### 代码签名（若分发需要花钱）

| 平台 | 不签名的后果 | 成本 |
|---|---|---|
| Windows | 每次打开弹 SmartScreen 警告 | $100–400/年 |
| macOS | Gatekeeper 拦截 | $99/年 |

---

## 5. ⚠️ 已知更正（重要，别再搞错）

### 5.1 CommandCode 峰谷时段 ≠ DeepSeek 官方时段

**用户的实际计费提供方是 CommandCode，不是 DeepSeek。**

| | 峰时窗口 |
|---|---|
| DeepSeek 官方（**错误**，早期面板用的） | UTC 16:30–00:30 |
| **CommandCode 实际（正确）** | **UTC 01:00–04:00 与 06:00–10:00，仅周一–周五** |

换算：

| 时区 | 峰时（全价） |
|---|---|
| UTC | 01:00–04:00、06:00–10:00 |
| 北京 UTC+8 | 09:00–12:00、14:00–18:00 |
| **悉尼 UTC+10** | **11:00–14:00、16:00–20:00** |
| 悉尼夏令时 UTC+11 | 12:00–15:00、17:00–21:00 |

**折扣一律 −50%**（早期面板写的 −75% 是错的）。

**待办**：`~/.pi/agent/extensions/left-info-panel.ts` 里的峰谷逻辑仍是错的，
需要改成上表。设计稿里的 JS 用的是**正确**时段。

### 5.2 看图：主模型现在能直接看（用户已开启）

**❗重要更新（2026-09-11）：用户已经给主模型开启了图片输入。**
在会话里直接 `read` 一张 PNG 就能看到图，**不需要再调 see2.mjs / 视觉模型**。
这省掉了整个外部链路、也不再受 vision 模型上游挂掉的影响。

如果哪天图片又看不了了，再回到下面这套口径（低优先级的备案）：

| 模型 | 5h / 周 / 月 请求数 | 输入 / 输出（谷时 $/M） |
|---|---|---|
| `commandcode/deepseek/deepseek-v4-flash-vision-exp` | 6,080 / 15,200 / 30,400 | $0.22 / $0.66 |
| `commandcode/gpt-5.6-sol` | 414 / 1,040 / 2,070 | $5.00 / $30.00 |

> 这个便宜的 vision 模型当时上游挂过（`No available providers match the 'only' filter: deepseek`），
> 要显式指定 `commandcode/gpt-5.6-sol`。在它能用之前，先用主模型直接看图。

### 5.3 截图成本极低

实测：一张 1440×900 截图 → **277~337 输入 token**。

| 项 | 成本 |
|---|---|
| 一次截图审查 | **≈ $0.0002** |

**图片不是成本瓶颈，我自己（每轮重发全部上下文）才是。** 优化轮次，不优化截图次数。

### 5.4 套餐额度

GOAT 计划 $10/月 = $70 额度：$14 / 5小时，$35 / 周，$70 / 月。

### 5.5 ⚠️ 模型的 128k 上下文窗口是假的，已修复

**pi 给所有模型的 `contextWindow` 都是硬编码兜底的 128000，不是真实值。**

根因链路：

```
1. CommandCode /models API 返回        context_length: 1000000
2. pi 生成 ~/.pi/agent/models.json 时   只写 id / name / reasoning / input
                                        ❌ 丢掉 context_length
3. provider-composer.js:72 兜底         contextWindow ?? 128000   ← 假的
                                        maxTokens     ?? 16384
```

**实测验证**（直接调 CommandCode API，绕过 pi）：

| | 值 |
|---|---|
| pi 给的 | 128,000 |
| **实测服务端接受** | **896,758 tokens**（HTTP 200，发了 5,060,000 字符） |
| API 声明 | 1,000,000 |

**已修复** —— 用 `~/.pi/agent/.dev/sync-model-context.mjs` 按 API 声明回填全部 69 个模型：

```bash
node ~/.pi/agent/.dev/sync-model-context.mjs            # 预演
node ~/.pi/agent/.dev/sync-model-context.mjs --write    # 写入（自动备份）
node ~/.pi/agent/.dev/sync-model-context.mjs --write --verify
```

**`pi update` 后 `models.json` 会被重写 → 需重跑此脚本。**

回填后的窗口分布：

| contextWindow | 模型数 | 代表 |
|---|---|---|
| 1,050,000 | 3 | gpt-5.6-sol / terra / luna |
| 1,048,576 | 9 | gemini-3.7-flash、muse-spark |
| **1,000,000** | **34** | **deepseek-v4.1-flash、claude-sonnet-5、Qwen3.8-Max** |
| 500,000 | 2 | grok-4.5 / 4.6 |
| 400,000 | 4 | gpt-5.5 / 5.4 / 5.3-codex |
| 256,000–262,144 | 10 | Kimi-K2.7-Code、hy3 |
| 200,000 | 7 | GLM-5.1、MiniMax-M2.5 |

**连带影响：压缩阈值从 111,616 → 983,616 tokens**
（`compaction.js:163`：`contextTokens > contextWindow - reserveTokens`，reserve 默认 16384）

长任务不再每 11 万 token 被压缩一次，上下文能保住 8.8 倍。

**未做**：`maxTokens` 保持 pi 兜底 16384。没实测过输出上限，猜大了有的 API 会返 400。

**风险窗口**：声称 1M，实测到 896k，中间 10% 未验证。若在该区间报错，把 `contextWindow` 下调到 950000。

> 成对提醒：窗口变大不等于免费。长上下文每轮重发 token 更多，
> 但 DeepSeek 前缀缓存读只要 **$0.007/M**（输入的 1/21）。
> **保持会话前缀稳定**（别频繁改系统提示、别来回切模型）能让缓存持续命中。

---

## 6. 成本控制约定（用户明确在意额度）

| 策略 | 做法 |
|---|---|
| **少轮次** | 一次写完整文件，不做十次小修补。同样功能轮次差 3–5 倍 |
| **存档** | 本文件就是存档。上下文被压缩后读它恢复，不要重扫代码库 |
| **验证分级** | ①免费：`check.mjs`、类型检查、DOM 断言 → ②极低：截图（$0.0002）→ ③避免：用视觉模型做日常调试 |
| **挑时段** | 悉尼谷时：工作日 20:00–次日 11:00、14:00–16:00，周末全天。峰时全价 |
| **主力模型** | 继续用 `deepseek/deepseek-v4.1-flash`（30,800 请求/5h，套餐内最高）。仅截图审查那轮切 vision 模型 |

---

## 7. 已验证的自检工具链

### 静态检查（零成本，每次都跑）

```bash
node %USERPROFILE%\Desktop\pi-desktop\docs\design\check.mjs <设计稿路径>
```

### 截图

```bash
EDGE="/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
"$EDGE" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --screenshot="out.png" --window-size=1440,900 "file:///C:/path/to/page.html"
```

⚠️ `file://` 页面**不能加载同目录的图片子资源**（被 Chromium 拦截）。
需要裁剪/放大时，用 `%TEMP%\mkhtml.mjs` 把 PNG 转成 **data URI** 再嵌入：

```bash
node %TEMP%\mkhtml.mjs <源图> <输出html> <源宽> <缩放> <裁剪x> <裁剪y>
```

### 视觉校验

**首选：直接 `read` 那张 PNG。** 用户已给主模型开启了图片输入，
我直接看图就行（零额外成本、无需另调模型）。本会话的所有视觉校验都是这么做的。

**备案（仅在主模型看不了图时）：**

```bash
node %TEMP%\see2.mjs <图片> "<问题>" commandcode/gpt-5.6-sol
```

> ⚠️ 默认的 `deepseek/deepseek-v4-flash-vision-exp` 上游挂过
> （`No available providers match the 'only' filter: deepseek`），
> 要显式传 `commandcode/gpt-5.6-sol`（~$0.02/次，贵 ~20 倍但可用）。

已固化的经验：

- 加 `--tools ""` 禁用工具，否则它会试图自己去读文件而不看图
- 提问要**具体、可证伪**（"进度条左右是否与内边距对齐"），不要问"好不好看"
- 它会犯错，也会把非问题说成问题（已遇到 4 次误报）。
  **收到反馈后先回代码验证，再改**
- 明确告诉它"不确定就说不确定"

#### 比截图更可靠的：DOM 断言

本会话验证 v0.2 的交互时，**截图只用于看静态布局**，真正的对错靠 DOM 断言：

```bash
EDGE=".../msedge.exe"
"$EDGE" --headless --dump-dom "file:///.../prototype.html" > dom.html
grep -o 'class="tool[a-z ]*"' dom.html      # 智能展开是否有生效
```

要测交互（拖拽排序 / localStorage / 折叠），就**注入一段测试脚本、把结果写进 `document.title`**，
再 `--dump-dom` 读出来。用 `grep` 找 `<pre>` 里的中文输出会被转义坑到，`title` 不会。

> ⚠️ 两个坑：
> 1. 注入的测试脚本不要用 `const` 重复声明主脚本已有的变量（`panel` / `tree` 等），
>    整个 script 会 SyntaxError **且 `window.onerror` 捕不到**（解析期错误）。
>    包一层 IIFE。
> 2. git-bash 的 `/tmp` ≠ `C:/tmp`。用 `file:///C:/tmp/x.html` 会得到 Edge 的 404 页，
>    看起来像「脚本没跑」。一律用 `%TEMP%` 的真实路径。

### 其他临时工具（在 `%TEMP%`）

| 脚本 | 用途 |
|---|---|
| `see2.mjs` | 视觉校验（见上） |
| `crop.mjs` | 截图局部放大再截（`node crop.mjs <src> <out.html> <srcW> <scale> <x> <y> <vw> <vh>`）|
| `imgtok.mjs` | 测图片 token 成本 |
| `build-icons.mjs` | 图标构建（**已归入项目**：`docs/design/build-icons.mjs`）|
| `parse-reicon.mjs` | 解析 reicon 清单 |

---

## 8. 已发现的设计陷阱（别重犯）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 用 `direction:rtl` 截断 Windows 路径 | 反斜杠触发 bidi 重排，路径乱序 | 右对齐 + `overflow-wrap:anywhere` 让路径换行 |
| 中栏不加 `max-width` | 1920px 下行长失控 | `--w-stream:780px` + `margin:0 auto` |
| 上下文进度条用绿色 | 与"成功/新增"语义冲突，一栏出现 4 种强调色 | 用 `--accent` 蓝 |
| 状态面板先放用量/花费 | 读起来像账单后台而非编码工具 | 代码上下文（改动/检查）在上，计费沉底 |
| `--fg-mute` 用 `#5c5c66` | 对比度仅 2.8:1，小字看不清 | `#82828f`（4.9:1，过 AA） |
| 增删统计 `+14 −2` 用单列 | `+N` 数字纵向不对齐 | 拆成两个定宽 grid 列，无值留空占位 |
| 靠系统字体回退凑栅格 | 汉字≠2×ASCII，表格/边框/diff 全错位 | **内嵌字体**，选 0.5em 或 0.6em 步进的 |
| 图标 sprite 内部 id 冲突 | `<clipPath>` 串位，裁剪到别的图标 | 构建时给内部 id 加图标名前缀 |
| 解 reicon 的 `@preview` base64 | 拿到的是硬编码 `#e4e4e7`，深色下变死白 | 取模块里的 `O`/`F` 字段（`currentColor`）|
| `calc(var(--d) * …)` 做树缩进 | HTML 属性**不会**生成同名自定义属性，`var(--d)` 为空 → 整条 `padding` **静默失效** | 用属性选择器 `.tnode[data-d="2"]{…}` |
| `file://` 外部引 sprite | Chromium 拦截跨文档 `<use href="x.svg#id">`，图标全空 | 构建期内联子集（`embed-icons.mjs`） |
| `file://` 用 localStorage | opaque origin → 抛异常，整段排序逻辑中断 | 读写包 `try/catch`，降级为「仅本次会话有效」 |
| `@font-face` 路径挂在子目录页 | 相对路径是相对 **HTML 所在目录**，字体静默不加载 | 测试页放 `docs/design/`，别放 `preview/` |
| 字号取 12px（非整数像素格） | 汉字格 14.4px，逐字累积 0.234px 抖动 | 取 **12.5px**（汉字格 15.00px 整） |
| 树折叠用增量 add/remove `hidden` | 嵌套折叠时残留错状态 | 遍历一次、按祖先展开态**整体重算** |
| 注入测试脚本重复 `const` 声明 | 整段 SyntaxError，且 `onerror` 捕不到 | 包一层 IIFE |
| 用 git-bash 的 `/tmp` 当 `C:/tmp` | `file:///C:/tmp/…` 得到 Edge 404 页，误判成「脚本没跑」 | 一律用 `%TEMP%` 的真实路径 |
| `--dump-dom` 去 `grep` 中文 `<pre>` 输出 | HTML 转义/编码把结果弄花，看不到内容 | 把结果写进 `document.title` 再抓 |

### 阶段 0 新踩的坑（实现时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 入口模块用 top-level `await app.whenReady()` | **ESM 下死锁** —— Electron 要等整个模块图求值完才发 ready，而 ready 又等这个 await。表现为「跑起来什么都没发生」，也不报错 | 包成 `async function main()` 再调用 |
| grid 子项用默认 `auto` 列 | 列被撑到子元素 `max-content` —— 长会话名直接撑破 `.rail`（实测 224px 变 255px，右侧内容被裁） | `grid-template-columns: minmax(0, 1fr)`；flex 子项同理 `min-width: 0` |
| `<details>` 折叠时测溢出 | Chrome 用 `content-visibility:hidden` 隐藏内容，被隐藏元素仍有非零 rect → 溢出探针**假阳性** | 探针里用 `el.checkVisibility({contentVisibilityAuto:true})` 排掉 |
| 嵌套 `.memrow` 的操作按钮与来源标记抢同一格 | 下边距/行高跳动 | 两者共用 `grid-column:3`，悬停时用 `:has(.acts)` 隐掉 `.src` |
| 记忆分区标题用 `<button>` 包住可聚焦的拖拽把手 | 嵌套交互元素是**非法 HTML** | 外层用 `<div role="button" tabIndex={0} aria-expanded>` + 手写 Enter/Space |
| `npm install` 默认不跑依赖的 install 脚本 | **Electron 二进制不会下载**（`node_modules/electron/dist` 缺失），esbuild 同理 | `npm approve-scripts --all`，必要时手动 `node node_modules/electron/install.js` |
| `@vitejs/plugin-react@6` 要 `vite@8` | 与 electron-vite 允许的 vite 7 冲突，`npm install` ERESOLVE | 锁 `@vitejs/plugin-react@^5.2.0`（peer 支持 ^7 \|\| ^8） |
| 自绘标题栏忘了 `-webkit-app-region` | 按钮点不动（整条都被当拖拽区） | 标题栏 `drag`，内部可交互元素全部 `no-drag` |
| 字体 TTF 被 `.gitignore`（`*.ttf`）排除 | 新克隆仓库没有字体 → @font-face **静默回退**，汉字格不再是 15.00px，栅格塔掉**且不报错** | 加了 `npm run font`；`npm run probe` 会打印汉字格宽供验证 |

### 阶段 1 新踩的坑（接 RPC 时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 在 ESM 入口用 top-level `await app.whenReady()` | **死锁**（第 4 版已记） | 包成 `main()` |
| `spawn` 一个 `.cmd` 需要 `shell:true` | shell 会做引号解析 —— 提示词里的引号/反斜杠/中文被吃掉或注入 | 用 `process.execPath` + `ELECTRON_RUN_AS_NODE=1` 跑 pi 的 **JS 入口**，纯参数数组 |
| `return '错误'` 提前退出探针脚本 | **丢掉缓冲区**，只看到一行错误看不到前因 | 失败也把 `out.join()` 返回，并把行首打上 `✗` |
| `memoryList` 每次都从磁盘重读 | 刚改完立刻读会读到旧文件，**把内存里的改动回滚**（删掉的条目复活） | `load()` 先 `await this.writeQueue` |
| 退出时不等记忆写盘 | 用户点完「对」立刻关窗口 → **确认丢丢失** | `shutdown()` 里 `await memory.flush()`；`before-quit` 要 `preventDefault` 再等 |
| `overflow-y: auto` 连带把 `overflow-x` 变 `auto` | 纵向滚动条一出现就挤窄 `clientWidth` → 冒出一条横向滚动条 | `overflow: hidden auto` + `scrollbar-gutter: stable` |
| 把扩展 `notify` 直接堆成列表 | 真实扩展（如 `left-info-panel`）会反复通知 → **刷满屏幕遮住界面** | 去重 + 上限 3 条；`.notices` 容器 `pointer-events:none` |
| `YAN_PROMPT` 用 did-finish-load + 定时两个入口 | 提示词被发了多次 | 加 `fired` 标志，只发一次 |
| edit 工具的参数形状 | 按 `old_string/new_string` 渲染 → 显示原始 JSON 而不是 diff | pi 当前是 `{path, edits:[{oldText,newText}]}`；要兼容 legacy 顶层 `oldText`（见 extensions.md:2035） |
| `spawn` 不传 `stdio` 时 TS 推出 `never` | `child.stdout` 全部报错 | 显式标 `ChildProcessWithoutNullStreams` |
| `npm run test:live -- a b` 只跑了 `a` | `process.argv[2]` 只取第一个 | 用 `slice(2).filter(a => !a.startsWith('-'))` |
| 会改文件的测试不清理 | 真的写进了用户的 `~/.pi/agent/sessions/` 和 `memory.json`，造成脏数据 | 测试自带清理 + 按首条消息前缀删除自己造的会话 |

> 诊断心得：右栏那条横向滚动条在**空状态**下看不到（内容不够高，不出滚动条），
> 只有数据满了才复现 —— 所以探测脚本必须在**有真实数据**的实例上跑，不能只看空壳。

---

### 第 7 版的三个坑

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| grid 容器的裸 `1fr` | 轨道最小值是 `auto`(内容宽) → 内容一宽就撑破容器，**溢出的部分被右邻不透明列盖住**（"12m 被遮" / "按钮边框不见" / 右栏被裁一列） | `minmax(0, 1fr)` + 子元素 `min-width:0`。已加 `npm run lint:css` 自动拦。**这个坑出现 3 次了** |
| 验收测试与用户共用状态 | 测试改掉了用户的**右栏分区顺序**；更早还往会话目录塞过 6 个会话、往记忆里留过 5 条编造的「已确认事实」（会误导后续对话） | 测试全跑隔离沙盒：`YAN_USER_DATA` / `YAN_SESSIONS_DIR` / `YAN_DATA_DIR` 三个 env 分别隔离 localStorage / 会话 / 记忆；fixture 从真实会话**只读拷贝** |
| pi 的 `--session-dir` 不跟着 env 走 | 隔离时会出现「pi 写 A、左栏读 B」的诡异现象 | `SESSIONS_DIR` 同时喂给 pi 的 `--session-dir` 与列表扫描 |
| 扩展的 `panel_todos` 只在 TUI 可见 | agent 调了工具维护进度，桌面端什么都不显示 —— 工具白调，用户也看不到它在干什么 | 读会话里的 `custom` entry（`customType` 匹配 `task|todo`）→ 左栏顶部渲染 |
| 扩展的启动通知 | `left-info-panel` 每次启动都 notify「信息面板已启用（overlay 44 列）· /panel …」—— 讲的是 TUI 的 overlay，桌面端不适用，开机弹出来让人困惑 | 启动期的 info/warning 通知只进日志抽屉（`startupPhase` 标志，收到 `msg-add` 后结束） |
| 删 import 时删多了 | `homedir` 被删掉但还在用 → **主进程启动即崩**（`ReferenceError: homedir is not defined`） | 只删确认用不到的；`tsc` 在改完立刻跑一遍 |
| 顶层直接执行 + `const` | 函数声明会提升、`const` 不会 —— fixture 生成器在 TDZ 里被调用 | 执行逻辑包进 `main()`，最后 `await main()` |

---


### 阶段 2 新踩的坑（补功能时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| pi 的会话文件是**懒创建**的 | `new_session` 后文件不落盘，左栏扫不到 → 点「新对话」**毫无反应** | 左栏为「当前会话」补一条合成条目（没落盘也能看见） |
| `new_session` 在当前会话为空时**不新建** | 返回同一个 sessionId/文件，容易误判成 bug | 这是 pi 的行为，不必绕；但测试别假设「一定会多一个」 |
| `bash_execution_update` 的 `id` 就是请求 id | 要接流式就得**在发命令前**知道 id；`command()` 默认自己生成 id 就接不上 | `command()` 支持传 `opts.id` |
| bash 是流式的，但响应里也有完整 `output` | 只用响应会丢掉「边跑边看」；只用流式可能不完整（有些命令一次性吐完） | 两者取长者 |
| 直接执行的 bash 走了「成功长输出折叠」规则 | 用户主动跑的命令看不到结果，等于白跑 | 给 ToolCard 加 `defaultOpen`，`!` 直执行的默认展开 |
| 附件去重只比 `existing` | 一次传入两条相同的不去重（批内没互相比） | 用 Set 同时比对已有 + 批内 |
| 不检查「当前正在使用的会话」就删文件 | pi 还持有那个文件句柄 | 删除前拦掉当前会话 |
| `deleteSession` 不校验路径 | 可被 `../` 穿越删别处的文件 | 校验：必须在 sessions 目录内且是 `.jsonl` |
| 在真实会话目录里跑测试 | 污染用户数据（我确实造了脏数据，后来清理了） | `YAN_SESSIONS_DIR` 指向临时目录 |
| 测试「改文件」的副作用没清 | 我编造的 5 条**假记忆**（含 4 条"已确认事实"）留在用户记忆里 —— 会误导后续对话 | 测试只动自己造的数据；用完必须清 |
| 探测脚本在**真实数据**上跑才发现的问题 | 长会话才触发的 bug（横向滚动条）在空壳上看不见 | 探测脚本要同时覆盖「空」和「满」两种状态 |

> 一条经验：**先确认自己测的是不是对的东西**。
> 阶段 2 里我报了 4 个「bug」，其中 2 个是我的测试断言写错了
> （bash 折叠算 UX 问题、会话列表断言用了错的判据）。查代码之前先看证据。

---

### 第 8 版的坑（视觉重做）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 给 pi 传 `--session-dir` | pi **不再按 cwd 建项目子目录**，新会话被平铺到 sessions 根目录 → 与用户已有会话分居两处，左栏靠「合成条目」显示 | 平时不传，交给 pi 自己组织；只在测试隔离时传（sandbox 里平铺无妨）。见 `SESSIONS_DIR_IS_OVERRIDE` |
| 给 pi 传 `--name 砚` | 每个新会话标题都是「砚」，左栏里长得一模一样，**等于没标题** | 不传。让 pi 用首条用户消息当标题 |
| 历史消息的思考块显示「正在思考…」 | 会话一多，每条历史消息都挂着「正在思考」，看着像卡死 | 三种标题分开：`live` →「正在思考…」／有耗时 →「已思考 N 秒」／**无耗时（历史加载）** →「思考过程」 |
| 空名字重命名 | pi 的 `set_session_name` 对空串返回 `success:false` —— 名字**一旦设了就清不掉** | 主进程拦住空值并给出理由；不要在 UI 上放「清空」按钮（假的） |
| 空会话时三栏全空 | 界面一片死寂，这是「难看」的最大来源 —— 不是配色问题 | 空状态要给**内容**：符号 + 记忆现状 + 可点建议 + 命令提示，把空白变成入口 |

### 第 9 版的坑

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 复用主会话去「总结标题」 | ① 污染对话（用户会看到「总结这句话」）② **prompt cache 全失效**（代价比一次请求贵得多） | 起独立进程 `--no-session --no-extensions` + thinking 关掉，用完即走（见 `src/main/title.ts`） |
| 用 `translateX(百分比)` 做滑块 | 百分比基数是**元素自身宽度**而非容器 → 走短一半（实测 105px vs 正确 123px），滑块停在左边一档 | 用 `left` + 实测 `offsetLeft/offsetWidth`（档位文字宽度不等，等分算不准） |
| 用 `dispatchEvent('mouseover')` 测 `:hover` | **合成事件不产生真正的 hover 状态**，`:has(.x:hover)` 永远不匹配 —— 断言失败但功能是好的 | 涉及 `:hover` 的只能 `sendInputEvent`（需窗口可见），或在测试里验证「CSS 规则存在」 |
| `.outline` 有 `pointer-events:none` 却写 `.outline:hover` | 它自己永远收不到 hover（只有刻度可点） | `:has(.outline-tick:hover)` 反推 |
| 给左栏用 `grid-template-rows` 数固定行数 | 子元素「有时有有时没有」（搜索框按需出现）→ 隐式行撑歪高度（805px vs 容器 866px），底部空一截且不随窗口变化 | 用 **flex 列**，让需要滚动的区域 `flex:1` |
| 失败的工具卡自动展开 | 失败命令的输出经常几十行，全展开把后续对话推出视野 | 只有「正在跑」和「用户主动跑的 `!` 命令」自动展开；失败只在摘要行标红 |
| 同一件事放两个入口（侧栏开关） | 用户不确定该点哪个 | 只留标题栏最左上角一个 |
| 用 `write` 工具**整体覆盖** CSS 文件 | 丢了 2174 行（靠 git 提交恢复） | 大文件一律用「追加」或精确 edit；覆盖前先 commit |

> 关于 `:hover` 那条值得单独记：**DOM 断言测不了 CSS 伪类**。
> 这不是「测试写得不好」，是工具边界 —— 要么上 `sendInputEvent`，要么把断言拆成
> 「规则存在」（可测）+「视觉行为」（人工看一眼）。

---

### 第 10 版的坑（回合合并 / 大会话性能 / 发布准备）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| **位置跳转用 `behavior:'smooth'`** | **滚动完全不发生**（`scrollTop` 一动不动）。而同一行代码在探针里单独调却是好的 —— 因为平滑滚动会被**任何一次重渲染取消**，而这条路径上总有重渲染（`setStickNow` / `setHover` / 流式推送）。第一次修时我写「跳得远就瞬移、跳得近才平滑」——**只拆中一半**：阈值是 `box.height*1.5`=954px，实测跳动 594px 恰好小于它 → 又走回平滑 → 又不动 | 位置跳转**一律 `behavior:'auto'`**。「跳到第 N 轮」是定位不是看动画。见 `App.tsx` 的 `scrollToTurn` |
| 用 `execFileSync` 探测 pi 版本 | **阻塞主进程事件循环** —— 实测 183ms 内所有 IPC 排队（bootstrap 并发下发 11 个请求全在等它） | 改 `execFile`（异步）+ 结果缓存 + 重入保护 |
| 大上下文会话「打开卡」 | 真因不是渲染（实测空转 3.5ms/帧、滚动 4.5ms/帧）：**pi 的 `switch_session` 要 2780ms**，而直接解析同一份 JSONL 只要 **59ms**。17MB 文件里 26 行 >100KB 占 12.9MB，最长一行 4MB（含 base64 图片的 toolResult） | 先读文件铺内容（486ms）再让 pi 在后台切；超长文本/图片**有损降级**并如实标记「N 条被截断」 |
| 以为 `get_messages` = 会话全部消息 | 它**不含压缩前历史**（`docs/rpc.md` 写明；要完整的得用 `get_entries`）。所以大上下文会话在界面上只剩当前窗口（实测 70 条 vs 文件 1093 条） | 需要完整历史就直读 JSONL（`src/main/session-reader.ts`） |
| 原始 range 滑块做「思考强度」 | `::-webkit-slider-runnable-track`（14px 高）与 `::-webkit-slider-thumb`（16px）**不同源** → 视觉上是断的；再叠一层绝对定位的档位小方块，两套指示打架（用户报「有 bug」） | 用**离散档位按钮**（1:1 映射）。滑块本身是谎言：档位是离散的，滑动却没有中间态 |
| `background-size: 240px` + `no-repeat` 做扫光 | 那 240px 之外**什么都不画** → 一条 898px 的线只显示前 240px，看着像断了 | **两层背景**：`background-color` 铺满（保证线完整）+ `background-image` 只做高光带 |
| `highlight.css` 里把 `.hljs` 全局替换成 `.md pre code` | 选择器变成 `pre code.md pre code-keyword` 这种无意义的东西 —— **语法高亮从来没生效过**（实测 `span[class^="hljs"]` 数恒为 0），浅色下代码块还是写死的深色底 | 作用域限定 + 保留 `.hljs-*` 类名：`.md pre code .hljs-keyword`；底色改走 `--code-bg` 令牌 |
| 渲染端 `window.keydown` 接全局快捷键 | 会被**输入法组合态**、焦点不在 webContents、菜单 accelerator 吃掉（三种都实测到） | 主进程 `before-input-event` + 把动作名发回渲染端（协议知识不进主进程） |
| `patchSettings` 读-改-写用内存快照 | 写回的是**整个对象**，两个写入方互相覆盖 —— 实测把 `alwaysOnTop` 的改动抹掉了（查三次都写不进去） | 写入前 `invalidate()` 重读磁盘 |
| 测试直接写 `~/.pi/agent/auth.json` | 那里是用户的**真实密钥**。写坏了比污染会话目录/记忆文件严重得多（那两件已经各踩过一次） | 加 `YAN_PI_DIR` 覆盖，测试用临时目录 |
| 探针按会话**标题**找 fixture | 改成「每轮用模型重新生成标题」之后，前面的场景一跑就把 fixture 标题改了 → 后面按 `title.includes('YAN-TODO')` 找不到。**典型「只在全量跑时暴露」** | 按 `path` 找（文件名里的 fixture 标识不会变） |
| 探针用固定 `sleep(900)` 等左栏展开 | 左栏是「320ms 延迟 + CSS 过渡」，负载高时（连跑多个场景）不够 → 偶发失败，单独跑必过 | 一律用**轮询**（`until(fn, ms)`），不用固定 sleep 等 UI 状态 |
| 测试之间共享 `localStorage` | 前一个场景改了 `yan.rail-open` 会把后一个场景的初始状态改掉 | 探针自己 setup 需要的状态，不假设初始值 |

---

> 这一版最重要的两条抽象：
> **① 浏览器会因为重渲染取消平滑滚动** —— 所以「跳转」类操作一律瞬移。
> **② 「打开慢」要先量再修** —— 我一开始以为是渲染（量了帧率：3.5ms/帧，不卡），
> 又以为是切换（量了：95–236ms，不慢），最后量到 `switch_session` 的 2780ms。
> 三次都在猜，第四次才量对。**先测出 47 倍的差距，再动手。**

---

> 教训：**层次靠色阶差，不靠投影**。
> 旧色阶 `bg-0→bg-1` 只差 7 个亮度值，三栏糊成一片黑，
> 所以「卡片」这个概念在视觉上根本不存在。拉开到 ~10 就好了。

---



### 架构

```
Renderer (React, 无 Node 权限)
   ↓ contextBridge 白名单
Main (Node)
   · SessionHub: Map<sessionId, PiSession>
   · 自己写的 RPC JSONL 编解码 + 请求响应关联
   · SessionStore: 读 ~/.pi/agent/sessions/
   · SettingsStore: 读 ~/.pi/agent/settings.json
   ↓ stdin/stdout JSONL
pi --mode rpc  ×N（一个会话一个进程）
```

**三条原则**

1. 一个会话 = 一个 pi 进程（崩溃隔离、并行、扩展环境干净）
2. **自己写 RPC 客户端，不 import 内部模块** —— 包里的 `rpc-client.js`
   不在 `exports` 里，`pi update` 一次就可能炸。协议适配集中在 `protocol.ts` 一处
3. `nodeIntegration:false` + `contextIsolation:true`

### 分阶段

| 阶段 | 内容 | 交付 | 预估轮次 |
|---|---|---|---|
| **0** | 脚手架 + `protocol.ts` + 最小流式窗口 | 窗口里打字能看到流式回答 | 8–12 |
| **1** | 消息流 + Markdown + 工具卡 + diff + 输入区 + 队列 | **能真用来改代码的客户端** | 15–25 |
| **2** | 多会话 + 项目切换 + 模型/thinking 选择器 + 状态面板 + 扩展 UI 桥接 | 完整多会话客户端 | 15–25 |
| **3** | 原生菜单/托盘/通知/多标签（可选） | 桌面化 | — |
| **4** | 并排 diff、子 agent 可视化、命令面板（可选） | — | — |

**v1 严格停在阶段 2。**

### v1 范围

| ✅ 做 | ❌ 不做 |
|---|---|
| 流式对话 + Markdown + 代码高亮 | 子 agent 可视化 |
| 工具卡片（bash 输出、edit diff） | 并排 diff 编辑器 |
| 多会话（新建/恢复/改名/删除/fork） | 内嵌终端 |
| 模型 & thinking 切换、项目切换 | 插件市场、主题市场 |
| 扩展 UI 桥接（notify/confirm/select/input/editor） | 移动端、远程连接 |
| 状态面板（含 Corrected 峰谷） | 自动更新 |
| 中英文 i18n | 三平台签名 |
| 深色主题 | 浅色主题 |

### 数据归属

| 数据 | 方案 | 理由 |
|---|---|---|
| 会话 | **复用 `~/.pi/agent/sessions/`** | 与 TUI 互通（核心价值） |
| 设置 | 复用 `~/.pi/agent/settings.json` | 同上 |
| 密钥 | 复用 pi 的 auth 存储 | 免二次配置 |
| 桌面端独有 | `~/.pi/agent/desktop.json` | 窗口尺寸、主题、语言 |
| 遥测 | **零遥测** | 写进 SECURITY.md |

⚠️ 代价：耦合 pi 会话格式（现 `version:3`）。需加**格式版本检查**，
不认识的版本降级为只读。

### 扩展兼容性

| 扩展 API | 桌面端 | 实现位置 |
|---|---|---|
| `registerTool` / `registerCommand` / 事件 | ✅ 完全可用 | pi 侧原生 |
| `notify` | ✅ 通知（去重 + 上限 3 条） | `UiBridge.tsx` |
| `confirm` / `select` / `input` / `editor` | ✅ 模态框（**必须应答，否则扩展会卡住**） | `UiDialog` |
| `setStatus` | ✅ 底部状态条 | `StatusBar` |
| `setTitle` | ✅ | `agent.ts` |
| `set_editor_text` | ✅ 塞进输入框 | `Composer` |
| `setWidget` / overlay / `custom` | ❌ TUI 专属 → 写进日志（否则会静默丢失） | `agent.ts` |
| `registerShortcut` | ⚠️ 未映射（桌面端快捷键方案待定） | — |

用户已写的 `left-info-panel.ts`：`panel_todos` 工具和 `/panel` 命令照常工作，
它的 `notify` 会被通知系统接住（**已验证：不去重会刷屏**）。
侧栏由桌面端原生实现，**扩展文件不用改**。

> 扩展的加载方式是 `pi --extension resources/pi/yan-memory.ts` ——
> **不写进 `~/.pi/agent/extensions/`**，所以与用户已有扩展零冲突，也不需要安装步骤。

---

### 8.11 第 12 版的坑（2026-09-12：工具栏 / 用户档案 / 启动器）

| # | 坑 | 教训 |
|---|---|---|
| 1 | **收起面板后开关点不到** | 一条过时规则 `.app.rail-off .rail{opacity:1;pointer-events:auto}` 让透明左栏盖在展开把手上。探针实测 `elementFromPoint` 命中 `rail-brand-btn` 而不是把手。**透明 ≠ 不可交互** —— 隐藏内容要用 `visibility`/`display`，不要只改 opacity |
| 2 | **同一个开关的两个状态用了两个元素** | 收起是 `.rail-stub`（24px 绝对定位），展开是品牌按钮（padding 追随内容）→ 几何对不上（用户报「不对称」）。**开关类 UI 要用同一个元素 + 固定盒子**，只换内容 |
| 3 | **列宽 0 会把把手一起裁掉** | 面板收起宽度必须 ≥「内边距 + 按钮 + 内边距」。左栏取 50（12+26+12），右栏取 40 |
| 4 | **主进程热路径不能异步读盘** | 快捷键处理器用 `getSettings().then()` 才知道当前档，而写设置是「invalidate → 重读 → 写」的异步链 → 连按两下第二下读到旧值，算出同一档（**表现为「按键丢了」**）。改用内存里的当前值 |
| 5 | **探针假设起始状态** | panels/symmetry 假设「起始是展开的」，但前一个场景可能把它留在收起态 → 前后对比整个颠倒 → 全错。**单跑必过、全量才炸**（同一个坑本项目踩过 3 次）。显式置初始状态 |
| 6 | **量几何不等稳定** | 面板收放有 120–160ms 过渡，中途量出的矩形是随机的。改为「连读两次相同才算」 |
| 7 | **引导层晚出现** | 「到了就找关闭按钮，找不到就算了」→ 引导层在数据就绪后才渲染，于是它一直开着盖住界面 → `elementFromPoint` 量到遮罩 → 偶发失败。**轮询等它出现**再关 |
| 8 | **固定时间窗口等 UI** | zoom 场景用「采样 10 秒」，被前一个场景拖慢时最后一下按键还没到就结束 → 假失败。正是本项目明令禁止的模式。改成轮询到条件成立 |
| 9 | **断言精度 vs 浮点缩放** | 界面缩放开着时 `getBoundingClientRect` 返回 25.998 而非 26，`>= 26` 判定失败。**量几何的断言要留容差** |
| 10 | **每个条目 stat 一遍** | 文件树为了拿 size 对每个条目 `stat`，然后才截断到 400 条 —— 家目录上千条 = 上千次系统调用。`readdir(withFileTypes)` 已免费给出目录标志；**先排序、再截断、最后才取 size** |
| 11 | **.cmd 文件里的中文会毁掉整个脚本** | cmd.exe 按系统 OEM 代码页（GBK）读 .cmd，非 ASCII 字节在 `chcp` 生效**之前**就破坏行解析。实测整个 .cmd 一行都没执行成功。**.cmd 保持纯 ASCII**，中文由 node 打印 |
| 12 | **探针失败要留现场** | symmetry 失败过两次而单跑必过 —— 没有现场只能靠重跑碰运气。现在失败时会 dump 面板状态/缩放/打开的浮层/各元素矩形 |

### 8.12 第 13 版的坑（2026-09-12：宽度拖拽 / 分区排序 / 工具库）

| # | 坑 | 教训 |
|---|---|---|
| 1 | **排序静默失效** | 读 section 的 `data-sec`（`rp-queue`）而顺序数组里存的是 `queue` → `indexOf` 得 -1 直接 return。**两套 id 形态混用是最难查的一类 bug**：不报错、不崩，只是什么都不发生。现在用 `data-tool-id` 统一 |
| 2 | **`setPointerCapture` 会抛异常** | 它抛 NotFoundError 时后面的初始化全被跳过 → 拖拽永远不启动。**capture 只是便利，必须包 try**，而且要放在状态更新之后 |
| 3 | **命中区重叠** | 宽度把手（7px，绝对定位）压在分区排序把手左缘 → 拖排序变成拖宽度。两个相邻交互元素必须有**不重叠的命中区**（把手 5px + grip margin-left 6px） |
| 4 | **非法 CSS 值让整条声明失效** | 拖过头产生 `-9439px` → `grid-template-columns` 整条被丢弃 → 那一拖完全没反应，且读回 CSS 变量变 NaN。**范围限制要在产生值的那一侧就夹**，不能只在落盘时夹 |
| 5 | **元素宽度写死 → 拖拽看起来无效** | `.rail{width:300px}` 让「列宽」与「元素宽」脱钩：设置改了、列宽变了，元素还是 300px。**跟随容器宽度用 100%** |
| 6 | **声明了却没读的字段** | `CATALOG[].envVar` 写了但 `listAuthProviders` 从没读过 → 用环境变量配好 key 的用户被判「未配置」。**加了字段要搜一遍谁在读它** |
| 7 | **重构时丢失「空则不渲染」** | 改成注册表渲染后，外层容器变成无条件渲染 → 空区块又出现了。**行为约定要跟着数据一起搬家**（现在 isEmpty 声明在注册表里） |
| 8 | **场景之间共享状态文件** | 所有场景共用一个 desktop.json，某场景改了 cwd → 后面的 atPath 拿到家目录，断言全落空。**每个场景开跑前重置为已知状态** |
| 9 | **探针抢焦点** | 连续开十几个窗口，每次都把焦点从用户的另一个桌面抢走（用户明确要求别干扰）。用 `showInactive()`；代价是未聚焦窗口被 Chromium 降频，必须同时关掉后台节流（`disable-background-timer-throttling` 等，只在有 YAN_PROBE 时） |
| 10 | **断言绑「碰巧的数据」** | atPath 断言「目录带尾斜杠」用的是 `src/main/`（那里只有 .ts 文件、没有子目录），它一直靠 cwd=家目录时 `Desktop/` 里碰巧有子目录才通过。**断言要绑必然成立的性质** |
| 11 | **固定 sleep 等首帧** | virtual 场景 `await sleep(50)` 后量 DOM：单跑够（62ms），全量后半段首帧还没提交 → 量到 0 条，后面全挂。与第 8.9 条同源 |

### 8.13 第 14 版的坑（2026-09-12：拖拽自动收起 / 工具库拖拽 / 高度 / 压缩提示）

| # | 坑 | 教训 |
|---|---|---|
| 1 | **zustand 选择器里造函数 = 无限重渲染** | `useStore((s) => () => s.setRailPinned(false))` 每帧返回新函数，v5 用 `Object.is` 比较 → 整页空白。**选择器只能返回已有引用或原始值** |
| 2 | **新功能会推翻老测试的假设** | 加了「拖过窄就收起」之后，resize 场景的「极限左拖应停在最小宽」不再成立，而它留下的收起态又把后面的 panels 场景搞挂（按钮被 flex 挤成 14px，看着像「按钮变小了」）。**加新行为必须回头改依赖旧行为的断言** |
| 3 | **收起宽度写死的元素** | `.rightstub { width: 40px }` 让「收起」失效（列宽 8px 但元素占 40px）。**跟随容器一律用 100%** |
| 4 | **overflow:hidden 会把悬停按钮裁掉** | 收起态的展开按钮比那条 8px 的缝宽，被 .rail-slot 裁掉 → 按钮存在但看不见也点不到。**收起态要放行溢出** |
| 5 | **跨组件拖拽要监听 window** | 指针必然离开源元素（从库拖到工具栏），只听元素自己的事件收不到后续坐标 —— 实时预览就无从谈起 |
| 6 | **浮动标签要 pointer-events:none** | 否则它一直待在指针下方，`elementFromPoint` 永远命中自己，落点永远算不出来 |
| 7 | **固定 sleep 等面板渲染** | 设置面板渲染 19 行接入方式，全量跑的最后可能 >900ms；固定等待量到 `tabs=[]` 就一路挂。**所有等 UI 的地方都改轮询**（本轮又修 4 处） |
| 8 | **注入的假数据会被真实推送覆盖** | virtual 场景注入 240 条后，前面场景切会话引入的 `sync` 推送把它冲掉 → 页面 0 条。**注入类探针要能重注入（竞态在测试里也存在）** |
| 9 | **编辑脚本吞引号/吞文件尾** | 我用 node 脚本做批量替换时，引号被吃掉（`typeof v === number`）、JSDoc 里的 `# 交接文档 · pi desktop

> **给新会话的第一个指令**（直接复制粘贴的版本在 [`NEXT-SESSION.md`](NEXT-SESSION.md)）：
> `读 %USERPROFILE%\Desktop\pi-desktop\docs\dev\HANDOFF.md，然后继续。`
>
> 最后更新：2026-09-12（**第 14 版：宽度拖拽自动收起 / 工具库拖拽 / 分区高度 / 自动压缩提示 / 隐藏项开关 / 命令自动管理**）
>
> ⚠️ **第 4 版开头的「方向变更」先读** —— 代码按「砚 · 个人 agent」实现（不是编码工具）。

---

## 0. 一句话目标

做一个 **个人 agent 的桌面端**（产品名暂定「**砚**」）：Electron 外壳 + React 渲染层，
中文界面 + 完整 i18n 通道，**类终端调性**（全等宽、无投影、颜色即语义），可开源分发。

**记忆是第一公民**：界面刻意区分「已确认的记忆」（来源：你）与「我的印象」（来源：我，未确认），
「身份」区块只读（agent 不能改自己）。这是本产品区别于普通 agent 客户端的地方。

**当前阶段：阶段 0 已完成 —— UI 骨架已跑通（2026-09-11）。**

| 阶段 | 状态 |
|---|---|
| 设计稿 | ✅ v0.2（已改名「砚」，见下方 ⚠️ 方向变更） |
| 技术栈 | ✅ **Electron + Vite + React 19 + TypeScript**（已拍板） |
| 阶段 0：UI 骨架 | ✅ 完成 |
| 阶段 1：接 RPC | ✅ 完成 —— 应用真的连上了 pi，可用 |
| **阶段 2：功能补完** | ✅ **完成** —— 图片 / 斜杠命令 / !bash / fork·clone·导出·删除 / 模型选择器 / 开关 / 虚拟化 |
| **阶段 2.5：桌面化** | ✅ **完成**（2026-09-12）—— **内置 pi** / 工具栏（文件树·日志）/ 用户档案 / 界面缩放（DPI 取整）/ 启动器 / 面板开关对称 |
| **阶段 2.6：可调与可整理** | ✅ **完成**（2026-09-12）—— 左右栏**宽度拖拽** / 分区**拖拽排序** / **工具库**（收进库·取回）/ 引导检查本地已有 key（含环境变量） |
| **阶段 2.7：细节打磨** | ✅ **完成**（2026-09-12）—— 拖过窄**自动收起** / 收起态改**细缝** / 工具库**拖到工具栏**（带实时预览）/ 分区**高度可调** / **自动压缩触发点**可视 / 隐藏项开关 / 命令**自动管理+填充** |
| 阶段 3：打包分发 | ⏭ 下一步（唯一阻塞「给别人用」的）—— 内置运行时已就位，只差 electron-builder |

---

## ⚠️⚠️ 方向变更：从「编码工具」到「个人 agent」（**需用户确认**）

`docs/design/prototype.html` 在本次会话前已被改写为「**砚**」，
`docs/design/preview/` 里的 `yan-1440x900.png` / `yan-1920x960.png` 也是新版。
它与本文件前半部分（第 1–3 版）描述的**编码客户端**不是同一条路线：

| | 编码客户端（HANDOFF 前 3 版） | **砚（当前 prototype.html + 已实现代码）** |
|---|---|---|
| 右栏 | 状态面板（上下文/工作区/文件树/任务/计费） | **记忆**（身份 / 关于你 / 我的印象 / 人 / 项目 / 状态） |
| 中栏 | 三栏 = 会话列表 / 消息流 / 状态 | 三栏 = 话题+会话 / 对话+连续性 / 记忆 |
| 核心创新 | 工具卡智能展开、改动回滚 | **记忆的认识论可见化**（已确认 vs 我的印象） |
| 无 | diff、文件树、测试状态、工作区 | 这些概念**全部去掉** |

**已实现的应用代码按「砚」做。**
本文件里关于 diff / 文件树 / 工具卡 / 改动回滚的章节（§1.2、§1.3、§3.4、§9 v1 范围）
**当前不适用**，保留作历史参考 —— 需要用户确认是**彻底转向**还是**两条线并行**。

> 已明确不受影响的部分：设计令牌（§1.1 字体 / §2 字号）、图标、i18n 机制、
> 三栏布局骨架、RPC 事实（§11）、上下文窗口修复（§5.5）、成本约定（§6）。

---

## 1. 已确认的需求（用户已拍板，不要再问）

| # | 需求 | 状态 |
|---|---|---|
| 1 | 界面中文，但**保留完整多语言接口** | ✅ 已定 |
| 2 | 视觉：类 Codex 的现代化圆角，但 **UI 贴近终端风格** | ✅ 设计稿 v0.1 |
| 3 | **可能会分发**这款软件 | ✅ 已定 |
| 4 | 要**符合 GitHub 规范**（README / LICENSE / CI / Issue 模板等） | ✅ 已定 |
| 5 | 技术栈**暂缓决定** | ✅ 已定 **Electron**（见下 §2 #1） |
| 6 | **字体：Maple Mono CN**，基准 **12.5px**（代码块 12px） | ✅ 已定并实测，见 §1.1 |
| 7 | **图标：reicon**（MIT） | ✅ 已抓取 92 个 |
| 8 | **改动审阅：只做回滚**，不做接受/拒绝 | ✅ 已定，见 §1.2 |
| 9 | 左栏做**可折叠的任务**列表 | ✅ 已定 |
| 10 | 右栏**中下部作为文件树** | ✅ 已定 |
| 11 | 工具卡片**智能展开** | ✅ 已定，见 §1.3 |
| 12 | 只做**深色**，但保留浅色方案 | ✅ 已定 |
| 13 | 状态面板卡片**可调顺序**（拖拽） | ✅ 已定 |

### 1.1 字体（已实测）

**Maple Mono CN**，拉丁列宽 `0.600em`，汉字 `1.200em`，严格 **1:2**。

必须内嵌，**不能靠系统回退** —— 常见英文字体的 ASCII 宽不是整数倍步进
（Cascadia 1.707 / Consolas 1.819 / 雅黑 1.023），凑不出 2.000。

**基准字号 12.5px**（v0.1 曾写 12px，v0.2 上调）。决定性指标不是观感，
而是**汉字格宽是否为整数像素**：

| 字号 | 拉丁列宽 | 汉字格宽 | 整数像素 | 栅格最大偏差 |
|---|---|---|---|---|
| 12px | 7.200px | 14.400px | ✗ | 0.234px |
| **12.5px** | **7.500px** | **15.000px** | **✓** | **0.000px** |

只有 12.5px 落回整数像素栅格（DPR 1 与 DPR 2 都成立），
汉字笔画不发虚、字与字之间无细微分差。代价是 780px 每行少 4 列（108 → 104）。
代码块 / diff 仍用 12px。

字号阶：`10 / 11 / 12 / 12.5 / 14`，行高 `1.45 / 1.65`。

详见 [`docs/design/DESIGN.md`](docs/design/DESIGN.md) §2.1–2.2，
对比图 [`docs/design/preview/fontsize-compare.png`](docs/design/preview/fontsize-compare.png)。

### 1.2 改动审阅：只做回滚（用户已拍板）

pi 设计上**无权限弹窗、写入立即落盘**（`docs/usage.md:309`）。
因此不做"接受/拒绝"，而是：

1. 编辑照常立即写入
2. 桌面端从 **session JSONL 读回 `old` 内容**，提供「↺ 回滚此改动」
3. **"接受"是隐式的**
4. 加一条常驻汇总条：`3 处改动待确认 · [全部回滚] [查看汇总 diff]`

会话 JSONL 里 edit 工具的参数含 `oldText` / `newText`，足够复原。

### 1.3 工具卡片展开规则

| 情况 | 默认 |
|---|---|
| 失败（非 0 退出 / 报错） | **展开** |
| 含 diff（edit / write） | **展开** |
| 进行中 | **展开** |
| 成功的长输出（read / bash） | **折叠** |
| 成功的短输出 | 折叠 |

用户可手动展开，手动状态优先于默认。

---

## 2. 待用户决策（**开工前必须问清**）

| # | 问题 | 我的建议 | 状态 |
|---|---|---|---|
| 1 | ~~**技术栈**：Electron / Tauri / 纯浏览器~~ | ✅ **已定 Electron**（2026-09-11）。UI 层用 Vite + React 19 + TypeScript + 原生 CSS 变量（**不用 Tailwind，不用组件库** —— 设计稿是反设计系统的，见下方说明） | ✅ 已定 |
| 2 | **项目名** | UI 概念名已是「**砚**」，包名暂用 `yan-desktop` | ⏸ 待确认 |
| 3 | ~~用户说的"Codex"指哪个~~ | 已按网页版"干净卡片"风格起草 | ✅ 已定 |
| 4 | **开源还是私有仓库** | 公开（GitHub Actions 对公开仓库免费无限分钟） | ⏸ 待定 |
| 5 | **是否购买代码签名证书** | 仅小范围分享 → 不签，文档写明绕过方法 | ⏸ 待定 |
| 6 | **目标平台** | 先 Windows，后续再加 | ⏸ 待定 |
| 7 | ~~语言范围~~ | zh-CN + en-US | ✅ 已定 |
| 8 | ~~浅色主题~~ | v1 只做深色，令牌保留 | ✅ 已定 |
| 9 | 注释语言 | 注释中文 / 类型英文 / 对外文档中英双语 | ✅ 已按此实现 |
| 10 | **字体子集方案** | `cn-font-split` 切 woff2（~2–3MB）；**当前是 18.6MB 全量 TTF** | ⏸ 待定 |

> 真正阻塞开工的需求已全部解决。剩下一个**方向**问题（砚 vs 编码客户端，见开头）
> 和一个**产品名**问题，都不阻塞阶段 1（接 RPC 只改 store 层）。

### UI 技术选型（已定，附理由）

| 层 | 选型 | 为什么 |
|---|---|---|
| 渲染框架 | React 19 | 生态厚（markdown / 虚拟列表 / 高亮 / diff）；RPC 事件流用 store 接很顺 |
| 构建 | electron-vite 5 + Vite 7 | Electron 两边一套配置，HMR 快 |
| 样式 | **原生 CSS + CSS 变量** | 见下 |
| i18n | **自写 `t()` + 从 zh-CN.json 推导类型** | DESIGN §6 已这么规定；漏翻译编译报错 |
| 图标 | reicon sprite 内联 | 已生成，`npm run icons` 可重跑 |

**为什么不用 Tailwind / 组件库（Ant Design / MUI / shadcn）：**

1. DESIGN §4 明令禁止投影与渐变 —— 组件库的整个层级语言建立在它们上面，用了等于每写一个组件打一次架
2. 视觉真源是**自己实测出来的令牌**（12.5px / 0.6em 列宽 / `--fact` vs `--warn`），不是组件库的 theme token
3. **最大复用点是 `prototype.html` 里的 CSS** —— 已逐像素调好，抽成 `tokens.css` + `app.css` 后组件只负责加 class
4. `check.mjs` 靠正则扫 CSS/HTML 做禁止项检查，构建期类名生成会削弱这套零成本校验

**阶段 2+ 可能引入的依赖**（用到再加，不提前装）：`zustand`（流式状态）、`virtua`（消息虚拟化）、
`react-markdown` + `shiki`（正文渲染）、`@base-ui-components/react`（模态框无障碍）。

---

## 3. 当前产出

### 3.0 应用代码（**阶段 1 已完成 —— 可用**，2026-09-11）

```
%USERPROFILE%\Desktop\pi-desktop\
├── src\
│   ├── main\
│   │   ├── index.ts         窗口 + IPC + 生命周期（含稳定退出）
│   │   ├── protocol.ts  ⭐ 手写 pi RPC 客户端（JSONL / 请求关联 / 扩展 UI）
│   │   │                       并负责**定位 pi**（内置运行时 → 全局 → PATH）
│   │   ├── agent.ts     ⭐ 协议 → UI 归一化（唯一认识 pi 协议的地方）
│   │   ├── normalize.ts     归一化的纯函数部分（拆出来供 session-reader 复用）
│   │   ├── session-reader.ts 直读 JSONL（大会话打开 486ms，不走 pi RPC）
│   │   ├── memory.ts        记忆存储 + 认识论规则 + soul.md 只读
│   │   ├── sessions.ts      会话列表（只读扫描 sessions/*.jsonl）
│   │   ├── credentials.ts    模型接入（读写 pi 的 auth.json）
│   │   ├── title.ts         用独立 RPC 进程总结会话标题
│   │   └── settings.ts      桌面端设置（不碰 pi 的 settings.json）
│   ├── preload\index.ts     contextBridge 白名单（形态由 YanBridge 约束）
│   ├── shared\ipc.ts        共享类型 + MainPush（已归一化的 UI 补丁）
│   └── renderer\src\
│       ├── state\store.ts   zustand：只负责套用 MainPush
│       ├── components\        TitleBar Rail Continuity Message Composer
│       │                    MemoryPanel UiBridge EmptyStream Onboarding AuthTab …
│       ├── styles\          tokens app stage1 stage2 redesign settings electron highlight
│       └── i18n\            中英双语，类型安全
├── resources\
│   ├── pi\yan-memory.ts     ⭐ 注入给 pi 的记忆扩展
│   └── pi-runtime\          ⭐ 内置 pi 运行时（生成物，不入库；npm run vendor:pi）
├── scripts\                 项目工具（纯应用侧）
│   ├── probe-pi.mjs         单独验证「pi 能否被找到并启动」
│   ├── probe\*.js           在真实应用里跑的 DOM/交互断言（live/memory/sessions/e2e…）
│   ├── test-live.mjs        跑上面这些场景
│   ├── test-unit.mjs        纯逻辑单测（吸附 test-turns.mjs）
│   ├── test-turns.mjs       回合分组 / 段落拆分 / 缓存命中率
│   ├── lint-css.mjs         拦裸 1fr（这个坑出现过 3 次）
│   ├── vendor-pi.mjs        抽取内置 pi 运行时
│   └── shot.mjs             截图
├── docs\
│   ├── dev\                 HANDOFF.md / NEXT-SESSION.md（开发过程文档）
│   └── design\              设计稿与设计工具（prototype.html / check.mjs / icons…）
└── 根目录                    README.md / LICENSE / package.json / tsconfig* / electron.vite.config.ts
```

**验证结果（实测）：**

| 项 | 结果 |
|---|---|
| `npm run typecheck` | ✅ 两个 tsconfig 均无错 |
| `npm run build` | ✅ renderer 1.49MB + 字体 18.6MB |
| `npm run test:live -- live memory sessions` | ✅ 全绿，不烧 token |
| `npm run test:live -- e2e` | ✅ 真发消息、真工具调用全通过 |
| `npm run probe-pi` | ✅ 找到 pi（AppData\Roaming\npm）、contextWindow = 1000000 |

真实能力已验证：流式对话、markdown + 语法高亮、bash/edit 工具卡（含彩色 diff）、
工具结果回执、多会话切换、新建会话、记忆确认流程、审阅条。

截图：`docs/design/preview/yan-live-1440x900.png`（真实对话）、
`yan-live-diff.png`（edit 的 diff）、`yan-live-tools.png`

### 3.1 设计稿

```
%USERPROFILE%\Desktop\pi-desktop\
├── docs\dev\                        ← 开发过程文档（HANDOFF / NEXT-SESSION 已移到这里）
│   ├── HANDOFF.md                    ← 交接文档（本文件）
│   └── NEXT-SESSION.md               ← 新会话开场指令（可直接复制）
└── docs\design\
    ├── prototype.html               ← 可交互设计稿 v0.2（112KB，浏览器直接打开）
    ├── DESIGN.md                    ← 设计规范（令牌真源，已到 v0.2）
    ├── check.mjs                    ← 静态自检（9 类，零成本）
    ├── measure-design.mjs           ← 量设计稿溢出（npm run measure:design）
    ├── embed-icons.mjs               ← 把用到的图标 symbol 子集内联进设计稿
    ├── build-icons.mjs              ← 图标抓取 + 生成（可重跑，从 reicon 拉）
    ├── extract-icons.mjs            ← 设计稿 sprite → renderer 的 TS 模块（npm run icons）
    ├── archive\                     ← 旧设计稿（v0.2 编码版）与一次性修复脚本
    ├── icons\                       ← reicon 图标集（已生成）
    │   ├── reicon.svg               ← 239KB 完整 sprite，166 个 symbol
    │   ├── icons.ts / icons.json    ← 名称 ↔ symbol id 映射
    │   ├── preview.html             ← 92 个图标总览（已视觉验证）
    │   └── preview.png
    ├── font-test\                   ← 字体调研（已定：Maple Mono CN）
    │   ├── README.md                ← 选型结论 + 实测表
    │   ├── fonts.html               ← 三字体逐字测宽对比
    │   ├── maple-size.html          ← Maple 四档字号对比
    │   ├── compare.png / msize.png
    │   └── MapleMono-CN-Regular.ttf ← 18MB，已 gitignore
    └── preview\                     ← 渲染截图（⚠️ 文件名后来随 UI 重做换过，以实际为准）
        ├── ui-live.png / ui-settings.png   ← README 用的两张
        ├── yan-app-*.png                   ← 设计稿渲染
        ├── yan-live-*.png                  ← 真实对话截图
        └── redesign-before/after.png       ← 一次 UI 重做的前后对比
```

**设计稿状态：v0.2 已完成。用户 8 条意见全部落实，`check.mjs` 全绿，已逐张目视校验。**

### v0.2 相对 v0.1 的改动（全部已完成）

| # | 改动 | 落在哪里 |
|---|---|---|
| 1 | 字体换 Maple Mono CN，基准 **12.5px**（代码 12px） | 令牌 §0；DESIGN §2.1–2.2 |
| 2 | 图标全换 reicon sprite（25 个内联子集），删光 `◐ ↺ ✓ ✗` | `embed-icons.mjs` + ICON-SPRITE 标记 |
| 3 | 上下文窗口 `128,000` → `1,000,000`（29%） | 右栏上下文卡 |
| 4 | 左栏：时间分组可折叠 + 会话内嵌可折叠任务 + 整栏可折叠 | 左栏 + 标题栏最左钮 |
| 5 | 右栏中下部：文件树（目录折叠 + 改动 ±N） | 新增 §3.4 区 |
| 6 | 工具卡智能展开（手动优先，折叠留摘要行） | §3.1 规则 + JS |
| 7 | 状态面板分区可拖拽排序（Alt+↑↓ / localStorage / 重置） | §3.3 |
| 8 | 常驻汇总条「3 处改动待确认 · [全部回滚] [查看汇总 diff]」 | 输入区上方 |
| 9 | 每条 edit 卡加「回滚此改动」 | 编辑卡操作栏 |

### v0.2 已实现的内容（设计稿是活文档，不只是静态图）

- 三栏布局：会话列表(216) / 消息流(780 上限) / 状态面板(320)，左右两栏可折叠
- 消息流：用户气泡、助手正文、思考折叠块、工具卡片（read / edit+diff / bash 成功 / bash 失败 / 进行中）、错误条
- **工具卡智能展开**：失败/diff/进行中展开，成功长输出折叠，折叠时留一行摘要；点过一次就转手动（`data-manual`），不再被自动规则推翻
- **状态面板六区**：会话 / 上下文窗口 / 工作区 / 文件树 / 任务 / 计费，可拖拽排序 + `Alt+↑↓`，`localStorage` 持久化，「重置顺序」还原
- 文件树：目录折叠（可见性整体重算，非增量）、选中态、改动文件带 `+N −N`
- 汇总条：`3 处改动待确认` + 全部回滚 + 查看汇总 diff（接受是隐式的）
- i18n：`LOCALES` 含 zh-CN + en-US 各 **55 键**，右上按钮实时切换（含 `title` 属性）
- 深浅主题令牌（浅色未打磨）
- 峰谷指示器：**实时计算**，用 CommandCode 真实时段

---

## 4. 关键事实（已核实，勿重复调查）

### 环境

| 项 | 值 |
|---|---|
| Node | v24.19.0 |
| npm | 11.17.0 |
| Windows | 10.0.26200（Win11 24H2+） |
| git | 2.55.0 ✅，但 **`user.name` / `user.email` 未配置** |
| gh CLI | ❌ 未安装（分发需 `winget install GitHub.cli`） |
| Rust | 1.96.0（Tauri 可行） |
| WebView2 | 149.0.4022.80（Tauri 可行） |
| Edge | 151.0.4129.93（可用于 headless 截图） |
| 磁盘 | C: 剩 60G |

### pi 包

| 项 | 值 |
|---|---|
| 版本 | 0.85.1 |
| 路径 | `%USERPROFILE%\AppData\Roaming\npm\node_modules\@earendil-works\pi-coding-agent` |
| 许可证 | **MIT** → 可以打包分发，保留版权声明 |
| 体积 | 424 MB（`node_modules` 401MB，其中 `@esbuild` **284MB**） |
| `dist/bundle` | **7.7 MB**，但实测**仍需外部依赖**（报缺 `@earendil-works/chord`） |
| 运行时依赖 | `@earendil-works/{chord,pi-agent-core,pi-ai,pi-telemetry,pi-tui}` 共 15MB + openai/google 等 |
| 无独立二进制 | 只有 `dist/bundle/cli.js`（需 Node） |

**打包 pi 是待解问题**：`dist/bundle` 不自包含，`@esbuild` 占 284MB。
估算裁剪后可到 ~60MB。分发前必须先解决。

### 分发路径（三选一，未定）

| 方案 | 用户门槛 | 体积 | 复杂度 |
|---|---|---|---|
| A. 要求用户自己 `npm i -g pi` | 高 | ~0 | 低 |
| B. 打包进应用 | 低 | +60~100MB | 中 |
| C. 混合：检测到就用，没有用内置 | 低 | +60~100MB | 中高 |

建议：开发期用 A，分发时做 C。

### 代码签名（若分发需要花钱）

| 平台 | 不签名的后果 | 成本 |
|---|---|---|
| Windows | 每次打开弹 SmartScreen 警告 | $100–400/年 |
| macOS | Gatekeeper 拦截 | $99/年 |

---

## 5. ⚠️ 已知更正（重要，别再搞错）

### 5.1 CommandCode 峰谷时段 ≠ DeepSeek 官方时段

**用户的实际计费提供方是 CommandCode，不是 DeepSeek。**

| | 峰时窗口 |
|---|---|
| DeepSeek 官方（**错误**，早期面板用的） | UTC 16:30–00:30 |
| **CommandCode 实际（正确）** | **UTC 01:00–04:00 与 06:00–10:00，仅周一–周五** |

换算：

| 时区 | 峰时（全价） |
|---|---|
| UTC | 01:00–04:00、06:00–10:00 |
| 北京 UTC+8 | 09:00–12:00、14:00–18:00 |
| **悉尼 UTC+10** | **11:00–14:00、16:00–20:00** |
| 悉尼夏令时 UTC+11 | 12:00–15:00、17:00–21:00 |

**折扣一律 −50%**（早期面板写的 −75% 是错的）。

**待办**：`~/.pi/agent/extensions/left-info-panel.ts` 里的峰谷逻辑仍是错的，
需要改成上表。设计稿里的 JS 用的是**正确**时段。

### 5.2 看图：主模型现在能直接看（用户已开启）

**❗重要更新（2026-09-11）：用户已经给主模型开启了图片输入。**
在会话里直接 `read` 一张 PNG 就能看到图，**不需要再调 see2.mjs / 视觉模型**。
这省掉了整个外部链路、也不再受 vision 模型上游挂掉的影响。

如果哪天图片又看不了了，再回到下面这套口径（低优先级的备案）：

| 模型 | 5h / 周 / 月 请求数 | 输入 / 输出（谷时 $/M） |
|---|---|---|
| `commandcode/deepseek/deepseek-v4-flash-vision-exp` | 6,080 / 15,200 / 30,400 | $0.22 / $0.66 |
| `commandcode/gpt-5.6-sol` | 414 / 1,040 / 2,070 | $5.00 / $30.00 |

> 这个便宜的 vision 模型当时上游挂过（`No available providers match the 'only' filter: deepseek`），
> 要显式指定 `commandcode/gpt-5.6-sol`。在它能用之前，先用主模型直接看图。

### 5.3 截图成本极低

实测：一张 1440×900 截图 → **277~337 输入 token**。

| 项 | 成本 |
|---|---|
| 一次截图审查 | **≈ $0.0002** |

**图片不是成本瓶颈，我自己（每轮重发全部上下文）才是。** 优化轮次，不优化截图次数。

### 5.4 套餐额度

GOAT 计划 $10/月 = $70 额度：$14 / 5小时，$35 / 周，$70 / 月。

### 5.5 ⚠️ 模型的 128k 上下文窗口是假的，已修复

**pi 给所有模型的 `contextWindow` 都是硬编码兜底的 128000，不是真实值。**

根因链路：

```
1. CommandCode /models API 返回        context_length: 1000000
2. pi 生成 ~/.pi/agent/models.json 时   只写 id / name / reasoning / input
                                        ❌ 丢掉 context_length
3. provider-composer.js:72 兜底         contextWindow ?? 128000   ← 假的
                                        maxTokens     ?? 16384
```

**实测验证**（直接调 CommandCode API，绕过 pi）：

| | 值 |
|---|---|
| pi 给的 | 128,000 |
| **实测服务端接受** | **896,758 tokens**（HTTP 200，发了 5,060,000 字符） |
| API 声明 | 1,000,000 |

**已修复** —— 用 `~/.pi/agent/.dev/sync-model-context.mjs` 按 API 声明回填全部 69 个模型：

```bash
node ~/.pi/agent/.dev/sync-model-context.mjs            # 预演
node ~/.pi/agent/.dev/sync-model-context.mjs --write    # 写入（自动备份）
node ~/.pi/agent/.dev/sync-model-context.mjs --write --verify
```

**`pi update` 后 `models.json` 会被重写 → 需重跑此脚本。**

回填后的窗口分布：

| contextWindow | 模型数 | 代表 |
|---|---|---|
| 1,050,000 | 3 | gpt-5.6-sol / terra / luna |
| 1,048,576 | 9 | gemini-3.7-flash、muse-spark |
| **1,000,000** | **34** | **deepseek-v4.1-flash、claude-sonnet-5、Qwen3.8-Max** |
| 500,000 | 2 | grok-4.5 / 4.6 |
| 400,000 | 4 | gpt-5.5 / 5.4 / 5.3-codex |
| 256,000–262,144 | 10 | Kimi-K2.7-Code、hy3 |
| 200,000 | 7 | GLM-5.1、MiniMax-M2.5 |

**连带影响：压缩阈值从 111,616 → 983,616 tokens**
（`compaction.js:163`：`contextTokens > contextWindow - reserveTokens`，reserve 默认 16384）

长任务不再每 11 万 token 被压缩一次，上下文能保住 8.8 倍。

**未做**：`maxTokens` 保持 pi 兜底 16384。没实测过输出上限，猜大了有的 API 会返 400。

**风险窗口**：声称 1M，实测到 896k，中间 10% 未验证。若在该区间报错，把 `contextWindow` 下调到 950000。

> 成对提醒：窗口变大不等于免费。长上下文每轮重发 token 更多，
> 但 DeepSeek 前缀缓存读只要 **$0.007/M**（输入的 1/21）。
> **保持会话前缀稳定**（别频繁改系统提示、别来回切模型）能让缓存持续命中。

---

## 6. 成本控制约定（用户明确在意额度）

| 策略 | 做法 |
|---|---|
| **少轮次** | 一次写完整文件，不做十次小修补。同样功能轮次差 3–5 倍 |
| **存档** | 本文件就是存档。上下文被压缩后读它恢复，不要重扫代码库 |
| **验证分级** | ①免费：`check.mjs`、类型检查、DOM 断言 → ②极低：截图（$0.0002）→ ③避免：用视觉模型做日常调试 |
| **挑时段** | 悉尼谷时：工作日 20:00–次日 11:00、14:00–16:00，周末全天。峰时全价 |
| **主力模型** | 继续用 `deepseek/deepseek-v4.1-flash`（30,800 请求/5h，套餐内最高）。仅截图审查那轮切 vision 模型 |

---

## 7. 已验证的自检工具链

### 静态检查（零成本，每次都跑）

```bash
node %USERPROFILE%\Desktop\pi-desktop\docs\design\check.mjs <设计稿路径>
```

### 截图

```bash
EDGE="/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
"$EDGE" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --screenshot="out.png" --window-size=1440,900 "file:///C:/path/to/page.html"
```

⚠️ `file://` 页面**不能加载同目录的图片子资源**（被 Chromium 拦截）。
需要裁剪/放大时，用 `%TEMP%\mkhtml.mjs` 把 PNG 转成 **data URI** 再嵌入：

```bash
node %TEMP%\mkhtml.mjs <源图> <输出html> <源宽> <缩放> <裁剪x> <裁剪y>
```

### 视觉校验

**首选：直接 `read` 那张 PNG。** 用户已给主模型开启了图片输入，
我直接看图就行（零额外成本、无需另调模型）。本会话的所有视觉校验都是这么做的。

**备案（仅在主模型看不了图时）：**

```bash
node %TEMP%\see2.mjs <图片> "<问题>" commandcode/gpt-5.6-sol
```

> ⚠️ 默认的 `deepseek/deepseek-v4-flash-vision-exp` 上游挂过
> （`No available providers match the 'only' filter: deepseek`），
> 要显式传 `commandcode/gpt-5.6-sol`（~$0.02/次，贵 ~20 倍但可用）。

已固化的经验：

- 加 `--tools ""` 禁用工具，否则它会试图自己去读文件而不看图
- 提问要**具体、可证伪**（"进度条左右是否与内边距对齐"），不要问"好不好看"
- 它会犯错，也会把非问题说成问题（已遇到 4 次误报）。
  **收到反馈后先回代码验证，再改**
- 明确告诉它"不确定就说不确定"

#### 比截图更可靠的：DOM 断言

本会话验证 v0.2 的交互时，**截图只用于看静态布局**，真正的对错靠 DOM 断言：

```bash
EDGE=".../msedge.exe"
"$EDGE" --headless --dump-dom "file:///.../prototype.html" > dom.html
grep -o 'class="tool[a-z ]*"' dom.html      # 智能展开是否有生效
```

要测交互（拖拽排序 / localStorage / 折叠），就**注入一段测试脚本、把结果写进 `document.title`**，
再 `--dump-dom` 读出来。用 `grep` 找 `<pre>` 里的中文输出会被转义坑到，`title` 不会。

> ⚠️ 两个坑：
> 1. 注入的测试脚本不要用 `const` 重复声明主脚本已有的变量（`panel` / `tree` 等），
>    整个 script 会 SyntaxError **且 `window.onerror` 捕不到**（解析期错误）。
>    包一层 IIFE。
> 2. git-bash 的 `/tmp` ≠ `C:/tmp`。用 `file:///C:/tmp/x.html` 会得到 Edge 的 404 页，
>    看起来像「脚本没跑」。一律用 `%TEMP%` 的真实路径。

### 其他临时工具（在 `%TEMP%`）

| 脚本 | 用途 |
|---|---|
| `see2.mjs` | 视觉校验（见上） |
| `crop.mjs` | 截图局部放大再截（`node crop.mjs <src> <out.html> <srcW> <scale> <x> <y> <vw> <vh>`）|
| `imgtok.mjs` | 测图片 token 成本 |
| `build-icons.mjs` | 图标构建（**已归入项目**：`docs/design/build-icons.mjs`）|
| `parse-reicon.mjs` | 解析 reicon 清单 |

---

## 8. 已发现的设计陷阱（别重犯）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 用 `direction:rtl` 截断 Windows 路径 | 反斜杠触发 bidi 重排，路径乱序 | 右对齐 + `overflow-wrap:anywhere` 让路径换行 |
| 中栏不加 `max-width` | 1920px 下行长失控 | `--w-stream:780px` + `margin:0 auto` |
| 上下文进度条用绿色 | 与"成功/新增"语义冲突，一栏出现 4 种强调色 | 用 `--accent` 蓝 |
| 状态面板先放用量/花费 | 读起来像账单后台而非编码工具 | 代码上下文（改动/检查）在上，计费沉底 |
| `--fg-mute` 用 `#5c5c66` | 对比度仅 2.8:1，小字看不清 | `#82828f`（4.9:1，过 AA） |
| 增删统计 `+14 −2` 用单列 | `+N` 数字纵向不对齐 | 拆成两个定宽 grid 列，无值留空占位 |
| 靠系统字体回退凑栅格 | 汉字≠2×ASCII，表格/边框/diff 全错位 | **内嵌字体**，选 0.5em 或 0.6em 步进的 |
| 图标 sprite 内部 id 冲突 | `<clipPath>` 串位，裁剪到别的图标 | 构建时给内部 id 加图标名前缀 |
| 解 reicon 的 `@preview` base64 | 拿到的是硬编码 `#e4e4e7`，深色下变死白 | 取模块里的 `O`/`F` 字段（`currentColor`）|
| `calc(var(--d) * …)` 做树缩进 | HTML 属性**不会**生成同名自定义属性，`var(--d)` 为空 → 整条 `padding` **静默失效** | 用属性选择器 `.tnode[data-d="2"]{…}` |
| `file://` 外部引 sprite | Chromium 拦截跨文档 `<use href="x.svg#id">`，图标全空 | 构建期内联子集（`embed-icons.mjs`） |
| `file://` 用 localStorage | opaque origin → 抛异常，整段排序逻辑中断 | 读写包 `try/catch`，降级为「仅本次会话有效」 |
| `@font-face` 路径挂在子目录页 | 相对路径是相对 **HTML 所在目录**，字体静默不加载 | 测试页放 `docs/design/`，别放 `preview/` |
| 字号取 12px（非整数像素格） | 汉字格 14.4px，逐字累积 0.234px 抖动 | 取 **12.5px**（汉字格 15.00px 整） |
| 树折叠用增量 add/remove `hidden` | 嵌套折叠时残留错状态 | 遍历一次、按祖先展开态**整体重算** |
| 注入测试脚本重复 `const` 声明 | 整段 SyntaxError，且 `onerror` 捕不到 | 包一层 IIFE |
| 用 git-bash 的 `/tmp` 当 `C:/tmp` | `file:///C:/tmp/…` 得到 Edge 404 页，误判成「脚本没跑」 | 一律用 `%TEMP%` 的真实路径 |
| `--dump-dom` 去 `grep` 中文 `<pre>` 输出 | HTML 转义/编码把结果弄花，看不到内容 | 把结果写进 `document.title` 再抓 |

### 阶段 0 新踩的坑（实现时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 入口模块用 top-level `await app.whenReady()` | **ESM 下死锁** —— Electron 要等整个模块图求值完才发 ready，而 ready 又等这个 await。表现为「跑起来什么都没发生」，也不报错 | 包成 `async function main()` 再调用 |
| grid 子项用默认 `auto` 列 | 列被撑到子元素 `max-content` —— 长会话名直接撑破 `.rail`（实测 224px 变 255px，右侧内容被裁） | `grid-template-columns: minmax(0, 1fr)`；flex 子项同理 `min-width: 0` |
| `<details>` 折叠时测溢出 | Chrome 用 `content-visibility:hidden` 隐藏内容，被隐藏元素仍有非零 rect → 溢出探针**假阳性** | 探针里用 `el.checkVisibility({contentVisibilityAuto:true})` 排掉 |
| 嵌套 `.memrow` 的操作按钮与来源标记抢同一格 | 下边距/行高跳动 | 两者共用 `grid-column:3`，悬停时用 `:has(.acts)` 隐掉 `.src` |
| 记忆分区标题用 `<button>` 包住可聚焦的拖拽把手 | 嵌套交互元素是**非法 HTML** | 外层用 `<div role="button" tabIndex={0} aria-expanded>` + 手写 Enter/Space |
| `npm install` 默认不跑依赖的 install 脚本 | **Electron 二进制不会下载**（`node_modules/electron/dist` 缺失），esbuild 同理 | `npm approve-scripts --all`，必要时手动 `node node_modules/electron/install.js` |
| `@vitejs/plugin-react@6` 要 `vite@8` | 与 electron-vite 允许的 vite 7 冲突，`npm install` ERESOLVE | 锁 `@vitejs/plugin-react@^5.2.0`（peer 支持 ^7 \|\| ^8） |
| 自绘标题栏忘了 `-webkit-app-region` | 按钮点不动（整条都被当拖拽区） | 标题栏 `drag`，内部可交互元素全部 `no-drag` |
| 字体 TTF 被 `.gitignore`（`*.ttf`）排除 | 新克隆仓库没有字体 → @font-face **静默回退**，汉字格不再是 15.00px，栅格塔掉**且不报错** | 加了 `npm run font`；`npm run probe` 会打印汉字格宽供验证 |

### 阶段 1 新踩的坑（接 RPC 时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 在 ESM 入口用 top-level `await app.whenReady()` | **死锁**（第 4 版已记） | 包成 `main()` |
| `spawn` 一个 `.cmd` 需要 `shell:true` | shell 会做引号解析 —— 提示词里的引号/反斜杠/中文被吃掉或注入 | 用 `process.execPath` + `ELECTRON_RUN_AS_NODE=1` 跑 pi 的 **JS 入口**，纯参数数组 |
| `return '错误'` 提前退出探针脚本 | **丢掉缓冲区**，只看到一行错误看不到前因 | 失败也把 `out.join()` 返回，并把行首打上 `✗` |
| `memoryList` 每次都从磁盘重读 | 刚改完立刻读会读到旧文件，**把内存里的改动回滚**（删掉的条目复活） | `load()` 先 `await this.writeQueue` |
| 退出时不等记忆写盘 | 用户点完「对」立刻关窗口 → **确认丢丢失** | `shutdown()` 里 `await memory.flush()`；`before-quit` 要 `preventDefault` 再等 |
| `overflow-y: auto` 连带把 `overflow-x` 变 `auto` | 纵向滚动条一出现就挤窄 `clientWidth` → 冒出一条横向滚动条 | `overflow: hidden auto` + `scrollbar-gutter: stable` |
| 把扩展 `notify` 直接堆成列表 | 真实扩展（如 `left-info-panel`）会反复通知 → **刷满屏幕遮住界面** | 去重 + 上限 3 条；`.notices` 容器 `pointer-events:none` |
| `YAN_PROMPT` 用 did-finish-load + 定时两个入口 | 提示词被发了多次 | 加 `fired` 标志，只发一次 |
| edit 工具的参数形状 | 按 `old_string/new_string` 渲染 → 显示原始 JSON 而不是 diff | pi 当前是 `{path, edits:[{oldText,newText}]}`；要兼容 legacy 顶层 `oldText`（见 extensions.md:2035） |
| `spawn` 不传 `stdio` 时 TS 推出 `never` | `child.stdout` 全部报错 | 显式标 `ChildProcessWithoutNullStreams` |
| `npm run test:live -- a b` 只跑了 `a` | `process.argv[2]` 只取第一个 | 用 `slice(2).filter(a => !a.startsWith('-'))` |
| 会改文件的测试不清理 | 真的写进了用户的 `~/.pi/agent/sessions/` 和 `memory.json`，造成脏数据 | 测试自带清理 + 按首条消息前缀删除自己造的会话 |

> 诊断心得：右栏那条横向滚动条在**空状态**下看不到（内容不够高，不出滚动条），
> 只有数据满了才复现 —— 所以探测脚本必须在**有真实数据**的实例上跑，不能只看空壳。

---

### 第 7 版的三个坑

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| grid 容器的裸 `1fr` | 轨道最小值是 `auto`(内容宽) → 内容一宽就撑破容器，**溢出的部分被右邻不透明列盖住**（"12m 被遮" / "按钮边框不见" / 右栏被裁一列） | `minmax(0, 1fr)` + 子元素 `min-width:0`。已加 `npm run lint:css` 自动拦。**这个坑出现 3 次了** |
| 验收测试与用户共用状态 | 测试改掉了用户的**右栏分区顺序**；更早还往会话目录塞过 6 个会话、往记忆里留过 5 条编造的「已确认事实」（会误导后续对话） | 测试全跑隔离沙盒：`YAN_USER_DATA` / `YAN_SESSIONS_DIR` / `YAN_DATA_DIR` 三个 env 分别隔离 localStorage / 会话 / 记忆；fixture 从真实会话**只读拷贝** |
| pi 的 `--session-dir` 不跟着 env 走 | 隔离时会出现「pi 写 A、左栏读 B」的诡异现象 | `SESSIONS_DIR` 同时喂给 pi 的 `--session-dir` 与列表扫描 |
| 扩展的 `panel_todos` 只在 TUI 可见 | agent 调了工具维护进度，桌面端什么都不显示 —— 工具白调，用户也看不到它在干什么 | 读会话里的 `custom` entry（`customType` 匹配 `task|todo`）→ 左栏顶部渲染 |
| 扩展的启动通知 | `left-info-panel` 每次启动都 notify「信息面板已启用（overlay 44 列）· /panel …」—— 讲的是 TUI 的 overlay，桌面端不适用，开机弹出来让人困惑 | 启动期的 info/warning 通知只进日志抽屉（`startupPhase` 标志，收到 `msg-add` 后结束） |
| 删 import 时删多了 | `homedir` 被删掉但还在用 → **主进程启动即崩**（`ReferenceError: homedir is not defined`） | 只删确认用不到的；`tsc` 在改完立刻跑一遍 |
| 顶层直接执行 + `const` | 函数声明会提升、`const` 不会 —— fixture 生成器在 TDZ 里被调用 | 执行逻辑包进 `main()`，最后 `await main()` |

---


### 阶段 2 新踩的坑（补功能时）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| pi 的会话文件是**懒创建**的 | `new_session` 后文件不落盘，左栏扫不到 → 点「新对话」**毫无反应** | 左栏为「当前会话」补一条合成条目（没落盘也能看见） |
| `new_session` 在当前会话为空时**不新建** | 返回同一个 sessionId/文件，容易误判成 bug | 这是 pi 的行为，不必绕；但测试别假设「一定会多一个」 |
| `bash_execution_update` 的 `id` 就是请求 id | 要接流式就得**在发命令前**知道 id；`command()` 默认自己生成 id 就接不上 | `command()` 支持传 `opts.id` |
| bash 是流式的，但响应里也有完整 `output` | 只用响应会丢掉「边跑边看」；只用流式可能不完整（有些命令一次性吐完） | 两者取长者 |
| 直接执行的 bash 走了「成功长输出折叠」规则 | 用户主动跑的命令看不到结果，等于白跑 | 给 ToolCard 加 `defaultOpen`，`!` 直执行的默认展开 |
| 附件去重只比 `existing` | 一次传入两条相同的不去重（批内没互相比） | 用 Set 同时比对已有 + 批内 |
| 不检查「当前正在使用的会话」就删文件 | pi 还持有那个文件句柄 | 删除前拦掉当前会话 |
| `deleteSession` 不校验路径 | 可被 `../` 穿越删别处的文件 | 校验：必须在 sessions 目录内且是 `.jsonl` |
| 在真实会话目录里跑测试 | 污染用户数据（我确实造了脏数据，后来清理了） | `YAN_SESSIONS_DIR` 指向临时目录 |
| 测试「改文件」的副作用没清 | 我编造的 5 条**假记忆**（含 4 条"已确认事实"）留在用户记忆里 —— 会误导后续对话 | 测试只动自己造的数据；用完必须清 |
| 探测脚本在**真实数据**上跑才发现的问题 | 长会话才触发的 bug（横向滚动条）在空壳上看不见 | 探测脚本要同时覆盖「空」和「满」两种状态 |

> 一条经验：**先确认自己测的是不是对的东西**。
> 阶段 2 里我报了 4 个「bug」，其中 2 个是我的测试断言写错了
> （bash 折叠算 UX 问题、会话列表断言用了错的判据）。查代码之前先看证据。

---

### 第 8 版的坑（视觉重做）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 给 pi 传 `--session-dir` | pi **不再按 cwd 建项目子目录**，新会话被平铺到 sessions 根目录 → 与用户已有会话分居两处，左栏靠「合成条目」显示 | 平时不传，交给 pi 自己组织；只在测试隔离时传（sandbox 里平铺无妨）。见 `SESSIONS_DIR_IS_OVERRIDE` |
| 给 pi 传 `--name 砚` | 每个新会话标题都是「砚」，左栏里长得一模一样，**等于没标题** | 不传。让 pi 用首条用户消息当标题 |
| 历史消息的思考块显示「正在思考…」 | 会话一多，每条历史消息都挂着「正在思考」，看着像卡死 | 三种标题分开：`live` →「正在思考…」／有耗时 →「已思考 N 秒」／**无耗时（历史加载）** →「思考过程」 |
| 空名字重命名 | pi 的 `set_session_name` 对空串返回 `success:false` —— 名字**一旦设了就清不掉** | 主进程拦住空值并给出理由；不要在 UI 上放「清空」按钮（假的） |
| 空会话时三栏全空 | 界面一片死寂，这是「难看」的最大来源 —— 不是配色问题 | 空状态要给**内容**：符号 + 记忆现状 + 可点建议 + 命令提示，把空白变成入口 |

### 第 9 版的坑

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| 复用主会话去「总结标题」 | ① 污染对话（用户会看到「总结这句话」）② **prompt cache 全失效**（代价比一次请求贵得多） | 起独立进程 `--no-session --no-extensions` + thinking 关掉，用完即走（见 `src/main/title.ts`） |
| 用 `translateX(百分比)` 做滑块 | 百分比基数是**元素自身宽度**而非容器 → 走短一半（实测 105px vs 正确 123px），滑块停在左边一档 | 用 `left` + 实测 `offsetLeft/offsetWidth`（档位文字宽度不等，等分算不准） |
| 用 `dispatchEvent('mouseover')` 测 `:hover` | **合成事件不产生真正的 hover 状态**，`:has(.x:hover)` 永远不匹配 —— 断言失败但功能是好的 | 涉及 `:hover` 的只能 `sendInputEvent`（需窗口可见），或在测试里验证「CSS 规则存在」 |
| `.outline` 有 `pointer-events:none` 却写 `.outline:hover` | 它自己永远收不到 hover（只有刻度可点） | `:has(.outline-tick:hover)` 反推 |
| 给左栏用 `grid-template-rows` 数固定行数 | 子元素「有时有有时没有」（搜索框按需出现）→ 隐式行撑歪高度（805px vs 容器 866px），底部空一截且不随窗口变化 | 用 **flex 列**，让需要滚动的区域 `flex:1` |
| 失败的工具卡自动展开 | 失败命令的输出经常几十行，全展开把后续对话推出视野 | 只有「正在跑」和「用户主动跑的 `!` 命令」自动展开；失败只在摘要行标红 |
| 同一件事放两个入口（侧栏开关） | 用户不确定该点哪个 | 只留标题栏最左上角一个 |
| 用 `write` 工具**整体覆盖** CSS 文件 | 丢了 2174 行（靠 git 提交恢复） | 大文件一律用「追加」或精确 edit；覆盖前先 commit |

> 关于 `:hover` 那条值得单独记：**DOM 断言测不了 CSS 伪类**。
> 这不是「测试写得不好」，是工具边界 —— 要么上 `sendInputEvent`，要么把断言拆成
> 「规则存在」（可测）+「视觉行为」（人工看一眼）。

---

### 第 10 版的坑（回合合并 / 大会话性能 / 发布准备）

| 陷阱 | 后果 | 正确做法 |
|---|---|---|
| **位置跳转用 `behavior:'smooth'`** | **滚动完全不发生**（`scrollTop` 一动不动）。而同一行代码在探针里单独调却是好的 —— 因为平滑滚动会被**任何一次重渲染取消**，而这条路径上总有重渲染（`setStickNow` / `setHover` / 流式推送）。第一次修时我写「跳得远就瞬移、跳得近才平滑」——**只拆中一半**：阈值是 `box.height*1.5`=954px，实测跳动 594px 恰好小于它 → 又走回平滑 → 又不动 | 位置跳转**一律 `behavior:'auto'`**。「跳到第 N 轮」是定位不是看动画。见 `App.tsx` 的 `scrollToTurn` |
| 用 `execFileSync` 探测 pi 版本 | **阻塞主进程事件循环** —— 实测 183ms 内所有 IPC 排队（bootstrap 并发下发 11 个请求全在等它） | 改 `execFile`（异步）+ 结果缓存 + 重入保护 |
| 大上下文会话「打开卡」 | 真因不是渲染（实测空转 3.5ms/帧、滚动 4.5ms/帧）：**pi 的 `switch_session` 要 2780ms**，而直接解析同一份 JSONL 只要 **59ms**。17MB 文件里 26 行 >100KB 占 12.9MB，最长一行 4MB（含 base64 图片的 toolResult） | 先读文件铺内容（486ms）再让 pi 在后台切；超长文本/图片**有损降级**并如实标记「N 条被截断」 |
| 以为 `get_messages` = 会话全部消息 | 它**不含压缩前历史**（`docs/rpc.md` 写明；要完整的得用 `get_entries`）。所以大上下文会话在界面上只剩当前窗口（实测 70 条 vs 文件 1093 条） | 需要完整历史就直读 JSONL（`src/main/session-reader.ts`） |
| 原始 range 滑块做「思考强度」 | `::-webkit-slider-runnable-track`（14px 高）与 `::-webkit-slider-thumb`（16px）**不同源** → 视觉上是断的；再叠一层绝对定位的档位小方块，两套指示打架（用户报「有 bug」） | 用**离散档位按钮**（1:1 映射）。滑块本身是谎言：档位是离散的，滑动却没有中间态 |
| `background-size: 240px` + `no-repeat` 做扫光 | 那 240px 之外**什么都不画** → 一条 898px 的线只显示前 240px，看着像断了 | **两层背景**：`background-color` 铺满（保证线完整）+ `background-image` 只做高光带 |
| `highlight.css` 里把 `.hljs` 全局替换成 `.md pre code` | 选择器变成 `pre code.md pre code-keyword` 这种无意义的东西 —— **语法高亮从来没生效过**（实测 `span[class^="hljs"]` 数恒为 0），浅色下代码块还是写死的深色底 | 作用域限定 + 保留 `.hljs-*` 类名：`.md pre code .hljs-keyword`；底色改走 `--code-bg` 令牌 |
| 渲染端 `window.keydown` 接全局快捷键 | 会被**输入法组合态**、焦点不在 webContents、菜单 accelerator 吃掉（三种都实测到） | 主进程 `before-input-event` + 把动作名发回渲染端（协议知识不进主进程） |
| `patchSettings` 读-改-写用内存快照 | 写回的是**整个对象**，两个写入方互相覆盖 —— 实测把 `alwaysOnTop` 的改动抹掉了（查三次都写不进去） | 写入前 `invalidate()` 重读磁盘 |
| 测试直接写 `~/.pi/agent/auth.json` | 那里是用户的**真实密钥**。写坏了比污染会话目录/记忆文件严重得多（那两件已经各踩过一次） | 加 `YAN_PI_DIR` 覆盖，测试用临时目录 |
| 探针按会话**标题**找 fixture | 改成「每轮用模型重新生成标题」之后，前面的场景一跑就把 fixture 标题改了 → 后面按 `title.includes('YAN-TODO')` 找不到。**典型「只在全量跑时暴露」** | 按 `path` 找（文件名里的 fixture 标识不会变） |
| 探针用固定 `sleep(900)` 等左栏展开 | 左栏是「320ms 延迟 + CSS 过渡」，负载高时（连跑多个场景）不够 → 偶发失败，单独跑必过 | 一律用**轮询**（`until(fn, ms)`），不用固定 sleep 等 UI 状态 |
| 测试之间共享 `localStorage` | 前一个场景改了 `yan.rail-open` 会把后一个场景的初始状态改掉 | 探针自己 setup 需要的状态，不假设初始值 |

---

> 这一版最重要的两条抽象：
> **① 浏览器会因为重渲染取消平滑滚动** —— 所以「跳转」类操作一律瞬移。
> **② 「打开慢」要先量再修** —— 我一开始以为是渲染（量了帧率：3.5ms/帧，不卡），
> 又以为是切换（量了：95–236ms，不慢），最后量到 `switch_session` 的 2780ms。
> 三次都在猜，第四次才量对。**先测出 47 倍的差距，再动手。**

---

> 教训：**层次靠色阶差，不靠投影**。
> 旧色阶 `bg-0→bg-1` 只差 7 个亮度值，三栏糊成一片黑，
> 所以「卡片」这个概念在视觉上根本不存在。拉开到 ~10 就好了。

---



### 架构

```
Renderer (React, 无 Node 权限)
   ↓ contextBridge 白名单
Main (Node)
   · SessionHub: Map<sessionId, PiSession>
   · 自己写的 RPC JSONL 编解码 + 请求响应关联
   · SessionStore: 读 ~/.pi/agent/sessions/
   · SettingsStore: 读 ~/.pi/agent/settings.json
   ↓ stdin/stdout JSONL
pi --mode rpc  ×N（一个会话一个进程）
```

**三条原则**

1. 一个会话 = 一个 pi 进程（崩溃隔离、并行、扩展环境干净）
2. **自己写 RPC 客户端，不 import 内部模块** —— 包里的 `rpc-client.js`
   不在 `exports` 里，`pi update` 一次就可能炸。协议适配集中在 `protocol.ts` 一处
3. `nodeIntegration:false` + `contextIsolation:true`

### 分阶段

| 阶段 | 内容 | 交付 | 预估轮次 |
|---|---|---|---|
| **0** | 脚手架 + `protocol.ts` + 最小流式窗口 | 窗口里打字能看到流式回答 | 8–12 |
| **1** | 消息流 + Markdown + 工具卡 + diff + 输入区 + 队列 | **能真用来改代码的客户端** | 15–25 |
| **2** | 多会话 + 项目切换 + 模型/thinking 选择器 + 状态面板 + 扩展 UI 桥接 | 完整多会话客户端 | 15–25 |
| **3** | 原生菜单/托盘/通知/多标签（可选） | 桌面化 | — |
| **4** | 并排 diff、子 agent 可视化、命令面板（可选） | — | — |

**v1 严格停在阶段 2。**

### v1 范围

| ✅ 做 | ❌ 不做 |
|---|---|
| 流式对话 + Markdown + 代码高亮 | 子 agent 可视化 |
| 工具卡片（bash 输出、edit diff） | 并排 diff 编辑器 |
| 多会话（新建/恢复/改名/删除/fork） | 内嵌终端 |
| 模型 & thinking 切换、项目切换 | 插件市场、主题市场 |
| 扩展 UI 桥接（notify/confirm/select/input/editor） | 移动端、远程连接 |
| 状态面板（含 Corrected 峰谷） | 自动更新 |
| 中英文 i18n | 三平台签名 |
| 深色主题 | 浅色主题 |

### 数据归属

| 数据 | 方案 | 理由 |
|---|---|---|
| 会话 | **复用 `~/.pi/agent/sessions/`** | 与 TUI 互通（核心价值） |
| 设置 | 复用 `~/.pi/agent/settings.json` | 同上 |
| 密钥 | 复用 pi 的 auth 存储 | 免二次配置 |
| 桌面端独有 | `~/.pi/agent/desktop.json` | 窗口尺寸、主题、语言 |
| 遥测 | **零遥测** | 写进 SECURITY.md |

⚠️ 代价：耦合 pi 会话格式（现 `version:3`）。需加**格式版本检查**，
不认识的版本降级为只读。

### 扩展兼容性

| 扩展 API | 桌面端 | 实现位置 |
|---|---|---|
| `registerTool` / `registerCommand` / 事件 | ✅ 完全可用 | pi 侧原生 |
| `notify` | ✅ 通知（去重 + 上限 3 条） | `UiBridge.tsx` |
| `confirm` / `select` / `input` / `editor` | ✅ 模态框（**必须应答，否则扩展会卡住**） | `UiDialog` |
| `setStatus` | ✅ 底部状态条 | `StatusBar` |
| `setTitle` | ✅ | `agent.ts` |
| `set_editor_text` | ✅ 塞进输入框 | `Composer` |
| `setWidget` / overlay / `custom` | ❌ TUI 专属 → 写进日志（否则会静默丢失） | `agent.ts` |
| `registerShortcut` | ⚠️ 未映射（桌面端快捷键方案待定） | — |

用户已写的 `left-info-panel.ts`：`panel_todos` 工具和 `/panel` 命令照常工作，
它的 `notify` 会被通知系统接住（**已验证：不去重会刷屏**）。
侧栏由桌面端原生实现，**扩展文件不用改**。

> 扩展的加载方式是 `pi --extension resources/pi/yan-memory.ts` ——
> **不写进 `~/.pi/agent/extensions/`**，所以与用户已有扩展零冲突，也不需要安装步骤。

---

### 8.11 第 12 版的坑（2026-09-12：工具栏 / 用户档案 / 启动器）

| # | 坑 | 教训 |
|---|---|---|
| 1 | **收起面板后开关点不到** | 一条过时规则 `.app.rail-off .rail{opacity:1;pointer-events:auto}` 让透明左栏盖在展开把手上。探针实测 `elementFromPoint` 命中 `rail-brand-btn` 而不是把手。**透明 ≠ 不可交互** —— 隐藏内容要用 `visibility`/`display`，不要只改 opacity |
| 2 | **同一个开关的两个状态用了两个元素** | 收起是 `.rail-stub`（24px 绝对定位），展开是品牌按钮（padding 追随内容）→ 几何对不上（用户报「不对称」）。**开关类 UI 要用同一个元素 + 固定盒子**，只换内容 |
| 3 | **列宽 0 会把把手一起裁掉** | 面板收起宽度必须 ≥「内边距 + 按钮 + 内边距」。左栏取 50（12+26+12），右栏取 40 |
| 4 | **主进程热路径不能异步读盘** | 快捷键处理器用 `getSettings().then()` 才知道当前档，而写设置是「invalidate → 重读 → 写」的异步链 → 连按两下第二下读到旧值，算出同一档（**表现为「按键丢了」**）。改用内存里的当前值 |
| 5 | **探针假设起始状态** | panels/symmetry 假设「起始是展开的」，但前一个场景可能把它留在收起态 → 前后对比整个颠倒 → 全错。**单跑必过、全量才炸**（同一个坑本项目踩过 3 次）。显式置初始状态 |
| 6 | **量几何不等稳定** | 面板收放有 120–160ms 过渡，中途量出的矩形是随机的。改为「连读两次相同才算」 |
| 7 | **引导层晚出现** | 「到了就找关闭按钮，找不到就算了」→ 引导层在数据就绪后才渲染，于是它一直开着盖住界面 → `elementFromPoint` 量到遮罩 → 偶发失败。**轮询等它出现**再关 |
| 8 | **固定时间窗口等 UI** | zoom 场景用「采样 10 秒」，被前一个场景拖慢时最后一下按键还没到就结束 → 假失败。正是本项目明令禁止的模式。改成轮询到条件成立 |
| 9 | **断言精度 vs 浮点缩放** | 界面缩放开着时 `getBoundingClientRect` 返回 25.998 而非 26，`>= 26` 判定失败。**量几何的断言要留容差** |
| 10 | **每个条目 stat 一遍** | 文件树为了拿 size 对每个条目 `stat`，然后才截断到 400 条 —— 家目录上千条 = 上千次系统调用。`readdir(withFileTypes)` 已免费给出目录标志；**先排序、再截断、最后才取 size** |
| 11 | **.cmd 文件里的中文会毁掉整个脚本** | cmd.exe 按系统 OEM 代码页（GBK）读 .cmd，非 ASCII 字节在 `chcp` 生效**之前**就破坏行解析。实测整个 .cmd 一行都没执行成功。**.cmd 保持纯 ASCII**，中文由 node 打印 |
| 12 | **探针失败要留现场** | symmetry 失败过两次而单跑必过 —— 没有现场只能靠重跑碰运气。现在失败时会 dump 面板状态/缩放/打开的浮层/各元素矩形 |

### 8.12 第 13 版的坑（2026-09-12：宽度拖拽 / 分区排序 / 工具库）

| # | 坑 | 教训 |
|---|---|---|
| 1 | **排序静默失效** | 读 section 的 `data-sec`（`rp-queue`）而顺序数组里存的是 `queue` → `indexOf` 得 -1 直接 return。**两套 id 形态混用是最难查的一类 bug**：不报错、不崩，只是什么都不发生。现在用 `data-tool-id` 统一 |
| 2 | **`setPointerCapture` 会抛异常** | 它抛 NotFoundError 时后面的初始化全被跳过 → 拖拽永远不启动。**capture 只是便利，必须包 try**，而且要放在状态更新之后 |
| 3 | **命中区重叠** | 宽度把手（7px，绝对定位）压在分区排序把手左缘 → 拖排序变成拖宽度。两个相邻交互元素必须有**不重叠的命中区**（把手 5px + grip margin-left 6px） |
| 4 | **非法 CSS 值让整条声明失效** | 拖过头产生 `-9439px` → `grid-template-columns` 整条被丢弃 → 那一拖完全没反应，且读回 CSS 变量变 NaN。**范围限制要在产生值的那一侧就夹**，不能只在落盘时夹 |
| 5 | **元素宽度写死 → 拖拽看起来无效** | `.rail{width:300px}` 让「列宽」与「元素宽」脱钩：设置改了、列宽变了，元素还是 300px。**跟随容器宽度用 100%** |
| 6 | **声明了却没读的字段** | `CATALOG[].envVar` 写了但 `listAuthProviders` 从没读过 → 用环境变量配好 key 的用户被判「未配置」。**加了字段要搜一遍谁在读它** |
| 7 | **重构时丢失「空则不渲染」** | 改成注册表渲染后，外层容器变成无条件渲染 → 空区块又出现了。**行为约定要跟着数据一起搬家**（现在 isEmpty 声明在注册表里） |
| 8 | **场景之间共享状态文件** | 所有场景共用一个 desktop.json，某场景改了 cwd → 后面的 atPath 拿到家目录，断言全落空。**每个场景开跑前重置为已知状态** |
| 9 | **探针抢焦点** | 连续开十几个窗口，每次都把焦点从用户的另一个桌面抢走（用户明确要求别干扰）。用 `showInactive()`；代价是未聚焦窗口被 Chromium 降频，必须同时关掉后台节流（`disable-background-timer-throttling` 等，只在有 YAN_PROBE 时） |
| 10 | **断言绑「碰巧的数据」** | atPath 断言「目录带尾斜杠」用的是 `src/main/`（那里只有 .ts 文件、没有子目录），它一直靠 cwd=家目录时 `Desktop/` 里碰巧有子目录才通过。**断言要绑必然成立的性质** |
| 11 | **固定 sleep 等首帧** | virtual 场景 `await sleep(50)` 后量 DOM：单跑够（62ms），全量后半段首帧还没提交 → 量到 0 条，后面全挂。与第 8.9 条同源 |

 被当成替换模式（首行变成 `NaN`）、整段函数尾被删。**含引号/注释的代码改动一律用编辑工具，不要拼 shell 字符串** |
| 10 | **别写死「外部工具」的常量** | 压缩阈值不是 16384 而是 `contextWindow - reserveTokens`，且 reserveTokens 用户可配（pi 的 settings.json）。**读它的配置，不要抄一个数字** |

---

## 10. 待解决的开放问题

| # | 问题 | 说明 |
|---|---|---|
| 1 | ~~**pi 怎么分发**~~ | ✅ **已解决**：不自己打包，搬运 pi 自带的 `dist/bundle` + 补 6 个最小依赖 = **20MB**。见下方 §10.1 |
| 2 | **RPC 协议稳定性** | 非公开稳定 API。`docs/rpc.md` 1618 行，47 命令 / 20+ 事件 |
| 3 | ~~改动审阅流程~~ | ✅ **已定：只做回滚**，见 §1.2 |
| 4 | 扩展 UI 桥接的完整度 | `select/confirm/input/editor` 需映射成模态框 |
| 5 | 无 `tsc` / `bun` | 只能用 `esbuild` 做语法检查，无法全量类型检查 |
| 6 | 状态面板应显示真实 contextWindow | pi TUI 看不到；当它与 API 声明不一致时给提示（见 §5.5） |
| 7 | **字体子集方案** | Maple CN 全量 17.7MB → 用 `cn-font-split` 切 woff2 |
| 8 | **Q4 三栏宽度与折叠** | 宽度保持 `216:1fr(上限780):320`，**两栏已实现可折叠**；拖动调宽留到实现阶段。**已按建议实现，待用户点头** |
| 9 | **Q5 助手消息无容器** | 保留不对称（贴近终端，靠左侧符号槽区分角色）。**已按建议保留，待用户确认** |
| 10 | **Q9 工作区「检查状态」** | 保留，但必须说清是**启发式**（从 bash 命令名推），不是真实测试框架集成。**待用户确认** |
| 11 | **左栏歧义** | ✅ 已按「左栏 = 时间分组（可折叠）+ 选中会话内嵌可折叠任务；右栏中下部 = 文件树」实现，见 DESIGN §3.4 末尾 |
| 12 | **字号** | ✅ 已定 **12.5px**（代码 12px），依据是汉字格整数像素 + 栅格零偏差 |

---

## 10.1 内置 pi 运行时（已决策：搬运 pi 自带的 bundle）

**结论：不要自己做 esbuild 单文件。** 以下四条是**打包器内联不了**的硬边界，
pi 官方自己用 esbuild 也打了四道补丁 —— 所以它的 `dist/bundle/` 本身就是
「打包能做到的极限」，直接搬运它比重造一个更碎、更脆。

| # | 内联不了的东西 | 实证 |
|---|---|---|
| ① | **顶层 external 包** | ESM 顶层 `import` 只能留 external。pi 自己把 `@earendil-works/chord` / `typebox` / `undici` 留在外（我的实验：删掉 `node_modules` 直接 `ERR_MODULE_NOT_FOUND: @earendil-works/chord`） |
| ② | **`.wasm` 二进制** | `@silvia-odwyer/photon-node`（图片缩放/EXIF）。pi 在 bundle 里写了 `patchPhotonWasmRead()` 去 `process.execPath` 旁边找 `photon_rs_bg.wasm` |
| ③ | **`worker_threads` 独立文件** | `new Worker(new URL("./image-resize-worker.js", import.meta.url))` —— worker 必须是独立文件，单文件装不下 |
| ④ | **运行时 `readFileSync` 的资产** | 主题 `dark.json` / `light.json` / export-html 模板 / `clankolas.png`。不是 import，打包器看不见。实测报错 `ENOENT: ...\dist\modes\interactive\theme\dark.json` |

还有第 ⑤ 条（只在加载 `.ts` 扩展时才暴露）：**`jiti`**。bundle 里静态扫不到它，
是扩展加载器在运行时 `require` 的 —— 我们自己那个 `resources/pi/yan-memory.ts` 就靠它编译。

### 方案

| 项 | 值 |
|---|---|
| 抽取脚本 | `scripts/vendor-pi.mjs`（`npm run vendor:pi`） |
| 产物 | `resources/pi-runtime/`（**已入 .gitignore**，20MB） |
| 组成 | `dist/bundle` + `dist` 资产 8.1MB；`node_modules` 子集 8.0MB |
| node_modules 子集 | `@earendil-works/chord` / `typebox` / `undici` / `@silvia-odwyer/photon-node` / `jiti` / `esbuild`（chord 的传递依赖） |
| 解析 | `src/main/protocol.ts` 的 `bundledRoots()`，优先级：`piBin` 设置 → `YAN_PI_BIN` → **内置运行时** → 全局安装 → PATH shim → shell 兜底 |

对比三个方案：依赖捆绑 **424MB** ／ esbuild 单文件（**不可行**，上面 5 条）／ 内置搬运 **20MB**。

### ⚠️ 维护要点

1. **`vendor-pi.mjs` 会自校验**：扫 bundle 的裸 import，若出现 `MUST_HAVE`/`OPTIONAL` 未覆盖的包就**非零退出**（pi 升级后新增依赖时这一步能拦住）。
2. 脚本末尾**真跑**一次 `cli.js --version` + 一次 `get_state` RPC 握手 —— 不做「拷贝成功即通过」的假验证。
3. pi 升级后要重跑 `npm run vendor:pi`。**不要手工改 `resources/pi-runtime/` 里的文件**（下次抽取会被覆盖）。
4. `@aws-sdk/signature-v4-crt` / `signature-v4a` 在 `OPTIONAL` 里 —— pi 自己也没装，缺失时降级（Bedrock 的 CRT 签名加速）。
5. **砍掉 `typebox`+`undici` 也能启动**（实测过），但它们是顶层 import，风险在运行时才炸；留着（ +6.2MB）换确定性。

### 验证（本次实测）

| 验证 | 结果 |
|---|---|
| `npm run probe-pi` | ✅ 解析到 `resources/pi-runtime/dist/bundle/cli.js`，model=DeepSeek V4.1 Flash |
| `npm run check` | ✅ **15/15 场景全绿**（全部跑在内置 pi 上，不是用户全局那个） |
| `npm run test:live -- image` | ✅ 真调模型，模型认出"红色" → **wasm 路径可用** |
| 扩展加载（jiti） | ✅ `--extension resources/pi/yan-memory.ts` 无线无错 |

---

## 11. 关键路径速查

```
pi 包        %USERPROFILE%\AppData\Roaming\npm\node_modules\@earendil-works\pi-coding-agent
pi 配置      %USERPROFILE%\.pi\agent
pi 会话      %USERPROFILE%\.pi\agent\sessions\--C--Users-Name--\*.jsonl
pi 扩展      %USERPROFILE%\.pi\agent\extensions\left-info-panel.ts
pi fork      %USERPROFILE%\.pi\fork\   （侧栏 fork，README.md 在里）
项目         %USERPROFILE%\Desktop\pi-desktop\
RPC 文档     <pi包>\docs\rpc.md
扩展文档     <pi包>\docs\extensions.md
边缘浏览器   C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
临时工具     %TEMP%\see.mjs（视觉校验）、%TEMP%\mkhtml.mjs（图片裁剪）
模型同步     %USERPROFILE%\.pi\agent\.dev\sync-model-context.mjs
模型配置     %USERPROFILE%\.pi\agent\models.json（备份 .bak）
```

### RPC 事实

- 启动：`pi --mode rpc --no-session`，JSONL over stdin/stdout
- 47 个命令：`prompt` `steer` `follow_up` `abort` `get_state` `get_messages`
  `set_model` `get_available_models` `set_thinking_level` `compact` `bash`
  `get_session_stats` `export_html` `switch_session` `fork` `clone` `get_entries`
  `get_tree` `get_commands` `select` `confirm` `input` `editor` `notify` …
- 事件：`agent_start/end/settled`、`turn_start/end`、`message_start/update/end`、
  `tool_execution_start/update/end`、`bash_execution_update`、`queue_update`、
  `compaction_*`、`auto_retry_*`、`extension_error`
- 扩展 UI 请求：`{"type":"extension_ui_request","method":"notify","message":"..."}`
- 响应：`{id, type:"response", command, success, data}`
- 流式中途发 prompt 必须带 `streamingBehavior:"steer"|"followUp"`
- 带图片：`{"type":"prompt","message":"...","images":[{"type":"image","data":"<base64>","mimeType":"image/png"}]}`
- **`RpcClient` 在 `dist/modes/rpc/rpc-client.js`，不在 `package.json` exports 里** —— 只能当协议参考，不能 import

### 侧栏 fork（独立项目，已完成）

```
%USERPROFILE%\.pi\fork\
├── README.md            用法与限制
├── setup-fork.mjs       一键重建（复制 + junction + 打补丁）
├── apply-patches.mjs    幂等补丁器（10 处补丁 + 1 新文件）
├── bin\launch.mjs       启动器（自动注入 --tui-mode fullscreen）
└── coding-agent\        dist 副本
```

`pi update` 后重新执行 `node ~/.pi/fork/setup-fork.mjs`。
侧栏默认**右侧**，`PI_SIDEBAR_SIDE=left` 可覆盖；宽度 `PI_SIDEBAR_WIDTH`（默认 38）。
配置持久化在 `~/.pi/agent/left-panel.json`。

**注意：桌面客户端不需要这个 fork** —— 侧栏在桌面端是普通 CSS。

---

## 12. 立即要做的事

### ✅ 已完成

1. ~~让用户审阅设计稿~~ → 已审阅，收到 8 条修改意见
2. ~~字体选型~~ → 已定 **Maple Mono CN，基准 12.5px**（代码 12px），实测栅格零偏差
3. ~~图标库~~ → 已抓 **92 个 reicon 图标**，已视觉验证
4. ~~128k 上下文窗口问题~~ → 已修复（§5.5）
5. ~~**设计稿 v0.2**（用户 8 条意见 + 字体/图标落地）~~ → **已完成**，见 §3；`check.mjs` 全绿，
   三套分辨率 + 浅色主题已逐张目视校验
6. ~~验证工具链升级~~ → `check.mjs` 9 类检查；新增 `embed-icons.mjs`；DOM 断言 + 注入式功能自测
7. ~~**定技术栈**~~ → ✅ **Electron + Vite + React 19 + TypeScript**
8. ~~**阶段 0：UI 骨架**~~ → ✅ **已完成**，见 §3.0。29 条交互断言全绿

### ⏭ 下一步

#### ✅ 阶段 1 + 2 已交付

| 交付 | 位置 |
|---|---|
| 手写 RPC 客户端（JSONL 分帧 / 请求关联 / 扩展 UI / 自定义请求 id） | `src/main/protocol.ts` |
| 协议 → UI 归一化（流式组装 / toolCallId 关联 / 16ms 节流 / bash 流式） | `src/main/agent.ts` |
| 记忆存储 + 认识论规则 + soul.md 只读 | `src/main/memory.ts` |
| 记忆扩展（remember/recall/forget + 系统提示词注入） | `resources/pi/yan-memory.ts` |
| 会话列表（含 `session_info` 名字）+ 切换 / 新建 / 重命名 / 删除 / fork / clone / 导出 | `src/main/sessions.ts` + `Rail.tsx` |
| markdown / 高亮 / 工具卡 / 彩色 diff / 智能展开 | `Message.tsx` |
| 输入区四模式（文本 / `/` 命令 / `!` shell / 图片）+ 自动长高 | `Composer.tsx` |
| 模型 + 思考档选择器、自动压缩/重试开关 | `MemoryPanel.tsx` |
| 扩展对话框 + 通知限流 + 状态条 + 日志抽屉 | `UiBridge.tsx` |
| 长会话虚拟化（阈值 80） | `App.tsx` + virtua |
| 三层测试（单元 19 条 / 真实应用 5 场景 / 真模型 3 场景） | `scripts/` |

#### 用户已提的都已完成（2026-09-12）

D（引导检查本地 key，含环境变量）/ E2（左右栏宽度拖拽）/
F（分区拖拽排序）/ G（工具库）已全部落地，见 §8.12。

#### 阶段 3 候选（按优先级）

| # | 任务 | 为什么 |
|---|---|---|
| 1 | ~~**打包分发**~~ | ⏭ **进行中**：pi 内置已落地（§10.1）；剩下 electron-builder 配置 + 字体子集化 |
| 2 | 会话树浏览（`get_tree` / `navigateTree`） | 协议已支持；能在分支间跳转 |
| 3 | 扩展 `registerShortcut` 映射 | 用户自己的扩展如果注册了快捷键，现在按不动 |
| 4 | 浅色主题打磨 | 令牌齐全，未调 |
| 5 | 设置面板（语言/主题/cwd 之外的东西落成 UI） | 目前散落在标题栏和右栏 |

> 阶段 1 与 2 的合计代码量约 8.5k 行（不含生成物）。

---

## 13. 另一条线的悬置问题（v0.2 遗留，若继续做编码客户端才需要）

| # | 问题 | 当前实现 |
|---|---|---|
| Q4 | 三栏 `216 : 1fr(上限780) : 320` OK 吗？ | 「砚」版已改为 `224 : 1fr(760) : 328` |
| Q5 | 助手消息无容器（不像用户气泡）的不对称保留吗？ | **保留**（已实现：你 = `›`，砚 = `✦` 符号槽） |
| Q9 | 工作区「检查状态」是启发式猜的，保留吗？ | 砚版已**去掉整个工作区概念** |

> 实现时可直接把 `DESIGN.md` 的 §2 令牌原样搬成 `styles/tokens.css`，
> §3 的组件约定已足够下笔，不需要重看设计稿。
> 设计稿里那些交互（智能展开 / 拖拽排序 / 树折叠）的**正确写法已写在 DESIGN.md**，
> 包括哪些坑不能踩。
