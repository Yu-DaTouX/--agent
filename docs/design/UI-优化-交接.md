# UI 优化：进度交接

> 给接手的人（或下一个对话）看的。**先读这份，再读 `UI-优化方案.md`。**
>
> 方案原文：`docs/design/UI-优化方案.md`（前面是方案本身，末尾是「附录：实施进展」逐项记录）。
> 相关工具产出：`docs/design/CSS-归属表.md`、`docs/design/CSS-令牌清单.md`。

---

## 1. 一句话现状

**P0 三项全部完成；P1 的 4.1 / 4.3 / 4.4 完成，4.2 完成大半；P2 的性能部分已实测并得出结论（不需要优化）。**
剩下的都是「要么需要新判断、要么需要截图基线、要么方案自己要求独立评估」的项，列在第 4 节。

用户报的**卡死（长会话白屏）已定位并修复**，见 3.1 —— 那不是一个样式问题，是布局 bug，值得先了解。

---

## 2. 先跑什么

```bash
cd "C:/Users/YuDaTou/Desktop/pi-desktop"

npm run typecheck      # tsc（node + web）+ CSS lint，17 个文件
npm run test:unit      # 189/189
npm run build

# 改动 UI 后跑几个相关场景（跑之前必须 unset，见第 5 节）
unset ELECTRON_RUN_AS_NODE && npm run test:live -- live narrow panels tools virtual perf railsearch toolrow

npm run check          # 完整回归（很长，含上面这些场景）
```

全部命令当前状态：**通过**。最近一次 `test:live` 15 个场景全绿。

---

## 3. 已完成（别重复做）

### 3.1 长会话白屏卡死（用户报的，已修）★ 先看这条

**根因**：`.center` 是 grid，行**按子元素出现顺序**分配（`auto minmax(0,1fr) auto auto`），
而子元素是 `[会话标题 / 连接条 / 对话区 / 输入区]`，其中**连接条只在连接异常时渲染**。
连接条一出现就抢走 `1fr` 行，对话区被挤到 `auto` 行：

| | 实测 | 结果 |
| --- | --- | --- |
| 修复前 | `connbar(484) stream(56)` | `.stream` 仅 56px，**渲染 0/240 条**，耗时 **24733ms** |
| 修复后 | `connbar(33) stream(507)` | 渲染 6/240 条，耗时 **0ms** |

普通列表靠内容撑开 `auto` 行所以看不出来；**虚拟化列表项是绝对定位**，那一行直接塌成
padding 高度。触发条件：**连接异常 且 会话长到触发虚拟化**。

**修复**（`styles/chat.css` / `layout.css` 里的 `.center`）：行定义改 `auto auto 1fr auto`
并给 `.stream`、`.composer-wrap` **显式绑行**。

⚠️ 同一类 bug 在 `.rightpanel` 上也存在（已修，用 `:has(> .tool-lib)` 分支）。
**改任何「按子元素顺序分行/分列」的容器时，声明的行数必须与实际渲染元素数一致**，
否则 `1fr` 会落到错误的元素上。`layoutdiag` 探针专门报这个形状，改完跑一下。

### 3.2 P0-3 快捷键与弹窗一致性
新 `lib/modalLayer.ts`（模态栈 + `useModalLayer` + `useFocusTrap`）；
`setHotkeyGuard` 通道（shared/ipc → preload → main 的 `before-input-event`）：
有模态时放行 Shift+Tab / Ctrl+P；`yan:renderer-ready` 重置。接入
Settings / UiDialog / Onboarding / Rail 删除确认框。

### 3.3 P0-2 窄屏布局
删掉 `app.css` 里失效的 `@media (max-width:900px)`（被后面的规则覆盖，从未生效）；
按实测倒推（`composer ≈ center − 73px`，可用下限 `center ≥ 393px`）在 `layout.css` 加 900px 档。

### 3.4 P0-1 样式收敛（主体完成）
- `sidebar-review.css` 拆成 `layout/shell/rail/chat/composer/tools/browser/dialog.css`（放在加载链末）。
- `redesign.css` **4734 → 1274 行**；`stage2.css` 淘汰；`stage1.css` 578 → 204 行。
- 令牌重复定义 **7 → 0**（`--r-*` 归 `tokens.css`，`--w-rail*` 归 `layout.css`）。
- **剩余部分见第 4.1 节**（不能盲改，需要先升级校验器）。
- 工具：`scripts/css-inventory.mjs`、`css-split-check.mjs`、`css-migrate.mjs`、
  `css-layer-check.mjs`、`css-consolidate.mjs`、`css-dead-rules.mjs`、`css-tokens.mjs`。

