# Superpowers (Vidably Fork)

> **This is [Vidably's](https://vidably.com) fork of [obra/superpowers](https://github.com/obra/superpowers).** We add custom skills on top of the excellent upstream workflow. Everything in the upstream 14-skill library works as-is — our additions are prefixed with `vidably-*` to avoid conflicts.

## What Vidably Adds

We build AI video evidence for e-commerce. Our engineering philosophy: **exhaustive research before decisions, multi-agent review for every plan and PR, and metrics to prove the process works.** These skills encode that philosophy.

### Custom Skills

| Skill                             | Status  | What It Does                                                                                                                                                                                                                                                                                                  |
| --------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vidably-exhaustive-research`     | Planned | Before any consequential decision (architecture, vendor choice, schema, services), search official docs and trusted sources, generate 4+ options with tradeoffs, rate on maintainability/complexity/extensibility/UX, present for approval. Type 1/Type 2 triage prevents over-triggering on trivial choices. |
| `vidably-multi-agent-plan-review` | Planned | After writing a plan, dispatch it to multiple AI models (Claude, Codex, Gemini) in parallel. Collect findings, apply consensus scoring (unanimous = fix, majority = likely fix, split = judgment call). Prevents architectural gaps before any code is written.                                               |
| `vidably-multi-agent-code-review` | Planned | Before opening a PR, dispatch the diff to multiple models. Same consensus scoring. Catches bugs across model blind spots (research shows 53% → 80% bug detection with multi-model debate).                                                                                                                    |

### Supporting Infrastructure

| Component            | Location   | What It Does                                                                                                                                                                         |
| -------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Agent definitions    | `agents/`  | Shared prompt templates for plan review, code review, and consensus scoring. Reusable across local skills and CI.                                                                    |
| Multi-model dispatch | `scripts/` | CLI runner that probes Codex/Gemini availability, handles input transport, timeouts, and output normalization. Gracefully degrades to Claude-only if external CLIs aren't available. |

### How We Use This

The fork is added to our app repo as a **git submodule** at `tools/superpowers/`. Custom skills are symlinked into `.claude/skills/` so Claude Code auto-discovers them with zero plugin installation. Edits to skills flow directly to this repo via git push.

```
our-app/
  tools/superpowers/                    ← this repo (submodule)
  .claude/skills/
    vidably-exhaustive-research/        ← symlink → tools/superpowers/skills/vidably-exhaustive-research/
    vidably-multi-agent-plan-review/    ← symlink → tools/superpowers/skills/vidably-multi-agent-plan-review/
    vidably-multi-agent-code-review/    ← symlink → tools/superpowers/skills/vidably-multi-agent-code-review/
```

### Upstream Sync

We sync with `obra/superpowers` monthly. Custom skills use the `vidably-*` prefix so merges are conflict-free:

```bash
git fetch upstream && git merge upstream/main
```

---

## Original Superpowers README

> Everything below is from the upstream [obra/superpowers](https://github.com/obra/superpowers) project by Jesse Vincent.

# Superpowers

Superpowers is a complete software development methodology for your coding agents, built on top of a set of composable skills and some initial instructions that make sure your agent uses them.

## How it works

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it _doesn't_ just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do.

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest.

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY.

Next up, once you say "go", it launches a _subagent-driven-development_ process, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward. It's not uncommon for Claude to be able to work autonomously for a couple hours at a time without deviating from the plan you put together.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers.

## Sponsorship

If Superpowers has helped you do stuff that makes money and you are so inclined, I'd greatly appreciate it if you'd consider [sponsoring my opensource work](https://github.com/sponsors/obra).

Thanks!

- Jesse

## Installation

**Note:** Installation differs by platform. 

### Claude Code Official Marketplace

Superpowers is available via the [official Claude plugin marketplace](https://claude.com/plugins/superpowers)

Install the plugin from Anthropic's official marketplace:

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code (Superpowers Marketplace)

The Superpowers marketplace provides Superpowers and some other related plugins for Claude Code.

In Claude Code, register the marketplace first:

```bash
/plugin marketplace add obra/superpowers-marketplace
```

Then install the plugin from this marketplace:

```bash
/plugin install superpowers@superpowers-marketplace
```

### OpenAI Codex CLI

- Open plugin search interface

```bash
/plugins
```

Search for Superpowers

```bash
superpowers
```

Select `Install Plugin`

### OpenAI Codex App

- In the Codex app, click on Plugins in the sidebar.
- You should see `Superpowers` in the Coding section. 
- Click the `+` next to Superpowers and follow the prompts.


### Cursor (via Plugin Marketplace)

In Cursor Agent chat, install from marketplace:

```text
/add-plugin superpowers
```

or search for "superpowers" in the plugin marketplace.

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

To update:

```bash
gemini extensions update superpowers
```

## The Basic Workflow

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.

2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.

4. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Testing**

- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**

- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Collaboration**

- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed implementation plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**

- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

Read [the original release announcement](https://blog.fsck.com/2025/10/09/superpowers/).

## Contributing

The general contribution process for Superpowers is below. Keep in mind that we don't generally accept contributions of new skills and that any updates to skills must work across all of the coding agents we support.

1. Fork the repository
2. Switch to the 'dev' branch
3. Create a branch for your work
4. Follow the `writing-skills` skill for creating and testing new and modified skills
5. Submit a PR, being sure to fill in the pull request template.

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

Superpowers updates are somewhat coding-agent dependent, but are often automatic.

## License

MIT License - see LICENSE file for details

## Community

Superpowers is built by [Jesse Vincent](https://blog.fsck.com) and the rest of the folks at [Prime Radiant](https://primeradiant.com).

- **Discord**: [Join us](https://discord.gg/35wsABTejz) for community support, questions, and sharing what you're building with Superpowers
- **Issues**: https://github.com/obra/superpowers/issues
- **Release announcements**: [Sign up](https://primeradiant.com/superpowers/) to get notified about new versions
