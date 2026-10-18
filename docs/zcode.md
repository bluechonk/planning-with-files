# ZCode Setup

ZCode discovers this fork through `.zcode-plugin/plugin.json`. The plugin keeps
one shared `hooks/hooks.json` descriptor for ZCode and Claude Code; the hook
logic stays in `hooks/claude-hook.sh`.

## Why every hook is one command string

ZCode's `type: "command"` hook executes only the `command` field. An `args`
array is implemented only for `type: "process"`, so a descriptor that splits
the interpreter and the script path across `command` and `args` runs a bare
`sh`. That process reads the hook JSON on stdin as shell input and exits
non-zero.

The shared descriptor therefore writes each hook as a single command string:

```json
"command": "sh \"${CLAUDE_PLUGIN_ROOT}/hooks/claude-hook.sh\" pre-tool-use"
```

Claude Code accepts the same shape, so both hosts execute the identical
descriptor. The timeout is 20 seconds, which covers the Git Bash cold path on
Windows hosts without the Python fast path.

## Event coverage

| Event | ZCode | Notes |
| --- | --- | --- |
| SessionStart | yes | `startup|resume|clear|compact` |
| UserPromptSubmit | yes | injects the selected plan |
| PreToolUse | yes | plan reminder before matched tools |
| PostToolUse | yes | progress reminder after writes |
| Stop | yes | completion gate |
| PreCompact | no | ZCode has no PreCompact event |

`PreCompact` lives in `hooks/claude-precompact.json` and is declared only by
`.claude-plugin/plugin.json`. This keeps ZCode free of an unsupported-event
diagnostic while Claude Code keeps its pre-compaction reminder. ZCode restores
context after compaction through the `SessionStart` matcher `compact`.

## Install from the fork

The repository root is a local marketplace: `.claude-plugin/marketplace.json`
lists the plugin with source `./`.

1. Clone the fork.
2. In ZCode, open Settings, then Plugins, then Marketplaces.
3. If an upstream `planning-with-files` marketplace already exists, remove it
   first. This fork keeps the same marketplace and plugin name, so the two
   cannot be installed side by side.
4. Add the fork directory as a marketplace, then install or update
   `planning-with-files`.
5. Open the plugin detail and confirm the five ZCode hooks are listed and no
   unsupported-event diagnostic is shown.
6. Start a new session. ZCode registers plugin hooks when the session is
   created, so an open session does not pick up an update.

The plugin ID is `planning-with-files@planning-with-files`.

## Verify

Create a project with `task_plan.md` and `progress.md`, then start a new
session in that directory. The first prompt should inject the plan. Tool calls
should not produce `hook.run.failed` entries in the ZCode log.

## Upstream updates

```bash
git fetch upstream
git merge upstream/master
python scripts/bump-version.py <version>
```

`.zcode-plugin/plugin.json` is part of the version parity set, so the bump
script keeps it aligned with the other manifests. Reinstall or update the
plugin after a version change.
