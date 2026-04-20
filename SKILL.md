---
name: word-master-skill
description: >
  AI-driven Word document generation and formatting system. Converts Markdown or text
  into professionally formatted .docx files with Chinese typography standards, punctuation
  correction, document type detection, and template support. Use when user asks to
  "生成Word", "写文档", "创建文档", "排版", "标点纠错", "生成报告", "生成简历",
  "写公文", "写小说", or mentions "word-master", "wpm".
---

# Word Master Skill

> AI-driven Word document generation system. Converts Markdown/text into professionally formatted .docx files with Chinese typography, punctuation correction, and template support.

**Core Pipeline**: `Content (Markdown/Text) → Generate .docx → Format (Chinese Typography) → Punctuation Check → Output`

## CLI Commands

This skill uses the `wpm` CLI (installed via `pip install word-master-cli`).

| Command | Purpose |
|---------|---------|
| `wpm create <input> [-r ref.docx] [-o output.docx]` | Generate Word from Markdown/text |
| `wpm format <input.docx> [-r ref.docx] [-o output.docx] [--type TYPE]` | Apply Chinese typography (auto-detect document type) |
| `wpm check <input.docx> [--fix] [--ai]` | Punctuation check & fix |
| `wpm template list` | List available templates |
| `wpm template import <docx>` | Import template from existing .docx |
| `wpm config show` | Show current config |
| `wpm config set <key> <value>` | Set config value |

**Global flags**: `--json` (structured output for agents), `--quiet` (suppress output)

## Input Modes

`wpm create` accepts three input modes:

| Mode | Example |
|------|---------|
| File path | `wpm create report.md -o output.docx` |
| Direct text | `wpm create "## 标题\n正文" -o output.docx` |
| Stdin pipe | `echo "## 标题" \| wpm create - -o output.docx` |

> **No AI configuration required**. The tool processes text/Markdown directly. AI backends (OpenAI/Claude) are optional for enhanced punctuation checking only.

---

## Workflow

### Step 1: Content Preparation

🚧 **GATE**: User requests a Word document or provides content.

**Determine content source**:

| User Provides | Action |
|---------------|--------|
| Markdown file | Use directly |
| Text description | Use as direct text input |
| Conversation content | Generate Markdown first, then pipe to wpm |
| Reference .docx | Note for style extraction |

> **Language rule**: Match the user's language. If user writes in Chinese, generate Chinese content.

> **Content generation**: Generate complete Markdown content based on user's request. Include proper heading hierarchy, paragraphs, lists, and tables as appropriate.

**Checkpoint — Content ready. Proceed to Step 2.**

---

### Step 2: Document Generation

🚧 **GATE**: Step 1 complete; Markdown content is ready.

Generate the Word document:

```bash
# From Markdown file
wpm create content.md -o output.docx

# From direct text
wpm create "<markdown_content>" -o output.docx

# With reference document styles
wpm create content.md -r reference.docx -o output.docx

# JSON output for agent parsing
wpm create content.md -o output.docx --json
```

**Default output**: Same directory as input, with `.docx` extension. No `-o` needed for same-location output.

**Intermediate files**: Uses `_wip` suffix during processing to prevent data loss on interruption.

**Checkpoint — .docx generated. Proceed to Step 3.**

---

### Step 3: Typography Formatting

🚧 **GATE**: Step 2 complete; .docx file generated.

Apply Chinese typography standards:

```bash
# Format with auto-detected document type (interactive prompt)
wpm format output.docx

# Format with specific document type (skip detection)
wpm format output.docx --type resume
wpm format output.docx --type novel
wpm format output.docx --type official
wpm format output.docx --type general

# Format with reference document styles
wpm format output.docx -r reference.docx

# Format to separate output
wpm format output.docx -o formatted.docx
```

**Document type presets** (auto-detected or via `--type`):

| Type | Body Font | Body Size | Line Spacing | Indent | Use Case |
|------|-----------|-----------|--------------|--------|----------|
| `general` | 宋体 | 12pt (小四) | 1.5x | 2 chars | General documents (default) |
| `resume` | 宋体 | 10.5pt (五号) | 1.25x | none | Resumes/CVs |
| `novel` | 宋体 | 12pt (小四) | 1.5x | 2 chars | Fiction/literature |
| `official` | 仿宋 | 16pt (三号) | 1.0x | 2 chars | Government docs (GB/T 9704) |

**Detection features**: Resume (keywords + contact info), Novel (chapter headings + dialogue), Official (formal language + closing phrases). Falls back to `general` if confidence is low.

**Interactive behavior**: In terminal mode, prompts user to confirm detected type. With `--json` or `--quiet`, auto-applies detected type. Use `--type` to skip detection entirely.

**Checkpoint — Formatting applied. Proceed to Step 4.**

---

### Step 4: Punctuation Check

🚧 **GATE**: Step 3 complete; document formatted.

Check and fix punctuation:

```bash
# Check only (report issues)
wpm check output.docx

# Auto-fix issues
wpm check output.docx --fix

# Check result in JSON
wpm check output.docx --json
```

**Rule-based checks** (no AI needed):
- Chinese/English punctuation mixing
- Full-width/half-width consistency
- Unmatched paired symbols (brackets, quotes)
- Extra spaces around punctuation
- **Consecutive punctuation (context-aware)**:
  - Same punctuation repeated (`。。`→`。`, `！！！`→`！`)
  - Terminal punctuation followed by non-terminal (`。，`→`。`, `！；`→`！`)

**Checkpoint — Document finalized.**

---

## Full Pipeline (One-shot)

For agent workflows, combine all steps:

```bash
# Generate → Format (auto-detect type) → Check
wpm create "## 报告标题\n正文内容" -o report.docx && \
wpm format report.docx && \
wpm check report.docx --fix

# Generate → Format (specific type) → Check
wpm create "resume.md" -o resume.docx && \
wpm format resume.docx --type resume && \
wpm check resume.docx --fix
```

Or with JSON output for each stage:

```bash
wpm create content.md -o report.docx --json && \
wpm format report.docx --json && \
wpm check report.docx --fix --json
```

## Template System

### Using Templates

```bash
# List available templates
wpm template list --json

# Create with template
wpm create content.md -t official_report -o output.docx
```

### Importing Templates

```bash
# Import from existing .docx
wpm template import reference.docx -n my_template
```

Templates are searched in this order:
1. CLI-specified path
2. Project `.wpm/templates/`
3. User `~/.wpm/templates/`
4. Package bundled templates

## Output Conventions

| Scenario | Exit Code | Output |
|----------|-----------|--------|
| Success | 0 | Result to stdout (--json) or stderr (human) |
| Error | 1 | `[ERROR]` to stderr, JSON error to stdout (--json) |
| Warning | 0 | `[WARN]` to stderr |

**JSON error format**:
```json
{"error": "message", "code": "error_code", "suggestions": ["fix suggestion"]}
```

## Reference Documents

- `references/chinese-typography.md` — Chinese typography standards and conventions
- `references/document-types.md` — Common document types and structures
- `workflows/create-from-conversation.md` — Workflow for generating docs from conversation

## Notes

- **No AI required**: All core features work without AI provider configuration
- **In-place editing**: `wpm format` and `wpm check --fix` modify files in-place by default
- **Safe writes**: Intermediate `_wip` files prevent corruption on interruption
- **Agent-friendly**: Always use `--json` for reliable parsing in agent workflows
