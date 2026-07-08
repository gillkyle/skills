---
name: env-vars-are-config
description: Use when adding, reviewing, or changing environment variables, .env files, process.env access, deployment secrets, runtime configuration, feature flags, CLI flags, command arguments, boolean toggles, or setup docs. Encourages keeping env vars for configuration that genuinely varies across environments, and using CLI flags, arguments, config files, code-level constants, or application toggles for ordinary behavior choices.
---

# Env Vars Are Config

Use environment variables for values that configure the same software differently across environments. Do not add env vars as a casual bucket for every boolean, behavior switch, local preference, or one-off mode.

## Decision Rule

Before adding or preserving an env var, ask:

- Does this value need to differ between dev, test, staging, preview, production, CI, or deployments?
- Is it secret or infrastructure-provided?
- Would operators reasonably set it outside the codebase?
- Would changing it without a code change be an operational need, not just a developer convenience?

If the answer is mostly no, prefer a non-env mechanism.

## Prefer These Instead

- Use CLI flags for command behavior: `--dry-run`, `--force`, `--limit 20`, `--verbose`.
- Use command arguments for explicit inputs: file paths, IDs, dates, output formats.
- Use config files for named project settings that should be reviewed with the repo.
- Use code constants for stable product behavior and defaults.
- Use typed application settings or feature-flag systems for runtime product toggles.
- Use test helpers or fixture options for test-only behavior.

## Good Env Vars

- Secrets and credentials: API keys, tokens, passwords, signing secrets.
- Environment-specific service locations: database URLs, queue URLs, hostnames, ports, buckets, regions.
- Deployment/runtime metadata supplied by the platform: commit SHA, build ID, service name, CI markers.
- Operational config that really changes by environment: log sink, telemetry endpoint, allowed origin, external callback URL.

## Env Var Smells

- A boolean that only changes how a script behaves once.
- A feature switch that should be owned by product logic or a feature-flag system.
- A value that every environment sets to the same thing.
- A hidden alternate code path that is hard to discover from help text or docs.
- A test escape hatch that should be a test helper, fixture option, or explicit argument.
- A local developer preference that belongs in a local config file or shell alias.

## Review Checklist

- Challenge new env vars and ask what environment boundary they represent.
- Prefer explicit flags/arguments for scripts and CLIs; make `--help` reveal behavior.
- Keep env var names narrow and operational when they are justified.
- Document required env vars in the existing project convention.
- Avoid adding boolean env vars unless they are truly deployment-level switches.
- When replacing an env var, preserve compatibility only when callers may already depend on it, and document the migration.

## Examples

Prefer:

```bash
script --dry-run --limit 100
```

Over:

```bash
DRY_RUN=true LIMIT=100 script
```

Prefer:

```ts
const ENABLE_FAST_RETRY = true;
```

Over:

```ts
const ENABLE_FAST_RETRY = process.env.ENABLE_FAST_RETRY === "true";
```
