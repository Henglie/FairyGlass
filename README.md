# FairyGlass

> 微蓝暗色 · 跨浏览器液态玻璃设计系统（Visual Identity System）

FairyGlass 是从「剪贴板里有什么？」与「优雅快存 FairySave」两个项目中抽取、统一的一套视觉系统，作为作者今后个人项目的主题底座。任何项目接入两个 CSS 文件即可获得一致的微蓝暗色液态玻璃外观；折射引擎可选。

## 设计基调（不可漂移）

- **微蓝暗色底**，绝不用纯黑、绝不用紫。蓝色来自背后光斑，不是给玻璃染色。
- **玻璃是中性无色的**（白色极低不透明度填充），通透感来自背后背景被采样/折射，而非白底。
- **动效高初速、果断、迅速滑停，干净利落，不回弹**。只动 `transform`/`opacity`。
- **不用彩色 emoji**；保留 `● ✓ ← ▼` 等黑白几何符号。
- 标题/标语用衬线（Noto Serif SC），正文/界面用无衬线。

## 文件结构

```
FairyGlass/
  css/tokens.css      令牌层：颜色 / 字体 / 玻璃材质 / 动效 / 背景 / 滚动条 / 入场动画
  css/glass.css       组件层：完整 VIS（外壳 / 导航 / 折叠 / 表单 / 反馈 / 数据展示 / 折射层样式）
  js/liquidGlass.js   折射引擎（可选，UMD，挂 window.FairyGlass）
  demo.html           全组件预览 + 折射实例（侧栏分面板展示）
  README.md           本文件
  AGENTS.md           给接手 AI 的硬性规范
```

## 快速接入

### 1. 普通网页（推荐）

```html
<link rel="stylesheet" href="FairyGlass/css/tokens.css" />
<link rel="stylesheet" href="FairyGlass/css/glass.css" />
<!-- 折射可选 -->
<script src="FairyGlass/js/liquidGlass.js"></script>
<script>FairyGlass.hydrateGlass(document);</script>
```

只引前两个 CSS 就已经是完整的磨砂玻璃主题；折射引擎是锦上添花。

### 2. file:// / webview（如 FairySave）

webview 加载本地 `file://`，**经典 `<script>` 不能用 ES 模块的 `import`**。`liquidGlass.js` 是纯 UMD，正好直接 `<script>` 引入，挂到 `window.FairyGlass`。CSS 建议内联进单文件 HTML（webview 下相对路径与缓存都更省心）。

### 3. ES 模块项目（如剪贴板项目）

```js
import "./FairyGlass/js/liquidGlass.js";  // 副作用导入，挂 window.FairyGlass
const { hydrateGlass } = window.FairyGlass;
hydrateGlass(document);
```

## 折射引擎用法

给元素加 `data-glass="<preset>"`，再调 `FairyGlass.hydrateGlass(root)` 批量注入：

| preset | 用途 | 折射强度 |
|---|---|---|
| `button` | 主按钮 / CTA | 最强（bezel 22 / scale 72） |
| `card` | 浮空卡片 / 弹窗（默认） | 中（bezel 18 / scale 60） |
| `chip` | 小药丸 / 胶囊 | 轻（bezel 12 / scale 44） |
| `bar` | 贴边长条（顶/底/侧栏） | **跳过折射**，保留磨砂 |

`hydrateGlass` 幂等（已处理元素跳过），且自动跳过 `bar` 与滚动容器（`overflow-y:auto/scroll`）。

### 折射的铁律（性能 + 黑边底线）

折射透镜（`filter:url()` 折射层）**只给浮空、数量少**的元素。原因：

1. **贴视口边缘**的长条会采到视口外纯深色 → **黑边**。所以顶/底/侧栏走磨砂（`bar` 或 `.fg-bar`）。
2. **滚动容器**的折射层会随内容滚走并引发重绘 → 自动跳过。
3. **大批量密集卡片**（几十上百张）上折射会拖垮性能 → 走磨砂（`.fg-card` 或 `.fg-card--flat`）。

一句话：**浮空元素用折射透镜，贴边长条 / 滚动容器 / 密集卡片用磨砂**。

## 跨浏览器原理（为什么 Firefox 也有折射）

折射走 `filter:url()` 而非 `backdrop-filter:url()`：

- `backdrop-filter:url()` 折射「元素背后已渲染像素」→ **仅 Chromium 支持**，Firefox 永远没折射。
- `filter:url()` 作用于「元素自身内容」→ **所有引擎都支持** `feDisplacementMap`。

做法：给玻璃元素注入 `.lg-refract` 子层，该层用 `background-attachment:fixed` 画一份与页面背景（`--glow-field`）按视口对齐的光斑拷贝，再施加 SVG 位移图。于是「透过玻璃看到的光斑」与真实背景严丝合缝，仅边缘被弯曲，三系引擎跑同一份代码。

## 来源

- 剪贴板项目（WhatsInYourClipboard）：液态玻璃主题的原始打磨地（折射引擎、动效令牌、跨浏览器统一均在此定型）。
- 优雅快存（FairySave）：仪表盘形态的实战检验（侧边栏 + 密集软件卡 + 开关）。

版本 v0.2。组件库已补齐为完整 VIS（应用外壳、导航、折叠、表单、反馈、数据展示一整套），可独立支撑一个完整界面。
