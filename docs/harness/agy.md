# pstack in agy

pstack skills use Cursor names. In agy (the Antigravity CLI), read this file once per session, then map each Cursor name in a skill with the tables below. `<pstack>` means the folder that holds `skills/` and `docs/`. For the installed plugin it is `~/.gemini/config/plugins/pstack/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | agy |
| --- | --- |
| `Task` | `invoke_subagent` with `TypeName` set to an agent from Agent types below and `Workspace: 'inherit'`. One call takes up to 16 subagents, so a "spawn all in one message" step is one `invoke_subagent` call. |
| `AskQuestion` | Ask in plain text, with the options as a numbered list. |
| Open a todolist | Keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | `view_file` `<pstack>/skills/<name>/SKILL.md` and follow it. When you tell the user to type it, write `/pstack:<name>`. |
| `Read`, `Grep`, `Glob`, `Shell` | `view_file`, `grep_search`, `find_by_name`, `run_command`. |
| Resume a subagent | Spawn a new one and paste the earlier result into its brief. |

## Agent types

agy fixes a subagent's model in its agent definition, not per call, and reasoning effort is a session setting with no per-subagent value. So a model role is an agent type. The plugin ships these:

| Agent type | Model tier | Writes files | Use for |
| --- | --- | --- | --- |
| `poteto-agent` | flash (Gemini 3.8 Flash) | yes | Code delegates: feature, refactoring, bug-fix, perf-issue, hillclimb, swarm workers. |
| `poteto-agent-pro` | flash (Gemini 3.8 Flash) | yes | The hardest changes, and the first seat of arena and architect runners. Normally the pro tier (Gemini 4 Argon); mapped to flash while the Google AI Plus plan has no Argon access. |
| `pstack-scout` | flash | no | Search: how explorers, why investigators, recall miners, the reflect tooling and divergent lenses, extra panel seats. |
| `pstack-judge` | flash | no | Judgment and prose: how explainer, why synthesizer, reflect judgment and synthesizer, the arena cross-judge, the first interrogate reviewer. Normally the pro tier; mapped to flash under the Plus plan, like `poteto-agent-pro`. |
| `comment-sicko` | flash | comments only | The no-comments pass. |

**Current plan configuration: every seat runs on flash.** The account is on Google AI Plus, which cannot use Gemini 4 Argon (paid Pro / Ultra only), so `poteto-agent-pro` and `pstack-judge` are defined with `model: flash`. Panels have no tier diversity in this configuration; do not report a pro/flash split. To restore the pro tier after returning to a paid Pro or Ultra plan, set `model: pro` again in those two agent definitions. On a pro-capable plan, note that pro quota runs out often and agy does not fall back on its own: a pro-tier subagent that hits a quota or capacity error stops with an error, so rerun the same brief once on the flash counterpart (`poteto-agent-pro` → `poteto-agent`, `pstack-judge` → `pstack-scout` with the judge's role written into the brief) and say in the reply which seats ran on flash. Never retry the pro seat in a loop, and never switch on paid credits.

Map a skill's model roles and subagent fields with this table:

| Cursor | agy |
| --- | --- |
| A `claude-*` slug, `judgment and prose`, or `hardest tasks` | `pstack-judge` for read-only work, `poteto-agent-pro` for work that edits files. |
| A `grok-*`, `gpt-*`, `composer-*`, or other fast slug | `poteto-agent` for work that edits files, `pstack-scout` for read-only work. |
| `auto` or `inherit-parent` | The same split by whether the work edits files. |
| A panel of two or more seats | One lead seat (`poteto-agent-pro` / `pstack-judge` by write access) and the rest flash. On a pro-capable plan the lead seat is the pro tier, so the panel keeps two model tiers; under the current Plus configuration every seat is the flash model. |
| `subagent_type: generalPurpose` | `poteto-agent`, or `poteto-agent-pro` for the pro seat. |
| `readonly: true` | `pstack-scout` or `pstack-judge`. Their tool lists cannot write files, and the brief still says "do not edit files". Check `jj st` afterward. |
| `environment: "cloud"` | No cloud workers. Run `jj workspace add <path> -r <base>` yourself for each writing worker, put the path in the brief, and tell it to work only there and report with `jj diff`. |
| `environment: "local"`, `run_in_background` | Omit. |
| `file_attachments` | List the paths in the brief. |
| The `pstack-models.mdc` rule, `/setup-pstack` | Not used. The agent definitions above are the model configuration. |

## Version control

Every repo on this host is a colocated jj repo, and git HEAD is detached there by design. Map each git step in a skill to jj. Never run a git write command (`add`, `commit`, `checkout`, `switch`, `branch`, `stash`, `rebase`, `reset`, `worktree`, `push`). Read-only `git` (`log`, `show`, `diff`, `merge-tree`, `remote get-url`) and `gh` are fine in the main repo.

| Cursor (git) | agy (jj) |
| --- | --- |
| `git worktree add <path> <b>` | `jj workspace add <path> -r <b>` |
| `git worktree remove --force <path>`, `git worktree prune` | `jj workspace forget <name>` from the main repo, then `rm -rf <path>`, then `jj workspace list` to confirm. `jj undo` restores a forget. |
| `git fetch && git reset --hard origin/<b>` | `jj git fetch` then `jj new <b>@origin` |
| `git checkout <sha>` | `jj new <sha>`, or `jj edit <sha>` to change that commit itself |
| `git add <files> && git commit -m` | `jj commit <files> -m "..."`, with explicit paths when the skill says not to stage everything |
| amend | `jj squash` (into `@-`) or `jj squash --into <change>` |
| `git rebase` onto trunk, reorder a stack | `jj git fetch` then `jj rebase -d 'trunk()'`; reorder with `jj rebase -s <change> -d <parent>`. Descendants follow. |
| split a commit | `jj split <files>` |
| `git push`, `git push --force-with-lease` | `jj bookmark set <name> -r @-` then `jj git push -b <name>`. After a rebase, move every bookmark in the stack and push them together (`jj git push -b <a> -b <b>`). Ask the user before moving a bookmark someone else pushed to. |
| `gh pr create` | Push the bookmark first, then `gh pr create --head <name> --base <parent-or-trunk>`. gh cannot infer the branch from a detached HEAD. |
| `git diff <base>...HEAD`, `git diff main...HEAD` | `jj diff -r '<base>..@'`, base as a revset such as `trunk()` or `<b>@origin`. `git diff` against HEAD misses the working-copy change `@`. |
| `git diff` (working tree), `git status` | `jj diff`, `jj st` |
| `git log <base>..HEAD` | `jj log -r '<base>..@'` |
| `git show origin/<b>:<path>` | `jj file show -r <b>@origin <path>` |
| current branch | `jj log -r @- --no-graph -T 'bookmarks'` |

## Transcripts and memory

agy keeps each conversation's steps in a binary database, so read its index instead of the steps.

| Cursor | agy |
| --- | --- |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.gemini/antigravity-cli/conversation_summaries.db` has one row per conversation with `conversation_id`, `title`, `preview`, `workspace_uris`, and `last_modified_time`. Query it read-only (`sqlite3` URI with `mode=ro`, for example through `python3 -c`), filtered by the workspace path and ordered by `last_modified_time`. `~/.gemini/antigravity-cli/history.jsonl` has every user prompt with its `workspace` and `timestamp`. When an answer needs a step's contents, say they were not readable. |
| Past sessions and decisions (`recall`, `why`, session pickup) | Search the shared memmini memory first with `memory_smart_search`, using the user's wording, then read the conversation index. memmini is a lead, not a citation. |

