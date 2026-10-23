# agy-pstack

[pstack](https://github.com/cursor/plugins/tree/main/pstack) (MIT, Lauren Tan) adapted to the Antigravity CLI (`agy`) on this host. It starts from the Claude Code fork's skill set, so the jj, memmini, and ponytail adaptations carry over, and swaps the Claude Code adapter for [`docs/harness/agy.md`](docs/harness/agy.md).

## What changes from upstream

- **Models are agent types.** agy fixes a subagent's model tier in its agent definition, and reasoning effort is a session setting. So each model role maps to one of five agents:

  | Agent | Tier | Writes | Role |
  | --- | --- | --- | --- |
  | `poteto-agent` | flash | yes | code delegates, swarm workers |
  | `poteto-agent-pro` | pro | yes | hardest changes, pro seat of arena and architect runners |
  | `pstack-scout` | flash | no | explorers, investigators, extra panel seats |
  | `pstack-judge` | pro | no | explainers, synthesizers, judges, first reviewer |
  | `comment-sicko` | flash | comments only | no-comments pass |

  Panels keep two tiers: one pro seat, the rest flash. Change a role by editing `model:` in `agents/<name>.md` and reinstalling.
- **Delegation** is `invoke_subagent` with `TypeName` and `Workspace: 'inherit'`, up to 16 per call.
- **Version control** maps git steps to jj. Parallel writers get `jj workspace add`.
- **Transcripts.** agy stores steps in a binary database, so `recall` and session pickup read `conversation_summaries.db` and `history.jsonl`, after searching memmini.
- **Skills.** 49 of upstream's 57. Left out: `make-bot-ui`, `automate-me`, `setup-pstack`, `statechart`, the two formal verification skills, `typescript-best-practices`, and `bro`. Task skills are model-invocable; the 24 principle skills stay hidden.

## Install

```bash
agy plugin validate ~/projects/agy-pstack --strict
agy plugin install ~/projects/agy-pstack
```

agy copies the plugin to `~/.gemini/config/plugins/pstack/`. After editing this repo, bump `version` in `plugin.json`, reinstall, and start a new conversation. Type `/pstack:poteto-mode` to start.
