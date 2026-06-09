---
name: let-cursor-cook
description: Orchestrate Cursor CLI agents from Codex, Claude Code, or another external coding harness. Use when managing Cursor/agent as cheap, fast technical engineers for codebase research, implementation, command execution, debugging, and code review while the host harness owns requirements, coordination, user communication, and final verification.
---

# Let Cursor Cook

Use this skill when the current assistant can run shell commands and should manage Cursor CLI agents as delegated technical engineers.

Default command: `agent`.
Expanded fallback: `cursor agent`.

## Role Split

Act as the manager/team lead.

You own:
- User-facing requirements clarification
- Acceptance criteria
- Architecture and scope decisions
- Task decomposition and coordination
- Final integration judgment
- Final user-facing summary
- Final E2E/signoff when possible

Delegate technical work to Cursor agents by default because they are cheap, fast, and provide the desired Cursor/Composer engineering path:
- Codebase research
- Implementation
- Command execution
- Debugging
- Test fixing
- Code review

Use native subagents from the host harness mainly as context-hygiene sidecars, such as E2E verification support, log summarization, broad non-code research, or independent checks. Do not use native subagents as the primary implementation or code-review path when Cursor agents are available and performing.

If Cursor fails twice to meet the manager's explicit requirements or quality bar for the same task, stop looping on Cursor and escalate to a native subagent or local host implementation path. Before that threshold, do not patch code locally just because the fix looks small.

## Workflow

1. Clarify requirements with the user.
2. Write concise acceptance criteria.
3. Delegate codebase research to Cursor if local context is insufficient.
4. Decide the plan as manager.
5. Delegate implementation to one or more Cursor agents.
6. For non-trivial work, delegate code review to a separate Cursor reviewer.
7. Send review findings that require code changes back to a Cursor implementer/fixer.
8. Run host-owned verification/E2E using available tools.
9. Summarize the outcome to the user.

The manager may inspect diffs, judge findings, run verification, update the task ledger, and write user-facing summaries. The manager must not make code edits directly unless the user explicitly asks for host implementation or the escalation threshold is met.

Prefer serial orchestration unless tasks are independent. Parallel Cursor agents are appropriate when write scopes are disjoint, research questions are independent, or competing approaches should be isolated.

## Delegation Briefs

Give every delegated agent a compact written brief. Include:
- Role label, such as `Cursor researcher`, `Cursor implementer`, `Cursor reviewer`, or `Native verifier`
- Objective
- Workspace path
- Relevant files, modules, or areas to inspect
- Constraints and non-goals
- Acceptance criteria
- Expected or allowed commands
- Whether edits are allowed
- Required report format

Do not include organization-specific project-management assumptions or require any external tracker.

Tell Cursor agents they report to the manager, not the end user. Their output should be operational, not polished final prose.

## Command Patterns

Use headless print mode for scriptable orchestration.

For read-only codebase research:

```bash
agent -p --trust --workspace /path/to/repo "Inspect the codebase and report the likely files and implementation plan. Do not edit."
```

Use plan/ask modes for read-only work:

```bash
agent -p --trust --workspace /path/to/repo --mode=plan "Inspect and propose a plan only. Do not edit."
agent -p --trust --workspace /path/to/repo --mode=ask "Review these changes for bugs. Report findings first by severity with file/line references."
```

Use explicit sandboxing for implementation, debugging, test fixing, or any task where Cursor should run local commands:

```bash
agent -p --trust --sandbox enabled --workspace /path/to/repo "Implement the scoped change. Run relevant local tests. Report changed files, commands, results, tests, and risks."
```

If `--sandbox enabled` still blocks necessary local commands in headless mode, escalate to `-f` while keeping the sandbox enabled and the prompt tightly scoped:

```bash
agent -p -f --trust --sandbox enabled --workspace /path/to/repo "Implement the scoped change. Allowed commands: npm test and npm run lint. Do not install dependencies, run migrations, deploy, touch secrets, or edit outside the scoped files without reporting back first."
```

`-f` / `--force` means force-allow commands unless explicitly denied. `--yolo` is an alias for `--force`. Do not use `--sandbox disabled` by default.

Use an isolated Cursor worktree for risky, parallel, or competing implementation:

```bash
agent -p --trust --sandbox enabled --workspace /path/to/repo -w feature-spike "Implement the scoped change in this isolated worktree. Run relevant local tests. Report changed files, commands, tests, and risks."
```

