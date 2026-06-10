# English Learning Review Website

## Project Overview

小学英语复习网站，单文件 HTML 应用（无构建工具），提供听写练习、语法练习和打印功能。

## Files

| File | Purpose |
|------|---------|
| `6B_review.html` | 6B 复习网站主文件（单文件应用，包含 HTML/CSS/JS 和所有数据） |
| `6B知识梳理单.html` | 6B U1-U6 原始学习资料（数据源） |
| `6B知识梳理单.pdf` | 6B U1-U6 原始学习资料 PDF 版 |
| `6B U7 知识梳理.docx` | 6B U7 Summer holiday 知识梳理（数据源） |
| `6B U8 知识梳理.docx` | 6B U8 Our dreams 知识梳理（数据源） |

## Architecture

`6B_review.html` 是一个自包含的单文件应用，结构为三层：

1. **CSS**（`<style>` 标签内）— 响应式布局 + 打印样式
2. **HTML**（`<body>` 内）— 侧边栏导航 + 顶部标签 + 内容区容器
3. **JavaScript**（`<script>` 标签内）— 数据 → 状态管理 → 渲染引擎

### JS 代码结构（按出现顺序）

```
DATA 对象           -- 所有练习数据（words / phrases / sentences / grammar / dictation / grammarReview）
state 对象          -- 当前 UI 状态（unit / tab / mode / answers）
loadProgress()      -- 从 localStorage 恢复进度
saveProgress()      -- 持久化进度到 localStorage
selectUnit()        -- 侧边栏单元切换
selectTab()         -- 顶部标签切换
getItems(tab)       -- 按当前 unit 过滤数据
normalize()         -- 答案标准化（小写/标点/空格）
checkAnswer()       -- 答案校验（容错比较）
speak()             -- Web Speech API TTS 朗读
render()            -- 总调度，按 tab 分派到具体渲染函数
renderDictation()   -- 渲染听写卡片（单词/词组/句子共用）
submitAnswer()      -- 提交听写答案
showHint()          -- 句子提示（逐步显示首字母）
renderGrammar()     -- 渲染语法练习
submitGrammar()     -- 提交选择题答案
submitGrammarFill() -- 提交填空题答案
renderPrint()       -- 渲染打印界面
doPrint()           -- 生成练习卷并触发 window.print()
showDictationPrint()      -- 专项默写打印：勾选界面
toggleAllDictationCheckboxes() -- 全选/取消全选默写勾选框
doPrintDictation()        -- 生成专项默写练习卷并触发打印
renderGrammarReview()     -- 渲染语法汇总练习（跨 Unit 1-8 综合复习）
submitGrammarReview()     -- 提交语法汇总选择题
submitGrammarReviewFill() -- 提交语法汇总填空题
clearGrammarReviewProgress() -- 重做全部语法汇总
doPrintGrammarReview()    -- 生成语法汇总练习卷并触发打印
resetProgress()     -- 重置所有进度
```

## Data Format

### words / phrases / sentences

```js
{u: <unit_number>, en: "<english>", zh: "<chinese>"}
```

- `u`: 单元编号（整数，如 1-8）
- `en`: 英文内容（用户需要输入的答案）
- `zh`: 中文提示（显示给用户的内容）

### dictation

```js
{id: <序号>, en: "<english>", zh: "<chinese>"}
```

- `id`: 序号（整数）
- `en`: 英文句子（正确答案）
- `zh`: 中文提示（显示给用户）

专项默写是跨单元的综合句子练习，独立于 unit 分组。

### grammarReview（语法汇总）

```js
{u: <unit>, topic: "<语法专题>", type: "choice"|"fill", q: "...", opts: [...], ans: <索引或字符串>, exp: "..."}
```

跨 Unit 1-8 的综合语法复习题（数据源：`六年级下册语法归纳.doc` 的"学以致用"+各单元知识点）。
- `topic`: 语法专题标签（如"形容词与副词"、"be going to"），用于展示分组
- 字段同 grammar；通过侧边栏「📋 语法汇总」入口进入，支持在线练习和打印（仅题目）
- 进度 key 格式：`greview-<索引>`

### grammar

```js
// 选择题
{u: <unit>, type: "choice", q: "<题目>", opts: ["A", "B", "C", "D"], ans: <正确选项索引0-3>, exp: "<中文解析>"}

// 填空题
{u: <unit>, type: "fill", q: "<题目，用____表示空格>", ans: "<正确答案字符串>", exp: "<中文解析>"}
```

## How to Add a New Semester / Book

以 7A 为例，完整步骤：

### 1. 创建新文件

复制 `6B_review.html` 为 `7A_review.html`，然后修改：

### 2. 替换 DATA 对象中的数据

在 `<script>` 标签内找到 `const DATA = {`，替换四个数组：

```js
const DATA = {
  words: [
    {u:1, en:"...", zh:"..."},
    // ...
  ],
  phrases: [
    {u:1, en:"...", zh:"..."},
    // ...
  ],
  sentences: [
    {u:1, en:"...", zh:"..."},
    // ...
  ],
  grammar: [
    {u:1, type:"choice", q:"...", opts:["A","B","C","D"], ans:0, exp:"..."},
    {u:1, type:"fill", q:"...", ans:"...", exp:"..."},
    // ...
  ]
};
```

### 3. 更新侧边栏导航

在 HTML 的 `<nav class="sidebar">` 中修改单元按钮：

```html
<h2>📖 7A复习</h2>
<button class="nav-item active" onclick="selectUnit('all')">全部单元</button>
<button class="nav-item" onclick="selectUnit(1)">Unit 1 标题</button>
<button class="nav-item" onclick="selectUnit(2)">Unit 2 标题</button>
<!-- 按需增减 -->
```

### 4. 更新页面标题和 localStorage key

- `<title>` 标签改为新名称
- 搜索 `6b_review_progress` 改为 `7a_review_progress`（避免与旧数据冲突）

### 5. 从知识梳理单提取数据的方法

数据源文件（如 `6B知识梳理单.html`）的结构规律：

- 单词在 `<h2>一、 单词</h2>` 之后的 `<p class="item">` 标签中
- 词组在 `<h2>二、 词组</h2>` 之后
- 句子在 `<h2>三、 句型</h2>` 之后
- 每个 `<p class="item">` 格式为：`<span class="num">序号.</span> 英文 中文`
- 语法说明在 `<h2>四、 语法</h2>` 或 `知识点总结` 之后（需人工编写为练习题）

提取数据时让 Claude 读取源文件并按 DATA 格式输出 JSON。语法练习题需要根据语法说明手动编写，每个语法专题建议 8-10 题（6 选择 + 2-4 填空）。

## Conventions

- 所有内容在单个 HTML 文件中，不拆分文件
- 不使用任何外部依赖或 CDN
- CSS 变量定义在 `:root` 中
- 进度数据存储在 localStorage，key 格式为 `{book}_review_progress`
- 答案校验：大小写不敏感、忽略尾部标点、空格规范化
- 语法题的 `ans` 字段：选择题为选项索引（0-3），填空题为字符串
