---
name: verify-changes
description: Use after making code, configuration, data, or documentation changes when deciding how to verify the work. Prioritizes deterministic checks such as static analysis, formatting, linting, type checking, unit tests, integration tests, build commands, migrations, schema checks, and deterministic scripts before relying on manual inspection.
---

# Verify Changes

Use this skill to choose verification that proves the changed behavior as directly and deterministically as possible.

## Verification Hierarchy

Prefer checks in this order, stopping only when the remaining risk is low or the next check is not practical:

1. Static structure: parse, format, lint, dependency graph checks, generated-code checks.
2. Type and schema checks: TypeScript, database schemas, GraphQL/OpenAPI contracts, migrations, config validation.
3. Focused tests: unit or component tests that cover the changed branch.
4. Integration tests: real adapters, services, queues, database, routing, auth, or API boundaries when the change crosses them.
5. Build/package checks: production build, bundle validation, asset generation, CLI packaging.
6. Deterministic runtime probes: scripts, smoke tests, seeded flows, curl requests, replay tools, browser automation with stable assertions.
7. Manual inspection: only for visual, copy, UX, or exploratory behavior that deterministic tools cannot fully prove.

## Start From The Diff

Before choosing commands:

- Identify the files and modules changed.
- Identify the user-visible or system-visible behavior that should now differ.
- Find existing package scripts, test files, CI config, Make targets, task runners, and repo docs.
- Prefer the narrowest check that exercises the changed path, then add broader checks if the blast radius is shared or uncertain.
- Do not invent a command when a project-specific command already exists.

## Match Check To Risk

Use focused verification for narrow, local edits:

- Formatting-only change: formatter or snapshot-stable diff check.
- Type-only change: typecheck plus any affected generated-contract check.
- Pure function change: unit tests for changed branches.
- Component change: component test, story test, or browser assertion for the affected state.
- API or service change: route-level test, integration test, seeded script, or direct request against a local server.
- Migration or schema change: migration dry run, schema validation, generated types, and affected query tests.
- Build config change: production build and any package/export smoke test.

Escalate to broader checks when:

- The change touches shared utilities, auth, payments, data migrations, routing, build tooling, serialization, caching, permissions, or cross-package contracts.
- A focused check does not exist.
- The code path is easy to break silently.
- Prior failures suggest local runtime drift, stale generated files, or environment-specific behavior.

## Verification Plan Format

State the plan before running expensive checks:

```markdown
Verification plan:
- `command`: proves ...
- `command`: proves ...
- Manual/browser check: only for ...
```

If a command is skipped, say why:

```markdown
Skipped `pnpm test`: broad suite has known unrelated failures; ran focused tests instead.
```

## Interpret Results

- Treat failing deterministic checks as work, not as noise, until the failure is explained.
- Separate failures caused by the change from known unrelated failures with evidence.
- If a command cannot run because of missing services, credentials, packages, or runtime state, try the nearest deterministic fallback.
- If generated files are stale, run the generator and verify the generated diff.
- If a local server is required, prove that the server corresponds to the current checkout before trusting browser or curl results.

## Completion Standard

Finish with:

- The exact commands or scripts run.
- Whether each passed, failed, or was skipped.
- The behavior each check covered.
- Any residual risk that deterministic checks did not cover.

Do not claim full verification from a build alone when the changed behavior needs tests or runtime assertions.
