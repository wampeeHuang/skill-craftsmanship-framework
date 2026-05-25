# Anthropic Skill 工程化设计原则

> 来源：Anthropic 官方 `skill-creator` skill（SKILL.md 486行 + 4个agent指令 + schemas文档）
> 提取日期：2026-05-09

---

## 1. 核心设计哲学

### 原则 1：解释 Why，而非堆砌 MUST

不要用全大写的 ALWAYS/NEVER 或僵化的强制结构。当前的 LLM 具有强大的心理理论（theory of mind），当你给它一个好的 harness，它能超越机械指令真正把事做成。用解释代替命令——让模型理解**为什么**这件事重要，而不是只知道**要**做什么。

> 来源：SKILL.md L139 "Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs." / L302 "If you find yourself writing ALWAYS or NEVER in all caps, or using super rigid structures, that's a yellow flag — if possible, reframe and explain the reasoning so that the model understands why the thing you're asking for is important."

### 原则 2：渐进式信息披露（Progressive Disclosure）

Skill 采用三层加载体系，按需加载而非一次性倾倒。这是 Claude Code 独有的架构设计，目的是控制上下文窗口消耗：

1. **Metadata**（name + description）——始终在上下文中（约 100 词）
2. **SKILL.md body**——skill 触发时加载（理想 <500 行）
3. **Bundled resources**——按需加载（无限制，脚本可执行而无需加载到上下文）

> 来源：SKILL.md L86-93

### 原则 3：泛化优先，不过拟合

Skill 可能被调用百万次，分布在完全不同的 prompt 上。你和用户在迭代中反复打磨的那几个例子只是加速开发的工具——如果 skill 只对这些例子有效，它就是废品。遇到顽固问题，不要堆 fiddly overfitty 的补丁，试试换比喻、换工作模式。保持指令的通用性。

> 来源：SKILL.md L298 "if the skill you and the user are codeveloping works only for those examples, it's useless" / L139 "try to make the skill general and not super-narrow to specific examples"

### 原则 4：保持 Prompt 精简

删掉那些不出力的内容。读 transcript 而不只是最终输出——如果 skill 让模型浪费大量时间做无用功，砍掉导致问题的部分看看效果。不要因为写进去了就不舍得删。

> 来源：SKILL.md L300 "Keep the prompt lean. Remove things that aren't pulling their weight."

### 原则 5：不令用户惊讶（Principle of Lack of Surprise）

Skill 不得包含恶意代码、漏洞利用代码或任何可能危害系统安全的内容。如果向用户描述了 skill 的意图，其实际内容不应让用户感到意外。不要配合创建误导性 skill 或旨在促进未授权访问、数据泄露的 skill。

> 来源：SKILL.md L111-113

---

## 2. Skill 结构规范

### 2.1 三层加载系统

| 层级 | 内容 | 何时加载 | 规模建议 |
|------|------|----------|----------|
| L1 | name + description（YAML frontmatter） | 始终在上下文 | ~100 words |
| L2 | SKILL.md body（Markdown instructions） | skill 触发时 | <500 lines（超过则加子层） |
| L3 | Bundled resources（scripts/references/assets） | 按需（脚本可执行不加载） | 无限制 |

> 来源：SKILL.md L86-93

### 2.2 目录结构

```
skill-name/
├── SKILL.md (必需)
│   ├── YAML frontmatter（必需：name, description）
│   └── Markdown 指令
└── Bundled Resources（可选）
    ├── scripts/     —— 可执行代码（确定性/重复性任务）
    ├── references/  —— 按需加载到上下文的文档
    └── assets/      —— 输出中使用的文件（模板、图标、字体）
```

> 来源：SKILL.md L75-84

### 2.3 文件组织规则

- **SKILL.md 限制**：保持在 500 行以内。接近此限制时，增加层级并给出清晰的跳转指引。
- **大文件处理**：对于 >300 行的 reference 文件，需包含目录（table of contents）。
- **明确引用**：在 SKILL.md 中清晰引用 reference 文件，并说明何时应读取它们。
- **领域分离**：当 skill 支持多个领域/框架时，按 variant 组织：

