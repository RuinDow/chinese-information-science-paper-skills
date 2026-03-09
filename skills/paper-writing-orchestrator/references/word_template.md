# Word 排版规范：中文学术论文默认模板

本文件定义了生成 paper_final.docx 时的默认排版规范。如果用户提供了自定义 Word 模板（templates/ 目录），则自定义模板优先。

## 字体规范

| 元素 | 中文字体 | 英文/数字字体 |
|------|---------|-------------|
| 论文标题 | 宋体 | Times New Roman |
| 各级标题 | 宋体 | Times New Roman |
| 正文 | 宋体 | Times New Roman |
| 图表标题 | 宋体 | Times New Roman |
| 参考文献 | 宋体 | Times New Roman |

## 字号与样式规范

| 元素 | 字号 | 加粗 | 对齐方式 | 备注 |
|------|------|------|---------|------|
| 论文标题 | 三号（16pt） | 加粗 | 居中 | 论文最顶部 |
| 一级标题（## ） | 三号（16pt） | 加粗 | 左对齐 | 如 "1. 引言"、"2. 相关工作" |
| 二级标题（### ） | 小三（15pt） | 加粗 | 左对齐 | 如 "2.1 xxx方向" |
| 正文 | 小四（12pt） | 不加粗 | 两端对齐 | 首行缩进 2 字符 |
| 图表标题 | 五号（10.5pt） | 不加粗 | 居中 | "图 1"、"表 1" |
| 参考文献条目 | 五号（10.5pt） | 不加粗 | 两端对齐 | 悬挂缩进 |

## 段落格式

| 属性 | 值 |
|------|-----|
| 行距 | 1.25 倍行距 |
| 正文首行缩进 | 2 字符（约 24pt） |
| 段前间距 | 0 |
| 段后间距 | 0 |
| 标题段前间距 | 12pt（一级标题前适当留白） |

## 页面设置

| 属性 | 值 |
|------|-----|
| 纸张大小 | A4（210mm × 297mm） |
| 上边距 | 2.54cm |
| 下边距 | 2.54cm |
| 左边距 | 3.17cm |
| 右边距 | 3.17cm |
| 页眉 | 1.5cm |
| 页脚 | 1.75cm |

## docx-js 实现要点

使用 docx-js 生成时需注意：

### 字体设置（中英文混排）
```javascript
// 正文 TextRun 示例
new TextRun({
  text: "正文内容",
  font: {
    name: "宋体",           // 中文字体
    eastAsia: "宋体",       // 显式指定东亚字体
    ascii: "Times New Roman", // 英文/数字字体
    hAnsi: "Times New Roman"
  },
  size: 24  // 小四 = 12pt = 24 half-points
})
```

### 字号对照表（docx-js 使用 half-point 单位）
| 中文字号 | pt 值 | half-points（docx-js size 参数） |
|---------|-------|-------------------------------|
| 三号 | 16pt | 32 |
| 小三 | 15pt | 30 |
| 四号 | 14pt | 28 |
| 小四 | 12pt | 24 |
| 五号 | 10.5pt | 21 |

### 首行缩进
```javascript
new Paragraph({
  indent: {
    firstLine: 480  // 2字符 ≈ 480 DXA（小四字号下）
  },
  spacing: {
    line: 312  // 1.25倍行距 = 240 * 1.25 ≈ 300，但具体值需根据字号微调
  }
})
```

### 标题样式
```javascript
// 论文标题
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [
    new TextRun({
      text: "论文标题",
      bold: true,
      font: { name: "宋体", eastAsia: "宋体", ascii: "Times New Roman", hAnsi: "Times New Roman" },
      size: 32  // 三号
    })
  ]
})

// 一级标题
new Paragraph({
  alignment: AlignmentType.LEFT,
  children: [
    new TextRun({
      text: "1. 引言",
      bold: true,
      font: { name: "宋体", eastAsia: "宋体", ascii: "Times New Roman", hAnsi: "Times New Roman" },
      size: 32  // 三号
    })
  ]
})

// 二级标题
new Paragraph({
  alignment: AlignmentType.LEFT,
  children: [
    new TextRun({
      text: "2.1 xxx方向",
      bold: true,
      font: { name: "宋体", eastAsia: "宋体", ascii: "Times New Roman", hAnsi: "Times New Roman" },
      size: 30  // 小三
    })
  ]
})
```
