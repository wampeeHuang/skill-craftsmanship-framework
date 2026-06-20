# Zara Zhang's Code-Explainer: Skill 设计原则提取

> 来源：`C:\Users\Administrator\.claude\skills\code-explainer\` 下全部 6 个文件
> 提取日期：2026-05-09
> 用途：后续转化为 Skill 设计审计检查项

---

## 1. 核心设计哲学

### 原则 1：倒置传统教学 — Build First, Understand Later

传统 CS 教育：先背概念（年）→ 终于动手 → 最后才懂（大多数人第一步就 quit）。
这个 Skill 的模型：**先 build 出来 → 体验它 work → 再来理解它怎么 work。**

学习者已经用过这个 app，已经知道它做什么——课程只在"你知道那个按钮吧？点它的时候底层发生了什么"这条线上展开。

**来源：** SKILL.md 第 41-48 行（"Why This Approach Works"）；README.md 第 59-61 行

### 原则 2：Show, Don't Tell — 激进可视化

不是"图文并茂"，是**每屏至少 50% 视觉面积**。文本块上限 2-3 句，第四句不该出现——应该变成图表/动画/卡片/交互元素。课程应该更像信息图，不像教科书。

**来源：** content-philosophy.md 第 7-21 行；README.md 第 63-65 行

### 原则 3：Meet Them Where They Are — 从已知到未知

课程弧线永远从学习者已经知道的东西出发——用户可见的行为——再逐步 Zoom in 到代码底层。第一屏永远是一个具体的用户操作："想象你粘贴了一个 YouTube URL 然后点击 Analyze——下面就是底层发生的事。"

**来源：** SKILL.md 第 46-47 行、第 69 行、第 73 行

### 原则 4：每个模块先回答"Why Should I Care"

每个模块在讲"how"之前先回答"why"，且答案永远是实用主义的：因为这个知识能帮你更好地**指挥 AI**、**调试**、或**做架构决策**。如果一个模块不能帮学习者做到某件事，砍掉它或重构到能做到为止。

**来源：** SKILL.md 第 47 行、第 87 行

### 原则 5：Learn by Tracing — 沿数据流追溯

跟随学习者在 App 里每天都在做的操作，端到端追踪数据流。"你知道你点的那个按钮——你点它之后数据经历了怎样的旅程。"学习者已经体验过结果，现在看的是背后的"幕后花絮"。

**来源：** content-philosophy.md 第 43-44 行；SKILL.md 第 69 行

---

## 2. 输出质量标准（可操作的审计检查项）

### 视觉密度

| # | 规则 | 来源 |
|---|------|------|
| 1 | 每屏至少 50% 视觉面积（图/代码块/卡片/动画，不是段落） | content-philosophy.md 第 13 行 |
| 2 | 单个文本块上限 2-3 句；超过 3 句必须转成视觉元素 | content-philosophy.md 第 11 行 |
| 3 | 3+ 项的列表 → 转卡片；步骤序列 → 转流程图；"组件 A 和 B 通信" → 转动画 | content-philosophy.md 第 15-20 行 |
| 4 | 每个模块至少有 1 个"hero visual"——占屏幕主导地位的图解或动画 | content-philosophy.md 第 26 行 |
| 5 | 用 generous 间距制造呼吸感（`--space-8` 到 `--space-12` 段间距） | content-philosophy.md 第 23-24 行 |

### 代码呈现

| # | 规则 | 来源 |
|---|------|------|
| 6 | **代码必须用原件**，不修改、不简化、不裁剪。选择代码库中自然短（5-10 行）的片段而非截断长函数 | content-philosophy.md 第 33 行；README.md 第 75-77 行；gotchas.md 第 19-21 行 |
| 7 | 代码块**禁止横向滚动条**：`white-space: pre-wrap` + `overflow-x: hidden` | content-philosophy.md 第 31 行；design-system.md 第 369-383 行 |
| 8 | 每个代码片段必须有 side-by-side 的纯英文逐行翻译（左代码右白话） | content-philosophy.md 第 28-29 行；SKILL.md 第 99 行 |
| 9 | 翻译块内侧：解释"为什么"而非"是什么"——如"带上 API key 让服务器知道你是谁"而非"设置 Authorization header" | interactive-elements.md 第 112 行 |

### 隐喻设计

| # | 规则 | 来源 |
|---|------|------|
| 10 | **每个概念独立隐喻，禁止复用**。数据库是图书馆卡片目录，Auth 是门卫查 ID，API 限流是夜店容量限制 | content-philosophy.md 第 41-42 行；README.md 第 71-73 行 |
| 11 | **"餐厅/厨房"隐喻是红线**——整个课程最多出现一次。发现自己在复用，停下来换 | content-philosophy.md 第 42-43 行；gotchas.md 第 16-18 行 |
| 12 | 隐喻要让人感觉对这个概念是"不可避免的"，不是硬凑的 | SKILL.md 第 94 行 |

### 测验设计

| # | 规则 | 来源 |
|---|------|------|
| 13 | **测验考应用不考记忆**。最优题型是"What would you do?"场景题，最差题型是"API 的全称是什么？" | content-philosophy.md 第 65-78 行；README.md 第 67-69 行；gotchas.md 第 22-23 行 |
| 14 | 禁止考：定义、文件名回忆、语法细节、任何能通过往上翻页直接抄答案的题 | content-philosophy.md 第 75-78 行 |
| 15 | 每题让学习者停下来思考，不是选显而易见的答案 | content-philosophy.md 第 87 行 |
| 16 | 错题反馈要有教育性，教新东西，不说"错了，答案是 B" | content-philosophy.md 第 85 行 |
| 17 | 每模块 1 个测验，位置在模块末尾，3-5 题 | content-philosophy.md 第 87 行 |
| 18 | 非惩罚性语调，无分数——"thinking exercise, not an exam" | content-philosophy.md 第 81-85 行 |

### 术语/词汇

| # | 规则 | 来源 |
|---|------|------|
| 19 | 每个技术术语在每模块**首次出现时**必须有 tooltip（1-2 句纯白话定义） | content-philosophy.md 第 49-51 行；SKILL.md 第 101 行 |
| 20 | Tooltip 激进策略：只要非技术人员有 1% 可能不认识这个词，就标。包括软件名（Blender, GIMP）、日常开发词（REPL, JSON, flag）、编程概念（function, variable, class）、缩写（永远标） | content-philosophy.md 第 52-57 行 |
| 21 | Cursor 用 pointer 不用 help——question-mark 太 clinical | content-philosophy.md 第 62 行；interactive-elements.md 第 652 行 |
| 22 | 不 tooltip 学习者已经熟知的领域术语（如 AI/ML 概念对 AI 从业者） | gotchas.md 第 11-12 行 |

### 交互元素硬性要求

| # | 规则 | 来源 |
|---|------|------|
| 23 | **5 类必选元素，每个课程必须全部出现**：Group Chat 动画、Message/Data Flow 动画、Code-English 翻译块（每模块至少 1 个）、Quiz（每模块至少 1 个）、Glossary Tooltips | SKILL.md 第 96-102 行 |
| 24 | 每模块必须包含至少 1 个交互元素（quiz/visualization/animation） | SKILL.md 第 92 行 |
| 25 | 每模块 1-2 个"aha!"callout box | SKILL.md 第 93 行 |

### 课程结构

| # | 规则 | 来源 |
|---|------|------|
| 26 | 4-6 个模块。只有代码库确实有 7-8 个值得教的独立概念时才扩展。更少更深的模块 > 更多更薄的模块 | SKILL.md 第 71 行 |
| 27 | 每模块 3-6 屏 | SKILL.md 第 90 行 |
| 28 | 一屏教一个概念，不 cram | content-philosophy.md 第 35 行 |
| 29 | 模块顺序：从用户行为 → 角色介绍 → 通信方式 → 外部世界 → 巧妙技巧 → 出错时 → 全貌。这是 menu 不是 checklist，针对代码库挑选 | SKILL.md 第 74-86 行 |

### 视觉设计硬性约束

| # | 规则 | 来源 |
|---|------|------|
| 30 | 暖色调 off-white 背景（像老纸），暖灰色调。**禁止冷白、蓝色调** | SKILL.md 第 203 行；design-system.md 第 22-23 行 |
| 31 | 一个自信的强调色（vermillion/coral/teal），**禁止紫色渐变** | SKILL.md 第 204 行 |
| 32 | 标题字体：Bricolage Grotesque 或类似粗几何字体。**禁止 Inter/Roboto/Arial/Space Grotesk** | SKILL.md 第 205 行 |
| 33 | 偶数/奇数模块交替两种暖色背景制造视觉节奏 | SKILL.md 第 207 行；design-system.md 第 62 行 |
| 34 | 阴影用暖色 RGBA（44,42,40），**禁止纯黑阴影** | SKILL.md 第 209 行；design-system.md 第 163-171 行 |
| 35 | 滚动行为：`scroll-snap-type: y proximity`，**严禁 `mandatory`**（会困住用户在长模块里） | SKILL.md 第 188 行；gotchas.md 第 25-26 行 |
| 36 | 模块最小高度 `min-height: 100dvh`，fallback `100vh` | SKILL.md 第 189 行 |

### 工程架构约束

| # | 规则 | 来源 |
|---|------|------|
| 37 | CSS 和 JS 是预制参考文件，**绝不重新生成**——永远从 references 直接 copy | SKILL.md 第 186 行、第 130 行 |
| 38 | 输出是目录而非单文件：`styles.css`、`main.js`、`_base.html`、`_footer.html`、`modules/`、`build.sh` 组装成 `index.html` | SKILL.md 第 130-145 行 |
| 39 | 模块 HTML 文件只含 `<section class="module">` 块，不含 `<html>`/`<head>`/`<body>`/`<style>`/`<script>` | SKILL.md 第 162 行 |
| 40 | **不要先展示 curriculum 给用户审批——直接 build。** 用户想看的是课程，不是计划文档 | SKILL.md 第 105 行 |

---

## 3. 反模式 / 禁止事项

| # | 禁止事项 | 来源 |
|---|----------|------|
| 1 | **文本墙**：连续 2-3 句以上无视觉截断的段落 | gotchas.md 第 13-15 行 |
| 2 | **隐喻复用**："餐厅"或"厨房"出现超过一次 | gotchas.md 第 16-18 行；SKILL.md 第 94 行 |
| 3 | **代码修改**：修剪、简化、"清理"来自代码库的代码片段 | gotchas.md 第 19-21 行 |
| 4 | **记忆型测验题**："API 的全称是什么？""哪个文件处理 X？" | gotchas.md 第 22-23 行 |
| 5 | **`scroll-snap-type: y mandatory`**——必须用 `proximity` | gotchas.md 第 25-26 行；SKILL.md 第 188 行 |
| 6 | **一口气写完所有模块**——会导致后期模块质量下降（thin and rushed） | gotchas.md 第 28-30 行 |
| 7 | **缺少交互元素的模块**——只有文本和代码块，没有 quiz/动画/图表/拖拽 | gotchas.md 第 31-32 行 |
| 8 | **重新生成 styles.css / main.js**——永远从 references 复制 | SKILL.md 第 186 行 |
| 9 | **冷白或纯白背景 蓝色调** | SKILL.md 第 203 行 |
| 10 | **紫色渐变强调色** | SKILL.md 第 204 行 |
| 11 | **Inter / Roboto / Arial / Space Grotesk 做标题字体** | SKILL.md 第 205 行 |
| 12 | **纯黑阴影（必须用暖色 #2C2A28 RGBA）** | SKILL.md 第 209 行 |
| 13 | **代码块横向滚动条** | design-system.md 第 369-383 行 |
| 14 | **Tooltip 用 `position: absolute` 放在 overflow:hidden 的容器内**——会被裁剪。必须用 `position: fixed` 并 append 到 `document.body` | gotchas.md 第 7-9 行；content-philosophy.md 第 63 行 |
| 15 | **Tooltip 用 `cursor: help`**——必须用 `pointer` | content-philosophy.md 第 62 行 |
| 16 | **先展示课程大纲让用户审批再 build**——直接 build | SKILL.md 第 105 行 |
| 17 | **惩罚性测验语气 / 显示分数**（"You got 3/5!"） | content-philosophy.md 第 84 行 |
| 18 | **向用户询问"这个 app 是做什么的"**——自己读 README 和入口文件判断 | SKILL.md 第 67 行 |

---

## 4. 入口 / 触发设计

### 触发策略

Skill 使用 **多信号叠加** 的触发策略：

1. **description 字段即自动触发器**：Claude Code 的 skill 系统根据 description 中列举的关键词和场景自动匹配
2. **明确的触发短语列表**：README 和 SKILL.md 均列出触发短语
3. **First-Run Welcome 消息**：首次触发时自动介绍自己，告知用户三种输入方式

### 触发词

来自 SKILL.md 第 3 行 description 字段和 README.md 第 50-55 行：

- `turn this into a course`
- `explain this codebase interactively`
- `teach this code`
- `interactive tutorial from code`
- `codebase walkthrough`
- `learn from this codebase`
- `make a course from this project`

### 输入方式

来自 SKILL.md 第 16-23 行：

| 输入类型 | 示例 | 处理方式 |
|----------|------|----------|
| 本地文件夹 | `turn ./my-project into a course` | 直接分析 |
| GitHub 链接 | `make a course from https://github.com/user/repo` | 先 clone 到 `/tmp/` 再分析 |
| 当前项目 | `turn this into a course` | 使用当前工作目录 |

