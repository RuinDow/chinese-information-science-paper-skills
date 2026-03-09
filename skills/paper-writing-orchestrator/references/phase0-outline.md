# Phase 0：项目理解与大纲生成

## 目标
读取用户的项目素材（笔记、实验数据、代码），生成一份带丰富注释的结构化大纲 outline.md，并同步生成 paper_content.md 标题骨架。

## 执行步骤

### Step 1：扫描项目结构
```
view [project_root]/
```
了解项目包含哪些文件和目录，建立全局认知。

### Step 2：读取文字记录
按优先级读取 notes/ 目录下的文件：
1. 如果是 .md 或 .txt → 直接用 `view` 读取
2. 如果是 .docx → 用 pandoc 转换后读取：
   ```bash
   pandoc notes/xxx.docx -o /tmp/xxx.md
   ```
3. 如果文件很多，先列出文件名让用户指定优先级

### Step 3：读取实验结果与代码
浏览 experiments/ 目录：
- 读取结果文件（.csv, .json, .txt）中的关键数值
- 查看图表文件名（.png, .pdf），记录有哪些图可用
- 如果有 README 或说明文件，优先读取
- 浏览代码文件：关注模型定义文件的类名和关键结构、配置文件中的超参数、训练脚本的核心逻辑（不需要逐行阅读）

### Step 4：确认论文标题

在生成大纲之前，与用户确认论文标题。论文标题将写入 paper_content.md 的顶部。

"在生成大纲之前，请确认论文标题。根据你的项目素材，我建议以下标题：
1. [建议标题 A]
2. [建议标题 B]

你也可以直接提供你想要的标题。"

### Step 5：加载 outline generator skill
```
view skills/paper-outline-generator/
view skills/paper-outline-generator/SKILL.md
```
按照该 skill 的指引生成大纲。

### Step 6：丰富大纲注释
生成的大纲不应只是标题列表，而是一份**带上下文的写作蓝图**。每个小节都应包含：

```markdown
## 3. Method
### 3.1 模型架构
- **核心要点**：基于 Transformer 的 xxx 架构，加入了 xxx 模块
- **关键数据引用**：见 experiments/model_config.json
- **相关代码**：experiments/src/model.py 第 45-120 行（XxxModel 类）
- **待写内容**：需要解释为什么选择 xxx 而非 yyy
- **预计图表**：Figure 2 - 模型架构图（需要绘制 / 已有 experiments/arch.png）
- **预计引用方向**：Transformer 原始论文、xxx 相关改进工作

### 3.2 训练策略
- **核心要点**：两阶段训练，先预训练再微调
- **关键数值**：lr=1e-4, batch_size=32, epochs=50（见 experiments/config.yaml）
- **实验结果引用**：training_curves.png, ablation_results.csv
- **待写内容**：解释两阶段训练的动机，对比单阶段的效果
- **预计图表**：Figure 3 - 训练曲线对比
- **预计引用方向**：预训练范式相关工作、学习率调度策略
```

这些注释将在后续每章撰写时成为关键的上下文信息。

### Step 7：⏸️ 暂停 — 用户审阅

将大纲呈现给用户，并说明：

"大纲初稿已完成。请审阅以下内容：
1. 章节结构是否合理？是否需要增减章节？
2. 每个小节的要点是否准确？是否有遗漏？
3. 数据/代码/图表的引用是否正确？
4. 写作方向（待写内容）是否符合你的意图？

你可以直接在大纲上修改，也可以告诉我你的修改意见。"

### Step 8：迭代修改
根据用户反馈修改大纲，可能需要多轮。每轮修改后再次暂停确认。

### Step 9：输出 outline.md 和 paper_content.md

用户确认后：

**1. 保存 outline.md** — 将最终大纲保存到项目根目录。

**2. 生成 paper_content.md 标题骨架** — 从 outline.md 中提取所有标题层级，生成只含标题的骨架文件。规则：
- 第一行写论文标题（`# 论文标题`）
- 按 outline.md 的章节结构，写入所有 `##` 一级标题和 `###` 二级标题
- 每个标题下写 `[待撰写]` 占位符
- 用 HTML 注释标记每章的 STATUS 为 `pending`
- 末尾必须包含 `## 参考文献` 标题（内容为 `[待生成]`）
- 参考 SKILL.md 中的 paper_content.md 格式约定

示例：
```markdown
<!-- PAPER_TITLE: 基于图神经网络的交通流量预测方法 -->
# 基于图神经网络的交通流量预测方法

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
### 2.1 交通流量预测方法
[待撰写]
### 2.2 图神经网络
[待撰写]

...（其余章节同理）

<!-- === CHAPTER: References === -->
<!-- STATUS: pending -->
## 参考文献
[待生成]
```

提示用户：
"outline.md 和 paper_content.md（标题骨架）已生成。接下来请：
1. 开一个新对话
2. 上传项目文件（包含 outline.md 和 paper_content.md）
3. 使用启动模板进入阶段 0.5（文献检索）"

## 常见问题

**Q: 用户的笔记非常零散怎么办？**
A: 先将所有笔记内容读取并整理成一份内部摘要（不输出给用户），基于摘要生成大纲。整理过程中如果有不理解的内容，向用户提问。

**Q: 用户没有提供实验数据怎么办？**
A: 大纲中对应的实验部分标注为 "[待补充实验数据]"，其余部分正常生成。

**Q: 用户想用特定的论文模板结构（如 IEEE、ACM）怎么办？**
A: 在生成大纲前询问目标期刊/会议，按其常见结构组织章节。在 outline.md 头部注明目标 venue。
