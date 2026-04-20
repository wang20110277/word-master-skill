# Word Master Skill

Claude Code skill for AI-driven Word document generation and formatting. Contains workflow instructions, Chinese typography references, and the skill manifest used by `wpm`.

## Structure

```
word-master-skill/
├── SKILL.md              # Skill definition (loaded by Claude Code / Codex / Gemini)
├── README.md             # This file
├── references/           # Technical references
│   ├── chinese-typography.md   # Chinese typography standards
│   └── document-types.md       # Common document structures
├── workflows/            # Workflow guides
│   └── create-from-conversation.md  # Generate docs from conversation
└── templates/            # Example templates and schemas
```

## Usage

Install the skill into a target project:

```bash
# Copy skill to project's .claude/skills directory
cp -r /path/to/word-master-skill /your/project/.claude/skills/word-master

# Or add to CLAUDE.md
echo '- Read .claude/skills/word-master/SKILL.md when user asks about Word documents' >> CLAUDE.md
```

## Prerequisites

```bash
pip install word-master-cli
```

## Related

- [word-master-cli](../word-master-cli) — The core engine (`wpm` CLI)