### First-Run Welcome 设计

来自 SKILL.md 第 12-21 行：

- 一句定位：**"I can turn any codebase into an interactive course that teaches how it works — no coding knowledge required."**
- 三种输入方式各给一个自然语言示例
- 承诺输出：beautiful single-page HTML course with animated diagrams, plain-English code explanations, and interactive quizzes
- 结尾：**"The whole thing runs in your browser — no setup needed."**（消除 setup 焦虑）

### 关键设计决策

- **不向用户解释产品**——Skill 自己读 README 和入口文件来判断 app 是做什么的（SKILL.md 第 67 行）
- **不展示 curriculum 求审批**——直接 build（SKILL.md 第 105 行）
- **不做需求访谈**——用户不想被问一堆问题，直接给结果

---

## 5. 独特洞察

以下是 Zara 独有、其他 Skill 作者未明确表达过的观点：

### 洞察 1："Vibe Coder" 作为精确的学习者画像

不是泛泛的"非技术人员"，而是一个**精确的 archetype**：

- 用自然语言指挥 AI 编码工具 build 软件的人
- 没有传统 CS 教育背景
- 可能自己 build 了这个项目但没看过代码
- 目标不是成为软件工程师——是把 coding 作为**自己已有能力的放大器**
- 不需要写代码，需要**读**代码、**理解**代码、**指挥**代码

