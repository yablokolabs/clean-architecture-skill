# Clean Architecture — agent skill

A coding-agent skill that turns the architectural principles from Robert C. Martin's *Clean Architecture* into rules an agent can apply while it works: where to draw a boundary, which way a dependency points, and what to keep out of the core.

Works with any agent that discovers skills as `<root>/<name>/SKILL.md` bundles — DeepSeek Harness, Claude Code, Codex, Cursor, and similar.

## Why

Agents write a lot of code fast and will happily bury a database call inside a business rule. Architecture advice usually arrives as prose that nobody re-reads. This skill makes the rules short, imperative, and checkable — including a 10-question review checklist and a list of smells to grep for.

## What it covers

| Section | Content |
|---|---|
| The one rule | The Dependency Rule, and a concrete test for violations |
| Policy vs detail | Two values, maximising decisions not made, judging structure by cost-per-change |
| Where boundaries go | Axis of change; level = distance from inputs/outputs |
| SOLID | As architecture rather than class hygiene |
| Components | Cohesion (REP/CCP/CRP) and coupling (ADP/SDP/SAP) |
| Metrics | `I = Ce/(Ca+Ce)`, `A = Na/Nc`, `D = ||A+I−1||`, zones of pain and uselessness |
| The shape | Entities → use cases → adapters → frameworks; Main as the dirtiest component |
| Details | Database, web and frameworks are details |
| Testability | Humble object, the test boundary, testing API |
| Packaging | Package by layer / feature / component, ports and adapters |
| Costs | Partial boundaries, the three decoupling modes, when to skip |
| Review checklist | 10 questions to ask of a real codebase |
| Smells | 16 signals that the structure is failing |

## Install

Copy or symlink the repo so that `SKILL.md` sits directly inside a scanned skill root.

```bash
# shared across agents
git clone https://github.com/yablokolabs/clean-architecture-skill.git \
  ~/.agents/skills/clean-architecture

# or for Claude Code specifically
git clone https://github.com/yablokolabs/clean-architecture-skill.git \
  ~/.claude/skills/clean-architecture
```

The directory name must match the `name:` in the frontmatter (`clean-architecture`). A skill must be a directory containing `SKILL.md` at the top level of the scanned root — nested `**/SKILL.md` files are not discovered.

## Use

The skill is model-invocable, so an agent loads it when a task involves structural decisions. You can also ask for it directly:

```
Use the clean-architecture skill on ./src and give me the 10-question review.
```

## Attribution

The principles encoded here originate in *Clean Architecture: A Craftsman's Guide to Software Structure and Design* by Robert C. Martin (Prentice Hall, 2017). This repository is an independent summary written in original wording, with chapter references so readers can consult the source. It is not affiliated with or endorsed by the author or publisher. See [NOTICE](NOTICE).

The MIT license below covers this repository's own text. It does not grant any rights in the book.

## License

MIT — see [LICENSE](LICENSE).
