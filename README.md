# PRD Creator (a Claude skill)

A Claude skill that turns a feature idea into a build-ready PRD through four layers: codebase recon before any questions, a structured one-question-at-a-time interview with opinionated defaults, a claims ledger that backs every factual assertion with a file:line citation, and an adversarial review stage. The review stage runs two blind fresh-context AI reviewers plus a third structured-pass reviewer; when that third arm catches a class of issue the blind reviewers missed, the lesson is folded back into the blind-review prompt so the review keeps getting stronger over time. It produces a dense machine-ready Markdown spec for an AI coding agent plus a formatted Word PRD for human readers.

It scales from a one-page fast-path spec for small personal tools up to a full 18-section treatment for hardened, public-facing applications (security checklist, audit hooks, monitoring, rollout and rollback plans).

## Download and install

Download [prd-creator.skill](./prd-creator.skill) (a zip archive with a different extension).

- **Claude desktop app:** Settings > Capabilities > add the .skill file. Or click "Save skill" when the file is shared inside a Claude chat.
- **Claude Code:** extract the archive into `~/.claude/skills/` so the recipe lands at `~/.claude/skills/prd-creator/SKILL.md` (Windows: `C:\Users\<you>\.claude\skills\prd-creator\SKILL.md`).

Then say "Write a PRD for ..." and the skill takes over.

## What's inside

- `SKILL.md` -- the workflow: ceremony dial, recon, interview, claims ledger, draft, blind review, fold-in, build contract
- `references/defaults.md` -- best-practice defaults offered during the interview (table scaffolds, error envelope, rate limits, UI states, role hierarchy)
- `references/spec-template.md` -- the 18-section spec template plus the claims-ledger appendix
- `references/review-prompts.md` -- the blind-review prompt and the adjudication rules for reviewer findings
- `scripts/md_to_docx.py` -- converts a finished spec to a Word document (requires python-docx)

## Feedback

Found a bug, or have a suggestion? [Open an issue](../../issues/new/choose). Bug reports, improvement ideas, and "the PRD it wrote missed X" reports all directly shape future versions. Please keep anything private out of public issues.

Version 2 (2026-06-03). Updated 2026-06-27: added a money-path gate and a decision-vs-conversation reconciliation check, and a third structured review arm whose blind-missed catches ratchet back into the blind-review prompt.
