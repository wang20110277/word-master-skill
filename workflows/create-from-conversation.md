# Workflow: Generate Document from Conversation

## 适用场景

用户在对话中描述了文档需求，但没有提供现成的 Markdown 文件。AI 需要根据对话内容生成 Markdown，再通过 wpm 生成 Word 文档。

## 步骤

### 1. 确认文档类型和内容

根据用户的描述，判断文档类型：

| 关键词 | 推荐类型 |
|--------|----------|
| 报告、汇报、总结 | report |
| 简历、履历 | resume |
| 会议、纪要 | meeting_minutes |
| 方案、计划 | proposal |
| 合同、协议 | contract |

向用户确认：
- 文档类型
- 大致内容范围
- 是否有参考文档（样式模板）

### 2. 生成 Markdown 内容

根据确认的类型和内容，生成结构化的 Markdown：

```markdown
# 文档标题

## 摘要
简短摘要...

## 一、第一部分
正文内容...

### 1. 子节内容
- 要点一
- 要点二

## 二、第二部分
正文内容...

| 项目 | 数值 |
|------|------|
| ... | ... |

## 三、结论
总结内容...
```

### 3. 使用 wpm 生成文档

```bash
# 直接文本模式
wpm create "<生成的 Markdown>" -o output.docx

# 或先保存为文件再转换
# 保存 Markdown → wpm create file.md -o output.docx
```

### 4. 排版和检查

```bash
# 原地排版
wpm format output.docx

# 标点检查和修复
wpm check output.docx --fix
```

### 5. 确认输出

```bash
# 查看文件信息
ls -la output.docx

# JSON 模式查看结果
wpm check output.docx --json
```

## 注意事项

- 生成 Markdown 时遵循中文排版习惯（标题编号、段落结构）
- 内容生成后先让用户确认再生成文档
- 如果用户有参考文档，使用 `-r ref.docx` 提取样式
- 大文档建议分章节生成再合并
