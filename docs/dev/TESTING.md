# 测试指南（含「测试用哪个模型」）

> 本文是**测试约定**的单一真源。每次加/改测试场景前先读一读，
> 尤其是「测试模型」一节 —— 它决定真实调用会不会花钱。

## 三层测试

| 层 | 命令 | 特点 |
|---|---|---|
| 纯逻辑 | `npm run test:unit`（node） | 快、确定、能断言边界。不启动 Electron、不碰 pi、不花 token |
| UI + 接线 | `npm run test:live -- <场景>`（真实 Electron） | 完整主进程 / preload / IPC / pi 子进程。默认不调模型 |
| 真行为 | 带 `cost: 1` 的场景（见下） | 真模型、真工具、真图片 |

源码改动后先 `npm run build`，再跑 test:unit / test:live；live 不自动构建。运行 Electron 前，在 PowerShell 执行 `Remove-Item Env:ELECTRON_RUN_AS_NODE -ErrorAction SilentlyContinue`。

完整门槛为 `npm run check`，具体执行链以 [package.json](../../package.json) 为准；纯文档改动只核对内容、链接和命令，不必构建应用。
完整场景清单在 [`scripts/test-live.mjs`](../../scripts/test-live.mjs) 的 `CASES`。

## 测试模型（重要）

### 默认模型（以脚本配置为准）

所有**真实调用模型**的测试场景默认使用：

| 项 | 值 |
|---|---|
| 供应商 provider | `commandcode` |
| 模型 id | `inclusionai/ling-3.0-flash-sante:free` |
| 传给 pi 的 `--model` | `commandcode/inclusionai/ling-3.0-flash-sante:free` |
| 费用 | 脚本默认选择带 free 标识的模型；实际供应商计费与可用性需在调用前确认 |

默认配置用于避免跟随用户日常模型误耗额度；配置名不保证供应商长期免费或模型能力不变。

### 覆盖方式

```powershell
$env:YAN_TEST_MODEL = "provider/modelId"
npm run test:live -- e2e
Remove-Item Env:YAN_TEST_MODEL
```

- 变量：`YAN_TEST_MODEL`（默认见上）。
- 凭证与模型配置从真实目录只读复制到隔离 sandbox，测试仅修改副本；详见“隔离与安全”。
- 注入点：[`src/main/agent.ts`](../../src/main/agent.ts) 在启动 pi 时把 `YAN_TEST_MODEL`
  变成 `--model <值>`。正常运行时该变量为空，不影响用户。

### 例外：需要视觉的场景

Ling 3.0 Flash Sante 是**纯文本**模型（`input: ["text"]`），发图会失败。
需要视觉的场景在 `CASES` 里用 `model:` 单独覆盖，默认用：

| 项 | 值 |
|---|---|
| 场景 | `image`（把图片真的发给模型） |
| provider | `commandcode` |
| 模型 id | `deepseek/deepseek-v4.1-flash`（`input: ["text", "image"]`） |
| 覆盖变量 | `YAN_TEST_VISION_MODEL` |

**以后加需要视觉的新场景**：在 `CASES` 对应项里写 `model: TEST_VISION_MODEL`。

## 哪些场景会真的调模型（花钱/耗额度）

在 `CASES` 里标了 `cost: 1` 的场景：

| 场景 | 做什么 | 用哪个模型 |
|---|---|---|
| `tokens` | 用量 / 速度 | 默认（Ling） |
| `conn` | 连接状态竞态 | 默认（Ling） |
| `e2e` | 真发一条消息（流式 + 工具） | 默认（Ling） |
| `queue` | 排队 + Esc 回收 | 默认（Ling） |
| `ask` | **问答功能**：模型主动提问 → 弹窗 → 回答 → 回填；含自主模式不弹窗 | 默认（Ling） |
| `image` | 图片真的发给模型 | **视觉模型** |

不标 `cost`（或 `cost: 0`）的场景按设计不主动调用模型；新增或改动探针时需核对实际调用链。

## 隔离与安全

`test:live` 每个批次建一个临时 sandbox，把：

- `YAN_USER_DATA`（localStorage / cache）
- `YAN_SESSIONS_DIR`（会话文件）
- `YAN_DATA_DIR`（`desktop.json` 设置）
- `YAN_PI_DIR`（设置面板读写的凭证目录）

指到临时目录，**不碰真实数据**。

### pi 的凭证与模型目录（只读复制）

隔离让 pi 读不到 `~/.pi/agent`，直接后果是 **pi 起不来** —— 所有依赖「pi 就绪」的
场景（`runnerselect`、任何 `cost: 1`）都会失败或跳过。所以启动前会把这三个文件
**只读复制**进 sandbox：

| 文件 | 作用 | 少了会怎样 |
|---|---|---|
| `auth.json` | 凭证 | pi 只能起一个 `{id:'unknown'}` 空模型 |
| `models.json` | 自定义 provider 定义 | 本机的 `commandcode`（69 个模型）来自这里，**不在** pi 内置目录中；少了它 pi 解析 `--model commandcode/...` 会 `Model not found` **并直接退出**，表现为 conn 一直卡在 `starting` |
| `models-store.json` | 目录缓存 | 多一次网络拉取（不致命） |

安全边界（用户要求：测试可以用，**打包切勿放进去**）：

- 只**读**源文件；sandbox 在系统临时目录，跑完（含 Ctrl+C / SIGTERM）自动删除；
- `auth` 场景改写/删除的也只是副本；
- **不会进发布包**：`electron-builder.yml` 的 `files` / `extraResources` 只收
  `out/`、`build/icon.png`、`package.json` 和 `resources/pi-runtime`，临时目录不在其中；
- `.gitignore` 已忽略 `auth.json`。

不要关闭隔离来绕过测试失败；优先检查 sandbox 的配置、凭证副本与连接状态。

## 问答功能怎么测

- **纯逻辑**（`npm run test:unit`）：`scripts/test-question.mjs` 直接 import
  内置扩展 `resources/pi-extensions/question.js`，喂假 `pi` API，断言
  系统提示随自主模式切换、自主模式不弹 UI、select/自定义/取消的回填。
- **端到端**（会调模型）：`npm run test:live -- ask`。流程：
  发一条要求提问的消息 → 等 UiBridge 弹窗 → 选答案 → 断言
  `question` 工具行完成、答案回填进模型回复；最后开自主模式，断言**不再弹窗**。

## 内置 pi 功能怎么核

- `npm run vendor:pi:check`：内置运行时能不能启动、RPC 握手是否正常。
- `npm run probe-pi`：单独验证「pi 能不能被找到并启动」。

## 相关文件

| 文件 | 作用 |
|---|---|
| `scripts/test-live.mjs` | live 场景注册表（`CASES`）+ 隔离 + 测试模型注入 |
| `scripts/test-unit.mjs` | 单元测试入口（纯逻辑） |
| `scripts/probe/*.js` | 各 live 场景的探针脚本（在真实渲染进程里执行） |
| `resources/pi-extensions/question.js` | 内置「提问」扩展（问答功能的模型侧） |
