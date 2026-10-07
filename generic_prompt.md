# Prompt: audit my Claude Code setup and write an optimization plan

Paste everything below the line into Claude Code, started in an empty folder you
are happy to keep notes in. It changes nothing on your machine. It explores your
setup, asks you a few questions, and writes a plan you can edit and then execute
yourself, one reviewed step at a time.

Tested with Claude Code v2.1.x. Some commands and settings fields may differ in other
versions, so the plan tells Claude to verify them first.

---

You are helping me optimize my Claude Code environment: faster and longer
autonomous loops, fewer wrong tool choices, lower always-loaded context, and
guardrails that keep Claude from doing things I never want it to do. Your job in
this session is to produce a PLAN, not to execute it.

## Rules for this session

1. Read-only. The only files you may create are `claude-env-audit.md` and
   `claude-optimization-plan.md` in the current folder. Do not modify, delete,
   install, uninstall, enable, disable or reconfigure anything else, and do not
   run anything that sends a message, comments, merges or pushes.
2. Redact secrets everywhere: API keys, tokens, passwords, env var values, auth
   headers, internal hostnames and account IDs. Write `<REDACTED>`. Be careful
   with commands that print configuration: for example `claude mcp get <name>`
   prints environment values. Prefer reading structure (key names, counts) over
   values, and filter output before it reaches the screen.
3. Locate files instead of assuming paths. If something is not where you expect,
   search for it and note where you found it.
4. Use subagents in parallel only where it keeps your context clean (for example
   one for transcript mining). Otherwise work inline.
5. If a step fails or a tool is missing, record that and continue.
6. Ask me one question at a time when you need a decision. Prefer a recommended
   default and a short reason.
7. Report facts faithfully. If you estimated something, say it is an estimate. If
   you could not check something, say so.
8. Do not copy anyone else's setup. Everything in the plan must come from what you
   find on this machine and what I tell you.

## Step A: Discover (read-only)

Build an inventory. Record path, source (user, project, plugin, built-in, synced),
approximate size, and a one-line purpose for each item.

1. Basics: `claude --version`, OS, shell, the default model, and whether I usually
   run in a permission mode that skips prompts (settings, or ask me). This matters
   because allow rules do nothing in such modes while hooks and deny rules still do.
2. Settings: user-level `settings.json` and any local, project and managed
   settings you can find. Summarize permission allow, deny and ask rules, env var
   names only, hooks, enabled plugins and the status line. Flag malformed rules
   (shell fragments saved as permissions), broad file-read grants (a whole home
   folder or a whole code folder), and stale per-project settings files.
3. Instruction files: `~/.claude/CLAUDE.md` and every `CLAUDE.md`, `AGENTS.md` and
   `CLAUDE.local.md` in the folders I tell you to include (ask me which top-level
   folders hold my projects). For each: path, token estimate, headings, one-line
   summary. Flag duplication and contradictions.
4. Agents: user, project and plugin agents with name, description, tools and model.
   Flag files with missing or malformed frontmatter.
5. Skills and commands: every user, project, plugin and synced skill and every
   custom command, with its description, size, and whether it bundles scripts or
   reference files. Note skills that are not real skills (wrong file name case, no
   frontmatter). Find loose scripts sitting next to skills.
6. Plugins and marketplaces: installed, enabled, version, source, and what each one
   contributes (skills, agents, commands, hooks, MCP servers). Use the plugin CLI
   (`claude plugin --help`) to see component inventories and token cost.
7. MCP servers: `claude mcp list`, project `.mcp.json` files, and connectors that
   come from my account. For each: name, scope, transport, purpose, health. Flag
   servers that fail to connect, duplicates (a standalone server and an account
   connector for the same product), and servers with credentials in their config.
8. Hooks: every hook from settings and plugins with event, matcher, command
   (redacted) and what it appears to do.
9. Measured context cost. Do not estimate when you can measure. `/context all`
   works headlessly: run `claude -p "/context all"` from a neutral folder and read
   the tables for skills, agents, MCP tools, memory files and system tools. Record
   per-item tokens. Remember that built-in skills and skills synced from my account
   count too, even though no local file controls them.