### 3.5 P1 部分
- **4.1 左栏**：搜索清空后焦点回输入框、Esc 关闭后焦点回开关（以前掉到 `body`）；
  状态图标本来就带 `title`。探针 `railsearch`。
- **4.2 消息/推理/工具（大部分）**：
  - 失败的**工具组**原来埋在「调用了 N 次命令」里、标题不说有东西挂了。
    现在：**有失败 → 默认展开 + 标题变红 + "N 个失败"角标**。探针 `toolrow`。
  - `ToolRow.canExpand` 补上 `failed`（那个变量以前定义过却从未使用）。
  - 推理/终端都已有高度上限（`chat.css` 的 `height` 写法 + 终端 resize 记忆）。
- **4.3 输入区**：`sendKey` 设置（auto / enter / ctrlEnter，默认 `auto` 不改用户习惯）+ 常显发送规则。
- **4.4 工具栏**：默认顺序改为 `todo, context, files, quota, queue, ext, log, actions`。

### 3.6 P2 性能：已实测，结论是「不需要优化」
探针 `perf`，240 条消息（虚拟化、挂载 7 个回合）：

| 指标 | 实测（中位） | 阈值 |
| --- | --- | --- |
| 流式更新（每次 delta 重算分组 + 重渲染） | **11.6ms**（最坏 34.7ms） | < 50ms |
| 右栏收放（grid 列宽变化 → 重排） | **11.6ms** | < 250ms |
| 左栏收放 | **10.3ms** | < 250ms |
| 虚拟化窗口 | 240 回合 → **14 个 DOM 节点** | — |

**所以方案 5.2 列的「增量分组 / 状态订阅细化 / 渲染隔离」不做** —— 数字不支持。
顺带删了无消费者却每次滚动都写 store 的 `scrollProgress`。

面板收放便宜是因为 `.workspace` 的 `grid-template-columns` 标了 `transition: none`。
**别给它加动画** —— 那会让整段对话在每一帧重新折行。

### 3.7 顺带修的
- `piCall()` 归一化 20 处 pi 命令调用 + `refreshSessions` / `reloadModels` / `reloadCommands` /
  `copyLastReply` 的失败处理（以前是 unhandled rejection）。
- 过时探针断言清理：`hotkeys`（订阅顺序：先订阅再等）、`narrow` / `panels` / `topbar`（48px 紧凑轨）、
  `slashcmd`（pi 未就绪时跳过）、`tools`（拖拽目标从实际渲染分区挑、catch 带堆栈）。
- 排查过所有 `useStore(...)` 选择器返回新引用的风险：**只有 1 处可疑**（`Rail.tsx` 的
  `descendantCount`，返回数字，安全）。共 195 处。

---

## 4. 待办

### 4.1 P0-1 尾款：剩下的行数压不动了（需要先造工具）

`stage1.css`（204 行）与 `redesign.css`（1274 行）里剩的，是**被其他文件也定义**的选择器，
以及多选择器规则（`.a, .b {}`）。

**为什么不能继续直接合并**：现有的 `css-layer-check.mjs` 按**选择器字符串**分组来比对
计算后声明。多选择器规则在它眼里是一个整体，它**看不到** `.modal, .settings` 与
`.settings` 之间的覆盖关系。基于它做合并，等价性无法证明。

**要做的**：
1. 把校验器升级为**逐选择器**（把 `A, B {}` 拆成 `A {}` + `B {}` 再比对）；
2. 纳入 `!important` 优先级（早期带 `!important`、后期不带 → 早期仍赢，这条现在没建模）；
3. 跨 `@media` 的覆盖无法静态判定 → **保持跳过**（宁可少合并）。

**已经踩过的坑**（写在这里免得重踩）：
- 解析声明前**必须抹掉注释**，否则 `/* … */ padding` 被当成属性名，覆盖判定**静默失效**
  （实测造成 8 处层叠不一致）。
- 因为证明不了，我删掉了写了一半的 `scripts/css-merge-dups.mjs`，**没有**纳入代码库。
  别把它捡回来，除非先把上面 1–3 做完。

**验证流程**：
```bash
cp -r src/renderer/src/styles /tmp/styles-bN
# 改 styles/
node scripts/css-layer-check.mjs /tmp/styles-bN src/renderer/src/styles
# 它会比对 1201 组 (媒体查询, 选择器) 的计算后声明
```

⭐ **CSS 加载顺序**（`App.tsx` 与 `css-migrate` / `css-inventory` / `css-layer-check` / `css-tokens`
里的 `ORDER` 数组必须一致）：