Use the expanded form if the shortcut is unavailable:

```bash
cursor agent -p --trust --sandbox enabled --workspace /path/to/repo "Implement the scoped change..."
```

## Sessions

Prefer explicit chat IDs for orchestrated work.

For non-trivial tasks, create or capture a chat ID when available:

```bash
agent create-chat
```

Then resume that exact engineer thread:

```bash
agent -p --trust --sandbox enabled --workspace /path/to/repo --resume <chatId> "Continue with the next requested fix. Report using the required format."
```

`agent -p` normal text output may not print a chat ID. Treat one-shot `agent -p` invocations as disposable unless you separately capture the session ID.

`agent --continue` resumes the latest Cursor agent session. Use it only for quick local continuation where ambiguity is acceptable. Prefer `--resume <chatId>` for serious orchestration.

Track chat IDs in the task ledger whenever available.

## Worktrees

Use `agent -w [name]` for:
- Parallel implementation
- Risky changes
- Competing approaches
- High-blast-radius tasks

For tiny single-agent fixes, the current workspace is acceptable.

When multiple agents are active, give each one a disjoint write scope. Tell agents they are not alone in the codebase and must not revert unrelated edits.

## Task Ledger

Maintain a lightweight ledger for multi-step work. Track:
- Requirement or acceptance criterion
- Assigned role
- Cursor chat ID, if available
- Workspace or worktree
- Current status
- Verification status
- Open risks or questions

The ledger can live in the conversation or a scratch file. Do not require a specific external tool.

## Required Cursor Report Format

Ask implementers and researchers to report:

```markdown
Summary:
Files changed:
Commands run:
Results:
Tests passed:
Tests failed:
Risks / unresolved questions:
Next recommended step:
```

Ask reviewers to report findings first:

```markdown
Findings:
- [Severity] file:line - issue and impact

Residual risk:
Recommended next step:
```

The manager decides which findings to accept.

## Guardrails

Use bounded autonomy. Cursor agents may run expected local commands such as tests, builds, linters, formatters, and safe repo inspection. Prefer `--sandbox enabled` for autonomous local command execution. Use `-f` only when required for headless execution, and keep the prompt explicit about allowed commands and write scope.

Require Cursor agents to pause or report before:
- Destructive actions
- Production deploys
- Secret handling
- Database migrations
- Network-heavy dependency changes
- Irreversible file operations
- Broad rewrites outside scope

Do not delegate final acceptance, final E2E signoff, secret handling, production deploys, or irreversible destructive actions.

When agents disagree, reduce the conflict to evidence: inspect the code, run targeted tests, ask a focused follow-up if useful, then make the call as manager. Do not average opinions or defer to confidence.

## Escalation

Cursor is the default technical delegate, but do not loop forever.

Count a failed Cursor attempt when the agent:
- Ignores explicit acceptance criteria
- Edits outside the assigned scope
- Cannot run or explain required verification
- Produces a change that fails the manager's review or tests
- Repeats the same misunderstanding after correction

Before two failed attempts, keep implementation and review-fix edits with Cursor. When a Cursor reviewer finds valid issues, send those findings to a Cursor implementer/fixer with a narrow brief. Do not patch the issues locally merely because they are small.

After two failed attempts on the same task, stop sending the same work back to Cursor. Choose the smallest effective escalation:
- Spawn a native worker/subagent from the host harness to implement the task
- Implement the fix locally as the manager only if the user explicitly allows host implementation or no suitable native worker is available
- Split the task into smaller Cursor-owned slices only if the prior failures were caused by task size or ambiguity

Before escalating, preserve the useful context: requirements, acceptance criteria, Cursor summaries, changed files, failed commands, review findings, and manager observations. The native worker or local implementation path should start from that evidence, not from another vague prompt.

Still run final manager-owned verification after escalation.

If the manager makes code edits before the threshold without explicit user permission, treat that as a process failure and disclose it.

## Verification

Cursor implementers can run tests, but the manager owns final confidence.

Use the tools available in the host harness:
- Unit/integration test commands
- Browser automation
- Playwright or app-specific E2E commands
- API smoke tests
- Screenshots
- Manual inspection through available UI tools

If verification would clutter the manager context, delegate support work to a native verifier subagent, but keep final signoff with the manager.
