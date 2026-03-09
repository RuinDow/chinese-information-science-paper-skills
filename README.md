# 中文情报学论文撰写 Skills

这是一个面向中文情报学论文写作的人机协同 skills 仓库。

其中，`skills/paper-writing-orchestrator` 是主 skill。实际使用时，建议直接调用它，由它作为总调度器，按阶段依次调度大纲、文献、引言、相关工作、方法、实验、结论摘要和 Word 导出等子 skills。

## 一、使用前准备

在使用本仓库之前，你首先需要有一个待撰写论文的项目目录。推荐项目结构如下：

```text
project_root/
├── notes/                       ← 原始笔记（.docx / .md / .txt）
├── experiments/                 ← 实验数据、结果图表、相关代码
├── templates/                   ← Word 模板（如有）
```

论文写作流程推进后，项目目录中会逐步生成以下文件：

```text
project_root/
├── notes/
├── experiments/
├── templates/
├── outline.md                   ← 对话 0 输出，带丰富注释的结构化大纲
├── literature_pool.md           ← 对话 0.5 输出，文献池
├── paper_content.md             ← 对话 1-5 累积输出，论文正文
└── paper_final.docx             ← 对话 6 输出，最终 Word 文档
```

## 二、仓库使用方式

请将本仓库中的各个 skill 目录整体放入你的论文项目目录下的 `skills/` 目录中使用，并保持相对位置不变。

推荐放置方式如下：

```text
project_root/
├── notes/
├── experiments/
├── templates/
└── skills/
    ├── paper-writing-orchestrator/
    ├── paper-outline-generator/
    ├── paper-introduction-generator/
    ├── paper-related-work-generator/
    ├── paper-method-generator/
    ├── paper-experiment-generator/
    ├── paper-conclusion&abstract-generator/
    └── docx/                    ← 需自行准备可用的 docx skill，推荐Claude官方skills：https://github.com/anthropics/skills
```

说明：

- 默认入口是 `skills/paper-writing-orchestrator/SKILL.md`
- 其他子 skills 由 orchestrator 按阶段自动调度

## 三、快速开始

可参考该提示词快速启动：

```text
请根据项目内容，遵循 paper-writing-orchestrator skills 的指示撰写情报学中文论文。
```

本仓库是一个人机协同的论文写作 workflow，而不是一次性自动生成整篇论文的黑盒工具。

在每个阶段，模型都会先产出当前部分的草稿，然后暂停，等待你审阅、修改和反馈。你可以：

- 直接对当前内容提出修改意见
- 让模型继续迭代润色
- 自己修改后，再让模型继续下一阶段

也就是说，中间每写完一部分都会停下来，你可以和它进行多轮对话修改；只有在你确认满意后，再进入下一段对话或下一写作阶段。

为了避免上下文长度过长后模型指令遵循能力下降，强烈建议在每一部分撰写完成后新开一个对话，再继续下一阶段。

建议在新对话中使用如下提示词：

```text
请帮我继续撰写论文。我的项目文件已上传。

请按以下步骤执行：
1. 读取 skills/paper-writing-orchestrator 的 SKILL.md
2. 根据项目中已有的文件判断当前阶段
3. 按照 orchestrator 的指引执行当前阶段
```

如果你希望指定某一章节，也可以在新对话中明确说明，例如只写引言、相关工作、方法或实验部分。

## 四、外部依赖

本仓库默认需要你自行配置以下 MCP 工具：

- Zotero MCP：`https://github.com/54yyyu/zotero-mcp`
- WebSearch MCP：`https://github.com/Aas-ee/open-webSearch`

如果未正确配置这些工具，涉及文献检索、文献库访问和 Web 搜索的阶段将无法完整运行。

## 五、仓库结构说明

当前仓库结构如下：

```text
.
├── README.md
└── skills/
    ├── paper-writing-orchestrator/
    ├── paper-outline-generator/
    ├── paper-introduction-generator/
    ├── paper-related-work-generator/
    ├── paper-method-generator/
    ├── paper-experiment-generator/
    ├── paper-conclusion&abstract-generator/
    └── docx/
```

各目录职责如下：

- `skills/paper-writing-orchestrator/`：主 skill，负责判断当前阶段并调度其他子 skills
- `skills/paper-outline-generator/`：生成论文大纲
- `skills/paper-introduction-generator/`：撰写引言
- `skills/paper-related-work-generator/`：撰写相关工作并组织文献
- `skills/paper-method-generator/`：撰写方法框架
- `skills/paper-experiment-generator/`：撰写实验与结果分析
- `skills/paper-conclusion&abstract-generator/`：撰写结论与摘要
- `skills/docx/`：负责最终 Word 导出，需用户自行提供

## 六、致谢

本仓库在设计过程中参考了以下项目，在此表示感谢：

- `https://github.com/luwill/research-skills/tree/main`
- `https://github.com/Galaxy-Dawn/claude-scholar/tree/main`
