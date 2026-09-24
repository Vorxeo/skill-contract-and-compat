# Contract and Compatibility

Claude Code skill: `contract-and-compat`

## What

Treat APIs, MCP tools, gRPC, and CLIs as contracts. Classify breaking vs additive; never return fields that imply enforcement when mode is warn/not_enforced; keep front-door parity.

## When to use

Changing request/response schemas, tool args, CLI flags, error codes, or setting resolvers.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/contract-and-compat
cp SKILL.md ~/.claude/skills/contract-and-compat/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`fail-closed-review`, `adversarial-qa`, `migration-and-data-safety`, `verified-delivery`, `reachability-audit`, `guards-that-scan`, `handoff-faber-rigor`, `code-that-holds`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
