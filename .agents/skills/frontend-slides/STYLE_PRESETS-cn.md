# 样式预设参考

为 `frontend-slides` 精心策划的视觉样式。

本文件用于：
- 必须的视口适配基础 CSS
- 预设选择与风格映射
- CSS 注意事项与验证规则

仅使用抽象形状。除非用户明确要求，否则避免使用插图。

## 视口适配是硬性要求

每个幻灯片必须完全适配单个视口。

### 黄金法则

```text
每个幻灯片 = 恰好一个视口高度。
内容过多 = 拆分为更多幻灯片。
绝对不要在幻灯片内部滚动。
```

### 密度限制

| 幻灯片类型 | 最大内容量 |
|------------|-----------------|
| 标题页 | 1个主标题 + 1个副标题 + 可选标语 |
| 内容页 | 1个标题 + 4-6个项目符号或2个段落 |
| 功能网格 | 最多6张卡片 |
| 代码页 | 最多8-10行代码 |
| 引述页 | 1条引语 + 署名 |
| 图片页 | 1张图片，理想高度不超过60vh |

## 强制基础 CSS

请将此代码块复制到每个生成的演示文稿中，然后在此基础上进行主题定制。

```css
/* ===========================================
   VIEWPORT FITTING: MANDATORY BASE STYLES
   =========================================== */

html, body {
    height: 100%;
    overflow-x: hidden;
}

html {
    scroll-snap-type: y mandatory;
    scroll-behavior: smooth;
}

.slide {
    width: 100vw;
    height: 100vh;
    height: 100dvh;
    overflow: hidden;
    scroll-snap-align: start;
    display: flex;
    flex-direction: column;
    position: relative;
}

.slide-content {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    max-height: 100%;
    overflow: hidden;
    padding: var(--slide-padding);
}

:root {
    --title-size: clamp(1.5rem, 5vw, 4rem);
    --h2-size: clamp(1.25rem, 3.5vw, 2.5rem);
    --h3-size: clamp(1rem, 2.5vw, 1.75rem);
    --body-size: clamp(0.75rem, 1.5vw, 1.125rem);
    --small-size: clamp(0.65rem, 1vw, 0.875rem);

    --slide-padding: clamp(1rem, 4vw, 4rem);
    --content-gap: clamp(0.5rem, 2vw, 2rem);
    --element-gap: clamp(0.25rem, 1vw, 1rem);
}

.card, .container, .content-box {
    max-width: min(90vw, 1000px);
    max-height: min(80vh, 700px);
}

.feature-list, .bullet-list {
    gap: clamp(0.4rem, 1vh, 1rem);
}

.feature-list li, .bullet-list li {
    font-size: var(--body-size);
    line-height: 1.4;
}

.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
    gap: clamp(0.5rem, 1.5vw, 1rem);
}

img, .image-container {
    max-width: 100%;
    max-height: min(50vh, 400px);
    object-fit: contain;
}

@media (max-height: 700px) {
    :root {
        --slide-padding: clamp(0.75rem, 3vw, 2rem);
        --content-gap: clamp(0.4rem, 1.5vw, 1rem);
        --title-size: clamp(1.25rem, 4.5vw, 2.5rem);
        --h2-size: clamp(1rem, 3vw, 1.75rem);
    }
}

@media (max-height: 600px) {
    :root {
        --slide-padding: clamp(0.5rem, 2.5vw, 1.5rem);
        --content-gap: clamp(0.3rem, 1vw, 0.75rem);
        --title-size: clamp(1.1rem, 4vw, 2rem);
        --body-size: clamp(0.7rem, 1.2vw, 0.95rem);
    }

    .nav-dots, .keyboard-hint, .decorative {
        display: none;
    }
}

@media (max-height: 500px) {
    :root {
        --slide-padding: clamp(0.4rem, 2vw, 1rem);
        --title-size: clamp(1rem, 3.5vw, 1.5rem);
        --h2-size: clamp(0.9rem, 2.5vw, 1.25rem);
        --body-size: clamp(0.65rem, 1vw, 0.85rem);
    }
}

@media (max-width: 600px) {
    :root {
        --title-size: clamp(1.25rem, 7vw, 2.5rem);
    }

    .grid {
        grid-template-columns: 1fr;
    }
}

@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.2s !important;
    }

    html {
        scroll-behavior: auto;
    }
}
```

## 视口检查清单

- 每个 `.slide` 都设置了 `height: 100vh`、`height: 100dvh` 和 `overflow: hidden`
- 所有排版使用 `clamp()`
- 所有间距使用 `clamp()` 或视口单位
- 图片设置了 `max-height` 限制
- 网格使用 `auto-fit` + `minmax()` 实现自适应
- 包含 `700px`、`600px` 和 `500px` 的低高度断点
- 如果感觉内容拥挤，请拆分幻灯片

## 风格与预设映射表

| 风格 | 适用预设 |
|------|--------------|
| 专业/自信 | Bold Signal, Electric Studio, Dark Botanical |
| 兴奋/活力 | Creative Voltage, Neon Cyber, Split Pastel |
| 平静/专注 | Notebook Tabs, Paper & Ink, Swiss Modern |
| 灵感/触动 | Dark Botanical, Vintage Editorial, Pastel Geometry |

## 预设目录

### 1. Bold Signal

