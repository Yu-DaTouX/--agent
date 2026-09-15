# 实现维护与排障

按问题查阅；当前任务和完成度见 [HANDOFF](HANDOFF.md)。本文只保留可复用的实现约束。

## 运行与测试

- live 场景使用 `out/`，不自动构建；先 build。Electron 启动前清除 `ELECTRON_RUN_AS_NODE`。隔离变量与模型配置统一见 [TESTING](TESTING.md)。
- 单实例锁随 userData 目录隔离。测试没有窗口或输出时，先检查环境变量、构建结果和实例目录。
- 测试按 fixture 路径定位会话，不依赖自动标题；用条件轮询等待连接、DOM 和几何稳定。
- sandbox 的凭证副本必须随退出、SIGINT/SIGTERM 清理；异常终止后检查残留，不打印凭证内容。

## 会话与能力状态

- 启动期 sessionId 会从临时值变为稳定 ID。模型、思考档位、命令等能力请求按运行实例与代次判过期，见 `state/capability-request.ts`。
- 启动时加载失败的能力列表，需要在连接就绪后实际补拉。核对调用链，不能只信“稍后重试”的注释。
- 模型未知时保留选择器和空态，避免多层 return null 使用户失去入口。
- 分支 entryId 只取自 `get_fork_messages`；JSONL 快读与 pi 的权威会话切换职责分开。
- 通知需要去重与限流；启动期扩展说明进日志，不反复打断用户。

## 浏览器原生视图

- 网页使用 `WebContentsView`，位于 renderer DOM 之上；加载/错误提示放工具栏，拖拽把手预留空间。
- DOM 的 CSS 坐标乘主窗口 `getZoomFactor()` 后才是 setBounds 所需 DIP。单列 grid 用 `minmax(0, 1fr)`。
- 加载会重置先前缩放；在 ready-to-show 后补设。避免在 did-finish-load 内同步 setZoomFactor，已有渲染进程崩溃记录。
- element ref 在文档替换后失效；普通 DOM 增删不全量清空。已脱离节点返回 STALE_ELEMENT。
- 点击先滚入视野，再重新测量坐标；切换外部 Chrome 目标后重新 observe。
- 内嵌浏览器与本机 Chrome 的 profile 独立。登录、下载与 Cookie 同步需实际目标网站验证，不能用 CDP 冒烟替代。

## 模型登录

- ChatGPT 登录实现在 `src/main/oauth.ts`：授权参数、PKCE、state、回调端口和 auth.json 格式必须对齐所分发的 pi 版本，不随意简化。
- 凭证写入要合并 provider 键；pi 重新读取凭证需按运行生命周期处理，不能打断忙碌会话。
- 其余 provider 不照搬 ChatGPT 协议；registerCommand 与 registerShortcut 分开看，后者在当前已记录的 pi RPC 中缺枚举/执行接口。

## 样式、扫描与打包

- 令牌先改 [DESIGN](../design/DESIGN.md)，再同步 tokens.css；旧名 stage1/stage2/redesign 不代表可删除。
- CSS 与死代码扫描先用已知正例自检；无外部 import 的 export 可能仍被文件内部使用，IPC 类型导出属于契约面。
- 不在 CASES 的 probe 可能由打包、审阅或手工 YAN_PROBE 入口驱动；删除前查调用者。
- 生成脚本后检查正则、反斜杠和 Windows 路径是否原样落盘，避免转义损坏造成扫描假阳性。
- pi runtime 保留其 bundle、依赖、WASM、worker 和动态资源；用 upgrade:pi 更新，不手工重打单文件。
- electron-builder 的 extraResources 要包含 runtime 的 node_modules；`out/test/**` 排除在发布包外。产物要查实际 asar，详见 [RELEASING](RELEASING.md)。
