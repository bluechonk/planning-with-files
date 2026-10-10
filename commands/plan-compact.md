---
description: "Manually flush planning state to disk before a context compaction. Use in runtimes (e.g. ZCode) that expose no PreCompact hook: run this right before /compact so progress.md and task_plan.md survive the summary."
disable-model-invocation: true
allowed-tools: "Bash, Read, Edit, Write"
---

You are about to compact the conversation. Before you do, make sure the
planning state is persisted to disk so it survives the summary. This command
exists because some runtimes (notably ZCode) fire no PreCompact hook, so the
automatic "flush before compaction" reminder never runs on its own.

Steps:

1. Emit the canonical pre-compaction reminder by running the plugin's own
   precompact path from the project root:
   - Linux/macOS/Git Bash: `sh ${CLAUDE_PLUGIN_ROOT}/scripts/inject-plan.sh --context=precompact`
   - Windows without Git Bash on PATH: locate Git Bash via `git.exe` (its
     `usr\bin\sh.exe` sibling) and run the same script with it.
   The script prints nothing when there is no active plan; that is expected.

2. If the three planning files exist in the current project directory, update
   them now, before compaction:
   - `progress.md` — append the recent actions taken since the last entry.
   - `task_plan.md` — set "## Current Phase" and each phase status so it
     reflects where work actually stands right now.
   - `findings.md` — record any discovery not yet written down.
   Keep status markers in their literal English form (`**Status:**
   in_progress`, `**Status:** complete`) — `check-complete.sh` greps for them
   verbatim.

3. Confirm persistence: list which of `task_plan.md`, `findings.md`,
   `progress.md` exist in the project directory and report them as
   `file ✓/✗`. These files stay on disk and will be re-read after compaction,
   so the plan does not depend on the model remembering the dropped turns.

4. Tell the user it is safe to compact now. Do not perform the compaction
   yourself — that is the user's action.

If none of the three files exist, say so and stop; there is nothing to flush,
and running /plan first would create them.
