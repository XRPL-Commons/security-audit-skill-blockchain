# Blockchain Clients and Protocol Implementations Hunting

#### When to use this file

Use this file for nodes, validators, full or archival clients, light clients, relayers, bridges, wallets that sign protocol objects, indexers, and other software whose inputs can affect protocol state, consensus, proofs, or asset accounting. Read [BLOCKCHAIN-PROFILES.md](BLOCKCHAIN-PROFILES.md) first when the protocol or client is unfamiliar. Add the existing binary, messaging, and availability companions only when their source-visible boundaries are present.

Split large targets by state transition, signing/proof verification, consensus, peer and synchronization paths, persistence/recovery, and upgrade boundaries. Select capability blocks below from the resolved protocol profile; capabilities compose.

## Core discipline (include in every agent prompt for this domain)

```
- Start with a source-grounded protocol profile. Name the pinned specification, reference, test vector, or executable model that defines the rule; a client is not a universal oracle.
- Separate consensus validity from local admission, mempool, indexing, API, and implementation choices. A difference outside the protocol rule is not a consensus split.
- For every claim, trace an untrusted transaction, peer, proposer, signer, proof, or snapshot through validation to the accepted state, authority decision, persisted state, or resource effect.
- State the protocol roles, membership, fault assumptions, thresholds, ordering, finality, and version/activation scope before calling a behavior unsafe. Do not infer defaults.
- Canonical bytes, signatures, proofs, and hashes must bind the intended object, context, key or role, version, and commitment; encoding validity alone is not authorization.
- Account for exact asset units, issuance, fees, rewards, burns, and rounding. Failed-operation effects follow the profile; do not import a generic transaction invariant.
- Bound any network or economic claim to the attacker capability and topology shown by source or a bounded fixture. A local failure does not establish a network-wide outcome.
- Require only profile-defined guarantees: do not impose local-pool equality, fork absence, a fixed quorum, all-or-nothing failed operations, arbitrary reorder invariance, or synchronized wall clocks.
- Keep decisive checks local and bounded. Differential agreement is corroboration, not proof, and a mismatch is a lead until the governing rule and security effect are established.
```

## State and cryptographic attack classes (subagent_type: `general`)

**State transition validity and determinism**
Find acceptance of an invalid state/input, rejection of a valid one, or divergent results for the same protocol-defined input. Record the initial state, exact input bytes, protocol context, canonical ordering, arithmetic, and expected result before running a bounded fixture. Distinguish required ordering from scheduler or container order; do not require a stronger determinism property than the profile defines.

**Canonical signing, proof, and hash binding**
Trace serialization, domain/network or chain identifiers, key and role selection, signature verification, proof paths, commitment roots, and version handling. Look for omitted fields, ambiguous encodings, malleability, replay across contexts, and proofs that authenticate a different object or state. A byte difference is not a vulnerability without a source-grounded rule and meaningful authority or state effect.

**Asset conservation and accounting**
Follow every debit, credit, mint, burn, fee, reward, escrow, withdrawal, and asset identifier through exact-unit arithmetic. Check sign, range, precision, rounding, ownership, and issuance authority at the final state transition. Establish the protocol's defined behavior for rejected or partially executed operations rather than assuming all effects disappear.

## Consensus, network, and lifecycle attack classes (subagent_type: `general`)

**Consensus safety and liveness under stated faults**
Map proposals, votes, locks, rounds or views, timeouts, leader changes, thresholds, membership snapshots, equivocation handling, and finality. Test only the safety and liveness claims actually made by the profile and its fault model. A local disagreement or unusual fork is not enough without the protocol condition and bounded effect.

**Malicious peer, synchronization, and fork-choice behavior**
Follow handshake identity, advertised headers or blocks, locators, proofs, checkpoints, peer scoring, invalid-data handling, reorg or fork choice, and recovery from an eclipsed or stale view. Name the malicious peer's permitted capability and show the affected state, authority, peer set, or resource; do not extrapolate a local peer result to the whole network.

**Resource amplification and operator cost**
Trace attacker-controlled bytes, transactions, proofs, fan-out, retries, work factors, storage, bandwidth, fees, and cache behavior to CPU, memory, disk, connections, or operator spend. Compare work and cost to source-visible limits and the protocol's economic controls. Use the availability companion when the result is a shared-service exhaustion path.

**Crash, recovery, pruning, and snapshot trust**
Review atomicity across state, indexes, journals, caches, and consensus metadata at crashes and restarts. Check replay, rollback, pruning, checkpoint or snapshot import, retention boundaries, and trust-root validation for stale, partial, or attacker-selected data. Confirm the resulting state or authority decision with a bounded local fixture.

