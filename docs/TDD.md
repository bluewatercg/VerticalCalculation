# 技术设计文档 (TDD) - 竖式计算练习页生成器

## 1. 文档概述
本技术设计文档（TDD）基于产品需求文档（PRD）V1.0.0，旨在为“竖式计算练习页生成器”的开发提供详细的技术实现方案。本文档将涵盖系统架构、前端设计、核心组件实现、数据流程以及关键技术点的具体解决方案。

**目标读者**: 前端开发工程师、测试工程师。

## 2. 总体设计
### 2.1 系统架构
本产品将采用纯前端（Client-Side）架构。所有的数据获取、逻辑处理和页面渲染都在用户的浏览器中完成。这种架构具备以下优点：
- **零后端依赖**: 无需服务器端语言、数据库或API，极大地简化了开发和部署。
- **高性能**: 静态资源可以被CDN高效缓存，用户访问速度快。
- **易于部署**: 可以部署在任何静态网站托管服务上（如GitHub Pages, Vercel, Netlify等）。

### 2.2 技术栈
根据PRD的建议，我们将采用最基础、最轻量的前端技术栈，以确保项目的简洁性和高性能。
- **语言**:
  - **HTML5**: 用于构建页面的基本结构。
  - **CSS3**: 用于定义页面样式，包括A4布局、网格和竖式框架。
  - **Vanilla JavaScript (ES6+)**: 用于处理所有逻辑，包括数据获取、DOM操作和事件处理。
- **开发工具 (可选)**:
  - **Prettier**: 用于代码格式化，保证代码风格一致。
  - **Live Server (VSCode Extension)**: 用于本地开发和实时预览。

### 2.3 核心模块职责
系统将遵循PRD中定义的功能模块进行划分，每个模块都有明确的职责：

- **`Data Loader` (数据加载模块)**:
  - **职责**: 负责通过 `fetch` API 异步获取 `陈凌睿作业.json` 文件。
  - **输出**: 返回一个包含所有题目对象的JavaScript数组。
  - **错误处理**: 捕获网络错误或JSON解析错误，并通知UI模块进行处理。

- **`Problem Parser` (题目解析模块)**:
  - **职责**: 接收原始的题目字符串（如 "76 ÷ 38 ="），并将其解析为结构化的数据对象。
  - **输入**: `problem: "76 ÷ 38 ="`
  - **输出**: `{ dividend: 76, divisor: 38 }`
  - **健壮性**: 需要能处理数字和运算符周围可能存在的空格。

- **`Page Renderer` (页面渲染模块)**:
  - **职责**: 核心渲染引擎。负责根据给定的题目数据，在页面上创建和管理所有DOM元素。
  - **子模块**:
    - **`A4Page`**: 创建模拟A4纸的容器。
    - **`Grid`**: 在A4容器内创建4x5的网格布局。
    - **`CalculationItem`**: 为每个题目创建竖式计算框架的DOM结构。

- **`Print Controller` (打印控制模块)**:
  - **职责**: 处理所有与打印相关的逻辑。
  - **功能**:
    - 绑定“打印”按钮的点击事件到 `window.print()`。
    - 通过 `@media print` 样式表，确保打印输出的纯净性。

### 2.4 项目文件结构
为了保持代码的组织性和可维护性，建议采用以下文件结构：
```

## 3. 前端设计
### 3.1 HTML 结构 (`index.html`)
HTML文件将保持极简，只包含必要的元素。
- 一个根容器 `#app` 用于挂载整个应用。
- 一个用于显示A4页面的容器 `#page-container`。
- 一个打印按钮 `#print-btn`。
- 一个用于显示加载/错误状态的区域 `#status-message`。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>竖式计算练习页生成器</title>
    <link rel="stylesheet" href="css/main.css">
    <link rel="stylesheet" href="css/print.css" media="print">
</head>
<body>
    <div id="app">
        <header class="app-header">
            <h1>竖式计算练习页</h1>
            <button id="print-btn" class="print-button">🖨️ 打印</button>
        </header>
        <main class="page-wrapper">
            <div id="page-container" class="a4-page">
                <!-- 题目网格将动态生成在这里 -->
            </div>
        </main>
        <div id="status-message" class="status-message"></div>
    </div>
    <script src="js/main.js" type="module"></script>
