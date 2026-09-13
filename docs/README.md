# 文档索引

工作目录的分类、源码导航、脚本用途及维护标注见 [工作目录导览](WORKSPACE.md)。

| 目录 | 内容 |
|---|---|
| [`design/`](design/) | **设计规范与设计稿**。`DESIGN.md` 是**设计令牌的唯一真源**（`styles/tokens.css` 必须与它一致），`prototype.html` 是可交互设计稿 |
| [`dev/`](dev/) | **开发过程文档**（非用户文档）。`HANDOFF.md` 是跨会话交接文档（需求 / 环境事实 / 踩过的坑），`NEXT-SESSION.md` 是新会话的开场指令 |
| [`archive/`](archive/) | **已完结阶段的工作记录**（目标 / 改动 / 验证 / 边界）。现状以下面 `dev/HANDOFF.md` 为准，这里用于追溯「改动是怎么来的」 |

## 想快速了解这个项目

| 你想知道 | 看哪个 |
|---|---|
| 这是什么、怎么跑起来 | [根目录 README](../README.md) |
| 界面设计为什么长这样 | [`design/DESIGN.md`](design/DESIGN.md) |
| 颜色 / 字号 / 间距的取值从哪来 | [`design/DESIGN.md`](design/DESIGN.md) §1–2（令牌真源） |
| pi 的 RPC 协议怎么用 | [`dev/HANDOFF.md`](dev/HANDOFF.md)「源码与协议边界」及 `src/main/protocol.ts` |
| 为什么**不要**自己做 esbuild 打包 | [`dev/HANDOFF.md`](dev/HANDOFF.md) 「维护中应保留的经验」 |
| 想打包分发 | 根目录 [`README`](../README.md) 的「打包分发」 + [`electron-builder.yml`](../electron-builder.yml)；坑见 [`dev/HANDOFF.md`](dev/HANDOFF.md) 「维护中应保留的经验」 |
| 有哪些反复踩的坑 | [`dev/HANDOFF.md`](dev/HANDOFF.md) 「维护中应保留的经验」 |
| 某个功能/修复当时是怎么做的 | [`archive/`](archive/)（按日期与主题索引） |
| 内置浏览器由哪些文件组成 | [`WORKSPACE.md`](WORKSPACE.md) 「内置浏览器结构」 |

## 设计目录里的脚本

都在 `design/` 下，可以直接跑（不依赖应用）：

| 脚本 | 做什么 | 命令 |
|---|---|---|
| `check.mjs` | 设计稿静态自检（令牌 / i18n / 禁止项 / 字体 / 图标） | `node docs/design/check.mjs` |
| `measure-design.mjs` | 量设计稿的布局溢出（要求 Electron） | `npm run measure:design` |
| `build-icons.mjs` | 从 reicon 拉取并生成完整 sprite（联网） | `node docs/design/build-icons.mjs` |
| `extract-icons.mjs` | 把设计稿的 sprite 抽成 renderer 用的 TS 模块 | `npm run icons` |
| `embed-icons.mjs` | 把用到的图标子集内联回设计稿 | `node docs/design/embed-icons.mjs` |

> `design/archive/` 放旧设计稿与一次性修复脚本，平时不需要跑。
> 顶层的 [`archive/`](archive/) 放开发过程的阶段记录，入口见 [`archive/README.md`](archive/README.md)。