这是对 AI 时代新用户群体的精确捕捉，定义了整个 Skill 的所有设计决策——隐喻设计、tooltip 激进策略、测验考应用不考记忆，全都源自这个 persona。

**来源：** SKILL.md 第 27-39 行

### 洞察 2：词汇即学习——词汇习得作为课程核心目标

不是"tooltip 提供辅助解释"，而是**词汇习得本身就是课程的核心教学目标之一**。原因是：vibe coder 需要精确的技术词汇来描述需求（知道说"namespace package"而不是"shared folder thing"），这决定了他们指挥 AI 的效果。每个 tooltip 的措辞要帮助学习者在**自己的指令中使用这个词**。

**来源：** SKILL.md 第 37 行；content-philosophy.md 第 59 行

### 洞察 3："检测 AI 出错" 作为显式学习目标

不是"理解代码"就完了，而是要让学习者能够**识别 AI 的幻觉、捕获坏模式、在代码 smells 时有所警觉**。这是针对 vibe coder 的特定痛点：他们依赖 AI 生成代码但无法判断质量。

**来源：** SKILL.md 第 33 行

### 洞察 4：故意拒绝"AI 紫渐变审美"

视觉设计有意识地和"典型的 AI 产品紫色渐变外观"拉开距离。选用暖色调 off-white 背景、粗几何字体、单一自信强调色——这不是偶然的偏好，是有意识的差异化设计决策。

