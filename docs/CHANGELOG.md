# Changelog

## v0.9.0 — Anti-Overcomplication & Coherence

### Simplicity Assessment (bmad-impl)

The Implementer now evaluates the architecture for overcomplication before writing code. It checks for unnecessary infrastructure, excessive dependencies, and components not traced to MVP user stories. This is an advisory check — the developer decides whether to simplify.

- **Enabled by default** — no config needed
- **Advisory only** — does not block implementation
- **Simplification decisions** are recorded in implementation notes

### Coherence & Scope Drift Check (bmad-qa)

The Quality Guardian now verifies big-picture coherence and detects scope drift during verification. It traces implemented components back to PRD user stories and checks for consistent patterns, missing integration points, and circular dependencies.

- **Enabled by default** — integrated into existing verification mode
- **Uses existing severity system**: scope drift = P1, circular dependency = P0
- **No new config options** — works with existing quality gate behavior

## v0.8.0 — Guardrails Enhancement

### Self-Verification Protocol

Fork-context roles (bmad-arch, bmad-impl, bmad-qa) now verify their output against upstream artifacts before handoff. Each role appends a **Traceability** section to its output document showing coverage of upstream requirements.

- **Enabled by default** — no action required
- **Disable per-project**: add `guardrails.self_check: false` to your `config.yaml`
- **Graceful degradation**: if upstream artifacts don't exist, self-verification is silently skipped

### validate-prd Default Changed

The greenfield workflow now defaults PRD Validation to **enabled** (previously disabled). When starting a new greenfield workflow, PRD Validation will be suggested as default-on.

- **No action required** — you can still opt out during greenfield setup
- **Existing configs preserved**: if your `config.yaml` has `validate_prd: false`, it takes precedence

### New Config Option

```yaml
# Add to your config.yaml if you want to disable self-verification
guardrails:
  self_check: false
```

### New Resource File

`plugin/resources/guardrails.md` — defines the self-verification protocol. Roles read this at runtime alongside `soul.md`.
