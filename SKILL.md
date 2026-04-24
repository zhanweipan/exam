---
name: exam-generator
description: Generate structured exam papers from a detailed item specification table (细目表). Reads an Excel file specifying question types, topics, points, difficulty levels, key competencies, and subject literacy requirements, then produces a complete Word document containing exam questions, answer key, and detailed solutions. Use when the user mentions 细目表, 出题, 命题, exam paper generation, or test creation from a specification table.
---

# Exam Generator from Specification Table

## Overview

This skill automates the creation of standardized exam papers from a 细目表 (detailed item specification table) in Excel format. It ensures every question aligns with the prescribed type, topic, difficulty, points, key competencies, and subject literacy targets.

## Input Requirements

The specification table must contain at least these columns:

| Column | Meaning |
|--------|---------|
| 题型 | Question type (e.g., 单项选择题, 多项选择题, 实验题, 计算题) |
| 题序 | Question number |
| 分值 | Points per question |
| 知识内容 | Topic / knowledge area covered |
| 难度要求 | Difficulty: 易 / 中 / 难 |
| 关键能力 | Key competencies (e.g., 理解能力, 推理论证能力, 模型建构能力, 实验探究能力, 创新能力) |
| 学科素养 | Subject literacy (e.g., 物理观念, 科学思维, 科学探究, 科学态度与责任) |

## Workflow

### Phase 1: Analyze the Specification Table

1. Read the Excel file with pandas or openpyxl.
2. Parse each row and build a structured question list.
3. Verify totals: sum of 分值 should equal the expected full score (commonly 100 or 150).
4. Note the difficulty distribution to guide question design.

### Phase 2: Draft Questions

For each row in the table, create a question that matches **all** constraints:

- **Question type** determines format:
  - 单项选择题: 4 options (A/B/C/D), only one correct
  - 多项选择题: 4 options, one or more correct; scoring rules (e.g., full / partial / zero)
  - 实验题: experimental procedure, data analysis, or design; typically fill-in or short-answer
  - 计算题: full calculation with required formulas and reasoning steps

- **Knowledge content** dictates the specific concept, law, or method being tested.
- **Difficulty** controls complexity:
  - 易: direct recall, single-step calculation, basic concept identification
  - 中: moderate reasoning, multi-step calculation, simple synthesis
  - 难: complex synthesis, critical thinking, multiple-model combination

- **Key competencies & subject literacy** guide the cognitive demand of the question.

**Writing guidelines:**
- Use clear, concise language. Avoid ambiguous phrasing.
- For calculation questions, provide all necessary data (mass, length, g, angles, etc.).
- If a question would normally require a diagram, replace "如图所示" with detailed textual descriptions of the physical setup, geometry, and spatial relationships.
- Ensure options for multiple-choice questions are plausible distractors rooted in common misconceptions.

### Phase 3: Prepare Answer Key and Solutions

After the questions, append a section titled "参考答案与解析":

- List correct answers for objective questions.
- Provide step-by-step solutions for calculation questions, including:
  - Stated physical principles / laws
  - Formula substitution
  - Numerical results with units
- For each question, write a brief 解析 explaining why the correct answer is right and why distractors are wrong.

### Phase 4: Generate Word Document

Use **docx-js** to create the `.docx` file.

**Document structure:**

```
Title (centered, bold, large font)
Exam info line (满分, 考试时间)

Section 1: 单项选择题 (bold header + scoring rule)
  Questions with indented options

Section 2: 多项选择题 (bold header + scoring rule)
  Questions with indented options

Section 3: 实验题 (bold header)
  Questions with sub-questions (1)(2)(3)

Section 4: 计算题 (bold header + solution requirements)
  Questions with sub-questions

Page break

参考答案与解析 (centered title)
  Answers and explanations per section
```

**Critical formatting rules for docx-js:**
- Never use `\n` inside TextRun; always use separate Paragraph elements.
- Set default font to 宋体 (or the user's preferred font), size 24 (12pt).
- Use indent for options and sub-questions (e.g., `{ left: 420 }` for options, `{ left: 840 }` for sub-questions).
- Use PageBreak inside a Paragraph to separate the exam from the answer key.

**Example snippet:**

```javascript
const { Document, Packer, Paragraph, TextRun, AlignmentType, PageBreak } = require('docx');
const fs = require('fs');

function p(text, opts = {}) {
  return new Paragraph({
    spacing: { before: opts.before ?? 100, after: opts.after ?? 100, line: 360 },
    alignment: opts.align ?? AlignmentType.LEFT,
    indent: opts.indent ? { left: opts.indent } : undefined,
    children: [new TextRun({ text, size: opts.size ?? 24, font: "宋体", bold: opts.bold ?? false })]
  });
}

const children = [];
children.push(p("试卷标题", { align: AlignmentType.CENTER, bold: true, size: 36 }));
// ... append all questions and answers ...
children.push(new Paragraph({ children: [new PageBreak()] }));
children.push(p("参考答案与解析", { align: AlignmentType.CENTER, bold: true, size: 32 }));

const doc = new Document({
  styles: { default: { document: { run: { font: "宋体", size: 24 } } } },
  sections: [{ properties: { page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } }, children }]
});

Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync("output.docx", buffer);
});
```

### Phase 5: Deliver the File

1. Save the generated `.docx` to the user's selected working folder.
2. Provide a `file://` link so the user can open it directly.
3. Summarize the exam structure (question counts, points, difficulty distribution) for verification.

## Verification Checklist

Before delivering the exam, confirm:
- [ ] Total points match the specification table.
- [ ] Every row in the table has a corresponding question.
- [ ] Difficulty labels (易/中/难) are correctly assigned.
- [ ] No "如图所示" remains without a textual description.
- [ ] All calculation questions include complete solutions in the answer key.
- [ ] Word document opens correctly and formatting is consistent.

## Common Pitfalls

- **Chinese quotation marks inside JS strings**: Characters `"` and `"` break JavaScript string literals. Replace them with「」or other brackets before writing docx-js code.
- **Missing docx dependency**: Install docx locally in the workspace (`npm install docx`) if the script fails with MODULE_NOT_FOUND.
- **Unicode encoding errors when reading Excel**: Write pandas output to a file with UTF-8 encoding instead of printing to console.
- **Formulas not evaluated**: docx-js creates formulas as text but does not compute them. If you embed formulas, use formula_processor.py or keep answers in static text.
