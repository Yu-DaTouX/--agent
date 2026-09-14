# UI 优化：进度交接

> 给接手的人（或下一个对话）看的。**先读这份，再读 `UI-优化方案.md`。**
>
> 方案原文：`docs/design/UI-优化方案.md`（前面是方案本身，末尾是「附录：实施进展」逐项记录）。
> 相关工具产出：`docs/design/CSS-归属表.md`、`docs/design/CSS-令牌清单.md`。

---

## 1. 一句话现状

**P0 三项全部完成；P1 全部完成；P2 的性能部分已实测（结论是不需要优化）、视觉部分已建立基线并
修掉两个真 bug。**
剩下只有「需要产品判断」的：4.2 的视觉微调（要你指认哪里挤、哪条线多余）。

两件值得先了解的事：

- **长会话白屏卡死**（用户报的）：不是样式问题，是**布局 bug**，见 3.1。
- **长会话卡顿**：主进程每帧重发整篇累积文本/输出（O(N²)），渲染端每帧重解析全部 markdown。
  已改成增量推送 + memo，见 3.8 —— 这是第二轮，`P2 不需要优化`那个结论只对**第一轮**的指标成立。

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
    现在：**有失败 → 默认展开 + 「N 个失败」角标（红）**。
    ⚠️ 标题文字**不染色** —— 用户后来明确要求「只显示 x 个失败（红色），
    其他的字改为之前的颜色」（整行变红会让人以为整组都坏了）。探针 `toolrow`。
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

### 3.8 长会话卡顿：增量推送 + 渲染隔离（第二轮）★

第一轮的 `perf` 探针说「不需要优化」，但那测的是 240 条消息的**合成**场景。真实会话里
有 1948 条消息、967 次工具调用 —— 数字完全不同。两个提交：

- **`Stream deltas instead of resending everything on every frame`**
  - 主进程每帧不再重发整篇 `text` / `thinking` / 工具输出，改发**增量**
    （`textDelta` / `thinkingDelta` / `outputDelta`，游标式）。全量快照仍在
    `message_end` / 中止 / `sync`，那是权威对齐（类型见 `src/shared/ipc.ts` 的 `MessagePatch`）。
  - 节流间隔随累积长度自适应：`FLUSH_MS`(16ms) → 上限 `MAX_FLUSH_MS`(120ms)。
  - `callIndex` / `callOwner` 把工具查找从「线性扫全部消息」改成 O(1)（工具事件是最高频路径）。
  - 守卫：`scripts/test-stream-deltas.mjs`（直接驱动 `AgentController`，断言「增量拼回来 == 全量」
    且单帧体积有界）+ `scripts/probe/deltas.js`（渲染端 append 路径）。
- **`Stop re-rendering the whole history on every stream frame`**
  - `groupIntoTurns` 每帧重建全部回合对象 → `TurnView` 的 memo 必然失效。
    下面挂 `Paragraph` / `ToolRow` / `ToolGroup` / `ReasoningCapsule` 的 **memo**（按值比较），
    markdown 结果加 LRU 缓存，关掉 `highlightAuto`。
    ⚠️ 代价：**没有语言标注的代码块不再自动上色**（原来每帧对半截代码逐个试语言，约 30ms/块）。
  - 虚拟化阈值改成「回合数 ≥ 80 **或** 消息数 ≥ 200」——真实会话是「回合少、消息多」，
    只看回合数等于没开虚拟化。
  - ⚠️ 改 `Paragraph` 的 props 要小心：**任何非原始值都会让这个 memo 失效**，
    退回成「每帧重解析整段会话」（实测一次 535ms）。

### 3.9 本轮顺带修的

- **`sessions` 探针重写**（`Rewrite the stale session probe assertions`）：左栏不再「悬停展开」，
  旧断言发个 `mousemove` 就当作展开了，功能删掉后它**永远等不到**；而收起时列表是 0 行 →
  探针以「会话太少」**假通过**。现在显式 `setRailPinned(true)` 并断言，依赖 pi 的部分显式跳过。
- **`light` 探针**：第 3 节断言的是废弃的 `.tool-head` / `.tool-sum`，选择器恒不匹配 →
  断言**静默走「跳过」分支**。改用 `.trow-head`，并新增第 5 节（终端窗口内部的深色覆盖）。
- **`shots.mjs` 支持 `YAN_SHOT_DIR`**：调 UI 时把基线导到临时目录，不碰仓库里那 4 张
  已发布的预览图（它们还带着用户未提交的本地修改）。
