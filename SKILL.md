---
name: contract-and-compat
description: >-
  Use when changing API, MCP, gRPC, or CLI contracts. Classify breaking vs
  additive; version and deprecate honestly; never return fields that imply
  enforcement when not enforced (not_enforced); grep all surfaces when changing
  a setting resolver; keep front-door parity. Client-visible lies are P0.
---
# Contract and Compatibility

APIs, MCP tools, gRPC services, and CLIs are contracts with clients. A governance product that lies about enforcement is worse than one that admits warn-mode.

## When this skill applies

- Any change to request/response schemas, tool args, CLI flags, error codes, or setting resolvers.
- Adding `not_enforced` / shadow / warn modes.
- Renaming or splitting authz-related fields.
- Before merge: pair with `fail-closed-review` and `verified-delivery`.

## Breaking vs additive

| Kind | Examples | Rule |
|------|----------|------|
| **Additive** | New optional field; new endpoint; new error code alongside old; new MCP tool | Safe if default preserves old behaviour |
| **Breaking** | Remove/rename field; change type/meaning; require new field; flip default to deny without version; change auth subject source | Needs version bump, dual-run, or coordinated deprecation |

**Failure mode silent-break**: ship breaking change on same version; clients fail closed or, worse, fail open on parse errors.

### Versioning and deprecation

1. Prefer additive on current major; put breaks behind new version or explicit flag.
2. Deprecation: announce → dual-support → remove after documented date.
3. Document migration in CHANGELOG / MCP tool description / `--help`.
4. Do not recycle field names with new semantics (semantic break disguised as rename).

## Enforcement honesty (P0)

**Never** return fields that imply enforcement when the control is not enforced.

Forbidden patterns:

- `blocked: true` / `decision: deny` while mode is `warn` / `shadow` / `not_enforced`
- `policy_applied: true` when policy was skipped
- Success body that looks like a hard gate after a soft check

Required:

- Expose mode truthfully: e.g. `enforcement: "warn" | "enforced" | "off"`
- In warn: report `would_deny` (or equivalent) **without** claiming blocked
- Client-visible lie = **P0** (`fail-closed-review`)

**Failure mode enforcement-lie**: dashboard green "protected" while flag is off.

## Grep all surfaces when changing a setting resolver

If you change how a setting is resolved (env, DB, header, org default):

1. Grep every call site: API, MCP, gRPC, CLI, workers, admin UI, tests.
2. List surfaces in the delivery receipt.
3. Ensure identical semantics (or document intentional differences with tests).

**Failure mode resolver-drift**: HTTP uses new resolver; MCP still uses old default → second-door / parity bug.

## Front-door parity

Same capability on multiple doors (REST + MCP + CLI + job) must share:

- Authn subject source
- Authz gate
- Enforcement mode honesty
- Validation of required fields (no omit-to-bypass on one door only)

**Failure mode front-door-only**: harden REST; leave MCP tool open.

Use **reachability-audit** to enumerate doors before claiming parity.

## Error and status contracts

- Stable machine-readable codes for deny vs validate vs unavailable.
- Do not map authz deny → 500 (hides gate; breaks client handling).
- Do not map gate failure → 200 with empty allow (except-as-allow).

## Checklist for contract PRs

- [ ] Classified: additive | breaking
- [ ] If breaking: version / deprecation plan written
- [ ] No enforcement-lie fields in warn/off modes
- [ ] Resolver change: all surfaces grepped and listed
- [ ] Front-door parity: doors listed; gates shared or exceptions documented
- [ ] Tests cover old client behaviour still works (additive) or fail loudly (breaking intentionally)
- [ ] Verified delivery receipt attached

## Failure modes (named)

| Name | Meaning |
|------|---------|
| **silent-break** | Breaking change, same version |
| **enforcement-lie** | Response implies enforced when not |
| **resolver-drift** | Surfaces disagree on setting meaning |
| **front-door-only** | One surface hardened, twins not |
| **semantic-reuse** | Same field name, new meaning |
| **deny-as-500** | Authz deny reported as server error |
| **optional-break** | "Optional" field now required without version |

## Interaction with other skills

- **fail-closed-review**: P0 on enforcement-lie; second-door checklist.
- **adversarial-qa**: omit optional fields; inverse routes; body authority across surfaces.
- **migration-and-data-safety**: warn→enforced flag honesty in APIs.
- **verified-delivery**: contract tests must be Verified, not Written.
- **handoff-faber-rigor**: Faber lists surfaces touched; Rigor refuses vague "API updated".
- **guards-that-scan** / **code-that-holds**: prefer typed contracts over stringly JSON hope.

## Anti-patterns

- "It's just an internal MCP tool" — still a contract; still a door.
- Returning `compliant: true` from a dry-run.
- Changing default from allow → deny in a patch release without changelog and dual-support.
- Documenting one schema while generating another from code without a check.
