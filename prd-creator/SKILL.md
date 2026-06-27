---
name: prd-creator
description: |
  Use this skill whenever the user wants to write a PRD, feature spec, or implementation spec for a software feature or new project. Trigger on: "write a PRD for", "spec out", "I want to build X", "help me write a spec", "create a feature spec", "document this feature", "I need a PRD for", "write up a spec", or any request to specify what needs to be built before development starts. Also trigger when the user has a feature idea or new project concept and wants something structured enough for an AI coding agent to implement in one pass.

  v2 (2026-06-03). Replaces the May 2026 interview-only version. Adds the three layers that interview alone cannot provide: codebase recon before the interview, a claims ledger verifying every factual assertion against the live code, and blind fresh-context adversarial reviews before the human review pass. Produces a dense CC-ready Markdown spec and a formatted Word PRD.
---

# PRD Creator v2

Objective: one-pass coding (OPC). A PRD good enough that the build needs zero spec-caused rework and ships zero latent spec-faithful bugs. The May 2026 experiment record: a carefully authored, twice-reviewed PRD still carried 7 verified blockers (a table-name collision, a false-positive matching trap, a nonexistent permission, and more) that only codebase verification and blind review caught. This skill exists so those layers are mandatory, not remembered.

Priority of layers (where quality leverage lives): 1. facts (recon + claims ledger), 2. adversarial review (blind subagents), 3. intent (interview), 4. format (template + outputs). Execution order is different from priority order; the phases below are in execution order.

## Phase 0: Ceremony dial

Decide the treatment before anything else and say which you chose:

- **Full treatment:** new feature or project, schema changes, new endpoints, state machines, anything over roughly a day of work. All phases below.
- **Fast path:** copy changes, small fixes, single-file features. Skip the interview ceremony and blind reviews; produce a one-page spec with four blocks: verified facts (with file:line), business rules, tests, executable definition of done. A skill that demands 400 lines for every task gets bypassed and then protects nothing.

Size gate: if the full-treatment spec implies more than ~a day of changes across many files, force a slice plan: separately shippable slices, each with its own definition of done. One-pass success probability decays fast with scope; slicing resets it per slice.

Money-path gate: any feature that sets, signs, rounds, routes, or reports a monetary value, or that bounds a balance window by date, takes the full treatment with blind review regardless of size. No fast-path for money. The record: a contained import change still carried two latent money bugs only the review pass caught (a date boundary anchored to local time instead of a stable UTC midnight, which threw a false balance mismatch at the day edge; and an overwrite that deleted from the wrong floor and double-counted an opening balance). Small code, real dollars; review is mandatory.

## Phase 1: Recon scan (BEFORE the interview)

For existing projects, read the codebase first so the interview is informed. 15 to 30 minutes of looking, not a full audit:

- What already exists that overlaps this feature? (Services, tables, routes, jobs. The most expensive PRD error is speccing a "new" thing that already exists under another shape.)
- The real names: permission unions, enums, status fields, error-code unions, file paths. Never guess these later; copy them now.
- The data model around the feature: which tables, which columns, what is nullable, what indexes exist.
- House patterns: error envelope, auth middleware, audit hooks, migration conventions, deploy script.

For new projects, recon is the ecosystem instead: what stack the user already runs, what they have deployed before, what conventions their other projects use.

## Phase 2: The interview

Run the intent capture and section interview. Read `references/defaults.md` first; lead every question with an opinionated recommendation so the user mostly confirms rather than invents.

Intent capture, one question at a time:

- Q1: New project or update to existing? (N/U). If U, recon from Phase 1 stands in for most context questions.
- Q2 (new only): Personal, business, or both? (P/B/X). Business outcome and success-metric questions are skipped for P.
- Q3 (new only): One-paragraph description: what does it do, who uses it? Follow up until you have the user pain, the user, and the key output.
- Q4 (update only): What does this feature do, in one sentence?
- Q5 (update only): What can't the user do today? Be specific.
- Q6 (B/X only): What metric should this move, and what does success look like? "X changes from Y to Z within W days."

Section interview. Walk the spec sections (user narrative, system context, data model, API, UI, auth, errors, performance, acceptance criteria, goals and non-goals, open questions). For each: present your recommendation grounded in recon ("you already have MatchingService with a score table; I recommend extending it with a check tier rather than a parallel matcher"), then ask to confirm or modify.

Etiquette (unchanged from v1, it was right):

- One question at a time. Ask, wait, then ask the next. Never bundle.
- Lead with your opinion: "My recommendation is X because Y. Does that work?"
- Accept agreement quickly; do not restate what was agreed.
- Follow implications: Stripe mentioned means raise idempotency keys and webhook signing; file uploads mean storage, size limits, scanning.
- Note deferred decisions in Open Questions and keep moving.
- Skip irrelevant sections, say why in one line.

## Phase 3: Deep archaeology and the claims ledger

Now verify everything the draft will assert. The rule: **every factual claim the PRD makes about the existing system carries a file:line citation, collected in the claims ledger (template Appendix A).** An unverified claim must look naked on the page.

Mandatory checks (each one is a real failure from the record):

