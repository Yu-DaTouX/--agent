# 样式规则归属表（自动生成）

> 由 `node scripts/css-inventory.mjs --md` 生成。改 CSS 后重新生成再对比。
>
> 加载顺序与 `App.tsx` 的 import 一致；**同特异性时后加载者胜**。

## 1. 各文件规模

| 顺序 | 文件 | 行数 | 唯一选择器 | 规则数 | @media | !important |
| ---: | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | `tokens.css` | 249 | 22 | 23 | 1 | 2 |
| 2 | `app.css` | 512 | 78 | 78 | 1 | 0 |
| 3 | `stage1.css` | 579 | 91 | 99 | 1 | 0 |
| 4 | `stage2.css` | 226 | 34 | 34 | 0 | 0 |
| 5 | `redesign.css` | 1954 | 232 | 247 | 5 | 2 |
| 6 | `motion.css` | 1364 | 179 | 184 | 5 | 11 |
| 7 | `settings.css` | 306 | 45 | 45 | 0 | 0 |
| 8 | `electron.css` | 35 | 11 | 12 | 0 | 0 |
| 9 | `highlight.css` | 173 | 81 | 81 | 0 | 0 |
| 10 | `layout.css` | 93 | 2 | 2 | 3 | 0 |
| 11 | `shell.css` | 200 | 30 | 30 | 0 | 0 |
| 12 | `rail.css` | 811 | 117 | 119 | 0 | 0 |
| 13 | `chat.css` | 709 | 103 | 103 | 0 | 0 |
| 14 | `composer.css` | 490 | 61 | 61 | 0 | 0 |
| 15 | `tools.css` | 855 | 142 | 143 | 0 | 0 |
| 16 | `browser.css` | 344 | 50 | 50 | 0 | 0 |
| | **合计** | **8900** | **1156** | | | |

## 2. 覆盖热力：每个文件「最终胜出」的选择器数

即：这些规则是这个文件说了算的（它是最后一个定义者）。

| 文件 | 最终胜出 |
| --- | ---: |
| `tokens.css` | 16 |
| `app.css` | 47 |
| `stage1.css` | 72 |
| `stage2.css` | 33 |
| `redesign.css` | 171 |
| `motion.css` | 176 |
| `settings.css` | 45 |
| `electron.css` | 10 |
| `highlight.css` | 81 |
| `layout.css` | 2 |
| `shell.css` | 30 |
| `rail.css` | 117 |
| `chat.css` | 103 |
| `composer.css` | 61 |
| `tools.css` | 142 |
| `browser.css` | 50 |

## 3. 被多个文件定义的选择器（覆盖链）

共 **103** 个选择器在 ≥2 个文件里出现。渲染顺序 = 从左到右，**最右那个胜出**。

| 选择器 | 定义它的文件（按加载顺序） |
| --- | --- |
| `:root` | tokens → redesign → motion → layout |
| `.rail` | app → redesign → electron → rail |
| `.titlebar` | app → redesign → electron |
| `.workspace` | app → redesign → layout |
| `.rail-body` | app → redesign → rail |
| `.msg-label` | app → stage1 → redesign |
| `.bubble` | app → redesign → electron |
| `.composer` | app → redesign → motion |
| `.composer textarea` | app → redesign → electron |
| `.composer-bar` | app → redesign → electron |
| `.status` | app → redesign → electron |
| `.md code` | stage1 → redesign → chat |
| `.md pre` | stage1 → redesign → chat |
| `.modal-scrim` | stage1 → redesign → motion |
| `.modal` | stage1 → redesign → motion |
| `.settings` | redesign → motion → settings |
| `.settings-scrim` | redesign → motion → settings |
| `html[data-theme='light']` | tokens → motion |
| `body` | tokens → electron |
| `::-webkit-scrollbar` | tokens → redesign |
| `::-webkit-scrollbar-thumb` | tokens → redesign |
| `::-webkit-scrollbar-thumb:hover` | tokens → redesign |
| `.tb-right` | app → redesign |
| `.search` | app → redesign |
| `.item` | app → redesign |
| `.item.sel` | app → redesign |
| `.item.sel .ico` | app → redesign |
| `.rail-foot` | app → redesign |
| `.center` | app → stage1 |
| `.stream` | app → redesign |
| `.stream-inner` | app → redesign |
| `.msg` | app → redesign |
| `.msg.user .gutter` | app → redesign |
| `.think` | app → redesign |
| `.think summary` | app → redesign |
| `.composer-wrap` | app → redesign |
| `.composer:focus-within` | app → redesign |
| `.composer textarea::placeholder` | app → redesign |
| `.send` | app → redesign |
| `.send:disabled` | app → redesign |
| `.cursor` | app → redesign |
| `.sect` | app → redesign |
| `.card` | app → redesign |
| `.connbar` | stage1 → motion |
| `.empty-stream` | stage1 → redesign |
| `.empty-mark` | stage1 → redesign |
| `.empty-title` | stage1 → redesign |
| `.empty-hint` | stage1 → redesign |
| `.cursor-inline` | stage1 → redesign |
| `.md p` | stage1 → chat |
| `.md pre code` | stage1 → highlight |
| `.notices` | stage1 → motion |
| `.notice` | stage1 → motion |
| `.notice:hover` | stage1 → motion |
| `.logdrawer` | stage1 → motion |
| `.stream-row` | stage1 → redesign |
| `.rail-empty` | stage1 → redesign |
| `.composer-wrap.dropping .composer` | stage2 → redesign |
| `.browser-tabs` | redesign → browser |
| `.browser-tab` | redesign → browser |
| `.browser-new-tab` | redesign → browser |
| `.browser-toolbar` | redesign → browser |
| `.browser-nav` | redesign → browser |
| `.browser-close` | redesign → browser |
| `.browser-nav.browser-chrome` | redesign → browser |
| `.browser-address` | redesign → browser |
| `.browser-address input` | redesign → browser |
| `.app.rail-off .rail` | redesign → rail |
| `.rail-top` | redesign → rail |
| `.rail-action` | redesign → rail |
| `.rail-section` | redesign → rail |
| `.proj-head` | redesign → rail |
| `.proj-path` | redesign → rail |
| `.srow-row` | redesign → rail |
| `.srow-text` | redesign → rail |
| `.srow-line` | redesign → rail |
| `.srow-origin` | redesign → rail |
| `.srow-menu-path` | redesign → rail |
| `.turn` | redesign → motion |
| `.srow-name` | redesign → rail |
| `.srow-wrap .srow-time` | redesign → rail |
| `.mt-pop` | redesign → motion |
| `.srow-menu` | redesign → motion |
| `.row-menu` | redesign → motion |
| `.prose` | redesign → electron |
| `.rp-body` | redesign → tools |
| `.rp-sec` | redesign → tools |
| `.rp-sec-head` | redesign → tools |
| `.rp-sec-body` | redesign → tools |
| `.rp-quota-plan` | redesign → tools |
| `.rp-meter i` | redesign → motion |
| `.rp-fs` | redesign → tools |
| `.rp-fs-row` | redesign → tools |
| `.rp-fs-size` | redesign → tools |
| `.rp-meter` | redesign → motion |
| `.rp-todo.active` | redesign → motion |
| `.rp-todo.active .rp-box` | redesign → motion |
| `.rp-todo.active .rp-text` | redesign → motion |
| `.rp-now-spin` | redesign → motion |
| `.outline-preview` | redesign → motion |
| `.op-title` | redesign → motion |
| `.op-answer` | redesign → motion |
| `.op-empty` | redesign → motion |