```
cloud-deploy/
├── SKILL.md（工作流 + 选择逻辑）
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

Claude 只读取相关的 reference 文件，不会加载无关内容。

> 来源：SKILL.md L96-109

### 2.4 命名规范

- 更新已有 skill 时，**保持原名不变**。目录名和 frontmatter 中的 `name` 字段都不改。例如安装的 skill 叫 `research-helper`，输出就是 `research-helper.skill`（不是 `research-helper-v2`）。
- 更新只读路径的 skill 时，先复制到 `/tmp/skill-name/` 编辑，再从那里打包。

> 来源：SKILL.md L438-443

---

## 3. 迭代开发流程

### 3.1 核心循环：Eval-Driven Loop

```
draft → test → review → improve → repeat
```

完整流程：

1. **Capture Intent**：理解用户想做什么、何时触发、输出格式、是否需要测试用例
2. **Interview & Research**：主动询问边界情况、输入输出格式、示例文件、成功标准、依赖——在写 test prompt 之前把这些搞清楚
3. **Write SKILL.md**：填充 name、description、compatibility、指令内容
4. **Write test cases**：2-3 个真实用户会说的 prompt，保存到 `evals/evals.json`（先不加 assertions）
5. **Run tests**：并行启动所有 subagent（with-skill + baseline），同时起草 assertions
6. **Review**：用 `generate_review.py` 生成评估浏览器，让用户查看定性输出和定量 benchmark
7. **Read feedback**：读取 `feedback.json`，聚焦用户有具体意见的测试用例
8. **Improve**：基于反馈泛化改进，进入下一轮迭代
9. **Repeat**：持续循环，直到用户满意、feedback 全空、或不再有实质性进展
10. **Package**：打包为 `.skill` 文件交付

> 来源：SKILL.md L10-19, L474-481

### 3.2 迭代终止条件

- 用户明确说满意
- 所有 feedback 为空（用户觉得没问题）
- 无法再取得有意义的改进

> 来源：SKILL.md L318-322

### 3.3 并行执行策略

**关键规则**：在同一轮中同时启动所有 with-skill 和 baseline 运行。不要先跑 with-skill 再回来跑 baseline——一次性全部启动，让它们差不多同时完成。这是为了减少等待时间并保证测试条件的可比性。

```json
// 每个测试用例两个 subagent：
// 1. with_skill → 保存到 workspace/iteration-N/eval-X/with_skill/outputs/
// 2. baseline  → 保存到 workspace/iteration-N/eval-X/without_skill/outputs/
//                 （或 old_skill/outputs/ 如果是在改进已有 skill）
```

> 来源：SKILL.md L169-186

### 3.4 改进 Skill 的思考框架

改进时遵循四个维度：

1. **从 feedback 泛化**：用户对你手上那几个例子给出的反馈，要抽象成通用规则
2. **保持 prompt 精简**：读 transcript，砍掉导致模型浪费时间的内容
3. **解释为什么**：把用户写得不满/沮丧的东西真正理解透，把这种理解传达进指令
4. **寻找重复工作**：如果所有测试用例里 subagent 都独立写了类似的 helper 脚本（如 `create_docx.py`），就该把这个脚本 bundle 进 skill 的 `scripts/` 里

> 来源：SKILL.md L298-304

---

## 4. 触发设计（Description Optimization）

### 4.1 Description 是主要触发机制

description 字段（SKILL.md 的 YAML frontmatter）**决定 Claude 是否调用这个 skill**。所有"何时使用"的信息都放在 description 中，不要放在 body 里。Body 只在 skill 被触发后才起作用。

> 来源：SKILL.md L67, L335

### 4.2 "Undertrigger" 问题与 "Pushy" 原则

Anthropic 明确指出了一个关键行为特征：**Claude 倾向于"欠触发"（undertrigger）skill**——即该用的时候不用。为了对抗这个问题，description 需要写得稍微 "pushy"（主动）：

- **不够好的写法**："How to build a simple fast dashboard to display internal Anthropic data."
- **更好的写法**："How to build a simple fast dashboard to display internal Anthropic data. **Make sure to use this skill whenever the user mentions dashboards, data visualization, internal metrics, or wants to display any kind of company data, even if they don't explicitly ask for a 'dashboard.'**"

> 来源：SKILL.md L67

### 4.3 底层触发逻辑

Skill 出现在 Claude 的 `available_skills` 列表中（name + description），Claude 基于 description 决定是否咨询一个 skill。

**关键洞见**：Claude **只对它自己难以处理的任务咨询 skill**。简单的单步查询（如"read this PDF"）即使 description 完美匹配也不会触发 skill，因为 Claude 可以直接用基础工具完成。**复杂、多步骤、专业化的查询**才可靠地触发 skill。

这一定义了 eval query 的底线：你的测试 query 必须足够实质性，Claude 才会真正受益于咨询一个 skill。

> 来源：SKILL.md L398-400

### 4.4 Trigger Eval 方法论

#### Query 设计原则

- **数量**：20 个——8-10 个 should-trigger + 8-10 个 should-not-trigger
- **真实感**：必须是 Claude Code 或 Claude.ai 用户**实际会输入的**内容，具体且有细节——文件路径、个人工作背景、列名和值、公司名、URL、一点故事背景
- **多样化**：长短混合，有些小写、缩略语、拼写错误、随意口语
- **聚焦边界**：重点是 edge cases，不是泾渭分明的案例

#### 好 query vs 坏 query

| 坏 query | 好 query |
|----------|----------|
| `"Format this data"` | `"ok so my boss just sent me this xlsx file (its in my downloads, called something like 'Q4 sales final FINAL v2.xlsx') and she wants me to add a column that shows the profit margin as a percentage. The revenue is in column C and costs are in column D i think"` |
| `"Extract text from PDF"` | 包含具体文件路径、背景故事、非正式表达 |
| `"Create a chart"` | 包含列名、业务场景、实际痛点 |

> 来源：SKILL.md L348-352

#### Should-trigger 设计

覆盖不同措辞表达同一意图——正式的、随意的。包含用户没有明确命名 skill 或文件类型但显然需要它的情况。加入一些不常见的用例，以及这个 skill 与其他 skill 竞争但应该胜出的场景。

> 来源：SKILL.md L354

#### Should-not-trigger 设计

**最有价值的负面用例是"近失"（near-misses）**——与 skill 共享关键词或概念但实际需要的是别的东西。应该测试：
- 相邻领域
- 本会触发朴素关键词匹配但不应触发的模糊表达
- query 触及了 skill 的能力范围但在上下文中另一个工具更合适的场景

**关键红线**：不要让 should-not-trigger 明显无关。对 PDF skill 写 `"Write a fibonacci function"` 太容易了——它什么都测试不了。负面用例必须真正棘手。

> 来源：SKILL.md L356-358

### 4.5 自动化优化循环

使用 `scripts/run_loop.py` 自动化 description 优化：

- **Split**：60% train / 40% held-out test
- **Evaluate**：每个 query 跑 3 次以获得可靠的触发率
- **Propose**：调用 Claude 基于失败案例提出改进方案
- **Iterate**：最多 5 轮
- **Select**：按 **test score**（而非 train score）选择最佳 description，避免过拟合
- 使用当前 session 的 model ID 进行测试，确保结果匹配用户实际体验

> 来源：SKILL.md L376-394

---

## 5. 测试方法论

### 5.1 并行对比测试

每个测试用例跑两个配置：

| 场景 | 配置对比 |
|------|----------|
| **创建新 skill** | with_skill vs without_skill（无 skill 的裸跑） |
| **改进已有 skill** | with_skill(新版) vs old_skill(旧版快照) |

> 来源：SKILL.md L183-186

### 5.2 测试目录结构

```
<skill-name>-workspace/
└── iteration-1/
    ├── eval-0/                      # 使用描述性名称，如 eval-0-extract-table
    │   ├── eval_metadata.json       # eval_id, eval_name, prompt, assertions
    │   ├── with_skill/
    │   │   ├── outputs/             # skill 产生的输出文件
    │   │   │   └── metrics.json     # 执行指标
    │   │   ├── timing.json          # 强制即时捕获（通知即失）
    │   │   └── grading.json         # grader 评估结果
    │   └── without_skill/           # 同上结构
    │       └── ...
    └── benchmark.json               # 聚合后的统计
