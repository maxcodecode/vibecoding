# VibePort 维护说明文档

这份文档说明如何维护本站：改配色、加功能分类、加构建工具、加项目、加广告、改统计代码等。

---

## 0. 站点结构与文件说明

| 文件 / 目录 | 作用 |
| --- | --- |
| `index.html` | 首页（项目列表 + 筛选 + 侧边栏「开发者故事 / 常见问题」） |
| `submit.html` | 提交项目页（复制邮件内容） |
| `privacy.html` | 隐私政策页 |
| `assets/vibeport.css` | 全站样式（明暗主题、配色、组件、响应式） |
| `assets/projects.js` | **项目数据源**（所有项目都在这里） |
| `assets/app.js` | 首页逻辑（筛选下拉框、分页、列表渲染、随便逛逛） |
| `assets/i18n.js` | 多语言文案（en / zh / fr） |
| `assets/theme.js` | 明暗主题切换 + Ctrl/Cmd+K 聚焦搜索 |
| `assets/ads.js` | 广告位配置 |
| `assets/stats.js` | 统计代码（Clarity / GA4） |
| `sitemap.xml` | 站点地图 |
| `llms.txt` | 面向 AI 的站点说明 |
| `zh/index.html`、`zh/submit.html` | 中文入口，自动跳转到 `/?lang=zh` |

> 语言说明：英文是主语言；下拉菜单可切换 中文 / Français。

---

## 1. 修改配色 / 主题

配色全部定义在 `assets/vibeport.css` 顶部的 `:root` 变量里，改一处全局生效。本站支持**明暗双主题**：

```css
:root {
    --bg-color: #FAFAF9;      /* 页面背景（浅色模式） */
    --card-bg: #FFFFFF;       /* 卡片背景 */
    --text-main: #1C1917;     /* 主文字 */
    --text-sub: #78716C;      /* 次要文字 */
    --primary: #6366F1;       /* 主色（靛） */
    --primary-hover: #4F46E5;
    --divider: #E7E5E4;       /* 边框/分割线 */
    ...
}

body.dark-theme {
    --bg-color: #0A0A0F;      /* 暗色模式背景 */
    --card-bg: #131318;
    --text-main: #F5F5F4;
    --primary: #8B5CF6;       /* 暗色模式主色（紫） */
    ...
}
```

- 改**主色（靛紫渐变）**：渐变分散在 `--primary`（用于普通态）以及 `.logo-icon`、`.hero h1 .highlight`、`.btn-submit`、`.footer-badge-icon` 里的 `linear-gradient(...)` 中，搜 `#6366F1` / `#8B5CF6` 即可定位。
- 改**背景色**：改 `--bg-color`（浅色）和 `body.dark-theme` 里的 `--bg-color`（暗色）。
- 主题切换按钮是 `#themeToggle`，逻辑在 `assets/theme.js`；选择结果存在 `localStorage['theme']`。

`404.html` 是独立页面，样式写在自己文件里，如需同步配色请单独改。

---

## 2. 添加一个“功能分类”

分类目前是**固定枚举**：`tool`（工具）、`fun`（娱乐）、`office`（办公）、`productivity`（效率）。
另外有一个特殊的“精选”（Featured）不是分类值，而是由项目里的 `isFeatured: true` 控制。

以新增分类 `design`（设计）为例，需要改 **4 个地方**：

### 2.1 在 `assets/app.js` 里加分类文案

找到 `CATEGORY_LABELS`，给三种语言各加一行：

```js
const CATEGORY_LABELS = {
    en: { tool: "Tool", fun: "Fun", office: "Office", productivity: "Productivity", design: "Design" },
    zh: { tool: "工具", fun: "娱乐", office: "办公", productivity: "效率", design: "设计" },
    fr: { tool: "Outil", fun: "Divertissement", office: "Bureau", productivity: "Productivité", design: "Design" }
};
```

### 2.2 在 `assets/app.js` 的 `renderFilterSelects()` 里加进筛选下拉框

找到 `catDefs` 数组，加一行（保持 `featured` 在最后）：

