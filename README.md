# Backlog Workflow Skill

A role-neutral workflow for using Backlog.md as the source of truth for task intake,
Spec/Plan, deliverable-based subtasks, agent progress, parent-task milestones, and
closeout.

## Scope

The skill uses **主 agent** as the generic coordinating role. It does not depend on
Router, Secretary, a particular repository, or a particular worker implementation.
Project-specific technical/TDD rules should remain in the consuming repository.

## Install for local agents

Install the skill directory at the user-level Agent Skills path:

```bash
mkdir -p ~/.agents/skills
codely skills install https://github.com/vinsli/backlog-workflow-skill.git --scope user
```

For manual installation or agents that do not provide an installer:

```bash
mkdir -p ~/.agents/skills/backlog-workflow
git clone https://github.com/vinsli/backlog-workflow-skill.git /tmp/backlog-workflow-skill
cp /tmp/backlog-workflow-skill/SKILL.md ~/.agents/skills/backlog-workflow/SKILL.md
```

Restart agents after installation so they rediscover the skill.

## Repository policy overrides

The skill is role-neutral, but each repository may add local policy overrides. For example,
a repository may require a specific Backlog CLI fork/version predicate. That local policy
must be checked before any Backlog read or write; never silently fall back to another CLI.

## Files

- `SKILL.md` — the installable skill
- `LICENSE` — MIT license

## Repository-local variants

- `variants/hpcsharp-compiler/SKILL.md` — the last in-repo state of this skill in the
  `hpcsharp-compiler` repository, where the version gate is pinned to that project's
  Backlog fork (`backlog --version` must contain `-thu.`) and examples use `THU-` task ids.
  See `variants/hpcsharp-compiler/NOTES.md` for provenance and the exact diff.
