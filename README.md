# Claude Agent Skills - Best Practices Guide

> Combined from [Anthropic's official docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) and [community refactoring lessons](https://www.reddit.com/r/ClaudeCode/comments/1opxf9f/i_was_wrong_about_agent_skills_and_how_i_refactor/).

---

## 1. The #1 Rule: Skills Are Context Engineering, Not Documentation

The single most important insight - validated by both official docs and painful community experience - is that **skills are not documentation dumps**. They are active workflow knowledge that must be context-engineered.

A developer learned this the hard way: 36 skills totaling ~15,000 lines caused context explosions, slow activation, and degraded output quality. After refactoring with progressive disclosure, initial load dropped by 85%, activation went from ~500ms to <100ms, and relevant information ratio jumped from ~10% to ~90%.

**The mantra:** Context engineering isn't about loading more information. It's about loading the right information at the right time.

---

## 2. The 200-Line Rule for SKILL.md

Official docs recommend keeping SKILL.md under 500 lines. Community experience sharpens this to **~200 lines** as the practical ceiling for the entry point.

Why 200 lines works:

- It's the sweet spot where Claude can efficiently scan and decide what to load next
- It forces you to write a table of contents, not a textbook
- Total loaded context stays at 400-700 lines of highly relevant content instead of 1,000+ lines of mixed relevance

If you can't fit the core instructions in 200 lines, you're putting too much in the entry point. Move the rest to reference files.

---

## 3. Progressive Disclosure Architecture

This is non-optional. Structure every skill in three tiers:

### Tier 1: Metadata (Always Loaded)

YAML frontmatter only (~100 words). Just enough for Claude to decide if the skill is relevant.

```yaml
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

### Tier 2: SKILL.md Entry Point (Loaded on Activation)

~200 lines max. Contains overview, quick start, navigation map. Points to references but doesn't include their content.

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features
**Form filling**: See [FORMS.md](FORMS.md)
**API reference**: See [REFERENCE.md](REFERENCE.md)
**Examples**: See [EXAMPLES.md](EXAMPLES.md)
```

### Tier 3: Reference Files (Loaded On-Demand)

200-300 lines each. Detailed, modular, focused on single topics. Claude reads only when needed.

### Directory Structure

    skill-name/
    ├── SKILL.md                # Entry point (≤200 lines)
    ├── references/
    │   ├── detailed-guide.md   # Loaded only when needed
    │   ├── api-reference.md
    │   └── examples.md
    └── scripts/
        ├── validate.py         # Executed, not loaded into context
        └── helper.sh

---

## 4. Organize by Workflow, Not by Tool

A critical reframing: skills should map to **what you actually do during development**, not to individual tools.

| Bad (tool-centric)                              | Good (workflow-centric)                            |
| ------------------------------------------------ | -------------------------------------------------- |
| `cloudflare/` (1,131 lines)                      | `devops/` (Cloudflare + Docker + GCloud + Vercel)  |
| `tailwind/` + `shadcn/` separate                 | `ui-styling/` (shadcn + Tailwind + design tokens)  |
| `nextjs/` + `turborepo/` + `remix/` separate     | `web-frameworks/` (Next.js + Turborepo + Remix)    |

Each skill teaches Claude **how to perform a specific development task**, not what a tool does. That's the difference between an encyclopedia and a capability.

---

## 5. Writing Effective Frontmatter

### Name Field

- Max 64 characters, lowercase letters/numbers/hyphens only
- Use gerund form for clarity: `processing-pdfs`, `testing-code`, `managing-databases`
- Avoid vague names: `helper`, `utils`, `tools`
- No reserved words: `anthropic`, `claude`

### Description Field

- Max 1024 characters, non-empty
- **Always write in third person** (it's injected into the system prompt)
- Include both what the skill does AND when to use it
- Include specific trigger terms

```yaml
# Bad
description: "Helps with documents"

# Good
description: "Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction."
```

---

## 6. Conciseness: Only Add What Claude Doesn't Know

Claude is already very smart. Challenge every piece of information:

- "Does Claude really need this explanation?"
- "Can I assume Claude knows this?"
- "Does this paragraph justify its token cost?"

```markdown
# Good (~50 tokens)
## Extract PDF text
Use pdfplumber for text extraction:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

# Bad (~150 tokens)
## Extract PDF text
PDF (Portable Document Format) files are a common file format...
To extract text from a PDF, you'll need a library...
There are many libraries available...
```

---

## 7. Set Appropriate Degrees of Freedom

Match specificity to the task's fragility:

| Freedom Level | When to Use                        | Example                                    |
| ------------- | ---------------------------------- | ------------------------------------------ |
| **High**      | Multiple approaches valid          | Code review guidelines, refactoring advice |
| **Medium**    | Preferred pattern exists           | Report templates with customizable sections|
| **Low**       | Fragile, consistency critical      | Database migrations, exact deploy commands |

Think of it as a path: narrow bridge with cliffs = exact instructions; open field = general direction.

---

## 8. Keep References One Level Deep

Claude may partially read files when they're referenced from other referenced files. Avoid nesting.

```markdown
# Bad: nested references
SKILL.md → advanced.md → details.md → actual info

# Good: flat references
SKILL.md → advanced.md (has the info)
SKILL.md → reference.md (has the info)
SKILL.md → examples.md (has the info)
```

For reference files over 100 lines, include a table of contents at the top so Claude can see the full scope even when previewing.

---

## 9. Workflows and Feedback Loops

### Use Checklists for Complex Tasks

Provide a checklist Claude can copy and track progress:

```markdown
## Deployment workflow

Copy this checklist:
- [ ] Step 1: Run tests
- [ ] Step 2: Build artifacts
- [ ] Step 3: Validate config
- [ ] Step 4: Deploy to staging
- [ ] Step 5: Verify staging
- [ ] Step 6: Deploy to production
```

### Implement Validation Loops

The pattern: **run validator → fix errors → repeat**

```markdown
1. Make edits
2. Validate immediately: `python scripts/validate.py`
3. If validation fails → fix → validate again
4. Only proceed when validation passes
```

---

## 10. Common Patterns

### Template Pattern

Provide output format templates. Use strict templates for API responses, flexible templates for reports.

### Examples Pattern

Provide input/output pairs (like few-shot prompting):

```markdown
**Input**: Added user authentication with JWT tokens
**Output**:
feat(auth): implement JWT-based authentication
Add login endpoint and token validation middleware
```

### Conditional Workflow Pattern

Guide Claude through decision points:

```markdown
**Creating new content?** → Follow "Creation workflow"
**Editing existing content?** → Follow "Editing workflow"
```

---

## 11. Scripts: Solve, Don't Punt

When writing utility scripts for skills:

- Handle errors explicitly with helpful messages, don't just let them crash
- Document all magic constants (no "voodoo constants")
- List all required package dependencies explicitly
- Make clear whether Claude should **execute** the script or **read** it as reference
- Use `pip install --break-system-packages` for Python packages

---

## 12. Iterative Development with Claude

The most effective development loop:

1. **Complete a task without a skill** - notice what context you repeatedly provide
2. **Ask Claude A to create a skill** from those patterns
3. **Review for conciseness** - cut anything Claude already knows
4. **Test with Claude B** (fresh instance) on real tasks
5. **Observe behavior** - where does it drift or miss?
6. **Refine with Claude A** based on observations
7. **Repeat**

### What to Watch For

- Does Claude read files in an order you didn't expect? → Restructure
- Does Claude miss references to important files? → Make links more prominent
- Does Claude over-read the same file? → Move that content to SKILL.md
- Does Claude never access a bundled file? → Remove it

---

## 13. Testing

- **Test the cold start**: Clear context, activate skill, measure. If >500 lines load on first activation, refactor.
- **Test with all target models**: What works for Opus may need more detail for Haiku.
- **Build evaluations before writing docs**: Identify gaps first, then write just enough to fix them.
- **Create at least 3 evaluation scenarios** per skill.

---

## 14. Anti-Patterns to Avoid

| Anti-Pattern                     | Why It's Bad                                         |
| -------------------------------- | ---------------------------------------------------- |
| Giant monolithic SKILL.md        | Context explosion, slow activation, irrelevant noise |
| Tool-centric organization       | Creates duplicates, doesn't match workflows          |
| Windows-style paths (`\`)       | Breaks on Unix systems                               |
| Multiple tool options            | Confuses Claude; pick a default, add an escape hatch |
| Time-sensitive info              | Becomes wrong; use "old patterns" sections instead   |
| Inconsistent terminology         | Confuses Claude; pick one term and stick with it     |
| Deeply nested references         | Claude partially reads nested files                  |
| Assuming tools are pre-installed | Always include install commands                      |

---

## 15. Quick Reference Checklist

### Before Sharing a Skill

- [ ] Description is specific, third-person, includes trigger terms
- [ ] SKILL.md body is under 200 lines (hard target) / 500 lines (absolute max)
- [ ] Heavy content is in separate reference files
- [ ] References are one level deep from SKILL.md
- [ ] No time-sensitive information
- [ ] Consistent terminology throughout
- [ ] Concrete examples, not abstract explanations
- [ ] Workflows have clear sequential steps
- [ ] Feedback loops for quality-critical tasks
- [ ] Tested with cold start activation
- [ ] Tested on real tasks, not just test scenarios

### For Skills with Code

- [ ] Scripts handle errors explicitly
- [ ] No magic constants (all values documented)
- [ ] Required packages listed with install commands
- [ ] Validation/verification steps for critical operations
- [ ] Clear distinction: execute vs. read-as-reference
- [ ] Forward slashes only in all file paths

---

## Key Metrics to Track

| Metric                     | Bad          | Good          |
| -------------------------- | ------------ | ------------- |
| SKILL.md lines             | 800+         | ≤200          |
| First activation load      | 1,000+ lines | <500 lines    |
| Relevant info ratio        | ~10%         | ~90%          |
| Token efficiency           | 1x (baseline)| 4-5x improved |
| Activation time            | ~500ms       | <100ms        |

---

*Sources: [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) · [Community Refactoring Post](https://faafospecialist.substack.com/p/i-was-wrong-about-agent-skills-and)*
