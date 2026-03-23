---
name: writing-skills
description: Use when creating a new skill, editing an existing skill, or helping someone structure a SKILL.md. Triggers on "write a skill", "create a skill", "skill for X", "SKILL.md", or when the user has instructions/best-practices to package as a reusable skill.
---

# Writing Skills

## Installation

**[AGENT] Detect platform and install:**

```bash
# Detect platform
if [ -d "$HOME/.claude/skills" ]; then
  PLATFORM="claude-code"
  SKILL_DIR="$HOME/.claude/skills/writing-skills"
elif [ -d "$HOME/.gemini/skills" ]; then
  PLATFORM="gemini-cli"
  SKILL_DIR="$HOME/.gemini/skills/writing-skills"
else
  PLATFORM="unknown"
  SKILL_DIR="$HOME/.claude/skills/writing-skills"
fi

# Install
mkdir -p "$SKILL_DIR/references"
curl -fsSL "https://raw.githubusercontent.com/jasonhp/jason-skills/main/writing-skills/SKILL.md" -o "$SKILL_DIR/SKILL.md"
curl -fsSL "https://raw.githubusercontent.com/jasonhp/jason-skills/main/writing-skills/references/testing-methodology.md" -o "$SKILL_DIR/references/testing-methodology.md"
echo "Installed to $SKILL_DIR (platform: $PLATFORM)"
```

**Definition of Done — verify ALL before confirming:**
- [ ] `SKILL.md` exists at `$SKILL_DIR/SKILL.md`
- [ ] `references/testing-methodology.md` exists at `$SKILL_DIR/references/testing-methodology.md`
- [ ] Both files are non-empty

**[AGENT] After install, confirm with:**
> writing-skills installed successfully to `[path]`. Skill includes 1 reference file (testing-methodology.md). Ready to use.

---

A step-by-step guide for writing high-quality SKILL.md files. Combines Google Cloud's 5 design patterns, three-level loading, Claude Search Optimization, and production-tested best practices.

**Core principle:** A well-written SKILL.md is all you need. Skills are Markdown instruction documents that teach agents how to use built-in tools. No code required unless wrapping a CLI.

## When to Create a Skill

**Create when:**
- Technique wasn't intuitively obvious and you'd reference it again across projects
- Pattern applies broadly, not just to one project (project-specific conventions belong in CLAUDE.md)
- Others would benefit from the same guidance

**Don't create for:**
- One-off solutions or standard practices well-documented elsewhere
- Mechanical constraints — if it's enforceable with validation/linting, automate it instead

---

## Background: Three-Level Loading

Structure content for progressive loading to minimize token usage:

| Level | What | Token budget | When loaded |
|-------|------|-------------|-------------|
| **L1 — Metadata** | Frontmatter (name, description) | ~100 tokens | Always — every conversation |
| **L2 — Instructions** | SKILL.md body | ~2000 tokens | When skill is triggered |
| **L3 — References** | Files in `references/` | Unlimited | When explicitly instructed in L2 |

```
your-skill/
├── SKILL.md              # L1 + L2
├── references/           # L3 — read on demand via explicit "Read 'references/x.md'" instructions
│   └── detailed-ref.md
└── assets/               # Templates (optional)
    └── template.md
```

**L3 is only loaded when the skill body contains an explicit imperative:** `"Read 'references/x.md' for ..."`. Passive mentions ("see references/x.md") are ignored by agents.

---

## Step 1 — Choose a Design Pattern

| Pattern | When to use | Key trait |
|---------|------------|-----------|
| **Tool Wrapper** | Wrapping an existing CLI or API | Loads reference docs on demand, applies rules to user's code |
| **Generator** | Creating artifacts (code, config, docs) | Fills a template with user-provided variables |
| **Reviewer** | Analyzing and providing feedback | Scores against a checklist by severity |
| **Inversion** | Agent must gather context before acting | Interviews the user, refuses to act until requirements are complete |
| **Pipeline** | Multi-step orchestrated workflow | Strict sequential steps with hard checkpoints |

**Decision tree:**
- Teach an agent a library/tool? → **Tool Wrapper**
- Consistent structured output? → **Generator**
- Evaluate/audit something? → **Reviewer**
- Agent tends to guess instead of asking? → **Inversion**
- Complex multi-step process with checkpoints? → **Pipeline**

**Patterns compose.** A Pipeline can include a Reviewer step. A Generator can start with Inversion to gather variables first.

## Step 2 — Write the Frontmatter (L1)

The frontmatter is **always in context** — every token counts.

```yaml
---
name: your-skill-name
description: Use when [specific triggering conditions]. Triggers on "phrase 1", "phrase 2", or when [situation].
---
```

**Rules:**
- `name`: Letters, numbers, hyphens only. Verb-first, active voice (`creating-skills` not `skill-creation`)
- `description`: Max 500 characters. Start with "Use when..." Third person. **ONLY describe triggering conditions — NEVER summarize the workflow**
- Include natural-language trigger phrases users would actually say

**Why never summarize workflow in description?** When a description summarizes the skill's process, agents follow that shortcut instead of reading the full skill body. The skill body becomes documentation the agent skips.

