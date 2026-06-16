# Migration runbook — executing the component-world split

The precise, ordered playbook to split `sciqnt/sciqnt` (the monorepo) into the
component world, when the owner gives the green light. Blueprint + rationale:
`sciqnt/sciqnt → research/component-world-blueprint.md`. The governing lesson
(Terraform): **stabilize the contract first, then split.** The monorepo stays the
source-of-truth (a virtual monorepo) until each component graduates.

## Pre-flight (one-time, owner-gated)

- [ ] **Org secrets** (needs `admin:org`): `CLAUDE_CODE_OAUTH_TOKEN`, `APP_PRIVATE_KEY`
      as **org-level** secrets (visibility: all repos). `APP_ID = 4059410` (org secret).
      Until these exist, only the inherited *CI* gate works on new repos; the agent
      gates (review/claude/triage) need them.
- [ ] **`sciqnt-bot` GitHub App** installed org-wide (already installed on `sciqnt`).
- [ ] Confirm GitHub plan for org rulesets / required workflows (Enterprise Cloud).
      Fallback without it: each repo opts into the gates via its own caller workflows
      (no *required* org-ruleset gate; weaker but workable).

## Phase 0 — harden the contract IN the mono (zero-risk, no new repos)

Do this entirely inside `sciqnt/sciqnt`; nothing splits yet.

1. [ ] Make `core/sq_schema` + the conformance harness (+ `core/sq_scaffold`) a clean,
       self-contained package with a **stable, semver'd public API** and a JSON-schema
       export. This is the `sciqnt-contract` distribution — our `tfplugin5.proto`.
2. [ ] Audit every connector/lib so it imports ONLY the contract's public surface
       (not deep internals). Add a test asserting the boundary (like the
       `test_connector_decoupling` dependency-direction test).
3. [ ] Tag the mono `contract-v0.x` so downstreams have a pin point.

## Phase 1 — governance + injection (DONE, pre-green-light)

Already live: `sq-constitution` (`@v1`), `.github` (defaults + workflow-templates),
`sq-connector-template`. New repos inherit the gates automatically (CI proven green).

## Phase 2 — split the contract (the hub)

1. [ ] Create `sciqnt/sq-contract`. Move the contract package's code in (it's already
       isolated from Phase 0). Wire the inherited workflows (copy `caller-templates/`).
2. [ ] Publish `sciqnt-contract` (PyPI, semver). This is the version everything pins.
3. [ ] **Virtual-monorepo option** (recommended during transition, cf. k8s
       publishing-bot): keep the mono authoritative and *publish* `sq-contract` from it
       (a Python/uv publishing step that guarantees the standalone repo installs
       clean), so the contract has one source of truth until it fully graduates.
4. [ ] Add Renovate (the shared preset) to every dependent so a new `sciqnt-contract`
       version auto-opens a bump PR.

## Phase 3 — split the loose leaves

Order by lowest coupling to the (now-stabilizing) contract; each is a leaf consumer.

1. [ ] `sq-math`, `sq-fmt`, `sq-performance`, `sq-secrets`, `sq-price-store`, `sq-fx`,
       `sq-compute` — for each: create the repo, move its package + tests, pin
       `sciqnt-contract` where needed, copy the caller workflows, publish to PyPI.
2. [ ] `sq-tui` (the app), `sq-config`, `sq-agents`.

## Phase 4 — build the sync, THEN detonate connectors

1. [ ] In `sq-contract`, wire the **contract-conformance** reusable (crater) with the
       downstream registry (the connector repo list), so a contract change reports the
       blast radius before merge.
2. [ ] Wire **dependency-migration** in every dependent (Renovate bump PR → the
       migration agent migrates + pushes, conformance-gated, human-merged).
3. [ ] **Detonate connectors — one repo each**, from `sq-connector-template`. The
       reverse-engineered ones (`sq-degiro`, `sq-robinhood`) can go early (leaf + the
       firewall is down — org-owned + disclaimed). Replace each skeleton test with real
       `conformance.check_snapshot()` against the published contract.

## Phase 5 — the standardization core + cleanup

1. [ ] `sq-portfolio` (the anti-corruption/aggregation layer) — highest contract
       coupling, so last.
2. [ ] The mono `sciqnt/sciqnt` becomes a thin meta/umbrella (or is archived) once
       every component has graduated.

## Per-component checklist (repeat for each)

- [ ] `gh repo create sciqnt/<name>` (from `sq-connector-template` for connectors).
- [ ] Move the package + tests; set `pyproject` name `sciqnt-<name>`, pin deps by version.
- [ ] Copy the caller workflows from `sq-constitution/caller-templates/`.
- [ ] Add `NOTICE.md` (connectors), accurate `manifest.yaml` (`risk_tier`, `contract`).
- [ ] CI green (conformance against the pinned contract). Tag + publish to PyPI.
- [ ] Add to the connector registry / `contract-conformance` downstream list (connectors).
- [ ] Update `connectors-index.json` / the org profile map.

## Honest residuals (the agentic era does NOT erase)

- Community/forked connectors you don't control self-test via conformance **with lag**
  (eventually consistent, not pre-merge-verified).
- The Python/uv publishing-bot guarantee is **not off-the-shelf** — build it.
- Org rulesets need Enterprise Cloud; agent migrations are non-deterministic (contained
  by conformance + human merge); never auto-migrate the money-core — escalate.
