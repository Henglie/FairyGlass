# PROGRESS · FairyGlass

> 项目进度与 AI 接管文档。**人看的介绍在 [README.md](./README.md)，本文件只给接手的 AI / 开发者。**
> 最后更新：2026-06-22

---

## ▍本文档规范（四个项目统一，接手前先读这一段）

作者把 FairyGlass / FairySave / WhatsInYourClipboard / ElegantHLK 四个项目纳入统一管理，
每个项目固定两份 md，文件名、结构完全一致：

| 文件 | 给谁看 | 写什么 |
|---|---|---|
| `README.md` | 人（使用者） | 项目介绍、功能、安装使用 |
| `PROGRESS.md` | AI / 开发者 | 进度、结构、踩坑、待办（**就是本文件**） |

**AI 接管流程（任意 AI 接手任意一个项目，照做即可）：**
1. 读本文件「技术栈红线」+「踩坑记录」两节 —— 这是别人用时间换来的，不要重新踩。
2. 看「项目结构」定位代码，看「已完成 / 待办」找当前进度。
3. 动手前确认「用户偏好」（提交署名、emoji、语言等硬约束）。
4. 干完活**只更新本文件的状态行**：把做完的 `[ ]` 改 `[x]`，新坑补进「踩坑记录」。

**写作规范（省 token，不许跑偏）：**
- 任务用 `[x]` 已完成 / `[ ]` 待办 / `[~]` 进行中 三种标记，一眼可扫。
- **不写流水账、不留历史快照式日志**。完成的任务并进「已完成」一句话带过，别堆每轮对话。
- 踩坑记录 = 结论 + 为什么 + 怎么绕开，三句以内。不要贴长堆栈。
- 写「关键文件:行号」让人能直接跳，不要大段粘代码。
- 中文书写（作者中文交流）。

---

## ▍这是什么

FairyGlass 是一套**微蓝暗色液态玻璃设计系统**，不是某个应用。它是作者所有个人项目的视觉底座，
只产出三样东西：令牌（`tokens.css`）、组件（`glass.css`）、折射引擎（`liquidGlass.js`）。
下游项目（剪贴板、FairySave 等）引入它，**不反向往里塞业务逻辑**。

> 真理来源是 FairyGlass。下游若与本系统冲突，以 FairyGlass 为准，并把分歧记回这里。

---

## ▍技术栈红线（设计基调不可漂移，违反即返工）

1. **微蓝暗色**。底色 `--bg-0/1/2` 是带蓝的深夜色，**绝不用纯黑**。
2. **绝不用紫**。强调色是天蓝 `--accent`；光斑 `--glow-a/b/c` 是蓝/青/深蓝。`--violet` 仅作「需与蓝区分的次要语义标签」（如插件标记），不得用作主色调。
3. **玻璃中性无色**。玻璃填充是白色极低不透明度（`--glass-fill`），**不给玻璃本体染蓝**。蓝只来自背后 `--glow-field` 光斑被采样/折射。
4. **动效不回弹**（默认）。高初速 → 迅速滑停。用 `--ease-out-quint`/`--ease-out-back`/`--ease-snap`，**禁止过冲/弹簧回弹**。只动 `transform`/`opacity`（GPU 合成）。
   - **受控例外**：`--ease-spring-back`（`cubic-bezier(.34,1.56,.64,1)`，y>1 段过冲）仅给**主动要求弹性**的微交互用，如 `.fg-search` 聚焦展宽。**不得污染主操作**（按钮/卡片/抽屉/入场），否则整体手感变松。
5. **不用彩色 emoji**。保留 `● ✓ ← ▼ ×` 等黑白几何符号（作者明确反感 emoji）。
6. **衬线给标题/标语，无衬线给界面**。`--font-mono` 里 CJK sans 必须排在 `monospace` **之前**，否则 Windows 把 CJK 兜底成宋体衬线（已知 bug）。

---

## ▍项目结构

```
FairyGlass/
  css/tokens.css      令牌层：颜色/字体/玻璃材质/动效/背景/滚动条/入场动画
  css/glass.css       组件层：基础件 + 完整 VIS（外壳/导航/折叠/表单/反馈/数据展示，前缀统一 fg-）
  js/liquidGlass.js   折射引擎（可选，UMD，挂 window.FairyGlass）
  demo.html           全组件预览 + 折射实例（改动必须在这里补预览实例）
  README.md           人看的介绍 + 接入方式
  PROGRESS.md         本文件
  AGENTS.md           指回本文件的指针（兼容 AGENTS 自动发现）
```

---

## ▍踩坑记录（折射引擎，违反任何一条都会黑边或掉帧）

