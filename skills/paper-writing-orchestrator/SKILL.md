---
name: paper-writing-orchestrator
description: "Orchestrate the full workflow of AI-assisted academic paper writing. Use this skill whenever the user mentions writing a paper, drafting a manuscript, academic writing, or thesis writing. Also trigger when the user mentions outline generation, literature review, paper sections (introduction, method, experiment, conclusion, abstract), or converting a paper draft to Word format. This is the master coordinator — it calls sub-skills (paper-outline-generator, paper-introduction-generator, paper-related-work-generator, paper-method-generator, paper-experiment-generator, paper-conclusion-abstract-generator) in sequence. Always use this skill first, even if the user only asks to write one section."
---

# Paper Writing Orchestrator

This skill coordinates the complete academic paper writing pipeline. It manages a multi-conversation workflow where each phase produces files that serve as input to the next phase.

## Architecture Overview

The workflow spans multiple conversations, connected by shared files:

```
对话 0   ── 项目理解 + 大纲生成 ──→ outline.md + paper_content.md（标题骨架）
                                        ↓ (人工修订确认)
对话 0.5 ── 文献广域检索 + 汇总 ──→ literature_pool.md
                                        ↓ (人工筛选确认)
对话 1   ── 写 引言      ──→ paper_content.md (填入对应标题下)
对话 2   ── 写 相关工作      ──→ paper_content.md (填入对应标题下) + 补充 literature_pool.md
对话 3   ── 写 方法框架            ──→ paper_content.md (填入对应标题下)
对话 4   ── 写 实验验证        ──→ paper_content.md (填入对应标题下)
对话 5   ── 写 结论 & 摘要 → paper_content.md (填入对应标题下)
对话 6   ── 整体润色 + 转 Word   ──→ paper_final.docx
```

## File Convention (所有对话必须遵守)

### 项目工作区结构
```
project_root/                    ← 用户上传的项目文件夹
├── notes/                       ← 原始笔记（.docx / .md / .txt）
├── experiments/                 ← 实验数据、结果图表、相关代码
├── templates/                   ← Word 模板（如有）
│
├── outline.md                   ← 对话 0 输出，带丰富注释的结构化大纲
├── literature_pool.md           ← 对话 0.5 输出，文献池
├── paper_content.md             ← 对话 1-5 累积输出，论文正文
└── paper_final.docx             ← 对话 6 输出，最终 Word 文档
```

### paper_content.md 格式约定

paper_content.md 在阶段 0 由大纲生成时同步创建，初始为**标题骨架**（只有标题、没有正文），后续每章撰写时将内容填入对应标题下方。

**初始骨架示例**（阶段 0 输出）：
```markdown
<!-- PAPER_TITLE: 基于xxx的xxx方法研究 -->
<!-- TARGET_VENUE: [期刊/会议名称] -->

# 基于xxx的xxx方法研究

<!-- === CHAPTER: Abstract === -->
<!-- STATUS: pending -->
## 摘要

[待撰写]

<!-- === CHAPTER: Introduction === -->
<!-- STATUS: pending -->
## 1. 引言

[待撰写]

<!-- === CHAPTER: Related Work === -->
<!-- STATUS: pending -->
## 2. 相关工作

### 2.1 xxx方向
[待撰写]

### 2.2 xxx方向
[待撰写]

<!-- === CHAPTER: Method === -->
<!-- STATUS: pending -->
## 3. 方法

### 3.1 整体框架
[待撰写]

### 3.2 xxx模块
[待撰写]

<!-- === CHAPTER: Experiment === -->
<!-- STATUS: pending -->
## 4. 实验

### 4.1 实验设置
[待撰写]

### 4.2 主实验结果
[待撰写]

### 4.3 消融实验
[待撰写]

<!-- === CHAPTER: Conclusion === -->
<!-- STATUS: pending -->
## 5. 结论

[待撰写]

<!-- === CHAPTER: References === -->
<!-- STATUS: pending -->
## 参考文献

[待生成]
```

**填入规则**：
- 每章撰写完成后，用 `str_replace` 将 `[待撰写]` 替换为实际内容
- 将 STATUS 从 `pending` 改为 `completed`
- 如果章节内容需要拆分放置（如 Related Work 的正文放在 §2 下，引用的参考文献放在 §参考文献 下），按标题位置分别填入
- 保持标题层级和编号与 outline.md 一致
- **论文标题**在骨架头部的 `# 标题` 处，必须在阶段 0 确定

### literature_pool.md 格式约定
```markdown
# Literature Pool

## 分类：[主题标签]
- [key: author2024keyword] Author et al., "Title", Venue Year
  摘要：...
  与本文关系：...
  适用章节：Introduction, Related Work, ...

- [key: ...] ...
```

## Workflow Execution

根据用户当前所处的阶段，读取对应的详细指引文件并执行。

### 判断当前阶段

当用户发起对话时，按以下逻辑判断该执行哪个阶段：

1. 如果用户明确说了要执行哪个阶段 → 直接执行该阶段
2. 如果项目中没有 outline.md → 执行阶段 0
3. 如果有 outline.md 但没有 literature_pool.md → 执行阶段 0.5
4. 如果有 literature_pool.md 但 paper_content.md 中所有章节 STATUS 为 pending → 执行阶段 1（Introduction）
5. 如果有 paper_content.md → 读取其中的 STATUS 注释，找到下一个 `pending` 状态的章节