**来源：** README.md 第 39 行；SKILL.md 第 203-204 行

### 洞察 5：目录输出 + 预制 CSS/JS 的架构决策

不是搞一个单 HTML 文件，而是目录结构：CSS 和 JS 是预制参考文件每次原样复制、各模块独立写成小 HTML 片段、通过 `build.sh` 组装。这是一个深思熟虑的架构决策：**CSS/JS 不重新生成 = AI 不浪费 token 在 boilerplate 上；模块独立 = 每个输出小而高质量；支持并行 agent 写作**。

**来源：** SKILL.md 第 49 行

### 洞察 6：Module Briefs 作为并行 Agent 写作的使能器

对于复杂代码库，先为每个模块写一个 brief——包含教学弧、预提取的代码片段（带文件路径和行号）、交互元素清单、所需参考文件章节。这些 brief 的关键价值是：**写作 agent 完全不需要读代码库**——代码片段已经预提取到 brief 里。这是一个关于 Agent 编排的具体工程洞察。

**来源：** SKILL.md 第 114-125 行

### 洞察 7：Learner Has Already Experienced the Result

传统教学中学生既不知道概念也不知道结果。但这个 Skill 的学习者已经用过 App、知道它做什么、甚至用自然语言描述过它的功能。教学起点不是"这是 X 概念"，而是"你知道那个你每天点的按钮？下面是它背后的故事"——利用已有的用户体验上下文作为教学脚手架。

