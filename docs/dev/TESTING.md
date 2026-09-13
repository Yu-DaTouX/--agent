# 测试指南（含「测试用哪个模型」）

> 本文是**测试约定**的单一真源。每次加/改测试场景前先读一读，
> 尤其是「测试模型」一节 —— 它决定真实调用会不会花钱。

## 三层测试

| 层 | 命令 | 特点 |
|---|---|---|
| 纯逻辑 | `npm run test:unit`（node） | 快、确定、能断言边界。不启动 Electron、不碰 pi、不花 token |
| UI + 接线 | `npm run test:live -- <场景>`（真实 Electron） | 完整主进程 / preload / IPC / pi 子进程。默认不调模型 |
| 真行为 | 带 `cost: 1` 的场景（见下） | 真模型、真工具、真图片 |

提交前跑 `npm run check`（typecheck + build + test:unit + 设计稿溢出 + 一批 live 场景）。
完整场景清单在 [`scripts/test-live.mjs`](../../scripts/test-live.mjs) 的 `CASES`。

## 测试模型（重要）

### 默认：commandcode 的 Ling 3.0 Flash Sante（**免费**）

所有**真实调用模型**的测试场景默认使用：

| 项 | 值 |
|---|---|
| 供应商 provider | `commandcode` |
| 模型 id | `inclusionai/ling-3.0-flash-sante:free` |
| 传给 pi 的 `--model` | `commandcode/inclusionai/ling-3.0-flash-sante:free` |
| 成本 | 0（免费） |

为什么固定用免费模型：真实场景（流式、工具调用、排队、问答）必须调模型才有意义，
如果每次都跟着用户当前选的模型跑，跑一次回归就可能花钱；固定成免费模型后可以放心反复跑。

### 覆盖方式

```bash
# 换一个模型跑（必须是 pi 的 "provider/modelId" 形式）
YAN_TEST_MODEL="commandcode/deepseek/deepseek-v4.1-flash" npm run test:live -- e2e
```

- 变量：`YAN_TEST_MODEL`（默认见上）。
- 约束：模型必须能从**真实的 `~/.pi/agent/auth.json`** 取到凭证 ——
  测试只隔离 `YAN_USER_DATA` / `YAN_SESSIONS_DIR` / `YAN_DATA_DIR`，
  **不隔离 pi 的 auth**（那是用户的密钥，不能拷）。
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

不标 `cost`（或 `cost: 0`）的场景不调模型，跑多少次都不花钱。

## 隔离与安全

`test:live` 每个批次建一个临时 sandbox，把：

- `YAN_USER_DATA`（localStorage / cache）
- `YAN_SESSIONS_DIR`（会话文件）
- `YAN_DATA_DIR`（`desktop.json` 设置）
- `YAN_PI_DIR`（设置面板读写的凭证目录）

指到临时目录，**不碰真实数据**。唯一会读到真实文件的是 pi 自己的
`~/.pi/agent/auth.json`（调模型要凭证），但只读。

`YAN_TEST_ISOLATED=0` 可直接跑真实环境，仅用于排查问题。

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