- **折射只给浮空、数量少的元素**：主按钮、hero 卡、弹窗、单独强调的胶囊。
- **贴视口边缘的长条（顶/底/侧栏）禁止折射** → 用 `data-glass="bar"` 或 `.fg-bar`。折射层的 `fixed` 光斑图在视口外是纯深色，贴边会采到 → **黑边**。
- **滚动容器禁止折射**（`hydrateGlass` 已自动跳过 `overflow-y:auto/scroll`）。折射层会随内容滚走并引发重绘。
- **大批量密集卡片（几十上百张）禁止折射** → 走磨砂 `.fg-card` 或更省的 `.fg-card--flat`。
- **背景必须静止**。`body::before` 的 `--glow-field` 绝不能加动画 —— 它是所有 `backdrop-filter` 的采样源，一动就逼每块玻璃每帧重采样+重模糊 → Chromium 持续掉帧。（剪贴板项目曾因 `drift` 动画掉帧，已移除，别加回来。）
- **入场动画终态必须 `transform:none`**，不能是 `scale(1)`。`fill:both` 残留的非 none transform 会建立包含块，永久破坏内部折射层 `background-attachment:fixed` 的视口对齐。
- **liquidGlass.js 必须是 UMD，文件里绝不能出现 `export` 语句**（注释里提到无妨）。含 `export` 的文件被经典 `<script>`（FairySave 的 webview `file://` 场景）加载会抛 `SyntaxError`，整段不执行。改引擎后用 `grep -nE '^\s*export ' js/liquidGlass.js` 自检。统一通过 `window.FairyGlass` 取用。
- **跨浏览器原理**：折射走 `filter:url()`（作用于元素自身内容，三系引擎都支持 `feDisplacementMap`），**不是** `backdrop-filter:url()`（仅 Chromium）。给玻璃注入 `.lg-refract` 子层，用 `background-attachment:fixed` 画一份与 `--glow-field` 视口对齐的光斑拷贝再施 SVG 位移图，于是透过玻璃看到的光斑与真实背景严丝合缝。

---

## ▍改动流程

1. 令牌变更（颜色/圆角/动效）只改 `tokens.css`，下游自动继承。
2. 新增组件加到 `glass.css`，类名前缀统一 `fg-`。
3. 任何改动后在 `demo.html` 补一个预览实例，并用浏览器实际打开核对（折射效果无法靠读代码验证）。
4. **跨浏览器自测**：至少 Chromium + Firefox 各开一次 `demo.html`，确认折射两边都出现、无黑边。

**下游同步：**
- **剪贴板（WhatsInYourClipboard）**：ES 模块，`import "./liquidGlass.js"` 副作用导入后从 `window.FairyGlass` 解构。它的 `css/` 是 FairyGlass 的原始母本，在那边继续打磨记得回灌 FairyGlass。
- **FairySave**：webview/`file://`，主题 CSS 内联进单文件 `ui/index.html`，引擎用经典 `<script>`。改 ui 后必须手动 `cp -r ui/* build/debug/bin/ui/`。

---

## ▍已完成