## Local equivalents

| Cursor | agy |
| --- | --- |
| `control-ui`, browser verification | The `web-verify` skill, or agy's own browser tools. |
| `control-cli`, launching the app | `run_command` against the real binary. |
| `/deslop` on code | The `ponytail-review` skill when it is available, else the `unslop` pass. |
| `/deslop` on prose | The `unslop` skill for English, `yomiyasu` for Japanese prose. |
| `/create-skill` | Write the skill folder directly, then `agy plugin validate` when it lives in a plugin. |
| Bugbot or an agentic review pass | The `interrogate` skill. |
| `/loop`, Automations routines and wakes | No equivalent. Skip with the line below. |
| Cursor cloud agent | No equivalent. Every subagent runs on this host. |

## Host

agy runs on Linux (Debian, arm64) and on macOS here.

| Cursor | agy |
| --- | --- |
| `npm`, `npx`, `pnpm`, `yarn` | `bun`, `bunx`. Package installs go through `pmg`. |
| `node <script>` | `bun <script>` |
| `pip`, `python` | `uv run`, `uv add` |
| `stat -f`, `date -r <epoch>` | On Linux, `stat -c '%Y %n'` and `date -d @<epoch>`. |
| Commit messages, PR text, code comments | English. Replies to the user follow the session language. |

## Comments

Agents on this host follow the ponytail working style, which marks a deliberate shortcut with a `ponytail:` comment that names its ceiling and upgrade path. Every comment pass keeps them.

## Cursor-only features

When a step needs a Cursor feature with no row above, put one line in your reply, `Skipped in agy: <step>. <reason>.`, then go on to the next step. Do not stop, ask, or retry.