```js
const catDefs = [
    { key: 'tool', label: getCategoryLabel('tool') },
    { key: 'fun', label: getCategoryLabel('fun') },
    { key: 'office', label: getCategoryLabel('office') },
    { key: 'productivity', label: getCategoryLabel('productivity') },
    { key: 'design', label: getCategoryLabel('design') },   // ← 新增
    { key: 'featured', label: t('filter.featured') }
];
```

### 2.3 在 `assets/i18n.js` 里加“页脚链接”用到的文案

在 en / zh / fr 三个语言对象里，各加一个 `cat.design`：

```js
en: { ..., "cat.design": "Design", ... }
zh: { ..., "cat.design": "设计", ... }
fr: { ..., "cat.design": "Design", ... }
```

### 2.4 （可选）在页脚加一个快捷链接

在 `index.html`、`submit.html`、`privacy.html` 的「功能分类」列里加一行：

```html
<a href="/?category=design" class="footer-link" data-i18n="cat.design">Design</a>
```

### 2.5 使用该分类

在 `assets/projects.js` 里给项目设置 `category: "design"` 即可。

---

## 3. 添加一个“构建工具”（例如 DeepSeek Harness）

**好消息：筛选下拉框里的工具列表是自动生成的**——它从 `assets/projects.js` 里所有项目的 `tool` 字段自动汇总并计数，**不需要单独注册**。

所以“添加构建工具” = 给某个项目写上这个工具名：

```js
tool: "DeepSeek Harness",
```

只要有一个项目用了 `"DeepSeek Harness"`，它就会自动出现在首页「构建工具」下拉框里，并显示数量。

### （可选）在页脚加一个快捷链接

在 `index.html`、`submit.html`、`privacy.html` 的「构建工具」列里加一行。

> 注意：URL 里的空格要写成 `+`。

```html
<a href="/?tool=DeepSeek+Harness" class="footer-link">DeepSeek Harness</a>
```

---

## 4. 添加一个项目

打开 `assets/projects.js`，在 `const projects = [ ... ]` 数组里加一个对象。字段说明：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `id` | ✅ | 唯一编号，不要重复 |
| `name` | ✅ | 项目名称 |
| `desc` | ✅ | 英文描述 |
| `descZh` | 可选 | 中文描述 |
| `descFr` | 可选 | 法语描述 |
| `category` | ✅ | 分类：`tool` / `fun` / `office` / `productivity`（或你新加的分类） |
| `tool` | ✅ | 构建工具名，如 `"DeepSeek Harness"` |
| `buildTime` | 可选 | 构建耗时（小时，数字）；不填则不显示 ⏱ |
| `buildCost` | 可选 | 构建费用（字符串），如 `"$5"`；不填则不显示 💰 |
| `isFeatured` | 可选 | `true` = 计入「精选」筛选 |
| `link` | ✅ | 项目网址 |
| `iconGradient` | ✅ | 图标渐变背景，两个色值逗号分隔，如 `"#8B5CF6, #EC4899"` |
| `iconChar` | ✅ | 图标显示的首字母/字符 |

示例（顺便演示了“DeepSeek Harness”这个新构建工具）：

```js
{
    id: 6,
    name: "DeepSeek Notes",
    desc: "AI note-taking assistant built with DeepSeek Harness.",
    descZh: "用 DeepSeek Harness 构建的 AI 笔记助手。",
    descFr: "Assistant de prise de notes IA construit avec DeepSeek Harness.",
    category: "productivity",
    tool: "DeepSeek Harness",
    buildTime: 6,
    buildCost: "$5",
    isFeatured: false,
    link: "https://your-product.com",
    iconGradient: "#8B5CF6, #EC4899",
    iconChar: "D"
}
```

> 提示：目前项目里的 `link` 是占位符 `"#"`，上线前请替换成真实网址。

---

## 5. 添加广告代码

广告配置在 `assets/ads.js`，广告位容器在 `index.html`。

### 5.1 开启并填写你的 AdSense 代码

打开 `assets/ads.js`：