</body>
</html>
```

### 3.2 CSS 设计
#### 3.2.1 A4页面布局 (`main.css`)
- 使用 `aspect-ratio: 210 / 297;` 来精确模拟A4纸的宽高比。
- 使用 `max-width` 和 `max-height` 确保页面在不同视口下的适应性。
- 使用 `box-shadow` 创造纸张悬浮的效果。
- 页面居中显示。

```css
/* main.css */
body {
    background-color: #f0f0f0;
    display: flex;
    justify-content: center;
    padding: 2rem;
}

.a4-page {
    background-color: white;
    width: 100%;
    max-width: 800px; /* 或者一个合适的宽度 */
    aspect-ratio: 210 / 297;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
    padding: 2cm; /* 模拟打印边距 */
    box-sizing: border-box;
}
```

#### 3.2.2 题目网格布局 (`main.css`)
- 使用 `display: grid;` 创建4列5行的网格。
- `grid-template-columns: repeat(4, 1fr);`
- `grid-template-rows: repeat(5, 1fr);`
- 使用 `gap` 属性来控制题目之间的间距。

```css
/* main.css */
.calculation-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-template-rows: repeat(5, 1fr);
    gap: 20px;
    height: 100%;
}
```

#### 3.2.3 打印样式 (`print.css`)
- 隐藏所有非打印内容，如按钮、页头。
- 移除A4页面的阴影和背景色，使其在打印时完全贴合纸张。
- 强制页面背景为白色。

```css
/* print.css */
@media print {
    body {
        background-color: white;
        padding: 0;
        margin: 0;
    }

    .app-header, #print-btn, .status-message {
        display: none;
    }

    .page-wrapper {
        margin: 0;
        padding: 0;
    }

    .a4-page {
        box-shadow: none;
        max-width: 100%;
        width: 100%;
        height: 100%;
        padding: 1.5cm; /* 调整实际打印边距 */
    }
}
```

### 3.3 JavaScript 逻辑流程 (`main.js`)
`main.js` 作为应用的入口点，将协调其他模块完成整个流程。

```javascript
// js/main.js (伪代码)
import { loadProblems } from './dataLoader.js';
import { renderPage } from './renderer.js';

document.addEventListener('DOMContentLoaded', async () => {
    const printBtn = document.getElementById('print-btn');
    const statusMessage = document.getElementById('status-message');

    printBtn.addEventListener('click', () => window.print());

    try {
        statusMessage.textContent = '正在加载题目...';
        const allProblems = await loadProblems('陈凌睿作业.json');
        
        // MVP: 只取前20道题
        const problemsForPage = allProblems.slice(0, 20);
        
        renderPage(problemsForPage);
        statusMessage.textContent = ''; // 清空状态信息

    } catch (error) {
        console.error('应用初始化失败:', error);
        statusMessage.textContent = `加载题目失败: ${error.message}`;
    }
});
```

## 4. 组件化设计与数据流

### 4.1 组件设计
我们将功能拆分为独立的、可复用的逻辑单元（模块/函数），即使不使用前端框架，也要有组件化的思想。

#### 4.1.1 `CalculationItem` 组件
这是最重要的UI组件，负责渲染单个竖式计算题。它不是一个真正的框架组件，而是一个函数，该函数接收题目数据并返回一个DOM元素。

- **输入 (Props)**: `(problemData, index)`
  - `problemData`: `{ dividend: number, divisor: number }`
  - `index`: 题目序号 (e.g., 0-19)
- **输出 (Return)**: `HTMLElement` (一个包含完整竖式框架的 `div` 元素)

**HTML结构 (示例):**
```html
<div class="calculation-item">
    <span class="item-index">1.</span>
    <div class="divisor">38</div>
    <div class="long-division-symbol">
        <div class="dividend">76</div>
    </div>
</div>
```

**CSS (`main.css`):**
```css
.calculation-item {
    position: relative;
    padding: 10px;
    font-size: 24px; /* 可调整 */
    font-family: 'KaiTi', 'STKaiti', serif; /* 使用楷体更像作业本 */
}

.item-index {
    position: absolute;
    top: 0;
    left: 0;
    font-size: 14px;
}

