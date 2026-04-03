# Vibe Workflow

Stay in the flow. Ship with confidence.

```
/vibe:feature "Add social login"
```

One command to orchestrate your entire dev pipeline — from brainstorming to code review, verification, and deployment.

[한국어](README_KOR.md)

---

## Sound familiar?

> "Claude started coding without asking anything — and went in a completely wrong direction"

> "It said it would write tests later. It never did."

> "I ran /clear and it forgot everything we were working on"

> "It recommended an approach with zero evidence. Turned out it was deprecated."

> "It auto-committed without asking me"

> "There were clearly relevant skills available, but it ignored them all"

Vibe Workflow solves each of these problems structurally.

---

## Key Features

### AI doesn't guess

Every approach suggestion is **backed by real research**. Official docs (context7), real-world cases (WebSearch), and GitHub implementations are searched in parallel before any recommendation.

```
Ingredients: verified sources only.
Cooking (combining and judging): go wild.
```

### You make every decision

Claude analyzes and proposes. You choose. Design approval, plan confirmation, commits, merges — every critical decision has a user gate. And you control how often you're asked.

```yaml
engagement_level: balanced    # high(~10x) / balanced(~4x) / low(~2x)
```

### Skills are actively used

At every Phase entry, available skills are checked. If there's even a 1% chance a skill is relevant, it gets invoked. "When in doubt, use it" is the principle.

### /clear without losing context

Keep sessions short without losing progress. Workflow state is tracked in YAML, history in a log file. New sessions auto-detect previous work.

```
"You have active workflows:
  (1) feat-payment — Phase 3 TDD, Task 5/8
  Which one to continue?"
```

### TDD is physically enforced

"I'll write tests later" doesn't fly. Code written without tests gets deleted (Iron Law). 12 rationalization patterns are preemptively blocked, and tdd-guard hooks act as a second safety net.

```
RED → GREEN → REFACTOR → COVERAGE(80%)
```

### Code review that actually works

Not a rubber stamp. Three internal agents review in parallel (quality / security / tests), then Codex provides an independent external review.

```
Internal: 3 Agent Teams in parallel (quality / security / tests)
External: /codex:review or /codex:adversarial-review
```

### Errors are solved together

Claude doesn't silently fail 3 times before telling you. Root cause analysis runs automatically, but **the fix direction is decided together**. Report timing adapts to severity.

```
Trivial:  Instant fix suggestion
Standard: Investigate → Report → Decide together
Complex:  Immediate report → Full collaboration
```

### Gets better the more you use it

After every workflow, a retrospective runs. Feedback on what was too much or too little gets recorded and reflected in config. Not a rigid template — an evolving system.

---

## 3 Modes

| Mode | Command | Path | Use case |
|------|---------|------|----------|
| **Feature** | `/vibe:feature` | Phase 1→2→3→(4)→5→6→7 | New feature |
| **Hotfix** | `/vibe:hotfix` | Phase 4→3→6→7 | Urgent bug fix |
| **Refactor** | `/vibe:refactor` | Phase 1→3→5→6→7 | Code restructuring |

### 7-Phase Pipeline

```
Phase 1  Brainstorming    Idea → Validated design (evidence-based search)
Phase 2  Planning         Design → Executable tasks (dependency graph)
Phase 3  TDD              RED → GREEN → REFACTOR → COVERAGE
Phase 4  Debugging        Evidence-based root cause analysis (severity triage)
Phase 5  Code Review      Internal parallel 3-team + Codex external review
Phase 6  Verification     Build/test/lint with actual execution proof
Phase 7  Wrap-up          Commit → PR/merge → Retrospective
```

### Scale-Adaptive

Process auto-adjusts to project size.

```
Small    One config change          →  35min  (abbreviated)
Standard Add social login           →  90min  (full pipeline)
Large    Build payment system       →  phased execution
```

Small skips Phase 2, uses single reviewer, QUICK verification.

---

## Install

```bash
# Add marketplace
/plugin marketplace add 0in11/vibe_workflow

# Install plugin
/plugin install vibe@0in11-vibe-workflow
```

### Optional: Codex External Review

```bash
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

---

## Quick Start

```bash
# New feature
/vibe:feature "Add user authentication API"

# Emergency fix
/vibe:hotfix "Login returns 500 error"

# Refactoring
/vibe:refactor "Split auth module responsibilities"
```

---

## Configuration

Create `.claude/workflow-config.yaml` in your project root for per-project settings.

```yaml
# User engagement level
engagement_level: balanced     # high / balanced / low

# Artifact paths
paths:
  specs: docs/specs
  plans: docs/plans
  workflows: .claude/workflows
  archive: .claude/workflow-archive

# Defaults
defaults:
  scale: auto                  # auto / small / standard / large
  checkpoint_interval: phase   # task / phase / end
  coverage_threshold: 80       # Coverage gate (%)
  review_mode: parallel        # parallel / single
  codex_review: true           # Codex external review

# TDD
tdd:
  framework: auto              # auto / jest / vitest / pytest / cargo / go
  guard_hook: true             # tdd-guard hook

# Git
git:
  base_branches: [main, master, develop]
  message_format: conventional
```

Works with defaults when no config file exists.

---

## Architecture

```
vibe_workflow/
  .claude-plugin/
    plugin.json             # Plugin manifest
  skills/
    feature/SKILL.md        # /vibe:feature
    hotfix/SKILL.md         # /vibe:hotfix
    refactor/SKILL.md       # /vibe:refactor
  workflow-config.yaml      # Default config template
```

---

## Benchmarks

Built on top of proven tools, improved where they fell short.

| Base | Adopted | Improved |
|------|---------|----------|
| superpowers:brainstorming | Hard Gate, question patterns | Scale branching, evidence-based search, MVP scoping |
| superpowers:writing-plans | No-placeholder policy, task decomposition | User confirmation gate, dependency graph, test specs |
| superpowers:tdd | RED-GREEN-REFACTOR | Coverage gate, Spike path, auto framework detection |
| superpowers:systematic-debugging | Evidence-based debugging | Severity triage, earlier user reporting |
| tdd-guard | Hook-based TDD enforcement | Skill + hook dual safety net |
| codex-plugin-cc | Codex review | Internal parallel + Codex 2-stage review |

---

## License

MIT