```

> 来源：SKILL.md L167, L188-197, L207-213, L225-232

### 5.3 Assertion 设计原则

#### 黄金规则

好的 assertion 是**可客观验证的**且**名字描述性强**——让人扫一眼 benchmark viewer 就立刻理解每一条在检查什么。

> 来源：SKILL.md L202

#### 区分性（Discrimination）原则

一条 assertion 真正的价值在于其**区分能力**：
- 当 skill 真正成功时通过
- 当 skill 确实失败时不通过

**弱 assertion 的危害**：一条总通过的 assertion 比没有 assertion 更糟糕——它制造虚假信心。如果 grader 发现某条 assertion 是 trivial 的，或者有重要输出没有 assertion 覆盖，就应该标记出来。

> 来源：grader.md L72-80

#### 评分标准

**PASS** 需要：
- Transcript 或 outputs 明确证明 assertion 为真
- 能引用具体证据
- 证据反映**实质性的任务完成**，而非表面合规（如文件存在且包含正确内容，而不是只是文件名正确）

**FAIL** 的情况：
- 找不到证据
- 证据与 assertion 矛盾
- 无法从可用信息中验证
- 证据是表面的——技术上满足了 assertion 但底层任务结果错误或不完整
- 输出似乎碰巧满足了 assertion 而非实际做了工作

**不确定时的判定**：通过的举证责任在 expectation 一方——不确定就判失败。

> 来源：grader.md L87-99

#### Claims 提取机制

除了预定义的 assertion，grader 还会从输出中提取**隐式声明**并验证：
- 事实声明（"The form has 12 fields"）
- 过程声明（"Used pypdf to fill the form"）
- 质量声明（"All fields were filled correctly"）

这捕获了预定义 assertion 可能漏掉的问题。

> 来源：grader.md L43-59

### 5.4 主观性 Skill 的特殊处理

写作风格、设计质量等主观性 skill **不应强加定量 assertion**。这些东西需要人的判断，强行量化适得其反。

> 来源：SKILL.md L202

### 5.5 时间数据捕获

**关键约束**：subagent 任务完成时的通知中包含 `total_tokens` 和 `duration_ms`，但**这些数据不会持久化到任何地方**。每个通知到达时必须即时保存到 `timing.json`——无法事后恢复。

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

> 来源：SKILL.md L207-219

### 5.6 Benchmark 分析要点

Analyzer 在查看聚合统计之外，需要关注：

- **无区分力的 assertion**：在两个配置中都 100% 通过 → 不能体现 skill 价值
- **高方差 eval**：可能是 flaky 或模型依赖的
- **反直觉结果**：with_skill 反而比 without_skill 差 → skill 可能在帮倒忙
- **时间/Token 权衡**：skill 增加了多少执行时间，值得吗？

> 来源：analyzer.md L189-275

### 5.7 盲比（Blind Comparison）

适用于 "新版到底比旧版好在哪里？" 这种需要严谨判断的场景：

- 将两个输出给独立 agent，**不告知哪个是哪个**
- Comparator 生成 rubric 打分（内容 + 结构两个维度）
- Analyzer 事后"解盲"，分析 winner 为什么赢、loser 差在哪里
- 输出具体的改进建议（按优先级分类：instructions/tools/examples/error_handling/structure/references）

> 来源：SKILL.md L325-329, comparator.md, analyzer.md

---

## 6. 反模式 / 注意事项

### 6.1 不过拟合测试用例

在几个例子上反复打磨时，避免写进只对这些例子有效的、过度约束的指令。Skill 要被调用百万次——让它在更广的分布上泛化，而非在 3 个 prompt 上完美。

> 来源：SKILL.md L298

### 6.2 不堆砌大写 MUST

用 ALWAYS 或 NEVER 全大写写指令是黄牌警告。重构为解释为什么——这是更人性化、更强大、更有效的方法。

> 来源：SKILL.md L302

### 6.3 不保留无用的 Prompt

大胆砍掉不出力的内容。读 transcript 确认——如果 skill 让模型浪费时间做无用功，删掉相应部分。

> 来源：SKILL.md L300

### 6.4 Eval Query 的质量红线

- **禁止**：抽象、短、无细节的 query（如 "Format this data"）——它们不会触发任何 skill
- **禁止**：明显无关的负面 query（如对 PDF skill 写 "write fibonacci"）——什么都测不出来
- **核心陷阱**：简单单步 query 不会触发 skill，即使 description 完美匹配——Claude 会直接用自己的基础工具处理

> 来源：SKILL.md L348-352, L358, L398-400

### 6.5 不跳过 Eval Viewer

在 Cowork 环境中尤其重要：**不要在你自己评估输入之前跳过生成 eval viewer**。先把结果送到人眼前。

> 来源：SKILL.md L451

### 6.6 不用其他测试 Skill

严格只用 skill-creator 自身的测试流程。不要使用 `/skill-test` 或任何其他测试 skill。

> 来源：SKILL.md L165

### 6.7 不要对主观 Skill 强加定量 Assertion

写作风格、艺术类 skill 用定性评估。强行加 assertion 制造虚假的客观性。

> 来源：SKILL.md L202

### 6.8 Grading.json 字段名必须精确

`expectations` 数组必须使用 `text`, `passed`, `evidence` 字段名——不能用 `name`, `met`, `details` 或其他变体。Viewer 依赖这些精确的字段名。

> 来源：SKILL.md L225

---

## 7. 写作风格指南

### 7.1 祈使句为主

指令优先使用祈使句（imperative form）。直接告诉模型做什么。

> 来源：SKILL.md L117

### 7.2 解释 Why，不堆砌 Must

核心写作原则在三处重复强调（L139, L302, L298）：

> "Try to explain to the model why things are important in lieu of heavy-handed musty MUSTs. Use theory of mind and try to make the skill general and not super-narrow to specific examples."

即使是用户给出的 terse 或沮丧的反馈，也要真正理解任务是什么、用户为什么这么写、他们实际写了什么，然后把这种理解传达进指令。

> 来源：SKILL.md L139, L302

### 7.3 输出格式定义方式

使用模板块定义输出格式：

```markdown
## Report structure
ALWAYS use this exact template:
# [Title]
## Executive summary
## Key findings
## Recommendations
```

> 来源：SKILL.md L119-127

### 7.4 Examples 模式

当 Input/Output 形式不合适时，可以偏离这个模板，但以下是推荐格式：

```markdown
## Commit message format
**Example 1:**
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