```
tokens, app, stage1, redesign, motion, settings, electron, highlight,
layout, shell, rail, chat, composer, tools, browser, dialog
```

### 4.2 P2 5.1 视觉密度与边框收敛（需要截图基线）

方案要求：减少重复边框、收敛强调色、统一图标与文字比例。
**这项我没有做，因为手上没有可对比的基线截图**，靠读 CSS 判断"密度是否合适"不可靠。

建议：先用 `npm run shot` / `npm run shots` / `docs/design/measure-design.mjs` 产出基线，
再逐块调。注意工作区里有用户自己的预览产物（见第 5 节），**不要动**。

### 4.3 P1 4.2 的零头

方案里还剩：
- 「推理和终端默认内容自适应，但给高度上限；用户主动调整过就记住」——终端已有 resize + 记忆，
  推理是靠 `height`（见 `chat.css` 的注释），**建议人工确认一遍短内容时是否仍占大面积空白**。
- 「消息标签、时间线、引用关系更轻」——属于视觉调整，与 4.2 一起看。

### 4.4 P1 4.5 任务状态语义（方案自己要求独立评估）

方案原文：涉及**共享类型和工具协议**，应独立评估，「避免与纯 UI 样式改造混在一起」。
具体疑点：单个任务失败与整批完成是否共用状态、部分成功是否明确、计数是否以工具协议为准。

**我没有碰它** —— 它会影响消息状态与工具协议，不该混在 UI 收尾里做。要动请单开一轮。

### 4.5 过时探针：`sessions`

`sessions` 探针还在断言「左栏悬停展开」的行为，而那个功能**已按用户要求删除**。
它现在是过时的，需要重写断言（不是产品 bug）。

---

## 5. 环境与约定（坑都在这里）

- 仓库 `C:/Users/YuDaTou/Desktop/pi-desktop`，分支 `main`，远端 `origin`（GitHub）。
- **跑 electron 前必须 `unset ELECTRON_RUN_AS_NODE`** —— 本机环境变量会让 electron 以 node 模式启动。
- **隔离测试环境里 pi 连不上**（`conn=exited`）。依赖 pi 的探针要**显式「跳过」**，不要报 ✗。
  反过来说：**探针大量失败时先怀疑自己的断言**，但**也不要轻易归因给「环境」**——
  那个白屏 bug 就是被"环境问题"糊弄过去的，它其实只在没连上 pi 时才复现。
- 改脚本不要用 `node -e` 处理带 `\n` 的字符串（转义踩过坑，写出过语法错误）；
  优先用编辑器工具改，改完 `node --check <file>`。
- 探针注入假消息要**等应用挂载完 + 轮询期间重复注入**：应用自己也会收 pi 的 `sync`，
  它才是权威的，会把注入冲掉（`virtual`、`toolrow` 都踩过）。
- 用户未提交的产物，**工作区里保留、不要提交也不要删**：
  `docs/design/preview/*.png`（M）、`docs/design/preview/live/`、`live-working.gif`、
  `ui-review-v2.png`、`docs/design/yan-onboarding-editable.html`、`scripts/live-preview.mjs`、
  `scripts/ui-review.js`、`tui-render-proposal.html`（D）。
- 提交风格：**英文祈使句**，与仓库既有一致；**只提交自己测过的改动**；每个提交都应可构建。
- 新增探针要在 `scripts/test-live.mjs` 的 `SCENARIOS` 里注册；
  纳入完整回归还要加进 `package.json` 的 `check` 脚本。

---

## 6. 提交记录（本阶段）

```
76b8a6e Surface failed tool calls instead of burying them
8e26ca7 Keep focus in the rail search instead of dropping it
eaf65f1 Measure performance instead of guessing
74d74a4 Match right panel grid rows to the elements actually rendered
cc8fd99 Fix the long-conversation white screen
bc7f50f Fix dead-rule detection and shrink redesign.css further
d481d0d Put tasks first in the default tool order
df20323 Make the send key explicit instead of implied by composer height
d7891aa Delete rules whose every declaration is overridden
fb67aed Retire stage2.css and move stage1 / redesign rules to their modules
147bf64 Merge duplicated design tokens into a single source
aeb92fd Consolidate duplicate rules and move the rest into per-module files
9ddd22f Normalize pi command failures instead of unhandled rejections
995b675 Split review styles into per-module files, fix narrow-window rules
64a18ce Let dialogs take back global hotkeys and unify focus behavior
```

全部已推送到 `origin/main`。