- 特点：自信、高冲击力、适合主题演讲
- 适用场景：融资演示（pitch deck）、产品发布、官方声明
- 字体：Archivo Black + Space Grotesk
- 配色：炭黑色底色、亮橙色焦点卡片、清晰白色文字
- 标志性设计：超大号章节编号、深色背景上的高对比度卡片

### 2. Electric Studio

- 特点：简洁、大胆、专业级打磨
- 适用场景：客户演示、战略复盘
- 字体：仅使用 Manrope
- 配色：黑、白、饱和钴蓝色点缀
- 标志性设计：双面板拆分、锐利的编辑级对齐

### 3. Creative Voltage

- 特点：活力充沛、复古现代、轻松自信
- 适用场景：创意工作室、品牌设计、产品叙事
- 字体：Syne + Space Mono
- 配色：电光蓝、霓虹黄、深藏青
- 标志性设计：半色调纹理、徽章、强烈对比

### 4. Dark Botanical

- 特点：优雅、高端、氛围感
- 适用场景：奢侈品牌、深度叙事、高端产品演示
- 字体：Cormorant + IBM Plex Sans
- 配色：近黑色、暖象牙白、腮红粉、金色、陶土色
- 标志性设计：模糊抽象圆形、细线边框、克制的动效

### 5. Notebook Tabs

- 特点：编辑风格、条理清晰、触感质感
- 适用场景：报告、复盘、结构化叙事
- 字体：Bodoni Moda + DM Sans
- 配色：炭黑背景上的奶油色纸张搭配淡色标签
- 标志性设计：纸张效果、彩色侧边标签、装订细节

### 6. Pastel Geometry

- 特点：亲和、现代、友好
- 适用场景：产品概述、新手引导、轻量化品牌演示
- 字体：仅使用 Plus Jakarta Sans
- 配色：淡蓝色背景、奶油色卡片、柔和粉/薄荷绿/淡紫色点缀
- 标志性设计：竖向胶囊形状、圆角卡片、柔和阴影

### 7. Split Pastel

- 特点：活泼、现代、创意
- 适用场景：机构介绍、工作坊、作品集
- 字体：仅使用 Outfit
- 配色：桃色+淡紫色拆分背景搭配薄荷绿徽章
- 标志性设计：拆分背景、圆角标签、轻量网格覆盖层

### 8. Vintage Editorial

- 特点：诙谐、富有个性、杂志风格
- 适用场景：个人品牌、观点类演讲、故事叙事
- 字体：Fraunces + Work Sans
- 配色：奶油色、炭黑色、暖调暗色系点缀
- 标志性设计：几何装饰、边框标注、醒目衬线大标题

### 9. Neon Cyber

- 特点：未来感、科技感、动态
- 适用场景：AI、基础设施、开发者工具、未来趋势类演讲
- 字体：Clash Display + Satoshi
- 配色：午夜藏青、青色、品红色
- 标志性设计：发光效果、粒子、网格、数据雷达动效

### 10. Terminal Green

- 特点：面向开发者、简洁黑客风格
- 适用场景：API、CLI 工具、工程演示
- 字体：仅使用 JetBrains Mono
- 配色：GitHub 暗色主题 + 终端绿色
- 标志性设计：扫描线、命令行边框、精确的等宽节奏

### 11. Swiss Modern

- 特点：极简、精确、数据导向
- 适用场景：企业演示、产品战略、分析报告
- 字体：Archivo + Nunito
- 配色：白、黑、信号红
- 标志性设计：可见网格、不对称布局、几何严谨性

### 12. Paper & Ink

- 特点：文艺、有深度、故事驱动
- 适用场景：随笔、主题演讲叙事、宣言类演示
- 字体：Cormorant Garamond + Source Serif 4
- 配色：暖奶油色、炭黑色、深红色点缀
- 标志性设计：引语突出、首字下沉、优雅分隔线

## 直接选择提示

如果用户已经知道自己想要的样式，请让他们直接从上述预设名称中选择，无需强制生成预览。

## 动效风格映射

| 期望感受 | 动效方向 |
|---------|------------------|
| 戏剧/电影感 | 缓慢淡入淡出、视差、大尺寸缩放进入 |
| 科技/未来感 | 发光、粒子、网格动效、文字打散效果 |
| 活泼/友好 | 弹性缓动、圆角形状、浮动动效 |
| 专业/商务 | 微妙的200-300ms过渡、简洁幻灯片切换 |
| 平静/极简 | 非常克制的动效、留白优先 |
| 编辑/杂志风格 | 强烈的层级感、交错的文字和图片联动 |

## CSS 注意事项：负值函数

绝对不要这样写：

```css
right: -clamp(28px, 3.5vw, 44px);
margin-left: -min(10vw, 100px);
```

浏览器会静默忽略这些写法。

请始终使用以下写法：

```css
right: calc(-1 * clamp(28px, 3.5vw, 44px));
margin-left: calc(-1 * min(10vw, 100px));
```

## 验证尺寸

至少在以下分辨率下测试：
- 桌面端：`1920x1080`、`1440x900`、`1280x720`
- 平板：`1024x768`、`768x1024`
- 手机：`375x667`、`414x896`
- 横屏手机：`667x375`、`896x414`

## 反模式

不要使用：
- 白底紫色的初创公司模板
- Inter / Roboto / Arial 作为视觉字体，除非用户明确要求实用中性风格
- 密集项目符号、过小字体、需要滚动的代码块
- 抽象几何图形足以满足需求时的装饰性插图
