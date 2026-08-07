---
name: save-notes
description: Save, update, or recall Adam's personal reference notes for a project, stored in ~/Documents/code/notes/<repo-name>/. Use when asked to "save these notes", "write this up so I can reference it later", "add this to my notes", or when asked what was previously noted about a topic ("what did I note about X?", "check my notes on Y"). Not for repo documentation (README, docs/) or code comments — those belong in the repo.
---

# Save Notes

Personal, long-lived reference notes that live **outside** the repo so they are never committed.
They capture cross-cutting explanations, gotchas, "why is it like this", open questions, and
proposed-but-unimplemented fixes — the things that don't belong in repo docs.

## Location

```
~/Documents/code/notes/<repo-name>/
├── INDEX.md                        # one line per note, kept current
├── migrations-and-deploy-tasks.md
└── squire-feature-flags.md
```

### Resolving `<repo-name>`

Use the **git repo name**, not the directory basename — Adam uses worktrees, where the directory
does not match the repo. Run this from anywhere inside the project:

```bash
name=$(basename -s .git "$(git remote get-url origin 2>/dev/null)" 2>/dev/null)
[ -z "$name" ] && name=$(basename "$(dirname "$(cd "$(git rev-parse --git-common-dir)" && pwd)")")
echo "$name"
```

The `origin` remote is shared across worktrees, so the first branch handles the common case. The
fallback uses `--git-common-dir`, which points at the *main* checkout's `.git` even from inside a
worktree. Both paths are verified against worktrees with and without a remote.

If the directory is not a git repo, ask where the notes should go rather than guessing.

## Sandbox

`~/Documents/code/notes/` is **outside the Bash sandbox write allowlist**. `mkdir` and any shell
writes there fail with `Operation not permitted` and need `dangerouslyDisableSandbox: true`.
Create the directory that way first, then use the normal Write/Edit tools for the files:

```bash
mkdir -p ~/Documents/code/notes/<repo-name>    # needs dangerouslyDisableSandbox: true
```

## Deciding: new note, or update an existing one?

Always list the project's notes folder first.

- **A note already covers this topic** → extend or revise that file. Do not create a near-duplicate.
  Re-verify any existing file:line refs you touch; correct stale ones and update the version stamp.
- **No matching note** → create a new one. Filename is kebab-case and descriptive of the topic
  (`migrations-and-deploy-tasks.md`, not `notes-2.md` or `vice-backend-notes.md`).
- **Recall request** ("what did I note about X?") → read the relevant notes and answer from them.
  Flag anything whose version stamp is old or whose line refs no longer match the code, rather
  than repeating it as current fact.

Then update `INDEX.md` — see below.

## Required conventions for every note

**1. Version stamp.** Immediately under the title, record what the note was accurate against, so
stale line numbers are self-evident later:

```markdown
All file references are relative to the `<repo>` repo root, as of the `<branch>` branch
on <YYYY-MM-DD> (`<short-sha>`).
```

Get it with `git rev-parse --short HEAD` and `git branch --show-current`.

**2. Verified `file:line` references.** Cite concrete paths and line numbers — they are most of
the value. **Re-read each one to confirm it is accurate before writing the note.** A wrong line
number is worse than none, because it will be trusted months later. If something can't be
verified, say so in the note instead of citing it.

**3. Quick-reference command block.** When the topic involves commands, end with a
copy-pasteable section. Include the commands actually used, with the non-obvious flags and any
"this one does NOT do X" caveats.

**4. Separate "proposed / not implemented" section.** Recommendations, open questions, and fixes
that haven't been made go in their own clearly-labelled section — never interleaved with how
things work today. Someone reading in six months must not mistake a suggestion for the status quo.

## Content guidance

Write the explanation, not a transcript. The note should stand alone for someone who wasn't in
the conversation.

- Lead with the thing that is actually confusing — the naming mismatch, the inverted default, the
  ordering that isn't what it sounds like.
- Prefer tables for anything with parallel structure (stage → directory → namespace).
- Call out landmines with a marker (⚠️ / 🚩) so they survive skimming.
- Keep the causal chain: not just "the flag defaults to false" but what breaks because it does.
- Include the dead ends and the "why not the obvious approach" reasoning — that is exactly what
  gets re-litigated later.

## INDEX.md

Every save updates it. One line per note, newest concerns first is fine; no frontmatter.

```markdown
# <repo-name> notes

- [Migrations & Deploy Tasks](migrations-and-deploy-tasks.md) — 3-stage pre/migrate/post pipeline,
  one_off vs repeated tasks, why integration envs silently drift
- [Squire Feature Flags](squire-feature-flags.md) — provider sourcing paths, pubsub sync
```

The hook after the em-dash should say what question the note answers, not restate the title.
Create `INDEX.md` if it doesn't exist yet.

## After saving

Tell Adam the path and give a short section-by-section summary of what went in, plus anything
deliberately left out or flagged as unverified.