> 来源：SKILL.md L129-135

### 7.5 "Fresh Eyes" 修订法

写完草稿后，用全新的眼光再看一遍并改进。不要只改一轮就停——认真思考，写一版修订稿，然后重新审视，再次改进。

> 来源：SKILL.md L139, L305-306 "I'd suggest writing a draft revision and then looking at it anew and making improvements."

### 7.6 通用性 > 特化

Skill 指令必须通用，不绑定到特定示例。换比喻、推荐不同的工作模式，而不是堆砌针对当前测试用例的补丁。

> 来源：SKILL.md L298

---

## 8. 独特洞察

以下洞察来自 Anthropic 的 skill-creator，在其他 skill 作者或社区方法论中未见明确表述。

### 8.1 Claude 的 "Undertrigger" 行为倾向

这是 Anthropic 作为模型开发者对自身产品行为特征的内部观察：**Claude 倾向于不调用 skill，即使调用会有帮助**。这不是 bug，而是一个需要显式对抗的行为模式。因此 description 需要 "pushy"——主动列出各种触发场景，甚至加上 "even if they don't explicitly ask for..." 这种看似冗余的指令。

> 来源：SKILL.md L67 "currently Claude has a tendency to 'undertrigger' skills -- to not use them when they'd be useful"

### 8.2 简单查询不触发 Skill 的底层逻辑