- [x] 令牌层 `tokens.css`：颜色/字体/玻璃材质/动效/背景/滚动条/入场动画全套。
- [x] 组件层 `glass.css`：卡/按钮/药丸/开关/输入/导航/弹窗/toast/徽章。
- [x] 折射引擎 `liquidGlass.js`：UMD，`hydrateGlass` 幂等，自动跳过 bar/滚动容器。
- [x] 跨浏览器折射统一（Chromium + Firefox + 国产内核像素级一致，`filter:url()` 方案）。
- [x] `demo.html` 全组件预览 + 折射实例。
- [x] 从剪贴板项目 + FairySave 抽取统一为独立设计系统。
- [x] git 仓库已建。
- [x] **`.fg-search` 弹性搜索框**：聚焦时宽度【弹一下】展开（`--ease-spring-back` 受控回弹），描边泛蓝 + 一圈淡蓝辉光。颜色/辉光仍走 quint 顺滑，避免颜色跟着抖。两个 CSS 变量 `--fg-search-width` / `--fg-search-width-focus` 控制收/展宽度，外部覆写即可。
- [x] **`.fg-drawer` / `.fg-drawer__inner` 抽屉**：`grid-template-rows: 0fr → 1fr` 高度过渡。选这条而不是 max-height 是因为后者必须写「假上限」，内容低则留白长、内容高又被截。0fr→1fr 让网格行的【自然高度】参与过渡，无视觉延迟。内层负责 `overflow:hidden` + `min-height:0`（让网格行真能压到 0）。
- [x] **`--ease-spring-back` 令牌**：`cubic-bezier(.34, 1.56, .64, 1)`，y>1 段过冲一下再回落。这是「不回弹」红线的【受控例外】，仅给主动要求弹性的微交互用（搜索框聚焦展宽），**不污染主操作**（按钮/卡片/抽屉/入场）。红线已在第 4 条更新。
- [x] **`.fg-card--accent` 降饱和**：从 0.14 / 0.35 降到 0.05 / 0.2。原值玻璃片偏鲜艳显廉价，新值是 FairySave 实战调出的克制档。下游若已用 0.14 老值，覆写无所谓；新接入直接用即可。
- [x] **demo 补 preset 参数表**：折射区下一张表列出 button/card/chip/bar 的 bezel/scale/blur/dispersion 档位（数值即 `liquidGlass.js:288-293` 硬编码），调用方挑参用。挑不准用 `card` 默认，要更猛传 opts 覆写。
- [x] **demo 补 `.fg-bar` 贴边长条预览**：药丸区后加一段磨砂长条实例，写明「不上折射、防黑边」的核心约定（之前 demo 通篇缺这个组件实例）。
- [x] **VIS 补全（v0.2）**：在基础组件之外补齐真实应用必用的「第二梯队」组件，glass.css 从 ~580 行扩到 ~1340 行。新增：应用外壳（`.fg-app` / `.fg-sidebar` / `.fg-topbar` / `.fg-main` / `.fg-main__scroll`，**浮块式**——侧栏/顶栏/内容区各自独立圆角玻璃，块间 gap、四周 padding，对齐 FairySave `#app` 形态，不做贴边方角）、折叠手风琴 `.fg-accordion`（剪贴板 `.cat-section` / 快存 drawer-head 的统一沉淀，grid 0fr→1fr）、标签页 `.fg-tabs`、分段控件 `.fg-segmented`、下拉 `.fg-select` + 浮层菜单 `.fg-menu`、文本域 `.fg-textarea`、复选/单选 `.fg-check` / `.fg-radio`（玻璃自绘）、标签 `.fg-tag`（5 语义色 + violet 次要）、tooltip、进度条 `.fg-progress`（含 indeterminate）、spinner、骨架屏、头像、提示横幅 `.fg-alert`、数据表格 `.fg-table`、列表 `.fg-list`、面包屑、分页、空状态、分隔线、键帽 `.fg-kbd`、步进器 `.fg-stepper`（正式组件化）。demo 重构为五面板侧栏导航全展示。
- [x] **修 `.fg-badge--privacy` 方角 bug**：早期版本圆角只写在 `.fg-badge` 上，privacy 徽章单用时拿不到 `border-radius` → 方角。已补 `border-radius:var(--radius-pill)` + border + font-size，独立可用。
- [x] **修 `.fg-num` 残留原生 spinner**：此前只去了 webkit 上下箭头，Firefox 原生 spinner 仍露在数字旁显「原始 UI」。已补 `appearance:textfield` 跨浏览器去除。
- [x] **「玻璃太艳」定位 + 解法**：FairyGlass 材质与剪贴板 `base.css` 逐字节相同（剪贴板是认可母本），艳不来自材质而来自**上下文**——980px 裸列把卡片压在视口中心最强叠加光斑上，`saturate(180%)` 顶起透光的蓝。解法不动材质（动则背离母本），而是套应用外壳让内容落回被前层玻璃过滤的弱光区。**材质红线不变**。

## ▍待办
- [ ] **回灌检查**：剪贴板项目的 `.cat-body`（grid 0fr→1fr 抽屉）已经是 `.fg-drawer` 的事实原型，但下游仍用 `.cat-body` 类名。下次接手剪贴板时可考虑把 `.cat-body` 重命名为 `.fg-drawer` 类（或并行支持），统一命名空间。**不强制改**，下游稳定运行的别动。
- [ ] **下游同步检查**：FairySave `ui/index.html` 的 `<style>` 是 FairyGlass 令牌的【内联复刻】（webview file:// 约束下没法 link 外部 CSS，复刻是预期）。若改 FairyGlass 的令牌/红线，记得一并改 FairySave 的 inline 副本，否则会漂移。
  - 当前漂移项（已对齐，仅记录）：FairySave 已加 `--ease-spring-back` 令牌、`.search` 已是弹性版（`--ease-spring-back`，宽度 240→300）、`.ctrl-card.master` 已降饱和（0.05/0.2）。这些在 FairyGlass 也已落定，两边一致。
- [ ] **demo 跨浏览器验证**：`.fg-search`（聚焦弹性展宽）/ `.fg-drawer`（0fr→1fr 抽屉）/ 本轮新增的 `.fg-bar` 长条与 preset 参数表，AI 没法验证视觉。下次有人值守时打开 `demo.html`，Chromium + Firefox 各一遍：核验回弹/抽屉顺滑无截断、`.fg-bar` 磨砂无黑边、折射两边都出现。

---

## ▍接手快捷入口（给下一个 AI）

- 想看本系统设计基调与红线 → 上面「技术栈红线」+「踩坑记录」两节，**先读再动手**。
- 想知道弹性微交互边界 → 红线第 4 条，受控例外只给 `--ease-spring-back`，仅微交互可用。
- 想看新组件怎么用 → `demo.html` 末尾「弹性搜索框」「抽屉」两节有可交互预览。
- 改完动效/颜色 → 下游 FairySave `ui/index.html` 内联 CSS 段需手动同步（webview 复刻约束）。

---

## ▍用户偏好（硬约束，四项目通用）

- 作者：恒烈 / Henglie / EternalBlaze。Windows 桌面开发者，熟 GDI+，中文交流。
- **反感 emoji**：回复和界面都克制，用纯文字/黑白几何符号。
- git 提交用 `Henglie <ebhenglie@gmail.com>` 身份，**绝不出现 Claude 署名 / Co-Authored-By / AI 痕迹**。
- 讲清「为什么」，给推荐而非罗列；能定的直接做，不确定才问。
