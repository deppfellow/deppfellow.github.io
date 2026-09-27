---
td: td-6a3f94
type: chore
priority: P3
ownership: agent-owned
blocked-by: [T1, T2]
spec: .docs/specs/now-page/SPEC.md §Ticket Decomposition slice T3
---

# T3: the DESIGN.md contract amendment

Delivered behavior: `DESIGN.md` documents the micro size, the Now line element, and the Now page, matching what T1 and T2 actually built.

## Objective

When this ticket closes, the design contract carries the initiative: the `typography` front matter lists `micro` with the exact token values, a Now line element entry sits with the other elements, and a Now page section records the page shape (prose at `65ch`, Updated line, absence behavior, no Search trigger, no FAB). The contract documents built reality; nothing here changes code.

## Interface Contract

- `DESIGN.md` front matter `typography` gains `micro` with `fontFamily` matching the note voice's serif stack and `fontSize: "0.6875rem"`, plus the line-height `1.4` in whatever structural form the existing entries use (mirror T2's token, never invent values).
- `DESIGN.md` body gains a Now line element paragraph (what it is, where it sits, micro size, iris hover, presence keyed on `NOW.md`) alongside the existing element entries, and a Now page section recording the reading-page prose treatment, the optional Scales Updated line under the h1, and that the page ships no Search trigger, no FAB, and stays out of the search index.
- `CONTEXT.md` already carries the Now page, Now line, and Logs entries from the grilling session - verify, do not rewrite.
- No new ADR: every decision in this initiative is feature-local and reversible (REQ-10). `.docs/adr/` gains nothing.

## Examples

- After the edit, a reader can answer from `DESIGN.md` alone: what size is fine print (micro, 0.6875rem), what links `/now/` (the Now line under the Observer's note), when does `/now/` exist (only when `NOW.md` is in the vault).
- Edge: `grep -i micro .docs/DESIGN.md` returns the front-matter entry with `0.6875rem` matching `global.css`; a value mismatch against `global.css` fails the ticket.

## Setup

T1 and T2 merged, so the built site already shows the behavior the prose must match. `global.css` is the value source of truth.

## Gate

```sh
npm run format:check && grep -c "0.6875rem" src/styles/global.css .docs/DESIGN.md .impeccable/design.json && git diff --stat .docs/adr/ | wc -l
```

The last check must print `0` (no ADR tree changes).

Verification skill: none - no driveable surface; built-HTML/CSS greps from T1/T2 already prove the behavior this prose documents.

## Acceptance Criteria

- L-AC-01 — `DESIGN.md` front matter carries a `micro` typography entry whose size value equals `global.css`'s `--text-micro` (REQ-07, REQ-10).
- L-AC-02 — `DESIGN.md` body contains a Now line element entry and a Now page section that mention the `65ch` prose measure, the Updated line, the absence behavior, and the no-Search-trigger/no-FAB rule (REQ-10).
- L-AC-03 — `git diff .docs/adr/` is empty against the pre-epic base; no ADR was added or edited (REQ-10).
- L-AC-04 — `CONTEXT.md` contains the Now page, Now line, and Logs entries with no further edits needed in this ticket (REQ-10).
- L-AC-05 — `npm run format:check` passes; `npm run lint` is untouched by this ticket (no code files changed).

## Specification Coverage

REQ-10 - matches the Spec's coverage row for T3 exactly.

## Preserved Invariants

- Every pre-existing `DESIGN.md` section stays verbatim; this ticket only appends the micro typography entry, the Now line element entry, and the Now page section.
- No source file under `src/` changes in this ticket.
- The dual-world and band rules are not restated or weakened by the new sections.

## Out of Scope

- Any code change, including comment or naming polish in `src/`.
- Rewriting the CONTEXT.md entries landed during grilling.
- ADR authoring; the verification-skill feature map (maintainer's job, never a ticket's).