**Upgrade, mixed-version, and feature-activation boundaries**
Trace version negotiation, feature flags, serialization changes, activation height/epoch, migration, rollback, downgrade, replay, and mixed-client behavior. A rule-changing fix requires coordinated activation and compatible validation; a compatibility or reliability mismatch without a security boundary is an `observations` ledger note, not a finding or severity.

## Capability blocks (select only when the profile says the capability is present)

**UTXO ownership and spend lifecycle**
Review output identifiers, input uniqueness, script or witness authorization, locktime and maturity, change, coinbase-like issuance, and spentness across validation, indexing, reorg, and recovery. Do not apply account or nonce assumptions to an output-based protocol.

**Account and VM execution**
Review account identity, nonce or replay protection, call context, authorization, gas or resource accounting, storage writes, nested calls, and failure/rollback semantics. Bind VM host functions and state roots to the protocol's exact execution and version rules.

**Chainwork and fork choice**
Where cumulative work, weight, checkpoints, or a comparable score selects history, trace its arithmetic, comparison, tie break, minimum-work policy, and interaction with finalized state. Use the profile's rule rather than assuming the heaviest or longest chain is authoritative.

**Weighted, quorum, and federated membership**
Review validator/member identity, weights, threshold calculation, uniqueness, membership snapshots, quorum changes, key rotation, and trust-list or federation updates. Record the actual fault and membership assumptions; never import a fixed threshold.

**Proposer selection, stake, slashing, and validator signing**
Review eligibility, randomness, stake/lockup, delegation, proposer and vote signing, equivocation evidence, slashing, rewards, withdrawals, and key rotation. PoS and BFT properties may both apply; select this block independently from consensus membership and finality blocks.

**Light-client proofs and verification**
Review header or state proofs, commitment roots, path completeness, update/replay windows, trust anchors, client checkpoints, and the boundary between verified and merely relayed data. Establish exactly what a light client is allowed to conclude without a full state view.

**Data availability and sampling**
Where a protocol separates validity from data availability, review erasure commitments, sampling, withholding detection, recovery, deadlines, and the consequence of unavailable data. Do not treat a valid commitment as proof that data is retrievable unless the profile says so.

**Rollup settlement and cross-domain finality**
Review batch ordering, inbox/outbox or bridge messages, proof/challenge windows, state roots, withdrawal authorization, replay domains, finality dependencies, and upgrade/admin paths. Bind every cross-domain effect to the exact source and destination domain and settlement rule.

## Universal moves (apply across the above)

- Build a protocol profile before judging behavior, then map input or peer message → validation → state/consensus decision → persistence or external effect.
- Use small initial states and exact fixtures that exercise valid, invalid, duplicate, replayed, reordered, boundary, and versioned cases only where the protocol defines those cases.
- Compare all in-repository clients, serializers, execution modes, recovery paths, and version gates. Treat disagreement as a lead until the governing rule and security consequence are proven.
- Re-check normal processing, reorg or rollback, restart, pruning/snapshot restore, synchronization, and mixed-version paths for the same invariant.
- For network-wide claims, state the topology, identities, stake/weight, resource, and adversary bounds that make the effect reachable; otherwise keep it local or unresolved.

## Validation rules (apply before reporting ANY finding here)

1. Cite the source rule and pinned reference basis, then provide a complete source trace from lower-trust input or authority to the state, consensus, proof, or resource effect.
2. For state or determinism claims, provide an exact initial state and input fixture, expected protocol result, observed result, and the bounded execution conditions that make the comparison meaningful.
3. For signing, proof, or hash claims, identify the canonical bytes, context/domain, key or role, commitment, and failure path that produces the unauthorized result.
4. For consensus, membership, fork-choice, or finality claims, state roles, fault model, threshold, ordering, version, and finality assumptions. Do not substitute a generic chain rule.
5. For accounting claims, show exact units and state deltas, including issuance, fees, rewards, burns, rounding, and the profile-defined failure behavior.
6. For peer, sync, cost, recovery, or upgrade claims, name the attacker capability and bounded affected scope. Local evidence cannot by itself prove a network-wide security effect.
7. Return `confirmed` only with source-complete evidence and a bounded meaningful result. Use `needs_validation` only for an exact external or unavailable fact, with a safe local or owner-observed plan.
8. Do not assign a finding or severity to a compatibility or reliability mismatch that has no demonstrated security boundary; record it as a hunter `observations` entry `{kind: conformance|reliability, coverage_id: assigned ID, description: source-grounded note citing source/checks}`.