Claude 只对**它自己难以处理的任务**咨询 skill。这意味着即使 description 完美匹配，简单的单步查询（如 "read this PDF"）也不会触发。**复杂度是触发的隐含前提**。这是一个架构级别的决策而非实现细节——直接影响 eval query 的设计策略。

> 来源：SKILL.md L398-400 "Claude only consults skills for tasks it can't easily handle on its own — simple, one-step queries like 'read this PDF' may not trigger a skill even if the description matches perfectly"

### 8.3 Description 作为唯一触发入口

Skill 系统的架构中，**description 字段是决定是否调用的唯一机制**。Body 内容只在触发后才有意义。所有 "when to use" 信息必须放在 description 中，不能放在 body 里。这与许多其他 agent 框架（通常用关键词匹配或路由器）根本不同。

> 来源：SKILL.md L67 "This is the primary triggering mechanism - include both what the skill does AND specific contexts for when to use it. All 'when to use' info goes here, not in the body."

### 8.4 时间数据的易逝性

Subagent 完成通知中的 `total_tokens` 和 `duration_ms` **完全不持久化**——错过通知就永久丢失。这是一个系统级约束，直接塑造了测试工作流：必须即时保存。这一设计细节意味着测试流程不是纯异步的，需要一条即时响应路径。

> 来源：SKILL.md L219 "This is the only opportunity to capture this data — it comes through the task notification and isn't persisted elsewhere."

