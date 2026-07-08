# skills

Agent skills for code quality, product copy, and deterministic verification.

## Install

```bash
npx skills add gillkyle/skills
```

Install a single skill:

```bash
npx skills add gillkyle/skills --skill verify-changes
```

## Skills

- **[env-vars-are-config](skills/env-vars-are-config/SKILL.md)** - keep environment variables focused on environment-specific configuration, and prefer CLI flags, arguments, config files, constants, or application toggles for ordinary behavior choices.
- **[quality-code](skills/quality-code/SKILL.md)** - write and review TypeScript/full-stack code with stronger domain types, derived contracts, realistic tests, useful observability, and conservative abstractions.
- **[write-better-error-messages](skills/write-better-error-messages/SKILL.md)** - review and rewrite user-facing error messages so people know what happened, what was preserved, and what to do next.
- **[verify-changes](skills/verify-changes/SKILL.md)** - choose deterministic verification for changes using linting, type checks, tests, builds, migrations, schema checks, scripts, and stable runtime probes.

## Credit

The `quality-code` and `write-better-error-messages` skills were adapted from ideas in [RhysSullivan/skills](https://github.com/RhysSullivan/skills). Thanks to Rhys Sullivan for the original skill collection.