- **`css-layer-check.mjs` 的 `!important` 建模 + `--selftest`**（见 4.1）。
- **设置页文案里的 `**正在运行**`**：`set-desc` 不渲染 markdown，星号被当明文画出来。
  已改文案，并在 `test:unit` 里加了「i18n 文案不含 `**`」的断言。
- **推理窗口高度**：固定 25vh 改成「内容自适应 + 上限」（见 4.3）。

---

## 4. 剩余项与结论

### 4.1 P0-1 尾款：校验器升级**已完成** —— 结论是「确实压不动了」

**已做**（提交 `Make the CSS layer checker model !important`）：
1. ~~逐选择器~~ —— 上一轮**已经做了**（`splitTopLevel` 按括号/引号深度拆 `,`；
   直接 `split(',')` 会把 `:is(.a, .b)` 拆坏）；
2. **`!important` 已建模** —— 这才是真缺口。旧版把 `!important` 当成值的一部分比较，
   既能误报，也会**假通过**：`{p:1px!important}{p:1px}{p:2px}` 与 `{p:1px}{p:2px}`
   被判为等价，而前者实际是 `1px`、后者是 `2px`；
3. 跨 `@media` 保持「各比各的」（保守）；
4. 新增 `--selftest`（10 个用例，含上面那条假通过），已接入 `npm run typecheck`。

**结论：`stage1.css` / `redesign.css` 已经没有可安全合并/删除的量**（实测）：
```bash
node scripts/css-dead-rules.mjs --file stage1.css --file redesign.css --dry
# → 完全被覆盖（可删）3 条；但 stage1.css 可删 0 条、redesign.css 可删 0 条
node scripts/css-consolidate.mjs stage1.css redesign.css --dry
# → 无需归并（它按完整 head 分组，`.md h1, .md h2` 与 `.md h1` 不算同名）
```

**为什么连「搬家」也不做**：本工具只保证「同一选择器自己的声明集不变」，
**不建模选择器之间的相对顺序与特异性**。搬家会改变规则之间的先后 —— 元素同时匹配
`.a`（被提前的那条）与 `.b` 时谁赢会变，而校验器看不见。要安全搬家得先建模特异性，
收益（行数）与风险不成比例。

⭐ **工具的边界**（也写在脚本注释里）：它够用来验证「同一批声明换个文件放」，
**不够**单独证明「改变规则之间相对顺序」是等价的。所以要合并规则时，除了跑它，
还要保证**同属性同选择器的相对顺序不变**。

**验证流程**：
```bash
cp -r src/renderer/src/styles /tmp/styles-bN
# 改 styles/
node scripts/css-layer-check.mjs /tmp/styles-bN src/renderer/src/styles
# 它会比对 1207 组 (媒体查询, 选择器) 的计算后声明；改动的每一处都会列出来
node scripts/css-layer-check.mjs --selftest   # 校验器自己的语义
```

⭐ **CSS 加载顺序**（`App.tsx` 与 `css-migrate` / `css-inventory` / `css-layer-check` / `css-tokens`
里的 `ORDER` 数组必须一致）：

```
tokens, app, stage1, redesign, motion, settings, electron, highlight,
layout, shell, rail, chat, composer, tools, browser, dialog
```

### 4.2 P2 5.1 视觉密度与边框收敛（基线已建；修了 2 个真 bug，微调待指认）

**基线有了**：`scripts/shots.mjs` 现在认 `YAN_SHOT_DIR`（提交 `Let screenshots go
somewhere other than the published previews`），把图导到临时目录，**不碰**仓库里
那 4 张已发布的预览图（它们还带着用户未提交的本地修改）。

```bash
YAN_SHOT_DIR=/tmp/uishot npm run shots   # 深色主界面 / 浅色 / 推理窗口 / 设置
```

（PNG 可以直接看：`read` 工具会把图附上来 —— 这两个 bug 就是这么查出来的，
靠读 CSS 看不出来。）

**已查出并修掉的两个真 bug**（都不是审美问题，是坏了）：

1. **浅色主题下终端窗口里的参数块是白底**（`Keep the terminal dark in the light theme too`）。
   `.term .tool-pre` 与 `.tool-pre.args` 同为 (0,2,0)，却写在前面 —— 后写者赢，
   于是整组「终端内覆盖」规则**全是死的**：参数块渲染成 `--bg-0` 白底 + `--border`
   下边框 + 中灰字，基本看不清。深色主题下 `--bg-0`≈`#0c0c0c`，所以只有浅色暴露。
   守卫：`light` 探针第 5 节（把终端内每个元素的颜色**合成到深底上**算亮度）。
