# Spec Template (Feature_Spec_Template v1.1)

Sections 1-15 are the established one-pass build standard. v1.1 adds Appendix A (claims ledger) and Sections 16-18 (build order, decision log, build notes). Fill every section; if one does not apply, write "N/A: [reason]" rather than omitting it. Mark each section REQUIRED/OPTIONAL as below.

Header block (above Section 1): feature name, board/issue item, version, date, author, review state, "builds on" list. The version line records what each version folded in and from which review.

---

## SECTION 1 — Feature Overview

- 1.1 Feature Identity (REQUIRED): table of name, board item, version, date, author, review state, builds-on.
- 1.2 Purpose and User Value (REQUIRED): one paragraph, plain English, the user's morning-after experience.
- 1.3 Scope Boundary (REQUIRED): explicit in-scope bullet list; out-of-scope list with the owning item/ticket for each deferral; known assumptions flagged by name (e.g., single-bank, single-tenant).
- 1.4 Business Rules (REQUIRED): numbered BR-n list. Each rule is testable, names the data that enforces it, and load-bearing rules carry a fence annotation: why the rule exists and what breaks if relaxed. Every BR maps to at least one named test in Section 10 (traceability is checked at review).
- 1.5 Client Email Description (REQUIRED for client-facing projects): 3-5 sentences for the non-technical client.

## SECTION 2 — Architecture and Component Map

- 2.1 Files Modified (REQUIRED): table of real, verified paths (claims ledger backs each) with the change per file. Include forced ripple changes: signature changes, type/select-clause widening, callers that must update.
- 2.2 New Files (OPTIONAL): each with one-line purpose and what it exports.
- 2.3 Architectural Constraints (REQUIRED): house invariants the feature must respect (scoping, transaction boundaries, no-block rules, degradation behavior). Note any existing code structure that must NOT be restructured, and why.
- 2.4 Dependencies (OPTIONAL): external APIs/fields used, verified present (SDK version checked if the field is new).

## SECTION 3 — Database Changes

- 3.1 Schema Changes (REQUIRED): full model blocks for new tables (after confirming no name collision, including @@map), column tables for modified tables, named constants the code shares with the schema (sentinels etc.). State which existing columns are reused rather than duplicated.
- 3.1.x subsections: any algorithmic spec tied to the schema (aggregation queries, scoring functions, counters) written out as runnable SQL/pseudocode with worked breakpoints.
- 3.2 Migration Script (REQUIRED): UP and DOWN, raw SQL. DOWN must never touch pre-existing objects; name them explicitly if there is any collision risk. Include new indexes with their justifying query.
- 3.3 Rollback Procedure (REQUIRED): exact steps, and what data survives rollback.
- 3.4 Seed Data (OPTIONAL): exact filter (verified discriminator columns, not proxies), idempotency mechanism spelled out, run order relative to migration.

## SECTION 4 — API Contract

- 4.1 Endpoint Overview (REQUIRED): table of method+path, purpose, permission. Permissions must be copied from the live permission union (cite it); never invent one. Bulk endpoints state their batching strategy.
- 4.2+ Endpoint Detail (REQUIRED for each non-trivial endpoint): request shape, every status code with error code, success behavior including state transitions (with invariant assertions written as code where the spec collapses a branch on purpose), idempotency mechanism named.

## SECTION 5 — State Machine

- 5.1 Applicability (REQUIRED).
- 5.2 States (REQUIRED): scoped state set; note where pre-existing lifecycles continue unchanged.
- 5.3 Valid Transitions (REQUIRED): table from/to/trigger. Every terminal state reachable; no unreachable branches (if a branch was removed on purpose, say so so nobody reinvents it).
- 5.4 Invariants (REQUIRED): each invariant testable; where an invariant justifies collapsed logic elsewhere, cross-reference it.

## SECTION 6 — UI Specification

- 6.1 Screens and Navigation (REQUIRED): per screen, what renders, where actions live, how rows enter/leave filtered views (name the filter predicate so nobody widens it).
- 6.2 Component States (REQUIRED): per-field and per-row states; any count shown to the user states whether it is exact or an upper bound.
- 6.3 Mobile Behavior (REQUIRED): or explicit deferral with owning item.
- 6.4 Exact Copy (OPTIONAL): literal strings for dialogs, pills, confirmations. Copy must match actual behavior (voided vs removed).

## SECTION 7 — Validation Rules

- 7.1 Field Validation Matrix (REQUIRED): field, rule, error code. Error codes verified against the live union; new codes marked NEW with the file they extend.
- 7.2 Cross-Field Validation (OPTIONAL).

## SECTION 8 — Audit Trail Hooks

