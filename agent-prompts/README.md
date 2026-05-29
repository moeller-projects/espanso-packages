# Agent Prompts

18 pipeline-driven prompts for AI coding agents (Claude Code, Codex, Cursor,
etc.) that wire together a custom skill set for Azure DevOps workflows:
implementing work items, reviewing PRs, debugging from stack traces,
refactoring, and producing decision artifacts.

## Triggers

|Trigger           |What it does                                       |
|------------------|---------------------------------------------------|
|`:agent-triage`   |Paste anything — routes to the right prompt below  |
|`:agent-implement`|Full work-item pipeline: spec → tasks → code       |
|`:agent-plan-spec`|Author a spec file interactively                   |
|`:agent-breakdown`|Existing spec → task graph → implementation        |
|`:agent-pr-desc`  |Draft a PR description from your branch + work item|
|`:agent-review-pr`|Review a PR; draft and post comments               |
|`:agent-respond`  |Triage and reply to feedback on your PR            |
|`:agent-refactor` |Review and refactor a class, method, or workflow   |
|`:agent-test`     |Design strategy and add coverage for existing code |
|`:agent-debug`    |Root-cause hunt from an error or stack trace       |
|`:agent-threat-model`|Read-only security exposure analysis            |
|`:agent-spike`    |Timeboxed exploration of an unknown                |
|`:agent-adr`      |File an Architecture Decision Record               |
|`:agent-docs`     |Write/refresh README, guide, or reference docs     |
|`:agent-migration`|Plan a risky change with per-slice rollback        |
|`:agent-modernize`|Legacy uplift: characterize, slice, refactor safely|
|`:agent-ship`     |Review or plan deployment path with rollback       |
|`:agent-onboard`  |One-time: scan a repo and stage AGENTS.md          |

Every trigger opens a form for its inputs (PR URL, work item ID, target branch,
etc.) before pasting the prompt.

## How the prompts work

Each prompt names the skills it drives, the phases they run in, the handoffs
between them, and the human checkpoints where the agent stops for approval.
The pipeline never silently codes against an unapproved spec or posts comments
you haven’t read.

The orchestration sits on top of the `moeller-projects/agent-skillz` skill set
(ado-gateway, spec-engine, delivery-engine, code-quality-engine, test-engine,
repo-engine, thinking-engine, doc-engine, ops-engine, caveman). Skills are
assumed to be installed and auto-discoverable by your agent — this package
provides only the prompts that drive them.

## Customising

Triggers use the `:agent-*` namespace to avoid collisions. To rename, edit the
`trigger:` field in the relevant `.yml`. To change a prompt’s phases, edit the
`replace: |` block. Espanso picks up changes on save (force with
`espanso restart`).

## See also

- `extras/config/wezterm.yml` — paste-shortcut override for TUI agents (Codex
  CLI, Claude Code) running inside wezterm. Required if you hit
  “Failed to paste image: no image on clipboard” when triggering inside the
  agent’s TUI.
- `extras/scripts/sync-to-claude-commands.py` — generates `.claude/commands/*.md`
  from this package so Claude Code’s native slash commands stay in sync.