### 阶段 0：项目理解与大纲生成

**详细指引**：`view references/phase0-outline.md`

**简要流程**：
1. 读取用户项目中的 notes/ 目录下的所有文字记录
2. 读取 experiments/ 中的关键结果
3. 加载 `paper-outline-generator` skill（先 view 其目录结构，再读 SKILL.md）
4. 与用户确认**论文标题**
5. 根据 skill 指引生成大纲初稿
6. ⏸️ **暂停点**：输出大纲供用户审阅修改
7. 用户确认后同时输出 `outline.md` 和 `paper_content.md`（标题骨架）

### 阶段 0.5：文献广域检索与汇总

**详细指引**：`view references/phase05-literature.md`

**简要流程**：
1. 读取 outline.md，提取各章节涉及的研究主题
2. 对每个主题进行文献检索（使用 web_search 或 Zotero 等工具）
3. 对检索结果进行去重、分类、摘要
4. 生成 literature_pool.md 初稿
5. ⏸️ **暂停点**：输出文献池供用户筛选补充
6. 用户确认后输出最终 `literature_pool.md`

### 阶段 1-5：逐章撰写

**详细指引**：`view references/phase1to5-chapters.md`

**简要流程（每章独立对话）**：
1. 读取 outline.md（了解全局结构和本章要点）
2. 读取 paper_content.md（了解已完成内容 + 定位本章标题位置，保持连贯性）
3. 读取 literature_pool.md（获取可引用的文献）
4. 加载对应章节的 generator skill（先 view 目录，再读 SKILL.md，按其指引执行）
5. 撰写本章内容
6. ⏸️ **暂停点**：输出本章内容供用户审阅
7. 用户确认后，将内容填入 paper_content.md 中对应标题下（替换 `[待撰写]`），如有参考文献则同步填入 `## 参考文献` 下

**章节与 skill 对应关系**：
| 章节 | Generator Skill | 对话编号 |
|------|----------------|---------|
| Introduction | paper-introduction-generator | 对话 1 |
| Related Work | paper-related-work-generator | 对话 2 |
| Method | paper-method-generator | 对话 3 |
| Experiment | paper-experiment-generator | 对话 4 |
| Conclusion & Abstract | paper-conclusion-abstract-generator | 对话 5 |

注意：Related Work 章节撰写时可能需要补充检索文献，此时更新 literature_pool.md。

### 阶段 6：整体润色与转 Word

**详细指引**：`view references/phase6-finalize.md`

**简要流程**：
1. 通读 paper_content.md 全文
2. 检查跨章节的连贯性、术语一致性、引用完整性
3. 润色修改
4. ⏸️ **暂停点**：输出润色后全文供用户最终确认
5. 读取 `skills/docx/SKILL.md`，按其指引将 Markdown 转为 Word
6. 读取 `references/word_template.md` 获取默认排版规范（用户如提供自定义模板则优先使用）
7. 输出 paper_final.docx

## Sub-Skill Invocation Protocol

加载任何子 skill 时，必须遵循以下协议：

```
步骤 1: view skills/[skill-name]/        ← 查看目录结构
步骤 2: view skills/[skill-name]/SKILL.md ← 读取主指引
步骤 3: 按 SKILL.md 中的指示调用 scripts/、references/ 等资源
步骤 4: 执行 skill 定义的完整流程
```

**关键原则**：
- 不要跳过步骤 1，子 skill 可能有 scripts/ 或 references/ 目录
- 严格按照子 skill 的 SKILL.md 执行，orchestrator 不越权干预具体写作逻辑
- 如果子 skill 需要外部工具（如文献检索），按其要求调用

## Pause Points (暂停点规则)

标记为 ⏸️ 的步骤是**强制暂停点**。到达暂停点时：
1. 将当前阶段的产出物呈现给用户
2. 明确告知用户："本阶段初稿已完成，请审阅。你可以：(a) 提出修改意见，我来修改；(b) 自行修改后告诉我继续；(c) 确认通过，进入下一阶段。"
3. **不得跳过暂停点自行继续下一阶段**

## New Conversation Startup Template

用户在每个新对话中可以使用以下模板快速启动：

---

**通用启动模板（用户复制粘贴）**：

```
请帮我继续撰写论文。我的项目文件已上传。

请按以下步骤执行：
1. 读取 skills/paper-writing-orchestrator 的 SKILL.md
2. 根据项目中已有的文件判断当前阶段
3. 按照 orchestrator 的指引执行当前阶段
```

**指定章节的启动模板**：

```
请帮我撰写论文的 [Method] 部分。

请按以下步骤执行：
1. 读取 skills/paper-writing-orchestrator 的 SKILL.md
2. 读取 outline.md 和 paper_content.md
3. 使用 paper-method-generator skill 撰写 Method 章节
4. 完成后将内容填入 paper_content.md 对应标题下
```

---

## Writing Style Defaults (可被子 skill 覆盖)

除非子 skill 另有规定，论文撰写遵循以下默认风格：
- 学术正式语体，优先使用被动语态
- 引用格式：[Author, Year] 或 [数字]（根据目标期刊要求）
- 公式使用 LaTeX 语法：$inline$ 或 $$display$$
- 图表引用：Figure X / Table X
- 段落之间有逻辑过渡，避免突兀跳转
- 每章开头有简要引导段，说明本章内容和组织结构
