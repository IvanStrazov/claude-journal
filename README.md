> [!IMPORTANT]
> **This repository has moved to [IvanStrazov/animus-journal](https://github.com/IvanStrazov/animus-journal)** and is now part of the Animus ecosystem. This copy is frozen at 0.2.0 and gets no updates.
>
> To switch: `/plugin marketplace add IvanStrazov/animus-journal`, then `/plugin install animus-journal@animus-journal`, then uninstall `claude-journal`. Your data is untouched.

# claude-journal

[![tests](https://github.com/IvanStrazov/claude-journal/actions/workflows/test.yml/badge.svg)](https://github.com/IvanStrazov/claude-journal/actions/workflows/test.yml)
[![version](https://img.shields.io/badge/version-0.2.0-blue)](CHANGELOG.md)
[![license: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
![dependencies: none](https://img.shields.io/badge/dependencies-none-brightgreen)
![python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)

**A minimal project journal for Claude Code, on a hard token budget.**

Most memory plugins record everything and pay for it in context. claude-journal does the opposite: it keeps a short, human-readable journal and keeps the cost low with limits enforced in code, not just in the prompt.

- **≤ 40-line project snapshot** (`STATE.md`) + the last 5 session lines are injected at session start. That's all.
- **≤ 20 lines per session entry**, written at wrap-up in a single shell call.
- **Lookup by task tag**: `find <tag>` returns only the 3 newest matching entries, not the whole history.
- **No hooks on every tool call or prompt.** One `SessionStart` hook, nothing else.
- **One stdlib Python file + plain Markdown.** No database, no vector store, no background process, no network, no extra model calls.

## Token budget (measured)

| Moment | What enters context | Worst case |
|---|---|---|
| Session start | `STATE.md` (≤ 40 lines) + 5 index lines | ~3.3 KB ≈ 800 tokens |
| Recording | the entry you write (≤ 20 lines) + a one-line ack | ≈ 150–300 tokens |
| Recalling a task | 3 matching entries | ≈ 200–1 000 tokens |
| Skill instructions | loaded only when the skill is used | ~2.4 KB ≈ 600 tokens |

Limits can be changed with environment variables: `JOURNAL_STATE_MAX`, `JOURNAL_ENTRY_MAX`, `JOURNAL_INDEX_TAIL`, `JOURNAL_SAVE_MAX_KB`.

## What it stores

```
<project>/.journal/
  STATE.md         current snapshot: goal, done, in progress, open questions, next steps
  INDEX.md         one line per session: date | #tags | title
  log/2026-11.md   session entries, one "## " section each
  scripts/ data/   helper scripts and small data files (≤ 1 MB) kept with the project
  .gitignore       "*" — the journal stays local and is never committed
```

Sessions without a project (home dir, temp dir, throwaway workspaces) go to a shared `~/journal/` (override with `JOURNAL_GLOBAL_DIR`).

An entry looks like this:

```
## 2026-11-16 19:47 #parser #api — Switched log parser to regex
Goal: parse nginx logs without the json module
Done: src/parse.py, tests/test_parse.py; commit 3f2a1c
Decided: regex over json — logs are not valid JSON
Next: handle multi-line stack traces
```

## Install

```
/plugin marketplace add IvanStrazov/claude-journal
/plugin install claude-journal@claude-journal
```

Requires Python 3.9+ on `PATH` (`python3` or `python`). Works on macOS, Linux and Windows (Git Bash, which Claude Code uses on Windows).

## Use

You mostly don't have to do anything:

- At session start Claude sees the snapshot and recent sessions.
- At the end of substantive work Claude records an entry and updates the snapshot. You can also ask: `/journal` or "write this to the journal".
- Ask "what did we do on the parser?" — Claude runs `find parser`.

Commands (Claude runs them; you can too): `add`, `state`, `find`, `tags`, `save`, `where`, `link` — see `python scripts/journal.py -h`.

## Works with claude-portrait (optional)

If [claude-portrait](https://github.com/IvanStrazov/claude-portrait) is installed, the journal offers to link the two, once. Nothing changes until you agree (`journal.py link on`, or say yes in the `portrait` skill).

When linked, the plugins share `~/.claude/ecosystem.json`:
- **Session start** adds a short digest of your portrait (default: Communication + Who, ≤ 15 lines). `portrait.py digest-cost` shows what each size costs you with prompt caching.
- **At wrap-up** the journal updates only the portrait's *Projects* and *Communication & preferences* sections from that session, and only when something changed. Sensitive topics are never touched.

`journal.py link off` undoes it. A missing or broken `ecosystem.json` never breaks session start.

## How it compares

If you want automatic capture of every action with semantic search, use a full memory system such as claude-mem. claude-journal is for people who want a small, readable log they control, with a predictable and low context cost.

## License

MIT
