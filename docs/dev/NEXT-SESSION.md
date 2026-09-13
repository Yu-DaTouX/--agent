# 新会话入口

先读 [HANDOFF.md](HANDOFF.md)，再按 [工作目录导览](../WORKSPACE.md) 定位代码。

可复制以下内容给新会话：

~~~text
阅读 docs/dev/HANDOFF.md 和 docs/WORKSPACE.md，检查 git status，保留工作区已有修改，然后继续用户指定的工作。

已确认：任务面板保持现状；记忆模块和提示词注入已移除，不恢复；会话树旧链路是主动移除，不自动补回。主题、设置、模型接入和 Windows 打包均已有实现。内置浏览器与本机 Chrome 接入已完成（`BrowserController` 的 `mode: 'external'`）；内置 pi 管理/升级、运行时校验入总检查、Ctrl+Shift+P 上一模型均已完成。

当前后续事项以 HANDOFF.md 的「尚未完成」表与「已知边界」为准（发布与账号能力：签名 / macOS·Linux 打包 / 登录）。不要把历史测试结果当作本次验收，也不要反复询问已确认的产品决定。
~~~
