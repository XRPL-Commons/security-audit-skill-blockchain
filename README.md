# Blockchain Security Audit Skill

A coding-agent skill for security reviews of blockchain clients and protocols, with a shared review method, selectable capability prompts, and source-grounded protocol and client profiles. It also retains the original audit support for services, APIs, libraries, and other software.

This repository extends [Cloudflare's security-audit skill](https://github.com/cloudflare/security-audit-skill), the starting point for the vulnerability discovery harness described in [Build your own vulnerability harness](https://blog.cloudflare.com/build-your-own-vulnerability-harness). The blockchain extensions are maintained here by XRPL Commons and apply across blockchain protocols; they are not specific to XRPL. The upstream MIT license and attribution are preserved.

## Blockchain coverage

The shared prompts cover state-transition validity, canonical encoding and cryptographic binding, accounting, consensus safety and progress, adversarial networking and synchronization, resource amplification, persistence/recovery, and protocol upgrades.

Compose only the capabilities present in the target: UTXO or account/VM execution, chainwork, voting and membership, proposer/signing responsibilities, proof verification, data availability, light clients, and settlement. Capabilities may overlap. The protocol profile supplies actual validity, ordering, finality, fault and activation rules; the client profile maps them to source, build configurations and local tests. Both are derived from pinned, available evidence for the target instead of a built-in list of chains or a universal reference client.

Independent clients accepting and rejecting the same input can be a meaningful disagreement. Temporary forks, differing mempools, permitted failure effects, and local policy differences are not automatically vulnerabilities. Confirmed findings require a violated security invariant and an observed consequence under the stated protocol assumptions.

## What it does

The skill runs a structured audit in six phases:

1. **Reconnaissance** -- map architecture, trust boundaries, input surfaces, prior evidence, and deterministic coverage in `architecture.md` and `coverage-ledger.json`.
2. **Coverage-led hunting** -- assign isolated hunters from ledger units, record their checks, and use coverage critics to find gaps.
3. **Candidate validation** -- give every unique candidate to a fresh verifier that tries to disprove it.
4. **Structured output** -- write `confirmed`, `needs_validation`, and `rejected` records to `findings.json` and validate them against `report-schema.json`.
5. **Independent record verification** -- fresh agents verify final source claims. Material replacements receive another independent verifier.
6. **Target-neutral reporting** -- derive `REPORT.md`, `FINDINGS-DETAIL.md`, and `NEEDS-VALIDATION.md` from the verified records and coverage ledger.

The parent runs `validate-coverage-ledger.cjs` after creating the ledger and after each later ledger update. It runs `validate-findings.cjs` in Phase 4 and again after every Phase 5 replacement.

The verdicts are distinct: `confirmed` has a complete source trace and bounded observed result, `needs_validation` has an exact unresolved fact and no severity, and `rejected` records a disproved candidate.

Multiple runs against the same repo are additive. The skill uses prior ledgers and findings to target gaps, revalidate changed source, and carry forward current-source evidence without treating stale or unresolved work as covered.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Setup, core principles, platform terminology, workflow overview, and audit anti-patterns |
| `RECONNAISSANCE.md` | Phase 1 reconnaissance prompts and synthesis instructions |
| `HUNTING.md` | Phase 2 orchestration, hunting methodology, and validation rules |
| `ATTACK-CLASSES.md` | Core, wildcard, and obvious-things attack prompts |
| `BLOCKCHAIN-CLIENTS.md` | Shared blockchain invariants, selectable capability prompts, and bounded validation rules |
| `BLOCKCHAIN-PROFILES.md` | Source-grounded protocol/client discovery prompts and structured evidence contract |
| `MEMORY-SAFETY-AND-BINARY.md` | Memory-safety, binary, and kernel hunting classes for native targets |
| `AI-AND-LLM.md` | Prompt-injection, agent/tool, and output-handling hunting classes for LLM-backed targets |
| `WEB-PROTOCOL-AND-AUTH.md` | HTTP request-framing, cache, and authentication-protocol hunting classes for HTTP-protocol and auth targets |
| `CLIENT-SIDE.md` | DOM-injection, messaging-trust, UI-redress, and prototype-pollution hunting classes for client-side/browser targets |
| `SUPPLY-CHAIN-AND-RELEASE.md` | Dependency, CI, release, signing, update, plugin, and extension hunting classes |
| `CLOUD-AND-DEPLOYMENT.md` | IAM, infrastructure-as-code, container, serverless, ingress, and runtime-configuration hunting classes |
| `PROTOCOLS-RPC-AND-MESSAGING.md` | RPC, serialization, queue, broker, webhook, and streaming-protocol hunting classes |
| `RESOURCE-EXHAUSTION-AND-AVAILABILITY.md` | Shared resource, quota, queue, worker, and operator-spend hunting classes |
| `DATA-ISOLATION-AND-LIFECYCLE.md` | Tenant isolation, cache, search, export, backup, migration, deletion, and restore hunting classes |
| `DESKTOP-MOBILE-AND-LOCAL-IPC.md` | Native app, deep-link, webview, exported-component, helper, daemon, and local-IPC hunting classes |
| `VALIDATION-AND-REPORTING.md` | Phases 3–6 candidate validation, structured output, record verification, and reporting |
| `report-schema.json` | JSON schema for all three `findings.json` verdicts |
| `validate-findings.cjs` | Zero-dependency validator for `findings.json` in Phases 4 and 5 |
| `validate-findings.test.cjs` | Findings-validator tests and producer-compatible fixture checks |
| `validate-coverage-ledger.cjs` | Zero-dependency validator for `coverage-ledger.json` in Phases 1–5 |
| `validate-coverage-ledger.test.cjs` | Coverage-ledger validator tests |

## Installation

Install the skill with the [Skills CLI](https://skills.sh):

```bash
npx skills add https://github.com/XRPL-Commons/security-audit-skill-blockchain \
  --skill security-audit
```

Use `--global` for a user-level installation:

```bash
npx skills add https://github.com/XRPL-Commons/security-audit-skill-blockchain \
  --skill security-audit \
  --global
```

Run `npx skills --help` for agent-selection and non-interactive options.

The skill retains the name `security-audit`; installing this fork updates that skill rather than introducing a second invocation name.

## Usage

Start your coding agent in (or pointed at) the codebase you want to audit, then ask it to do a security audit:

```
security audit this codebase
```

```
find security vulnerabilities in ./src
```

```
do a security review, output to ~/audits/my-project
```

For a focused protocol review:

```
Review this client's block validation and crash recovery. Derive the protocol rules
from the pinned specification and local references, and distinguish consensus
validity from mempool policy. Keep any unproven network-wide impact explicit.
```

The skill activates automatically when the request matches its trigger (security audit, find vulnerabilities, pen-test the code, etc.). A direct codebase audit or pen-test request uses full audit mode. Security questions and focused vulnerability work use guidance mode unless you request report artifacts. In full audit mode, an unspecified output directory defaults to `~/security-audit-skill/<repo-name>/run-<N>`. The workflow writes inside the target repository only when you explicitly select a directory that version control ignores.

## Requirements

- A coding agent with a model that supports tool use and parallel sub-agents
- Node.js for the zero-dependency findings and coverage-ledger validators
- An OS-enforced sandbox for target-controlled builds, tests, processes, browsers, emulators, fuzzers, and fixtures. It must disable external networking, use a sanitized allowlisted environment, enforce resource limits, and allow writes only to assigned scratch paths. Without these controls, the workflow keeps the lead as `needs_validation` instead of executing target code.

## Design principles

- **Only confirm established boundary failures.** Keep a source-grounded blocked lead as `needs_validation` with its exact unresolved fact.
- **Adversarial validation.** The agent that checks a finding is never the agent that found it.
- **Severity requires impact.** Likelihood x impact, not deviation from a checklist.
- **Defense-in-depth gaps are not vulnerabilities.** If Layer A prevents the attack, the absence of Layer B is a hardening note.
- **Multiple runs can improve coverage.** Carry forward source-grounded evidence and revisit gaps. No blockchain detection-rate claim is made for these prompts.

## Validation

Run the dependency-free validators' regression suites from the repository root:

```bash
node --test skills/security-audit/*.test.cjs
```

The findings format remains compatible with generic reports. Blockchain-unit records additionally use a structured `blockchain` context; confirmed records require reproducible initial state, inputs, rules, expected/observed results and impact scope. Schema validation establishes format, not consensus correctness.

## Contact

Report issues with this extension in [this repository](https://github.com/XRPL-Commons/security-audit-skill-blockchain/issues). Upstream security research and the original skill are maintained by Cloudflare.

## License

MIT -- see [LICENSE](LICENSE).
