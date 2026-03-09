# Phase 6：整体润色与转 Word

## 目标
对 paper_content.md 全文进行跨章节润色，然后按照 Word 模板转换为 paper_final.docx。

## 前置条件
- paper_content.md 包含所有章节（Introduction ~ Conclusion & Abstract）
- 所有章节 STATUS 标记为 completed
- literature_pool.md 存在（用于生成参考文献列表）
- 如有 Word 模板，应在 templates/ 目录中

## Part 1：整体润色

### Step 1：通读全文

读取 paper_content.md 全文，进行以下检查：

**连贯性检查**：
- 章节之间的过渡是否自然
- 是否有前后矛盾的表述
- 术语和缩写是否在首次出现时有定义，且全文一致
- 符号使用是否一致（如同一个变量不要用不同字母）

**引用完整性检查**：
- 正文中引用的 [key] 是否都能在 literature_pool.md 中找到
- literature_pool.md 中标记为 "核心论文（必引）" 的论文是否都被引用
- 图表引用编号是否连续（Figure 1, 2, 3... 无跳跃或重复）
- 表格引用编号是否连续

**数据一致性检查**：
- Abstract 中提到的数值是否与 Experiment 中一致
- Conclusion 中的总结是否与实际实验结果一致
- Introduction 中声明的贡献是否在 Method/Experiment 中得到了支撑

**语言质量检查**：
- 是否有过于口语化的表述
- 是否有重复冗余的句子
- 段落长度是否适中（不要超长段落）
- 被动语态和主动语态的使用是否合理

### Step 2：生成润色报告

将发现的问题整理为报告呈现给用户：

"全文通读完毕，发现以下问题：

**连贯性问题**（共 X 处）：
1. Section 2.3 和 Section 3.1 中对 xxx 的定义不一致...
2. ...

**引用问题**（共 X 处）：
1. [wang2024xxx] 在正文中被引用但不在文献池中...
2. ...

**数据问题**（共 X 处）：
1. Abstract 中说 accuracy 达到 95.3%，但 Table 2 中是 95.1%...
2. ...

**语言问题**（共 X 处）：
1. Section 3.2 第二段过长（约 15 句），建议拆分...
2. ...

我可以自动修复其中 [N] 处，其余需要你确认。是否继续？"

### Step 3：执行修复

用户确认后，使用 `str_replace` 逐一修复问题。
对于不确定的修改（如数据不一致），先询问用户哪个是正确值。

### Step 4：⏸️ 暂停 — 用户最终审阅

"润色修改已完成。请最终审阅 paper_content.md 全文。
这是转换为 Word 之前的最后一次审阅机会。
确认无误后，我将开始转换。"

---

## Part 2：生成参考文献列表

### Step 1：提取引用

从 paper_content.md 中提取所有 [key] 格式的引用，生成引用列表。

### Step 2：匹配文献信息

将引用 key 与 literature_pool.md 中的条目匹配，提取完整的文献信息。

### Step 3：格式化参考文献

根据目标期刊的引用格式要求（在 outline.md 头部应有注明），格式化参考文献列表。

常见格式：
- **IEEE**：编号式 [1], [2]... 按出现顺序排列
- **APA**：(Author, Year) 按字母顺序排列
- **ACM**：编号式或作者-年份式
- **Nature/Science**：上标编号

如果 outline.md 中未指定格式，询问用户。

### Step 4：替换引用标识符

将正文中的 [key] 替换为正式的引用格式：
- 如果是编号式：[key: wang2024xxx] → [7]
- 如果是作者-年份式：[key: wang2024xxx] → (Wang et al., 2024)

### Step 5：填入参考文献列表

将格式化后的参考文献列表填入 paper_content.md 中 `## 参考文献` 标题下（替换之前各章节逐步积累的条目，生成最终统一格式的完整列表）。

---

## Part 3：转换为 Word

### Step 1：加载 docx skill

```
view skills/docx/SKILL.md
```

按照 docx skill 的指引进行转换。

### Step 2：确定模板

**默认排版规范**：
读取 orchestrator 的 `references/word_template.md`，该文件定义了标准的中文学术论文 Word 排版规范（字体、字号、行距、缩进等）。

**如果用户提供了自定义 Word 模板**：
- 读取 templates/ 目录中的 .docx 模板
- 分析模板的样式定义（标题样式、正文样式、页眉页脚等）
- 自定义模板优先于 word_template.md 中的默认规范

**如果既没有自定义模板，也需要非中文排版**：
- 询问用户目标期刊/会议
- 根据期刊要求调整排版参数

### Step 3：处理特殊元素

**数学公式**：
- LaTeX 公式需要转换为 Word 可渲染的格式
- 可使用 pandoc 处理：`pandoc paper_content.md -o paper_final.docx`
- 或使用 docx-js 手动构建（对于复杂公式）

**图表**：
- 如果 paper_content.md 中引用了图片文件（如 experiments/xxx.png），插入到文档中
- 如果只有图表引用占位符，标注 "[INSERT FIGURE X HERE]" 供用户后续插入

**表格**：
- Markdown 表格转换为 Word 表格
- 确保表格格式整齐，列宽合理

### Step 4：验证输出

生成 paper_final.docx 后：
1. 使用 docx skill 中的验证工具检查文件完整性
2. 用 pandoc 或 LibreOffice 转为 PDF 预览（如果工具可用）
3. 检查页面布局、分页是否合理

### Step 5：输出最终文件

将 paper_final.docx 保存到项目根目录并呈现给用户。

"论文 Word 文档已生成：paper_final.docx

请检查：
1. 格式是否符合目标期刊要求
2. 图表位置是否正确（标注了 [INSERT FIGURE] 的位置需要手动插入）
3. 数学公式是否正确渲染
4. 参考文献格式是否正确

如有问题，请告诉我具体位置，我来修复。"

---

## 常见问题

**Q: pandoc 转换后公式显示不正常怎么办？**
A: 尝试使用 `--mathjax` 或 `--webtex` 选项。如果仍有问题，对有问题的公式使用 docx-js 手动创建 OMML 格式。

**Q: 用户要求双栏排版怎么办？**
A: 在 docx-js 中设置 section properties 的 column 属性。注意双栏排版下表格和图片可能需要特殊处理（跨栏显示）。

**Q: 参考文献数量很多，手动替换 key 容易出错怎么办？**
A: 写一个脚本自动完成替换。脚本读取 paper_content.md，建立 key → 编号 的映射，批量替换后输出。

**Q: 用户想用 LaTeX 而不是 Word 怎么办？**
A: paper_content.md 本身就是 Markdown + LaTeX 公式，可以相对容易地转为 .tex 文件。使用 pandoc：
```bash
pandoc paper_content.md -o paper.tex --template=[模板]
```
但这超出了本 orchestrator 的默认范围，需要用户额外说明。