2. **设置页文案里的 `**正在运行**` 被当明文画出来**（`Stop showing markdown asterisks…`）。
   `.set-desc` 直接把 `t()` 插进文本节点，不渲染 markdown。守卫：`test:unit` 断言
   i18n 文案不含 `**`。

**还没做的**：真正的「密度 / 边框 / 比例」微调。那需要你的偏好 —— 有基线了，
指哪打哪（哪块觉得挤、哪条线觉得多余）。

⚠️ 调完如果要刷新正式的那 4 张预览图，先想清楚：它们现在带着用户未提交的修改，
`npm run shots`（不带 `YAN_SHOT_DIR`）会盖掉它们。

### 4.3 P1 4.2 的零头（推理高度已改；其余待指认）

- **推理窗口高度已改成「内容自适应 + 上限」**（`Make the reasoning window fit its content`）。
  原来固定 `height: 25vh`：900px 窗口 → 227px，而一段 4 行推理只有 80px，下面**空 2/3**
  （截图实测）。现在 `max-height: min(25vh, 420px)`：短内容贴着内容，超过上限后内部滚动。
  守卫：`reasoning` 探针（短内容量到 54px、超长内容量到上限；退回固定高度就会 ✗）。
  ⚠️ 这与 `chat.css` 里「**用户要求**：固定大小」的旧注释冲突 —— 按你的裁决
  「依方案为主」改的（方案的 4.2 写的是「内容自适应，并设置高度上限」）。
- **「消息标签、时间线、引用关系更轻」**：属于视觉调整，与 4.2 一起看，需要你指认。

### 4.4 P1 4.5 任务状态语义（**已完成**）

提交 `Give tasks a real status instead of guessing one`。

**做了什么**：`SessionTodo` 加了可选 `status`（`pending` / `running` / `done` / `blocked`）；
`todo-snapshots.ts` 用**别名表**解析（`in_progress` / `doing` / `active` / `IN-PROGRESS` 都认），
**认不出来就不带 status**（退回 `done` 推断）—— 猜错的状态比没有状态更糟。
UI 里「哪条正在进行」改成两步：显式 `status === 'running'` 优先；没有显式状态时退回
「第一个未完成」，但**要求回合真的在跑**。

**为什么「回合在跑」是必需的**：只有 `{text, done}` 的老数据里，谁在做只能推断。
旧实现 `findIndex(x => !x.done)` 意味着只要还有没做完的，界面上就**永远**有一条在转 ——
agent 停了、报错了、用户中断了，照转。那是猜测，不是状态。

**守卫**：`test-todo-history.mjs`（别名映射 + `done`/`status` 冲突 + `sameTodos` 不看 status）、
`todos.js` / `todonew.js`（停下后不得有 active；显式 running 赢过更靠前的 pending）。

**方案里另两个疑点还没覆盖**（写在这里免得被当成已做）：
- 「单个任务失败与整批完成是否共用状态」—— `blocked` 现在会解析出来，但 UI 还没给它
  单独的样式（显示成「未完成」）。要看真实数据里到底有没有这个值再定；
- 「计数是否以工具协议为准」—— 目前计数就是 `done/总数`（清单自己的口径）。
  这需要另一个可与它对账的来源，目前没有。

⚠️ 写清单的是 **pi 侧的扩展**（`panel_todos`，不在这个仓库），所以这些字段名我们只能
**兼容**、不能规定。真实会话里现在一条 `panel_todos` entry 都没有（都是 fixture 合成的），
所以 `status` 的支持目前是「前瞻」：扩展哪天带了，界面当天就能用。

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

## 6. 提交记录

### 第一轮（P0 + P1 主体 + 性能实测）

```
3c02772 Keep diagnostic probes out of the full regression
4f4817c Write a handover for the UI work
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

### 第二轮（长会话卡顿 + 收尾）

```
5327279 Rewrite the stale session probe assertions
f519435 Stream deltas instead of resending everything on every frame
bdf12bc Stop re-rendering the whole history on every stream frame
4e08c99 Make the CSS layer checker model !important
580eb1d Let screenshots go somewhere other than the published previews
ee67d43 Make the reasoning window fit its content
95f9cc8 Keep the terminal dark in the light theme too
b3ebe46 Stop showing markdown asterisks in the settings copy
902c0c4 Give tasks a real status instead of guessing one
```

第一轮全部已推送到 `origin/main`；第二轮（上面那 10 个）目前只在**本地** `main` 上。
