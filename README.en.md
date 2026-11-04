# Full-Stack Stack Guide

The home of an **Agent Skill**: `fullstack-stack-guide` — a **technology stack selection guide** for full-stack projects (TypeScript track / Python track).

It answers two questions:

1. **Selection.** A full-stack project spans a dozen layers (frontend framework, backend, API style, database, ORM, auth, runtime, deployment, testing). Which combinations are genuinely incompatible, which are just community convention, and where the version numbers should be pinned — these are **verifiable facts**, not something to recall or guess.
2. **Terminology.** When a user says "frontend and backend", "interface", "migration", "deployment", they may mean something different from what an engineer means. The skill teaches the terminology *during* the selection conversation, so the user knows what they participated in deciding.

The skill stores **no version facts** — only **structural rules and verification paths**. It never states what the current version is; it records how to check official docs, how to judge compatibility, and how to store the evidence as files. Versions must be looked up and evidenced at the moment of selection. See [ADR 0001](docs/adr/0001-no-version-snapshots.md).

## Layout

```
skills/fullstack-stack-guide/
  SKILL.md                      Main skill: three-stage flow + boundaries + re-check
  references/
    artifacts.md                Artifact formats: decision table, version evidence, change impact
    constraints-python.md       Python track: layer candidates + hard constraints + conventions
    constraints-typescript.md   TS track: layer candidates + hard constraints + conventions
    database-engines.md         Database engine selection (shared by both tracks)
    glossary.md                 Full-stack term entries (term / plain language / why it matters / common misuse)
    version-verification.md     Version verification procedure (npm / PyPI field sets)
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

## Configuration

**No configuration needed** — no keys, no accounts, no setup step. Works right after install.

The skill only reads and writes files when triggered; it runs no resident process.

Version verification needs `npm` (TypeScript track) or `curl` and `pip` (Python track) locally. These are dependencies of the verification commands, not of the skill itself — without them the skill still works, but version evidence will be incomplete.

The skill creates a `docs/stack/` directory in the consuming project and may append a short stack section to that project's `AGENTS.md`. That is its only write to external state.

## What it leaves behind in your project

```
docs/stack/
  decision.md          Stack decision table (per-layer choice + rationale + cost)
  version-evidence.md  Version verification record (commands + output + date)
  change-impact.md     Change impact assessment (produced during development re-checks)
```

Plus a short section appended to the project's `AGENTS.md` pointing at `docs/stack/decision.md`, so later sessions can read the selection back.

## Scope and boundaries

**In scope**: stack selection, candidate comparison, combination compatibility, version compatibility verification, terminology explanation, and evaluating "continue / degrade / swap" when requirements change mid-development.

**Out of scope**: requirements clarification and alignment, architecture design, writing code, code review, deployment operations. Those belong to other skills or to your own judgement.

**Not covered**: the skill gives general stack guidance; it does not design a solution for a specific product. An existing, verified stack choice does not need it overturned — unless a requirement change touches a selection premise.

## Data and permissions

- The skill **collects and uploads nothing**. No telemetry, no reporting, no key storage.
- The only outbound requests are the read-only `npm view` / `curl https://pypi.org/...` calls used to verify versions.
- The only write to external state: creating files under `docs/stack/` in the consuming project, and appending a stack section to that project's `AGENTS.md`. Existing content is checked before writing.

## Compatibility

| Dimension | Status |
|---|---|
| Operating system | No platform lock. The skill is documentation and a workflow; macOS / Windows / Linux all work. Verification commands are written portably |
| Host | Any host supporting the Agent Skills directory convention (`SKILL.md` + frontmatter). Verified to be discovered after `npx skills add` |
| Runtime | No runtime dependency. The skill contains no scripts |
| Verification tools | TS track needs `npm`; Python track needs `curl` and `pip` (optional `jq`). Without them the skill still works, but version evidence is incomplete |
| Network | Version verification needs access to the npm registry and PyPI. Offline the skill still handles terminology and constraint checking, but versions must be supplied manually |
| No sub-agent capability | Supported. The skill is a workflow and reference set for a single agent; it requires no delegation |

## Design decisions

See `docs/adr/`. The three that most affect how it behaves:

- [0001](docs/adr/0001-no-version-snapshots.md) stores no version facts, only structural rules and verification paths — the skill does not track version evolution, so it never becomes a version table that needs continuous upkeep
- [0002](docs/adr/0002-constraint-layers.md) splits constraints into "technical hard constraints" and "ecosystem conventions"
- [0003](docs/adr/0003-scope-boundary.md) selection and terminology only, no requirements alignment

## License

MIT — see [LICENSE](LICENSE).
