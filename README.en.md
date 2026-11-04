# Full-Stack Stack Guide

The home of an **Agent Skill**: `fullstack-stack-guide` — a **technology stack selection guide** for full-stack projects (TypeScript track / Python track).

It answers two questions:

1. **Selection.** A full-stack project spans a dozen layers (frontend framework, backend, API style, database, ORM, auth, runtime, deployment, testing). Which combinations are genuinely incompatible, which are just community convention, and where the version numbers should be pinned — these are **verifiable facts**, not something to recall or guess.
2. **Terminology.** When a user says "frontend and backend", "interface", "migration", "deployment", they may mean something different from what an engineer means. The skill teaches the terminology *during* the selection conversation, so the user knows what they participated in deciding.

The skill stores **no version numbers**. It stores the **verification procedure**: how to check official docs, how to judge compatibility, how to record the evidence as files. Versions must be looked up and evidenced at the moment of selection.

## Layout

```
skills/fullstack-stack-guide/
  SKILL.md                      Main skill: three-stage flow + boundaries + re-check
  references/
    glossary.md                 Full-stack term entries (term / plain language / why it matters / common misuse)
    constraints-typescript.md   TS track: layer candidates + hard constraints + conventions
    constraints-python.md       Python track: layer candidates + hard constraints + conventions
    version-verification.md     Version verification procedure (npm / PyPI field sets)
    artifacts.md                Artifact formats: decision table, version evidence, change impact
    templates/                  Copy-paste artifact templates
```

## Install

```bash
# Global, available across projects
npx skills add meisijiya/full-stack-skill -g

# Project scope only
npx skills add meisijiya/full-stack-skill

# See what is installable
npx skills add meisijiya/full-stack-skill --list
```

The `skills` CLI installs into each agent's skills directory (`.agents/skills/`, `.claude/skills/`, …). Manual equivalent:

```bash
git clone https://github.com/meisijiya/full-stack-skill.git
cp -r full-stack-skill/skills/fullstack-stack-guide ~/.agents/skills/
```

## Usage

The skill is **model-invoked**: its description carries the trigger scenarios, so an agent loads it on its own when it hits selection, version compatibility, or terminology questions. You can also invoke it manually:

```
/skill:fullstack-stack-guide
```

Typical triggers:

- "I want to build a full-stack project, which stack should I use"
- "Can this combination work together"
- "Drizzle or Prisma"
- "The requirements changed — does the current stack still fit"
- "What is an ORM / what is RPC"

## What it leaves behind in your project

```
docs/stack/
  decision.md          Stack decision table (per-layer choice + rationale + cost)
  version-evidence.md  Version verification record (commands + output + date)
  change-impact.md     Change impact assessment (produced during development re-checks)
```

Plus a short section appended to the project's `AGENTS.md` pointing at `docs/stack/decision.md`, so later sessions can read the selection back.

## Design decisions

See `docs/adr/`. The three that most affect how it behaves:

- [0001](docs/adr/0001-no-version-snapshots.md) stores no version numbers, only the verification procedure
- [0002](docs/adr/0002-constraint-layers.md) splits constraints into "technical hard constraints" and "ecosystem conventions"
- [0003](docs/adr/0003-scope-boundary.md) selection and terminology only, no requirements alignment

## License

MIT — see [LICENSE](LICENSE).
