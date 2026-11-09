# Blockchain Protocol and Client Profiles

#### When to use this file

Use this file before every blockchain-client audit, including familiar protocols and clients, and revalidate it when relevant protocol rules, source revisions, build flags, or deployment paths change. It produces a source-grounded protocol profile and client profile, then composes [BLOCKCHAIN-CLIENTS.md](BLOCKCHAIN-CLIENTS.md)'s core discipline with only the capability blocks the source establishes.

## Profile discipline

- Derive facts from repository source, tests, fixtures, build metadata, locally available specifications, pinned references, vectors, or executable models. Record the exact path, revision, and line for each consequential fact.
- Keep protocol rules separate from client behavior: identify what must be true for consensus, what is local admission or operator policy, and what is merely an implementation choice.
- A reference client is one possible source basis, not a universal fixed implementation. Differential agreement is useful corroboration; disagreement needs the governing rule and a security consequence before it is a finding.
- When specifications, references, vectors, or models conflict, identify the authoritative basis and revision or record the exact blocker; do not resolve the conflict by assumed majority or client popularity.
- State unknowns explicitly. Mark an absent mechanism `not_applicable` with its source basis; reserve `unknown` for facts the evidence does not establish. Do not invent a chain, quorum, clock, ordering, failure, accounting, trust-root, or upgrade default to fill a blank profile.
- Select capability blocks independently and compose them when present (for example, stake-based proposer selection can coexist with BFT finality). Do not add a block because a dependency name or generic blockchain term appears.

## Source-grounded reconnaissance prompt

Use the existing four baseline reconnaissance agents to gather these facts in their normal assignments; do not create an extra mandatory profile agent. Each agent returns source facts and repository-relative paths/line numbers, plus explicit unknowns. A focused agent is optional only for a materially distinct subsystem and only if the existing budget gate permits it.

```text
Read the target and locally available protocol/client references at <target>. Do not use network access or run target code. Build two profile fragments from source evidence, not assumptions.

Protocol fragment: identify protocol/version and source of truth; roles and trust boundaries; fault model and membership/weight rules; state-validity and canonicalization rules; ordering, replay, fork-choice and finality; failure, rollback, crash and recovery semantics; asset/accounting and issuance/fee rules; upgrade, activation and mixed-version behavior; and trust roots/checkpoints/keys. Mark each unknown and cite path:line.

Client fragment: identify client role and revision; state, consensus, signing/proof, peer/sync, storage/recovery, pruning/snapshot, and upgrade paths; exact source paths; offline test and fixture commands; build flags/toolchain requirements; expected write locations; and locally selectable network or deployment modes. Do not run commands during reconnaissance. Mark missing sandbox or runtime facts and cite path:line.

Return only source-grounded facts, exact commands that a later bounded local check could use, and explicit unknowns. Distinguish consensus rules from local admission and implementation choices. Do not infer a protocol default, network-wide reachability, or security impact from a local mismatch.
```

Agent focus remains compatible with [RECONNAISSANCE.md](RECONNAISSANCE.md):

- Product/stack and local operation should collect client role, revision, paths, tests, fixtures, commands, and build flags.
- Principals/authority should collect protocol roles, membership, fault assumptions, trust roots, finality, and upgrade authority.
- Entry surfaces/copies/sinks should collect wire objects, state transitions, ordering, signatures/proofs, and accounting effects.
- Local execution/deployment visibility should collect recovery, pruning, snapshots, mixed-version paths, sandbox limits, and facts that remain external.

## Protocol profile template

Keep unresolved values as `unknown: <fact needed>`; do not replace them with a presumed chain default.

```text
Protocol profile
- protocol / version:
- source of truth and pinned reference basis:
- roles, authority, and trust roots:
- fault model, membership, weights, and thresholds:
- state-validity, canonicalization, and signing/proof rules:
- ordering, replay, fork choice, and finality:
- failure, rollback, crash, recovery, pruning, and snapshot rules:
- asset, issuance, fee, reward, and accounting rules:
- upgrade, activation, downgrade, and mixed-version rules:
- unresolved facts and safe owner/source checks:
```

## Client profile template

```text
Client profile
- client / role / revision:
- source paths for state, consensus, signing/proofs, peer/sync, and persistence:
- tests, fixtures, and exact offline commands:
- build flags, toolchain, generated inputs, and expected write locations:
- selectable network, deployment, and version modes:
- crash/recovery, pruning/snapshot, replay, and migration paths:
- local admission, API, indexing, and operator-policy differences:
- unresolved runtime/deployment facts and safe validation plan:
```

