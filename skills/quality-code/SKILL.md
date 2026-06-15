---
name: quality-code
description: Use when writing, reviewing, or refactoring TypeScript and full-stack application code. Emphasizes clear domain models, strong type boundaries, reliable tests, observable behavior, and abstractions that fit the codebase instead of premature generalization.
---

# Quality Code

Use this skill to keep code changes small, understandable, and difficult to misuse.

## Model The Domain

Make invalid states hard to represent.

- Use narrow types for domain identifiers, validated strings, money, dates, and state-machine values.
- Prefer discriminated unions over objects with many optional fields or Boolean flags.
- Parse and validate data at system boundaries, then pass trusted values through the rest of the code.
- Reuse project-native type helpers and schema libraries before inventing new ones.

```ts
type InvoiceState =
  | { kind: "draft"; editable: true }
  | { kind: "sent"; sentAt: Date }
  | { kind: "paid"; paidAt: Date; receiptId: string };
```

## Keep Types Flowing

Derive types from the source of truth whenever possible.

- Prefer `typeof`, `ReturnType`, `Awaited`, `Pick`, `Omit`, and schema inference over hand-written duplicate shapes.
- Preserve database, API, and client types across boundaries when the stack supports it.
- Do not widen precise domain values into plain strings or generic records without a reason.
- Use object parameters for multi-field calls so call sites remain self-documenting.

```ts
type Customer = Awaited<ReturnType<typeof getCustomer>>;

function sendReceipt(input: { customer: Pick<Customer, "id" | "email">; invoiceId: string }) {
  // ...
}
```

## Test Real Behavior

Prefer tests that exercise the real code path.

- Use real parsers, routers, database adapters, queues, and storage emulators when they are practical to run locally or in CI.
- Mock external services only at trust boundaries where a stable test environment is unavailable.
- Cover the failure path for risky changes, not only the happy path.
- Add regression tests when fixing a bug whose cause can be reproduced deterministically.

## Make Failures Observable

When behavior may fail in production, leave enough evidence to debug it.

- Add structured logs, spans, counters, or events at meaningful boundaries.
- Include stable identifiers such as request IDs, job IDs, account IDs, or integration names.
- Avoid logging secrets, raw tokens, payment details, or unnecessary personal data.
- Prefer OpenTelemetry or the project's existing observability stack over ad hoc print statements.

## Choose Boring Abstractions

Add abstractions only when they reduce real complexity.

- Follow established local patterns before introducing a new helper, framework, or layer.
- Extract repeated code when the duplication is meaningful and stable, not just visually similar.
- Keep one-off business rules near the feature that owns them.
- Name functions after domain outcomes, not implementation mechanics.

## Review Checklist

Before finishing, check:

- Can invalid input or impossible state still reach core logic?
- Are important types derived from existing sources of truth?
- Does the change have deterministic coverage for the behavior it changes?
- Will a production failure leave enough signal to diagnose it?
- Did the change fit the surrounding code instead of forcing a new style?
