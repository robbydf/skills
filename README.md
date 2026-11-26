# skills

My agent skills for Claude Code, packaged as a plugin so they're available in every project at the user level. No per-project setup or initialization is needed.

## Install

In Claude Code:

```
/plugin marketplace add robbydf/skills
/plugin install robbydf@robbydf
```

Choose **user scope** when prompted (the default) so the skills load in every project. To pick up new versions later, run `/plugin marketplace update robbydf`.

Plugin skills are namespaced, so they appear as `robbydf:<name>` (e.g. `/robbydf:grill-me`).

## Skills

| Skill | What it does |
| --- | --- |
| `grill-me` | Manual-only (`/robbydf:grill-me`): starts a grilling session. |
| `grilling` | Interviews you relentlessly about a plan or idea, in rounds, with a recommended answer for each question, until nothing is left assumed. |
| `design-alternatives` | "Design it twice": parallel sub-agents produce radically different designs for an interface, schema, architecture or workflow, then compares them and recommends one. |
| `prototype` | Builds a throwaway prototype to answer one design question: a single-file HTML logic demo, or switchable UI variants. |
| `review-plan-loop` | Independent reviewer sub-agent critiques a plan; fix and re-review until clean. |
| `review-code-loop` | The same loop for code changes. |

`grill-me`, `grilling`, `design-alternatives` and `prototype` are adapted from [mattpocock/skills](https://github.com/mattpocock/skills); see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Without the plugin

You can also symlink or copy any `skills/<name>` folder into `~/.claude/skills/`. Skills then use bare names (`/grill-me`).

## License

MIT, see [LICENSE](LICENSE). Adapted skills keep their original MIT notice in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