```yaml
# BAD: Summarizes workflow — agent follows this instead of reading the skill
description: Creates skills by choosing a pattern, writing frontmatter, then testing with subagents

# GOOD: Just triggering conditions
description: Use when creating a new skill, editing an existing skill, or structuring a SKILL.md
```

**Keyword coverage:** Use words agents would search for — error messages, symptoms, synonyms, tool names.

## Step 3 — Write the Body (L2)

### Required Sections

| Section | Purpose |
|---------|---------|
| **Overview** | What is this skill? Core principle in 1-2 sentences |
| **When to Use / Quick Reference** | Triggering conditions, key facts at a glance |
| **Common Workflows** | Step-by-step for the top 3-5 use cases |
| **Gotchas** | Non-obvious constraints and defaults |
| **Common Mistakes** | What goes wrong + fixes (can include troubleshooting table) |

### Additional Sections by Pattern

- **Tool Wrapper:** Step 0 (prerequisite check), Security section, Understanding Output section
- **Generator:** Template variables list, output format spec
- **Reviewer:** Rubric/checklist reference, severity level definitions
- **Inversion:** Phased questions with explicit gate conditions ("DO NOT proceed until...")
- **Pipeline:** Numbered steps with hard checkpoints and user confirmation gates

### Writing Guidelines

- One excellent example beats many mediocre ones. Pick the most relevant language and scenario
- Code inline if < 50 lines. Use `Read 'references/x.md'` imperative for longer content
- Flowcharts ONLY for non-obvious decision points. Use tables/lists for everything else
- Write instructions as imperatives ("Load the reference", "Check the code"), not narratives
- Cover high-frequency parameters in examples — agents read examples first and rarely consult full references unprompted

### CLI Wrapper Specifics

If wrapping a CLI tool:
- Never bundle binaries. Provide install commands + version check
- Add a Step 0 that runs on every trigger: check installed → check version → auto-upgrade if outdated
- Default to non-interactive mode in examples (agents run in non-TTY environments)
- Show `--env` / `--quiet` / `--non-interactive` flags prominently if they exist

```bash
# Step 0 pattern
if ! command -v your-cli >/dev/null 2>&1; then
  echo "NOT_INSTALLED"
else
  _LOCAL=$(your-cli --version)
  _LATEST=$(npm view your-cli version 2>/dev/null)
  if [ -n "$_LATEST" ] && [ "$_LOCAL" != "$_LATEST" ]; then
    echo "OUTDATED local=$_LOCAL latest=$_LATEST"
  else
    echo "OK $_LOCAL"
  fi
fi
```

## Step 4 — Self-Install Support (Optional)

Add an installation section so users can send one message to install:

> Read https://example.com/SKILL.md and follow the instructions to install

Include: platform detection, install commands, Definition of Done checklist, confirmation message the agent must send after install.

## Step 5 — Quality Check

Rate each dimension. Aim for 4/5+ on all:

| Dimension | What to check |
|-----------|---------------|
| **Completeness** | All required sections present? Top use cases covered? |
| **Reliability** | Failure modes caught? Troubleshooting covers common errors? |
| **Ease of use** | 5 minutes from install to first result? Minimal config? |
| **Safety** | Sensitive operations clearly marked? No silent destructive actions? |
| **Agent-friendliness** | Examples show realistic usage? Output interpretation clear? Gotchas prevent common mistakes? |
| **Token efficiency** | L1 < 100 tokens? L2 < 2000 tokens? Heavy reference in L3? |
| **Discoverability** | Description has "Use when..."? Keywords match search terms? Trigger phrases included? |

## Step 6 — Test with a Real Agent

**A skill that reads well to a human may mislead an agent.** Have an agent actually execute the workflows.

```bash
# Symlink for fast iteration (no push needed)
ln -s /path/to/your-skill ~/.claude/skills/your-skill
```

### What to Test

1. **Trigger words** — does the skill activate on expected phrases?
2. **Step 0** (if CLI wrapper) — correctly detects installed/not-installed/outdated?
3. **Each workflow** — run through the top 3-5 examples end-to-end
4. **Error paths** — trigger each troubleshooting scenario
5. **Agent interpretation** — does the agent correctly interpret command output?
6. **Cross-agent** — test with a different agent to catch assumptions

### Testing Methodology by Skill Type

Different skill types need different test strategies. Read 'references/testing-methodology.md' for the complete methodology including pressure scenario design, RED-GREEN-REFACTOR cycle, rationalization tables, and red flags lists.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Summarizing workflow in description | Description = triggering conditions ONLY |
| Passive L3 reference ("see references/x.md") | Use imperative: "Read 'references/x.md' for ..." |
| Bundling CLI binaries | Provide install command + version check |
| Only happy-path examples | Add troubleshooting table with cause → fix |
| Detailed docs in SKILL.md body | Move to `references/` (L3) |
| Missing high-frequency params in examples | If most users need it, show it in the example |
| Assuming TTY availability | Default to non-interactive mode |
| Narrative storytelling ("We discovered that...") | Write imperatives ("Check X", "Load Y") |
| Multi-language code examples | One excellent example in the most relevant language |
| Skipping agent testing | Agent interpretation differs from human reading — always test |