```js
window.ADS_CONFIG = {
    adsEnabled: true,          // ← 改成 true 才启用广告

    adTop: {                   // 列表顶部广告位
        client: "ca-pub-你的ID",
        slot:   "你的广告单元ID"
    },
    adBottom: {                // 列表底部广告位
        client: "ca-pub-你的ID",
        slot:   "你的广告单元ID"
    }
};
```

把 `client` 换成你的 AdSense 发布商 ID（形如 `ca-pub-1234567890123456`），`slot` 换成对应广告单元的 ID。

### 5.2 打开顶部广告位

首页 `index.html` 里，顶部广告位目前被注释掉了。去掉注释即可：

```html
<!-- 改前 -->
<!-- <div id="ad-top" class="ad-slot">Ad slot (top)</div> -->

<!-- 改后 -->
<div id="ad-top" class="ad-slot">Ad slot (top)</div>
```

底部广告位 `#ad-bottom` 默认已存在。启用后，脚本会自动把占位框替换成 AdSense 的 `<ins>` 标签。

> 提示：`adsEnabled` 为 `false` 时，页面只显示虚线占位框，不会请求任何广告，方便本地调试。

---

## 6. 修改统计代码

统计配置在 `assets/stats.js`，目前集成了 Microsoft Clarity，预留了 Google Analytics 4（GA4）。

### 6.1 修改 Microsoft Clarity（已启用）

把脚本里的项目 ID `yuyhnem3o5` 换成你自己的 Clarity 项目 ID：

```js
(function(c,l,a,r,i,t,y){ ... })("clarity", "script", "yuyhnem3o5");
//                                                ↑ 换成你的 Clarity ID
```

### 6.2 启用 Google Analytics 4（当前未启用）

GA4 目前是注释状态。要启用，把 `stats.js` 里这段改成你的 GA4 测量 ID（`G-XXXXXXXXXX`）：

```js
// 1) 动态加载 gtag.js
(function() {
    var s = document.createElement('script');
    s.async = true;
    s.src = 'https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX';
    document.head.appendChild(s);
})();

// 2) 初始化并配置
window.dataLayer = window.dataLayer || [];
function gtag(){ dataLayer.push(arguments); }
gtag('js', new Date());
gtag('config', 'G-XXXXXXXXXX');
```

把两处 `G-XXXXXXXXXX` 换成你的 GA4 测量 ID 即可。

### 6.3 事件打点

站点里统一通过 `window.trackEvent('事件名', { ... })` 打点，`stats.js` 会自动转发给 Clarity 和 GA4：

```js
trackEvent('visit_click',  { project_name: 'xxx', lang: 'en' });
trackEvent('share_click',  { project_name: 'xxx', lang: 'en' });
trackEvent('search',       { keyword: 'xxx', lang: 'en' });
trackEvent('filter_click', { category: 'tool', lang: 'en' });
trackEvent('tool_filter_click', { tool: 'DeepSeek Harness', lang: 'en' });
```

新的事件只需在对应交互处调用 `trackEvent(...)` 即可，无需改统计文件本身。

---

## 7. 常见问题

**Q：改了项目数据，首页没更新？**
A：项目数据在 `projects.js`，刷新浏览器（必要时强制刷新 Ctrl+F5）即可。筛选下拉框和数量会自动重新计算。

**Q：为什么我加的工具没出现在筛选下拉框里？**
A：确认至少有一个项目的 `tool` 字段用了这个名字，且没有拼写不一致（例如 `DeepSeek Harness` 和 `DeepSeek Harness ` 带空格会被当成两个工具）。

**Q：多语言怎么维护？**
A：所有界面文案都在 `assets/i18n.js` 的 `en` / `zh` / `fr` 三个对象里。新增文案时，三个语言各加一条同名 key 即可。

**Q：主题怎么切换 / 默认是深色？**
A：右上角 🌙/☀️ 按钮切换，选择保存在 `localStorage['theme']`。逻辑在 `assets/theme.js`；若想默认浅色，把 `theme.js` 里对 `prefers-color-scheme` 的判断去掉即可。