10. Transcripts: find my session transcripts (usually under `~/.claude/projects/`)
    and analyze roughly the last 60 days. Report counts and patterns only, never
    contents, code or client data. Validate your counting method first on an item
    you know I use, so a bug does not make every skill look unused. Identify:
    - which skills, commands, agents and plugins were actually used, and which never;
    - requests where I asked for subagents or parallel work by hand;
    - instructions I repeat across sessions (candidates for CLAUDE.md or a skill);
    - things I keep reminding Claude to do or not do (candidates for hooks);
    - common task categories and rough frequency;
    - tools that error or get denied often.
11. External tools I might use. Check which of these are installed and how they are
    authenticated, without printing credentials: the GitHub CLI, a cloud CLI (for
    example AWS), a database client, a CI client, a docs or notes connector (for
    example Notion), a chat connector, a browser connector, and a log or
    observability tool. For cloud CLIs, note whether sessions are interactive logins
    that expire, and check which role a profile resolves to (an admin role means
    "read-only" can only be enforced by tooling, not by permissions). Never write
    profile names or account IDs into your outputs.
12. Per-project summary for each project folder I named: stack, test and lint
    commands, CI system, and which Claude config it has or lacks.

Write the findings to `claude-env-audit.md` with these sections: executive summary
(ten bullets at most), inventory tables, overlap and context-cost analysis (a table
of items with overlapping purpose, vague or colliding descriptions, and measured
always-loaded tokens by source), usage patterns, per-project summary, and a final
list of everything that failed or could not be checked. Do not propose fixes in the
audit.

## Step B: Interview me

Using what you found, ask me short questions, one at a time, to personalize the
plan. Cover at least:

- What kinds of work do I do most (from the transcripts, confirm rather than ask)?
- What must Claude never do on its own? Typical answers: merge, force push, push to
  protected branches, message or email people, comment on tickets or PRs, request
  reviewers, change production infrastructure, run queries against production
  data. Which of these may be allowed in a narrow form (for example create or edit
  an issue, open a draft PR assigned to me, save a chat draft but not send it)?
- Which systems are sensitive (production cloud accounts, databases, customer data,
  logs) and how do I reach them today (tunnels, VPNs, SSO, bastion hosts)?
- Which repetitive flows would I like to run end to end (for example: read an issue,
  investigate, fix with tests, review, open a draft PR)?
- Where do I want to review every change, and where am I fine with Claude running
  alone for a while?
- Which skills, plugins or connectors do I use, never use, or want to keep out of
  every session to save context?
- Is anything in a project folder documentation I want to keep even if it looks
  unused?

## Step C: Design the plan

Write `claude-optimization-plan.md`. It must be something I can read, edit and run
step by step, so it has to be concrete: exact paths, exact commands, full file
contents for anything new, and a test or check for every step. Start the file with
the title, a one-line goal, a pointer to the audit, and the "How to execute this
plan" block below, copied verbatim.

### How to execute this plan (copy into the plan)

1. One step at a time, in order. Do not batch steps or skip ahead.
2. Show every change before applying it. New file: show the full content. Edited
   file: show a unified diff. Deletion, move, plugin or MCP operation: show the
   exact commands and the list of affected paths. Then wait for my explicit OK. An
   OK for one step does not approve the next.
3. Ask when anything is unclear, ambiguous or conflicts with what you find on disk.
   Do not guess. Each step lists its decision points; ask them even if they seem
   obvious. Ask one question at a time.
4. Verify before acting. The plan was written earlier. Re-check paths, names and
   current contents. For Claude Code features (plugin CLI syntax, hook output
   schema, agent frontmatter) check `claude --help`, `claude plugin --help` or the
   docs instead of assuming.
5. After each step, give a short summary of what changed, tick the checkbox, and
   commit if the changed folder is under version control.
6. Anything destructive gets an exact list and a backup first, and I confirm the
   list. Do not run broad cleanups (cache prunes, recursive deletes) that I did not
   ask for.
7. Never run anything that sends a message, comments or merges, even as a test.
   Hook tests use piped JSON only.
8. Interactive logins (SSO, OAuth) are run by me, not by you. Tell me the command.
9. Never print secrets. If a command might print one, filter it first.
10. Out of scope unless I say otherwise: per-project instruction files I marked as
    intentional, legacy projects, and my session transcripts.

### Phases to include (adapt each one to the audit and interview)

Include only the steps the findings justify, and say why each one is there. For each
step write: the problem it solves, the exact change, how to verify it, how to undo
it, and its decision points.

**Phase 0: Safety net**
- Back up the Claude config folder and the user-level JSON config (exclude
  transcripts, caches and history) into a dated archive; show size and file count.
