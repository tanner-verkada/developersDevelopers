# developersDevelopers — Verkada edition

A workflow plugin for Claude Code, forked from [vrennat/developersDevelopers](https://github.com/vrennat/developersDevelopers) and tuned for Verkada repos. Same lean confirm-only-when-ambiguous workflow; Verkada-specific verification gates, house rules, and Linear/PR conventions baked into the agents.

If you've used `obra/superpowers` and noticed you say "yes, do that" 19 out of 20 times, this is for you.

## What's Verkada-specific

- **`validator`** knows the per-repo quality gates: `just autofix` + `bazel test` (Verkada-Backend), `yarn check-typescript` + scoped `yarn test:unit` (Verkada-Web), Black 100 + Ruff (Verkada-Support), `terraform plan` (Support-Terraform), SUMMARY.md routing (Verkada-Support-Docs).
- **`fast-impl`** carries the house rules: match existing patterns, no conditional/lazy Python imports, strict TS with no `any`, minimal diffs, never delete a failing test.
- **`debug-genius`** reproduces under Bazel (never bare pytest) and checks CloudWatch before local hypotheses for Lambda services.
- **`brutal-code-reviewer`** blocks on customer data in logs/external services, missing authz on internal endpoints, and inline secrets.
- **`/impl`** knows "Fixes TEAM-123" vs "Relates to TEAM-123", that pushing a Verkada-Web branch IS a staging deploy, and that Support Bug comments carry critical context.
- `templates/AGENTS.md` ships pre-filled with the Verkada Linear workspace and support-org team prefixes.

Upstream tracking: `git remote add upstream https://github.com/vrennat/developersDevelopers` is already configured in the working clone; merge upstream periodically.

## What you get

```
/brainstorm "live spectator mode"   → idea becomes a spec at docs/specs/
/impl ERT-1234                      → ticket gets read, classified, executed
/impl docs/specs/foo-design.md      → spec gets routed and built
/research "is option A faster?"     → measurable experiment loop
```

`/impl` is the workhorse. It assesses every task on two axes — clarity and complexity — and **only asks you a question when there's genuine ambiguity**, not after every section of every plan. Touches one file? Direct edit. Touches five? Spawns subagents in parallel. Bug with a real error? Pulls in `debug-genius`. You decide nothing routine; the plugin decides nothing ambiguous.

## Install

```
/plugin marketplace add tanner-verkada/developersDevelopers
/plugin install developersDevelopers-verkada@tanner-verkada
```

Or, once merged into the Verkada support marketplace:

```
/plugin marketplace add verkada/verkada-support-skills
/plugin install developersDevelopers-verkada@verkada-support-skills
```

**Uninstall `obra/superpowers` first** — they collide on slash command names. The personal `developersDevelopers@vrennat` also collides on command names (`/impl`, `/plan`, ...); install one or the other, not both.

## First-time setup (per project)

```
/bootstrap
```

Creates `docs/specs/` and `docs/plans/`, detects legacy `docs/superpowers/` paths, prints copy-paste commands for migration, CLAUDE.md, and hooks. Idempotent.

## Inventory

### Slash commands

| Command | Use |
|---|---|
| `/impl <input>` | The default. Spec file, ticket ID, or freeform description → executed work. |
| `/brainstorm <idea>` | Idea → spec doc at `docs/specs/`. |
| `/research <question>` | Measurable experiment loop (THINK → TEST → REFLECT). |
| `/plan <input>` | *(opt-in)* Spec → written plan when you want one to review first. |
| `/tdd <description>` | *(opt-in)* Strict RED-GREEN-REFACTOR scaffold. |
| `/bootstrap` | Run once per project. |

### Subagents (used by `/impl`)

- `fast-impl` (haiku) — execute clear tasks
- `validator` (haiku) — typecheck + tests
- `brutal-code-reviewer` (sonnet) — review for risky/architectural changes
- `debug-genius` (sonnet) — deep bug investigation

### Auto-trigger skills

You don't invoke these — they fire on signals.

- `systematic-debugging` — fires on observed errors/test failures
- `verification-before-completion` — fires before any "done/fixed/passing" claim
- `onboarding` — surfaces `/bootstrap` for new installs and obra/superpowers migrations
- `plan-hunter` — tournament-style planning (4 lenses → 4 judges → synthesis) for substantive multi-week plans; triggers on `/plan-hunter <idea>` or open-ended planning asks

### Templates (opt-in `cp` from `templates/`)

- `hooks/git-sync-pre-edit.sh` — block edits when local main is behind origin (catches parallel-session divergence)
- `settings.example.json` — pre-wired `.claude/settings.json` for the hook above
- `AGENTS.md` — for non-Claude agents (Cursor, Codex)

## The design rule

**Clarity decides whether to ask. Complexity decides how to route. Never confuse the two.**

| Task | Clarity | Complexity | Behavior |
|---|---|---|---|
| Card backs render larger than fronts; fix it | clear | 1 file | Just do it. |
| Add card sorting to hand | ambiguous (sort by? UI?) | 2-3 files | One batched question, then proceed. |
| Refactor rules engine for layered effects | clear (architecture documented) | >3 files | Just do it via subagent team. |

"Many files touched" is complexity, not ambiguity. "Non-trivial implementation" is complexity, not ambiguity. "I'd like CYA approval" is the rubber-stamp anti-pattern this plugin exists to kill.

**Escape hatch:** destructive ops (force-push, deploys, money) always confirm. Those are blast-radius gates, not ambiguity gates.

## Contributing

Run `./scripts/lint-content.sh` before committing. It enforces the structural rules:

- **Length caps:** skills ≤ 80 lines, commands ≤ 150 lines, agents ≤ 60 lines
- **Frontmatter:** `name:` and `description:` required; description must be one sentence specific enough that no two skills match the same prompt
- **No graphviz dot blocks. No `EXTREMELY IMPORTANT/MUST/NEVER` repeated more than once per file.**
- **No skill references another skill by name in its body** — each skill is a leaf; chaining is the user's job

## License

MIT — see [LICENSE](LICENSE).
