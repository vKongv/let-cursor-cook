# Let Cursor Cook

A portable AI skill for Codex, Claude Code, and other external coding harnesses that can shell out to Cursor's `agent` CLI.

Use it when you want the host agent to act like a manager/team lead while Cursor agents handle the technical cooking: codebase research, implementation, command execution, debugging, and code review.

## Install

Install with [skills.sh](https://www.skills.sh/):

```bash
npx skills add vkongv/let-cursor-cook
```

Install for specific agents:

```bash
npx skills add vkongv/let-cursor-cook -a codex -a cursor -a claude-code
```

## What It Teaches

- Keep the host harness responsible for requirements, planning, user communication, and final verification.
- Keep the host harness in a manager/orchestrator role, not an implementer role.
- Delegate technical work to Cursor CLI agents by default.
- Use explicit Cursor chat IDs for serious orchestration.
- Use isolated Cursor worktrees for risky or parallel implementation.
- Use separate Cursor reviewer agents for non-trivial code changes.
- Send valid review-fix code changes back to Cursor instead of patching them locally.
- Run closed-loop Cursor passes with explicit goals, checks, exit conditions, and iteration caps.
- Keep native subagents available for context hygiene and verification sidecars.
- Escalate to native subagents or local implementation if Cursor fails twice on the same task.

## Requirements

- Cursor Agent CLI available as `agent` or `cursor agent`
- A host AI harness that can run shell commands

Check Cursor Agent:

```bash
agent --help
```

Recommended implementation command:

```bash
agent -p --trust --sandbox enabled --workspace /path/to/repo "Implement the scoped change and run relevant local tests..."
```

Use `-f` only when headless Cursor still blocks necessary local commands:

```bash
agent -p -f --trust --sandbox enabled --workspace /path/to/repo "Implement the scoped change. Allowed commands: npm test and npm run lint..."
```

## Skill

The skill lives in [SKILL.md](./SKILL.md).