- Before any `CREATE TABLE`: does a table with that name already exist? Check `@@map` directives too, not just model names.
- Before citing any permission, role, enum value, or error code: read the actual union/enum and copy it verbatim.
- Before "add matching/dedup/lookup on column X": what else writes to X? (The check-number trap: an auto-numbering job wrote sequence values into the same column as real check numbers.)
- Before "extend service Y": read Y's actual signature and call sites. List every signature, type, and select-clause change the extension forces.
- Before "reuse existing validator/flow Z": confirm Z exists and does what the PRD says. If it does not exist, the PRD says NEW, not "existing".
- For every threshold or time-window rule: name the column that stores the data enforcing it.

## Phase 4: Draft

Read `references/spec-template.md` and fill every section; write "N/A: [reason]" rather than omitting. Principles:

- Dense over readable. The .md is for a machine; every line is information.
- Explicit over shorthand. Never "standard CRUD"; write the four routes.
- Chesterton's fence annotations: every load-bearing constraint carries one line on why it exists and what breaks if an implementer relaxes it. Most one-pass failures are the builder "simplifying" something the spec author knew better about. The annotation is the message across that gap.
- Rule-to-test traceability: every business rule (BR-n) maps to at least one named test in the test section, and every test names its BR. A rule with no test is where spec-faithful bugs hide longest.
- Self-contained. The coding agent must not need to ask clarifying questions; unresolved items go in Open Questions with an owner.
- Reconcile decisions against the conversation. Before the draft is final, walk every decision and default in the spec against what was actually agreed in the interview and the originating conversation. Where the spec defaulted to something other than the agreed choice, flag it as an Open Question pointing at the divergence, do not silently ship the default. The blind reviewers verify the spec against the code; they are context-free and cannot see what was agreed, so spec-versus-agreement is the author's job alone. The record: a spec once defaulted to one UI design (a column-reflow editor) when the conversation had settled on a different one (a master-detail editor), two different builds; the blind reviews never flagged it because nothing in the code contradicted the spec.

## Phase 5: Blind adversarial review (full treatment only)

Spawn two fresh-context subagents with the prompt in `references/review-prompts.md`: one on the authoring model, one on a different model family if available. Give them the spec path and codebase access; forbid the paths holding review artifacts and prior spec versions. Fresh context is the point: the author reviewing their own spec checks "does the code match the rule" and is structurally blind to "is the rule right".

Then, in parallel with the two blind reviewers, run a third arm: a fresh-context **structured 5-pass reviewer** (factual accuracy / completeness / internal consistency / implementability / risk, in that order), same spec path, same codebase access, same forbidden paths, blind to the other two. This arm is retained deliberately, not as the primary review but as a generator: it periodically surfaces a class of issue the blind adversarial prompt does not yet target, and each such catch is folded back into the blind prompt so the blind layer keeps improving. In controlled trials each run of this arm has produced about one valid catch both blind reviewers missed.

Then, before adopting anything: **verify every reviewer claim against the code yourself.** Blind reviewers and the 5-pass both overreach (recorded examples: a claimed flag-corruption risk where the flag turned out to be write-only; a 5-pass false positive that a role could create a record when the route actually returned 403). Adjudication against code is the false-positive backstop for all three arms. Adopt verified findings, reject the rest with the evidence, and record both in the decision log.

The ratchet (the reason the 5-pass arm is kept): when the 5-pass surfaces a valid item both blind reviewers missed, fold the **general lesson** (not the hyper-specific instance) into `references/review-prompts.md` so the blind layer catches that class next time. Keep lessons general to avoid prompt bloat. Keep a short log of these fold-ins and the per-spec count of 5-pass-unique valid catches so the retirement streak can be tracked.

Retirement criterion: drop the 5-pass arm only after roughly five consecutive full-treatment specs where it surfaces zero valid items the blind reviewers missed. At that point the blind prompt has absorbed everything the 5-pass was catching and the arm is pure redundancy. The arm costs little (async, modest tokens), so when unsure, keep it.

Optional: an extra review pass by a second assistant, per your project's review protocol, at the user's call.

## Phase 6: Fold-in and outputs

Apply accepted findings. Bump the version, record what changed and why in the version header. Produce both outputs:

1. CC spec: `[Feature]_PRD_MMDDYY-N.md` (or the project's naming convention). Prior versions stay untouched for diffing; fold-ins create N+1.
2. Word doc: same basename `.docx`, generated via `scripts/md_to_docx.py` (`py md_to_docx.py <src.md> <dst.docx>`). Leads with the plain-English "what this is" paragraph for non-technical readers.

Ask once where the project's specs live and save both files there.

Writing style: house rules apply to every artifact (no em-dashes; see the user's global CLAUDE.md style list).

## Phase 7: Handoff and the build contract

- The human review pass happens on the post-fold-in version, never the raw draft.
- At build time the coding agent still enters plan mode. The spec's Build Order section is a coarse dependency skeleton with checkpoints; plan mode plans file-by-file against the tree as it exists at build time, which may have drifted since spec approval. The two must agree at the skeleton level; disagreement is signal that the spec missed a dependency or the code moved, and it gets resolved before coding, not silently.
- After the build, the coding agent fills the spec's Build Notes section: every deviation from spec and why. Build Notes are the feedback loop; deviations-per-build is the metric that tells you whether the PRDs are getting better. Target: under 2 spec-caused deviations at v1.0 of any spec.
