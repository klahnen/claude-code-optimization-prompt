# Claude Code optimization prompt

A prompt that makes Claude Code audit your own setup and write a plan to improve
it. It does not change anything on your machine. It explores your skills, agents,
plugins, MCP servers, hooks, settings and recent sessions, asks you a few
questions, and writes a plan you can edit and then run one reviewed step at a time.

## Use

1. Open an empty folder and start `claude` there.
2. Paste the part of [`generic_prompt.md`](generic_prompt.md) below the line.
3. Answer the questions, then read and edit the plan it writes.
4. When you are happy with it, ask Claude to execute Phase 0 of the plan. It shows
   every change and waits for your OK before applying it. Phase 0 installs the
   `config-curator` agent.
5. Restart Claude Code and say "use config-curator to continue the plan". It runs
   the remaining phases one reviewed step at a time.

## config-curator

[`agents/config-curator.md`](agents/config-curator.md) is a Claude Code agent that
keeps your setup in line with your plan. Modes: audit (default, read-only), apply
(only the changes you name) and execute plan (step by step, your OK per step). It
never prints secrets, never sends or merges anything, backs up before editing and
commits every change.

Manual install:

```sh
mkdir -p ~/.claude/agents
cp agents/config-curator.md ~/.claude/agents/
```

Then start a fresh session; new agents do not appear in a running one. Use it later
to audit drift ("use config-curator to audit my setup") or to fix a gap a session
exposed.

## What the plan covers

A safety net (backup and version control for your config), a guardrail hook that
blocks merges, force pushes and messages to people, allow rules for read-only
commands so agents do not stall on prompts, cleanup of unused plugins,
skills and servers, read-only agents and wrapper scripts for sensitive systems,
hygiene for stale settings and worktrees, and a first dry run on a real task.

Nothing in it is specific to one company or setup. Everything in the plan comes from
what Claude finds on your machine and what you tell it.

Tested with Claude Code v2.1.x. Some commands and settings fields may differ in other
versions, and the plan tells Claude to verify them first.

## License

MIT, see [LICENSE](LICENSE).