- Put the config folder under git with a `.gitignore` for transcripts, caches,
  runtime state, plugin caches and downloaded marketplaces; scan the staged list
  for secrets before the first commit; commit after every later step.

**Phase 1: Guardrail hook**
- A `PreToolUse` hook script with tiers: always deny (merge, force push, push to
  protected branches), contact (anything that reaches a human: ticket or PR
  comments, reviewer requests, chat or email sends, scheduled messages), and any
  narrow allowances I chose in the interview (draft PRs, self-assignment only,
  chat drafts, issue creation).
- It must match only text that can execute. Ignore heredoc bodies, quoted strings
  and comments, but still catch `bash -c`, `eval`, `ssh`, `xargs`, command
  substitution, and a heredoc fed to a shell. Plain-text matching blocks harmless
  commands that merely mention a blocked phrase (for example a commit message), and
  that destroys the speed the plan is meant to buy.
- Match connector tool names too, and check them against the tools actually present
  in this session: block writes and sends, allow reads and drafts, and show me the
  matched and unmatched lists.
- Register it in settings with a deny-rule safety net for the worst commands.
- Tests: a script that feeds piped JSON cases and prints a results table. Include
  hard-deny, contact, allow, false-positive regressions (blocked phrases inside
  heredocs, echo, grep, commit messages, comments) and bypass attempts that must
  still be denied (`bash -c`, `eval`, command substitution, quoted URLs).
- Live check after a restart: attempt one harmless blocked call and show the denial.
- Hooks fire inside subagents too; verify that later in the agents phase.

**Phase 2: Cleanup**
- Plugins: for each unused, broken or duplicate plugin, decide uninstall versus
  disable (uninstall is cleaner when reinstalling is one command). Keep ones I use.
  Use the plugin CLI, and show the resulting settings diff.
- MCP servers: remove failing ones and standalone servers that duplicate an account
  connector (for example a standalone notes or docs server next to the official
  connector). Never print their configuration; if it holds a token, tell me to
  revoke it.
- Skills and commands: delete or archive what I no longer use, one item at a time,
  with its measured token cost. Fix skills that cannot load (file name case, missing
  frontmatter). Give every command a trigger-oriented one-line `description`.
  Decide whether a rarely used flow should be a command rather than a skill, and
  remove references to tools that no longer exist from what remains.
- Instruction files: propose a short set of global rules from the repeated
  instructions found in transcripts (for example: where to put text meant for other
  people, read-only investigations by default, never merge or comment, prefer the
  dedicated file tools over shell equivalents). Keep it short, it loads every
  session. Show a diff.
- Report what account-level settings control (synced or built-in skills) and let me
  decide; you cannot change those from here.

**Phase 3: Agents, tools and workflows**
Choose agents from what I actually do, not from a template. Common, useful shapes:
a read-only investigator per source (logs and errors, cloud accounts, a query
engine), a reviewer of local changes before they are pushed, an infrastructure-code
reviewer, a browser validator for a staging environment, and a CI watcher. For each
agent write the full file and ask me before creating it.

- Agent files: verify the frontmatter fields in the docs first. Known points to
  check: `skills:` is a YAML list and injects the full skill body at startup, so
  preload only what an agent always needs and have it read large skills on demand;
  `tools:` can restrict which subagents an agent may spawn (nesting is supported,
  confirm the depth); `disallowedTools` enforces "do not edit". Give every agent a
  trigger-oriented description so Claude picks it correctly and does not collide
  with another one.
- Merge a skill and its only consumer (an agent) into one file when nothing else
  uses the skill.
- Symlinked agent files are discovered; confirm in a fresh headless session
  (`claude -p`), because the running session's agent list is frozen at startup.