- 8.1 Audit Event Inventory (REQUIRED): event, when, metadata. Note which existing events are unchanged.
- 8.2 Audit on Failure (OPTIONAL).

## SECTION 9 — Non-Functional Requirements

- 9.1 Performance Targets (REQUIRED): numeric targets, and the implementation strategy that makes each achievable (batching, indexes), cross-referenced to where it is specced.
- 9.2 Data Volume Limits (OPTIONAL): growth rate, pruning decision.

## SECTION 10 — Test Requirements and Acceptance Criteria

- 10.1 Unit Tests Required (REQUIRED): per service/module; every BR-n appears in at least one test; regression tests for every trap found during archaeology or review (construct the trap, assert it does not fire). Mark new test files as new.
- 10.2 Integration Tests Required (REQUIRED): end-to-end paths, partial-failure paths, and permanent regression tests for past incidents by name.
- 10.3 Acceptance Criteria / Definition of Done (REQUIRED): numbered, each independently checkable; ends with the executable DoD (Section 16 checkpoints reference these).

## SECTION 11 — Monitoring and Observability

- 11.1 Required Log Events (REQUIRED): with payload fields; note known coverage caveats (e.g., null FK on one path).
- 11.2 Alert Conditions (OPTIONAL): full plumbing spec, not just the metric name: query, where it runs, how it is stored/exposed.

## SECTION 12 — Email and Notification Spec

- 12.1 Applicability (REQUIRED): even if "no new sending"; name what intents are written and who owns transport.
- 12.2 Template (OPTIONAL).

## SECTION 13 — Third-Party Failure Modes

- 13.1 Failure Mode Matrix (REQUIRED): dependency, failure, behavior. Every degradation explicit; nothing blocks the critical path silently.

## SECTION 14 — Security Checklist

- 14.1 Security Requirements (REQUIRED): auth/scoping per endpoint, server-side revalidation of client-suggested ids, what the LLM prompt may receive, what unauthenticated surfaces expose, attribution rules (including reserved sentinels).

## SECTION 15 — Rollout Notes

- 15.1 Pre-Deploy Checklist (REQUIRED): including regression probes for review findings (e.g., "proposed-rules page shows ZERO after seeding").
- 15.2 Deploy Command (REQUIRED): the project's wrapped deploy, never the bare platform command.
- 15.3 Post-Deploy Verification (REQUIRED): see also Section 16 executable DoD.
- 15.4 Rollback Plan (REQUIRED).
- 15.5 Feature Flags (OPTIONAL): what is flagged, what is intentionally not (correctness fixes are not flagged).

## SECTION 16 — Build Order and Checkpoints (NEW in v1.1, REQUIRED)

A coarse dependency skeleton, NOT a file-by-file plan (plan mode owns that at build time, against the tree as it exists then):

| Step | Layer | Checkpoint ("green" means) |
| --- | --- | --- |
| 1 | e.g. migration + prisma generate | migrate dry-run clean; generate clean; no collision |
| 2 | e.g. services (pure logic + unit tests) | new unit tests pass; tsc clean |
| 3 | e.g. routes/jobs wiring | integration tests pass |
| 4 | e.g. UI | localhost review gate |
| 5 | executable DoD | literal commands with expected outputs |

Rules: a failed checkpoint stops the pass there. The executable DoD lists literal commands (`npx tsc --noEmit`, the curl probe, the spot-check query) with expected output, so post-build verification is mechanical. If plan mode at build time disagrees with this skeleton, resolve the disagreement before coding and record it in Build Notes.

## SECTION 17 — Decision Log and Open Questions (NEW in v1.1, REQUIRED)

- Decisions: each significant choice with the rejected alternative and why (one line each). This is what stops a builder under momentum from "fixing" a deliberate choice.
- Review adjudications: blind-review findings adopted (with verification evidence) and rejected (with the evidence that refuted them).
- Open Questions: anything unresolved, with owner ("user decides" / "coding agent decides, document in Build Notes").

## SECTION 18 — Build Notes (NEW in v1.1, filled by the coding agent post-build)

Every deviation from this spec and why; anything discovered that the spec got wrong; follow-ups created. Deviations-per-build is the OPC metric. Leave this section header in the shipped spec, empty until the build.

---

## APPENDIX A — Claims Ledger (REQUIRED, full treatment)

Every factual claim the spec makes about the existing system, one row each:

| # | Claim | Evidence (file:line) | Verified |
| --- | --- | --- | --- |
| 1 | e.g. Permission union is exactly [...] | permissions.ts:11 | yes |
| 2 | e.g. No table named X exists (incl. @@map) | schema.prisma full read | yes |

A claim without a citation does not go in the spec body. Reviewers spot-check this table first.