## Compose and retain the resolved profile

For each blockchain agent prompt, include the `Core discipline`, applicable shared attack-class blocks, selected capability blocks, `Universal moves`, and `Validation rules` from [BLOCKCHAIN-CLIENTS.md](BLOCKCHAIN-CLIENTS.md). Add selected ordinary blocks from [ATTACK-CLASSES.md](ATTACK-CLASSES.md) and other companions using the composition rules in [HUNTING.md](HUNTING.md), then the relevant protocol and client profile fragments. Keep exact block names so coverage-ledger references remain stable. Revalidate the profile from current source even for a familiar client; unknown facts remain unknown.

In full audit mode, the parent stores the resolved profiles as the run artifact `<output-dir>/blockchain-profiles.md` alongside the run's other retained artifacts. In guidance mode, keep the profile inline and do not create a file unless the user asks for one. Profile capture does not change inherited sandbox, write-isolation, budget, or no-live-network controls.

## Conditional `blockchain` evidence

Every blockchain companion-unit finding record retains a `blockchain` object. Generic findings outside such a unit may omit it. Use exactly these common fields; do not add chain-specific keys:

```text
protocol: string
rules: string
invariant: string
attacker_capabilities: non-empty array of strings
assumptions: array of strings (may be empty)
reference_basis: non-empty array of pinned reference strings
```

For a `confirmed` record, the same object also requires exactly these additional fields:

```text
network: string
client_revision: string
initial_state: string
inputs: ordered non-empty array of strings (exact raw/encoded input or replayable fixture references; preserve order and duplicates)
expected_result: string
observed_result: string
impact_scope: string
```

The confirmed fields must describe one bounded reproducible state transition, proof, consensus, accounting, or resource result. `client_revision` identifies every participating target and comparison client by name and exact revision, with relevant build flags. Describe the network parameters in `network`, or explicitly identify a network-independent harness; likewise state when a check needs no ledger state rather than inventing one.

Preserve repeated messages and schedule order in `inputs`; never deduplicate a replay or schedule fixture. State the encoding of binary inputs or the exact fixture reference and replay command in `execution.instructions`. Identify existing repository fixtures by pinned path/revision and generated fixtures by independently checked, parent-promoted artifact; include content digests where needed to identify them. A vague fixture label is not reproducible evidence. Raw strings, including empty or malformed inputs, remain valid native test data. `execution.observed_result` and `blockchain.observed_result` must describe the same observation; the verifier resolves any inconsistency before confirmation. `reference_basis` may identify a pinned specification, reference revision, vector, or executable model; it is not permission to assume that implementation is correct.

For `needs_validation` and `rejected`, the `blockchain` object still carries all six common required fields. Preserve known context, put precise unresolved portions in `rules` and `assumptions`, and keep the verdict's exact top-level `blockers` and validation plan, or `reason`, respectively; never fabricate a value. Optional experiment fields (`network`, `client_revision`, `initial_state`, `inputs`, `expected_result`, `observed_result`, `impact_scope`) may be included when known, while `confirmed` requires all of them; any `inputs` supplied here remains an ordered non-empty string array with duplicates preserved. If no source-grounded invariant or reference basis exists, retain the unit as `blocked` rather than creating an unsupported candidate. A compatibility or reliability mismatch without a security boundary receives no new finding verdict or severity; retain it as a hunter `observations` entry `{kind: conformance|reliability, coverage_id: assigned ID, description: source-grounded note citing source/checks}`.

## Bounded evidence rules

Use dummy keys, accounts, validators, peers, states, and fixtures. Run only local, bounded checks under the inherited sandbox and promotion controls; never contact a live network, public RPC, deployed validator, bridge, or shared service. If a decisive fact is external, state the exact blocker and owner-observed or safe local plan. One-node observation does not establish a network-wide deployed effect; a bounded multi-node or model counterexample can demonstrate a safety/liveness violation under explicit fault and topology assumptions, while live deployment impact remains conditional.

Before finalizing a protocol-specific record, verify the profile's rule, client path, exact fixture, expected/observed result, and affected scope agree. A source mismatch or test-vector disagreement alone is not a consensus split; establish the rule and the concrete security outcome.