### 8.5 Assertion 区分力 > Assertion 覆盖率

不仅要求 assertion 正确，更要求 assertion 具有**区分力**——能在 skill 成功时通过、失败时不通过。永远通过的 assertion 是毒药——制造虚假信心。Grader 被赋予双重职责：打分 AND 批评 eval 本身。这是一个元评估层。

> 来源：grader.md L69-80 "A passing grade on a weak assertion is worse than useless — it creates false confidence."

### 8.6 "不确定则判失败"的评估立场

Burden of proof 在 expectation 一方——不确定就判失败，不给 partial credit。这保证了 benchmark 数据的下限可靠性：通过的都是确凿通过的。

> 来源：grader.md L99 "When uncertain: The burden of proof to pass is on the expectation."

### 8.7 Automated Description Optimization 的 ML 方法论

将 prompt 工程视为一个优化问题，使用 train/test split、multiple runs per query、model-driven proposal generation——这是把机器学习方法论直接应用到 prompt engineering 上。`run_loop.py` 用 test score（而非 train score）选最优解，防止对 trigger eval 的过拟合。

> 来源：SKILL.md L376-394

### 8.8 寻找 Bundle 机会而非容忍重复

观察所有测试用例的 transcript，如果 subagent 在彼此独立的运行中都写了相同的 helper 脚本（`create_docx.py`, `build_chart.py`），这是明确的信号——该脚本应该进 skill 的 `scripts/`。这识别了一种其他方法论常忽视的优化类型：不通过直觉设计，而是通过观察实际执行 pattern 来发现 bundle 机会。

> 来源：SKILL.md L304

### 8.9 "Fresh Eyes" 作为编码化的元认知技术

不是泛泛的 "多改几轮"，而是明确地将"写完再以全新眼光审视"编码为具体指令。这一技术出现两次（draft skill 时 + 改进 skill 时），表明 Anthropic 认为这是 prompt 写作本身需要的一种纪律性思维模式。

> 来源：SKILL.md L139, L305-306

### 8.10 沟通降级：适应非技术用户

Skill creator 被设计为可被 "plumbers opening up their terminals" 和 "grandparents googling how to install npm" 使用。术语使用有明确分级："evaluation"和"benchmark"勉强可以；"JSON"和"assertion"需要用户表现出足够的技术线索才能不解释直接使用。这反映了一种产品化的用户共情，而非纯工程视角。

> 来源：SKILL.md L34-42

---

## 附录：核心概念速查表

| 概念 | 定义 | 来源 |
|------|------|------|
| Progressive Disclosure | 三层按需加载：metadata → body → resources | SKILL.md L86-93 |
| Undertrigger | Claude 倾向不调用 skill 的行为特征 | SKILL.md L67 |
| Pushy Description | 对抗 undertrigger 的 description 写作策略 | SKILL.md L67 |
| Fresh Eyes | 写完重新以全新视角审视的修订法 | SKILL.md L139, L305 |
| Eval-Driven Loop | draft → test → review → improve → repeat | SKILL.md L474-481 |
| Blind Comparison | 不让评判者知道输出来源的 A/B 对比 | SKILL.md L325-329 |
| Assertion Discrimination | assertion 区分成功/失败的能力 | grader.md L72-80 |
| Claims Extraction | 从输出中提取并验证隐式声明的元评估 | grader.md L43-59 |
| Near-miss Negatives | 与 skill 共享关键词但不应触发的负面测试用例 | SKILL.md L356 |
| Bundle Signal | 多个测试中重复出现的 helper 脚本→应 bundle | SKILL.md L304 |
