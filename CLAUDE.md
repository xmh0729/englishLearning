# English Learning Review Website

## Project Overview

小学英语复习网站，单文件 HTML 应用（无构建工具），提供听写练习、语法练习和打印功能。

## Files

| File | Purpose |
|------|---------|
| `6B_review.html` | 6B 复习网站主文件（单文件应用，包含 HTML/CSS/JS 和所有数据） |
| `7A.html` | 7A 复习专题（单文件应用；含单词/词组/句子/语法/闪卡/朗读/默写卷/打印八大模块，原名 `7A单词_review.html`） |
| `7A单词/` | 7A wordlist 截图数据源（1-6.jpeg，单词 + 词组） |
| `7A句子/` | 7A Notes 截图数据源（132-139.jpeg，重点句子 + 讲解短语） |
| `7A语法/` | 7A Grammar check 截图数据源（140-148.jpeg，每单元语法讲解） |
| `7A默写/` | 7A 默写卷 docx 数据源（`2026 7A U1默写卷.docx`，✍️ 默写卷 tab 的数据源） |
| `7A 单词录音、课文录音/` | 7A 教材配套 MP3（8 个单元子目录共 61 个文件，🎤 朗读 tab 的音频源；已提交进 git，Pages 线上可用） |

> 注：6B 的原始数据源文件（`6B知识梳理单.html/.pdf`、`6B U7/U8 知识梳理.docx`）已删除；6B 的全部数据已内嵌在 `6B_review.html` 中。

> `7A.html`（原名 `7A单词_review.html`）由 `6B_review.html` 模板裁剪而来：保留「单词听写 / 词组听写 / 句子听写 / 语法练习 / 打印练习」五个 tab 并新增「🎴 闪卡」「🎤 朗读」「✍️ 默写卷」三个 tab（共八个），去掉了专项默写（dictation）与语法汇总（grammarReview）；localStorage key 为 `7a_words_review_progress`（`state.unit` 也随进度保存，重新打开页面自动恢复上次选中的单元）。数据规模：words 352 / phrases 110 / sentences 75 / grammar 69（53 选择 + 16 填空）/ readings 61 段教材录音（Unit 1-8）+ DICTATION 默写卷 7 节（Unit 1）。各单元语法专题：U1 be动词一般现在时 · U2 行为动词一般现在时(三单) · U3 人称代词 · U4 时间介词+频度副词 · U5 可数与不可数名词 · U6 疑问句 · U7 some-any+there be · U8 现在进行时。

### 🎤 朗读 tab（7A 独有）

- 数据为 `READINGS` 数组（61 条）：`{u, part, title, file}`，`file` 是相对 `7A 单词录音、课文录音/7A Unit{u} 单词录音、课文录音/` 的文件名（⚠️ Unit 4 Reading 文件名是双空格 + 小写 `.mp3`，源文件如此）；音频地址用 `encodeURI()` 编码。
- 每段录音一张卡片：原生 `<audio>` 播放器（0.75x/1x 调速）+ 🎙️ 录音跟读（`getUserMedia` + `MediaRecorder`，浏览器本地录音）+ 标记完成。
- 录音 Blob 只存内存（模块级 `_recs`，刷新失效，不进 localStorage）；完成一次跟读自动标记已朗读。完成状态存 `state.readings`（key `"{u}:{part}"` → true），随 `loadProgress`/`saveProgress` 持久化。

### ✍️ 默写卷 tab（7A 独有）

- 数据为 `DICTATION` 数组，每单元多个分节（数据源 `7A默写/2026 7A U1默写卷.docx`）：`{u, sec, label, rev, words, phrases, sentences}`；U1 共 7 节（Welcome / Reading I / Reading II / Grammar & Pronunciation / Integration A-C / Integration D / 词组卷）。
- 每节按原卷顺序分四组：`rev`（英译中复习，👁 点击显示中文，不录入）、`words` / `phrases` / `sentences`（看中文默写英文，输入 + 校验）。条目格式 `{zh, en, alt?, given?}`：`alt` 为可接受的其他写法（`checkAnyAnswer` 任一命中即算对，主 DATA 的 words/phrases/sentences 同样支持 `alt`）；句子的 `given` 是原卷填空支架，显示为提示、答案仍是完整句。
- 进度 key `dict-{u}-{sec}-{type}-{i}`，存 `state.dictAnswers`（`{correct, userInput}` 或 `{revealed:true}`），随 `loadProgress`/`saveProgress` 持久化；重做分两级：顶部「重做本单元」（`clearDictProgress`，按当前单元清除）和每节节头的「↩️ 重做本节」（`clearDictSection`，只清该分节，有作答记录时才显示）。
- 「🖨️ 打印默写卷」按原卷版式生成可打印空白卷（Revision/Words(词性)/Phrases/Sentences 分组，填空句带支架），支持 `print-grid cols-2/3/5`。
- 默写卷的知识点已同步合并进主 DATA（U1：words +age，phrases +39 条，sentences +20 条），因此闪卡/单元听写/打印练习都能覆盖。

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

### DICTATION（✍️ 默写卷，仅 7A）

```js
// 每单元多个分节；rev = 英译中复习 {en, zh}；其余为 {zh, en, alt?, given?}
{
  u: <unit>, sec: "<分节id>", label: "<Welcome / Reading I / ...>",
  rev: [{en: "greet each other", zh: "互相问候"}],
  words: [{zh: "介绍", en: "introduce"}],
  phrases: [{zh: "在同一个班级", en: "in the same class", alt: ["..."]}],
  sentences: [{zh: "张晶准备好介绍她自己了吗？", en: "Is Zhang Jing ready to introduce herself?",
               given: "______ Zhang Jing ______ ______ ______ herself?"}]
}
```

- words / phrases / sentences 条目的 `alt`（可选数组）：同义写法/缩写展开任一命中即算对，主 DATA 同样支持。

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

**截图数据源（7A 方式）**：若数据源是教材截图（如 `7A单词/`、`7A句子/`、`7A语法/` 中的 .jpeg），让 Claude 直接用 Read 工具读取图片转写：

- wordlist 截图：headword（单个词，含连字符词如 `paper-cutting`）归入 `words`，缩进的多词条目（如 `be good at`、`come true`）归入 `phrases`；忽略音标和页码；中文取主要释义、去掉词性缩写。
- Notes / 重点句截图：编号的核心句归入 `sentences`，讲解中明确教授的短语补入 `phrases`（需与 wordlist 已有短语去重）。
- Grammar check 截图：仅提供语法讲解，不含练习——练习题须据此手写。

转写量大时可并行派发子代理（每张图一个），返回结构化 JSON 后合并。完成后用 node 校验：DATA 可解析、单元内无重复、选择题 `ans` 索引在选项范围内、`<script>` 语法 OK。

## Conventions

- 所有内容在单个 HTML 文件中，不拆分文件
- 不使用任何外部依赖或 CDN
- CSS 变量定义在 `:root` 中
- 进度数据存储在 localStorage，key 格式为 `{book}_review_progress`
- 答案校验：大小写不敏感、忽略尾部标点、空格规范化
- 语法题的 `ans` 字段：选择题为选项索引（0-3），填空题为字符串
