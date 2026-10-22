---
name: openspec-subagents
description: Run OpenSpec work (explore, propose, update, apply, archive, sync-specs) in subagents, each on a model matched to the task. Make sure to use this skill whenever the user mentions openspec, /opsx commands, an openspec/ directory or change, or asks to explore, propose, update, apply, implement, archive or sync a change or spec, even if they don't say "subagent" or name a model. It checks whether explore or propose is a big change and asks before spending Opus on it.
---

# OpenSpec via subagents

OpenSpec work has a very uneven cost of mistakes. A weak proposal poisons every task derived from it, while archiving is close to a script. So each OpenSpec task runs in its own subagent on the model that fits it, and the main conversation stays a thin coordinator: pick the task, pick the model, brief the subagent, relay the result.

The repos already ship the workflow itself as project skills (`openspec-explore`, `openspec-propose`, `openspec-update-change`, `openspec-apply-change`, `openspec-archive-change`, `openspec-sync-specs`). This skill does not replace them; it decides who runs them and on what model.

## Model table

| Task | Model | Prompt the user first? |
|---|---|---|
| explore | `sonnet` | Only if it looks like a big change, then offer `opus` |
| propose | `sonnet` | Only if it looks like a big change, then offer `opus` |
| update | `sonnet` | No |
| apply | `sonnet` | No |
| archive | `haiku` | No |
| sync-specs | `haiku` | No |

If the user names a model ("use opus for this"), use it and skip any prompt. A named model beats the table.

## Step 1: Identify the task

Map the request to one of the six tasks. `/opsx:<name>` commands map one to one. If the request is ambiguous (for example "work on the auth change"), look at `openspec list --json` and `openspec status --change "<name>" --json` to see which artifacts exist before choosing. If it is still unclear, ask.

Find the repo root that holds the `openspec/` directory. In this workspace that is usually `vice-backend`, `vice-frontend` or `workflow-engine` under `~/Documents/code`. If the user named a store, carry `--store <id>` through to the subagent.

## Step 2: Size check (explore and propose only)

Explore and propose are where judgment compounds, so this is the only place a bigger model changes outcomes. Decide whether the work is big. Signs of a big change:

- **Cross-cutting concerns**: auth, permissions, logging, error handling, data model or schema changes that ripple through many modules.
- **Multiple repos or services**: a contract change between backend, frontend and gateway, or anything where specs in more than one `openspec/` root are affected.
- **Many capabilities**: the change would touch three or more existing specs, or introduce several new ones.
- **Risky or hard to reverse**: migrations, public API changes, concurrency, rollout sequencing.
- **Unclear scope**: the user is still deciding what the change even is, and the answer depends on understanding a lot of existing code.

Skim the request and, if cheap, the relevant specs (`openspec list --specs`) before deciding. Two or more signs, or one strong one (multi-repo, migrations), means big.

- **Big**: ask with AskUserQuestion. Say in one line why it looks big, then offer `Sonnet` and `Opus` (put Sonnet first and recommend it only if it is close to the line; otherwise recommend Opus). Use the answer as the model.
- **Not big**: use `sonnet` without asking. Do not ask "should I use Opus?" for ordinary changes; the point of the default is that the user should not have to think about it.

Update, apply, archive and sync-specs never prompt. Apply stays on Sonnet by design: a good proposal constrains it, and if apply struggles, the usual fix is sharper tasks, not a bigger model.

## Step 3: Spawn the subagent

Use the Agent tool with `subagent_type: "general-purpose"` and the `model` from Step 1 or 2. Brief it like a colleague who has none of this conversation. Include:

1. **The task and the skill to use**: "Invoke the `openspec-<task>` skill (Skill tool) and follow it." If that skill is not available in the repo, tell it to run `openspec instructions <artifact|apply|archive> --change "<name>" --json` and follow the output.
2. **Where**: the absolute repo path, the change name (if any), and the store id (if any).
3. **The user's request, verbatim**, plus any context from the conversation that the subagent can't get from the repo (decisions already made, constraints, links).
4. **No live user**: the subagent cannot ask the user anything. If it hits a question that matters (ambiguous requirement, a confirmation prompt in the underlying skill, incomplete tasks during archive, a conflicting spec), it must stop and return the question instead of guessing or answering for the user. This matters most for propose, where a guessed requirement is worse than a short delay.
5. **What to return**: a short report, not a transcript. Per task:
   - explore: findings, options with tradeoffs, and open questions. Write no files unless the user already confirmed the scope.
   - propose / update: change name, artifacts created or changed, and any assumptions or open questions.
   - apply: tasks completed (checkboxes ticked in `tasks.md`), tasks remaining or blocked, files touched.
   - archive / sync-specs: what moved or merged, and any warning the CLI printed.

For **apply**, also add: do not run tests, rubocop, or any docker command; list the exact commands for the user to run instead. The user runs these themselves, and subagents don't inherit that rule unless told.

Run independent tasks in parallel (for example, archiving two finished changes), but keep tasks that depend on each other sequential (propose before apply).

## Step 4: Relay the result

Tell the user which model ran the task in one short line (for example "Propose ran on Sonnet"), then summarize the subagent's report. Surface its open questions prominently, since the user is the only one who can answer them.

Don't take the subagent's word on spec merges: after archive or sync-specs, glance at the resulting diff of `openspec/specs/` (or the report) because a bad merge is the one failure that stays silent.

## Follow-ups

Explore is conversational but a subagent is one-shot. To continue an exploration, answer its open questions by messaging the same subagent with SendMessage so it keeps its context, rather than spawning a new one. Start a fresh subagent only if the direction changed completely. If the user then wants to move from explore to propose, spawn a new subagent for propose and pass along the conclusions from explore in the brief.

If the user asks you to switch models mid-flow ("redo that on Opus"), spawn a new subagent on the requested model with the same brief and note what the first attempt produced.
