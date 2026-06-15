# sq-constitution

**The sciqnt org's constitution + the governance it injects across every repo.**

This repo is the single source of truth for *how every sciqnt component is governed*.
Two things live here:

1. **The constitution** — `PRINCIPLES.md` (the 18-principle constitution),
   `FOUNDATION.md` (the worldview), and `PRINCIPLE_REVIEW.md` (the rubric the
   principle-review agent applies). Every repo in the org is governed by *these*.

2. **The reusable workflows** (`.github/workflows/*.yml`, `on: workflow_call`) that
   every component repo inherits with a one-line caller — so CI, the conformance
   gate, the principle-review agent, `@claude` fix-mode, and issue-triage are defined
   *once, here*, and bound everywhere. Update the governance by bumping the ref.

> The `sciqnt/sciqnt` monorepo is the source-of-truth for the *code* during the
> component-world transition (see `research/component-world-blueprint.md` there).
> This repo is the source-of-truth for *governance*. Neither modifies the other.

## How injection works

A component repo (`sq-degiro`, `sq-portfolio`, …) carries **thin caller workflows**
that delegate to the reusable ones here (copy from `caller-templates/`):

```yaml
# <component>/.github/workflows/principle-review.yml
on:
  pull_request_target:
    types: [opened, reopened, ready_for_review, synchronize]
jobs:
  review:
    uses: sciqnt/sq-constitution/.github/workflows/principle-review.yml@v1
    secrets: inherit
```

The reusable `principle-review` checks **this** repo out at the pinned ref to read
`PRINCIPLES.md` + `PRINCIPLE_REVIEW.md` — so the agent always judges against the
*same* constitution, no per-repo copy to drift. (The org `.github` repo carries the
human-facing fallback files; those are display-only and not in clones, which is why
the CI agent reads the constitution from *here* instead.)

## What's provided

| Reusable workflow | What it does |
|---|---|
| `ci.yml` | Matrix Python CI — install + run the repo's tests/conformance |
| `principle-review.yml` | The advisory principle-review agent + auto-merge-on-aligned (trusted-author, bot-PRs human-gated) |
| `claude.yml` | `@claude <instruction>` fix-mode (maintainer-triggered) |
| `issue-triage.yml` | Iterative issue triage (close / needs-info / propose-PR) + labels |
| `contract-conformance.yml` | Crater-style: run downstream conformance against a candidate contract |
| `dependency-migration.yml` | Agent migrates (not just bumps) a contract version upgrade |

## Versioning

Tagged `v1`, `v2`, … (and a moving `v1` major alias). Component repos pin
`@v1`; breaking changes to the governance contract bump the major. Strict semver on
the governance surface, same as the data contract.

## License
MIT. Governance is open; the constitution binds by consent (DCO), never by CLA.