**来源：** SKILL.md 第 45-46 行；content-philosophy.md 第 43-44 行

### 洞察 8：Tooltip 的 `cursor: pointer` vs `cursor: help` 不仅是审美选择

用 `cursor: pointer`（clickable and inviting）而非 `cursor: help`（clinical question-mark）。这个看似微小的 CSS 属性选择背后是对学习者心态的理解：不应让他们觉得"你不懂，我来教你"，而是"这有个好东西，点开看看"。

**来源：** content-philosophy.md 第 62 行

### 洞察 9："自己做产品分析，别问用户"

Skill 明确指示：自己读 README、入口文件、UI 代码来判断 app 是做什么的——不要问用户。理由是用户可能也不熟悉这个项目（比如从 GitHub clone 了一个开源项目想学习）。这体现了 Skill 自主性的设计哲学。

**来源：** SKILL.md 第 67 行

### 洞察 10：复杂/简单代码库的双路径策略

不是所有代码库走同一条流水线。简单代码库（单一入口、5 模块以内）走 Sequential 路径直接写；复杂代码库（全栈应用、多服务、6+ 模块）走 Parallel 路径先写 brief 再并行派发 subagent。这种分支策略把"质量 vs 效率"的权衡做成了分层决策。

**来源：** SKILL.md 第 107-111 行

---

## 附录：关键数字速查

| 参数 | 值 | 来源 |
|------|-----|------|
| 模块数 | 4-6（最大 7-8） | SKILL.md 第 71 行 |
| 每模块屏数 | 3-6 | SKILL.md 第 90 行 |
| 文本块上限 | 2-3 句 | content-philosophy.md 第 11 行 |
| 视觉面积下限 | 50% | content-philosophy.md 第 13 行 |
| 测验题数 | 3-5 / 模块 | content-philosophy.md 第 87 行 |
| Code 片段理想行数 | 5-10 行 | content-philosophy.md 第 33 行 |
| Callout box 上限 | 1-2 / 模块 | SKILL.md 第 93 行 |
| Tooltip 字数 | 1-2 句 | interactive-elements.md 第 788 行 |
| 内容宽度 | 800px（标准）/ 1000px（宽） | design-system.md 第 134-135 行 |
| 字体比例 | 1.25 ratio | design-system.md 第 80 行 |
| 必选交互元素 | 5 类（Group Chat / Flow / Translation / Quiz / Tooltip） | SKILL.md 第 96-102 行 |
| 并行 agent 批量 | 一次 3 个 | SKILL.md 第 168 行 |