.long-division-symbol {
    border-top: 2px solid black;
    border-left: 2px solid black;
    padding-left: 10px; /* 为除数留出空间 */
    margin-left: 40px; /* 为除数留出空间 */
    height: 1.5em; /* 竖线高度 */
}

.divisor {
    position: absolute;
    left: 10px;
    top: 28px; /* 需要微调对齐 */
}

.dividend {
    padding-left: 10px;
}
```
*注意: 上述CSS是初步实现，需要仔细调试才能实现完美的对齐效果。*

### 4.2 数据流 (Data Flow)
数据流是单向的，清晰地展示了从数据获取到最终渲染的完整过程。

```mermaid
graph TD
    A[开始] --> B{main.js: DOMContentLoaded};
    B --> C[dataLoader.js: loadProblems];
    C --> D{fetch('陈凌睿作业.json')};
    D --> E[解析JSON为JS数组];
    E --> F[返回题目数组];
    F --> G{main.js: 接收题目数组};
    G --> H[截取前20道题];
    H --> I[renderer.js: renderPage];
    I --> J{遍历20道题};
    J --> K[problemParser.js: parseProblemString];
    K --> L{返回 {dividend, divisor}};
    L --> M[CalculationItem: 创建题目DOM];
    M --> N{将题目DOM添加到网格中};
    J -- 循环20次 --> N;
    N -- 完成后 --> O[结束];

    subgraph "错误处理"
        D -- 失败 --> P[抛出网络或解析错误];
        P --> Q{main.js: catch(error)};
        Q --> R[在 #status-message 中显示错误];
    end
```

## 5. 关键技术点详述

### 5.1 竖式除法符号的CSS实现
PRD中提到，除号（厂字头）可以用CSS绘制。上面的 `CalculationItem` 组件CSS提供了一种实现思路：
- 使用一个容器 `.long-division-symbol`。
- `border-top` 和 `border-left` 组合起来形成厂字头。
- 被除数 `.dividend` 放在这个容器内部。
- 除数 `.divisor` 使用绝对定位，放置在厂字头的左侧。
这种方法的挑战在于不同数字宽度导致的对齐问题，需要通过微调 `padding`, `margin` 和 `position` 来解决。

### 5.2 题目字符串解析
`problemParser.js` 模块需要一个健壮的函数来解析 "76 ÷ 38 =" 这样的字符串。

```javascript
// js/problemParser.js
export function parseProblemString(problemStr) {
    // 使用正则表达式匹配数字，忽略空格
    const match = problemStr.match(/(\d+)\s*÷\s*(\d+)/);
    if (!match) {
        throw new Error(`无法解析题目: ${problemStr}`);
    }
    const dividend = parseInt(match[1], 10);
    const divisor = parseInt(match[2], 10);
    return { dividend, divisor };
}
```

### 5.3 分页逻辑 (v2.0 储备)
虽然MVP版本只做第一页，但设计上应为分页预留接口。

- `main.js` 中可以增加一个 `currentPage` 变量。
- 每次渲染时，根据 `currentPage` 和 `itemsPerPage` (值为20) 来计算 `slice` 的起止位置。
  - `const start = (currentPage - 1) * itemsPerPage;`
  - `const end = start + itemsPerPage;`
- 需要增加“上一页”和“下一页”按钮，并绑定事件来修改 `currentPage` 并重新调用 `renderPage`。

### 5.4 模块化JavaScript
为了避免全局命名空间污染，所有JS文件都应作为ES模块。
- 在 `<script>` 标签中添加 `type="module"`。
- 使用 `export` 关键字从模块中导出函数。
- 使用 `import` 关键字在其他模块中导入函数。
这使得代码结构更清晰，依赖关系更明确。
/
|-- index.html             # 主HTML文件
|-- css/
|   |-- main.css           # 主要样式
|   |-- print.css          # 打印专用样式
|-- js/
|   |-- main.js            # 主逻辑，应用入口
|   |-- dataLoader.js      # 数据加载模块
|   |-- problemParser.js   # 题目解析模块
|   |-- renderer.js        # 页面渲染模块
|-- 陈凌睿作业.json        # 题目数据文件
|-- docs/
|   |-- ... (PRD, TDD等文档)