- Sensitive systems get one wrapper script each, and agents may only use the
  wrapper. Design every wrapper the same way:
  - validates the statement (read-only verbs only, one statement, no writes, DDL or
    file output), requires a bounded time window or a LIMIT, and caps rows;
  - runs through the right profile, tunnel or session itself, so no agent handles
    credentials, and refuses to run if the target is not the intended read-only one;
  - limits local concurrency across all terminals (a small slot directory with stale
    entry cleanup), gives a clear "busy" exit code, uses a unique socket and port
    per call, and cancels queries after a time limit;
  - logs each statement with a timestamp and, where available, bytes scanned;
  - never logs in for me: an expired session exits with a message and I run the
    login;
  - ships with an offline test suite that uses fake binaries for the CLI and client,
    covering concurrency, busy, timeout, expired session, failure cleanup, output
    capping and validation; then one small live query that I approve.
  - Plan for bulk exports from the start. A row cap and a short time limit make
    full-population exports (audit evidence, user lists) impossible, and the agent
    then falls back to handing me SQL to run by hand, with no way to verify the
    file. Give database wrappers a second, separate export mode: one SELECT, paged
    by primary key in chunks that each fit the time limit, streamed to a file in the
    output folder, with its own total-row cap, the same read-only checks, the same
    logging, and a printed row count and checksum. Tell the query agent about it and
    allow it with its own rule. Handing me a query is only for systems no agent or
    wrapper can reach.
  - A hook rule then denies direct use of the underlying client or CLI against that
    system, so agents must go through the wrapper. Add allow rules for the wrapper
    in both the `~` and absolute path forms.
- Cloud CLIs: always pass an explicit profile flag and never rely on environment
  variables; keep profile names out of tracked files (read them from a config file at
  run time). Resolve which role a profile uses; if it is an admin role, say that
  "read-only" is enforced by the wrapper and hook, not by permissions, and let me
  decide whether to ask for a read-only role. Any change in a production account
  (a new workgroup, a limit, a log setting) is a decision for me, not a step you
  execute. Do not create it for me.
- Notes and docs connector (for example Notion): keep the official connector, treat
  comment creation as human contact, allow reads, page edits and drafts as I decide
  in the interview, and never keep a duplicate standalone server with its own token.
- End-to-end flow skill, if I want one (for example fix an issue): hidden from the
  model with `disable-model-invocation: true` so it runs only when I invoke it. Its
  steps: understand the issue, investigate with the right agents (one query agent at
  a time), isolate in a worktree and branch from the up-to-date default branch,
  implement with tests, review, validate, commit with my commit convention, push,
  open a DRAFT PR assigned to me with no reviewers, watch CI only when the repo has
  CI or the tests could not run locally, and hand off with the PR body on my
  clipboard. Hard rules inside it: never merge, never comment, never request
  reviewers, never push to protected branches, switch to the right CLI account
  before the first call, and ask when acceptance criteria are unclear.
- Load check: after a restart, run `/context all` again and compare with the audit
  measurements. Report real numbers, not estimates.
- Delegation check: in a fresh headless session, have one agent delegate to another
  and confirm the hook denies a blocked call inside the delegated agent.

**Phase 4: Hygiene**
- Per-project settings files: list them with rule counts and last session date.
  Remove only what I confirm, keeping folders that hold documentation. Remove
  malformed rules. If I run in a permission mode that skips prompts, say that these
  allow rules do nothing there, and offer to delete the files.
- Broad file-read grants: show them and propose narrower ones.
- Stale worktrees: for each one list uncommitted changes and commits that exist on
  no remote, and check whether the commits live on local branches. Stop and ask if
  any work would be lost. Removing a worktree keeps its branches.

**Phase 5: First run and wrap-up**
- Dry run: I pick a low-risk issue. Run the flow, then report where it hesitated,
  what it asked, which agents it used, how long it took, and every hook block.
  Fix the agents, the skill or the hook rules from what we learn.
- Final summary: the full commit log of the config folder, everything deleted or
  archived, backups still on disk (and when to delete them), open items for me
  (tokens to revoke, roles to tighten, optional account-level changes), and
  anything left undone. Everything must end committed.

### Known gotchas to bake into the plan

- Measure context with `/context all`; estimates were wrong by a wide margin.
- Hook false positives from plain text matching are the most common source of
  blocked work. Test for them.
- The GitHub CLI can fail on `issue view` and `pr edit` with a "Projects (classic)"
  deprecation error. Read issues and assign through `gh api` instead, and let the
  hook allow self-assignment only.
- Agents and skills added during a session do not appear until a fresh session.
  Test in `claude -p`.
- Do not widen scope silently. If a cleanup would also affect other projects (a
  global prune, a shared cache), ask first.
- Do not store hostnames, account IDs or passwords in tracked config. If I choose to
  keep a password in a local file, it is my decision; note the trade-off once.

## Step D: Finish

When both files are written, tell me their paths and approximate sizes, list the
decisions you made on my behalf (there should be none that are not marked as a
default), and list the questions that remain open. Do not start executing the plan.
I will review it, change what I want, and tell you when to begin.
