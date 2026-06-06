# Blind Review Prompts

Run in Phase 5 (full treatment only). Spawn two fresh-context subagents in parallel via the Agent tool: one on the authoring model, one on a different model family (e.g. `model: "sonnet"` when the author is Opus). Fresh context is the mechanism; do not summarize the spec for them or mention any known concerns, prior reviews, or suspected weak spots. Leading hints contaminate the experiment.

## Prompt template

Replace the bracketed values; keep everything else verbatim.

```
You are an independent senior reviewer doing a pre-implementation review of a PRD for [project one-liner: stack, domain, hosting].

Review this document: [absolute path to the spec .md]

You may and should consult the codebase to verify the PRD's claims: [repo root] ([key dirs: src/, prisma/schema.prisma, tests/]). Verify that referenced files, columns, enums, and services actually exist and behave as the PRD asserts.

STRICT SCOPE RULE: do NOT read anything under [paths holding review artifacts, prior spec versions, task queues]. Those contain unrelated material that must not influence this review. If you accidentally open one, discard what you saw and say so in your output.

Your job: find problems an implementer would hit or, worse, would NOT hit because the code would faithfully implement a flawed spec. Be skeptical of every business rule, state transition, schema change, and query. Check internal consistency between sections (business rules vs schema vs state machine vs API vs UI copy vs tests). Check that every behavior described is actually implementable against the real schema and that every threshold/window has the data to enforce it.

Output format:
1. BLOCKING ISSUES: numbered list of issues that must be resolved before implementation starts. For each: what's wrong, where (section), why it matters, and a recommended fix.
2. IMPLEMENTATION NOTES: numbered list of smaller items to carry into implementation.
3. One-line overall verdict.

Return the review as raw markdown text. Aim for high precision: only report things you have verified or reasoned through concretely, and say which sections/code you checked.
```

## Adjudication rules (the step after the reviews return)

1. Verify every reviewer claim against the code yourself before adopting it. Blind reviewers overreach too; the record includes a claimed flag-corruption risk where the flag turned out to be write-only, and a suggested alternative column that was wrong.
2. Adopt verified findings into the spec; bump the version; cite the verification evidence in the Section 17 decision log.
3. Reject unverified or refuted findings explicitly, also in Section 17, with the evidence. Silent rejection loses the reasoning.
4. Where the two reviewers disagree, the code decides, not seniority of model.
5. Expect partial overlap, not duplication. Two reviewers finding different things is the system working; it is the reason there are two.
