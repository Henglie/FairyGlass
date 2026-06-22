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
4. **动效不回弹**。高初速 → 迅速滑停。用 `--ease-out-quint`/`--ease-out-back`/`--ease-snap`，**禁止过冲/弹簧回弹**。只动 `transform`/`opacity`（GPU 合成）。
5. **不用彩色 emoji**。保留 `● ✓ ← ▼ ×` 等黑白几何符号（作者明确反感 emoji）。
6. **衬线给标题/标语，无衬线给界面**。`--font-mono` 里 CJK sans 必须排在 `monospace` **之前**，否则 Windows 把 CJK 兜底成宋体衬线（已知 bug）。

---

## ▍项目结构

```
FairyGlass/
  css/tokens.css      令牌层：颜色/字体/玻璃材质/动效/背景/滚动条/入场动画
  css/glass.css       组件层：卡/按钮/药丸/开关/输入/导航/弹窗/toast/徽章（类名前缀统一 fg-）
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

## ▍待办

- [ ] 建 git 仓库（目前四项目中唯一没有 .git 的）。
- [ ] 沉淀 `data-glass` preset 的参数表到 demo（button/card/chip/bar 的 bezel/scale 数值）。
- [ ] 把下游已打磨但未回灌的样式（若有）合并回 tokens/glass。

---

## ▍用户偏好（硬约束，四项目通用）

- 作者：恒烈 / Henglie / EternalBlaze。Windows 桌面开发者，熟 GDI+，中文交流。
- **反感 emoji**：回复和界面都克制，用纯文字/黑白几何符号。
- git 提交用 `Henglie <ebhenglie@gmail.com>` 身份，**绝不出现 Claude 署名 / Co-Authored-By / AI 痕迹**。
- 讲清「为什么」，给推荐而非罗列；能定的直接做，不确定才问。
