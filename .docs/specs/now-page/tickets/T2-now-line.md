---
td: td-c4470c
type: feature
priority: P2
ownership: agent-owned
blocked-by: T1
spec: .docs/specs/now-page/SPEC.md §Ticket Decomposition slice T2
---

# T2: the Now line and the micro token

Delivered behavior: the home page shows the Now line - the serif sentence "See the now page for the overview of what I'm doing now." with "now" linked to `/now/` - under the Observer's note, set at the new `--text-micro` token, present only when `NOW.md` exists.

## Objective

When this ticket closes, the 11px micro size exists as a token in `global.css` mirrored through the design contract files, and the home page renders the Now line as the Now page's single discovery point. The line disappears on its own when the vault has no `NOW.md`, and survives when `ABOUT.md` is absent.

## Interface Contract

- `src/styles/global.css`: the token slots gain `--text-micro: 0.6875rem` and `--text-micro--line-height: 1.4`, following the existing `--text-*` naming and line-height companion pattern.
- `.impeccable/design.json` and the `DESIGN.md` front matter `typography` section gain a `micro` entry whose values match `global.css` byte for byte.
- `src/pages/index.astro`: after the about section, inside the same `max-w-275` container, a paragraph renders the exact sentence `See the now page for the overview of what I'm doing now.` where only the word "now" directly after "the" is an anchor to `/now/`; the final "now" of the sentence stays plain text. The paragraph is sized `var(--text-micro)` with the token line height; the link hover moves to `var(--color-iris)` with the house `120ms steps(2, end)` color transition.
- Presence is keyed on the meta `now` entry existing (the same condition that builds `/now/`). The line renders regardless of whether the about section renders; when `ABOUT.md` is absent the line sits at the top of `main`.

## Examples

- Fixture build → `dist/index.html` contains `See the <a href="/now/">now</a> page for the overview of what I'm doing now.` (whitespace-insensitive match on the sentence, exactly one anchor to `/now/`).
- Edge: rename `fixtures/vault/NOW.md` away, rebuild → the sentence is gone from `dist/index.html`, the about section still renders, 49 HTML pages.
- Edge: also rename `ABOUT.md` away → the sentence still renders (no about section above it), and `/now/` still builds.

## Setup

T1 merged (the meta `now` loader and fixture `NOW.md` exist). Commands run with `WIKI_PATH=fixtures/vault`.

## Gate

```sh
npm run format && npm run format:check && npm run lint && npm run check && npx tsc --noEmit -p tsconfig.json && WIKI_PATH=fixtures/vault npm run build && grep -c "href=\"/now/\"" dist/index.html
```

Verification skill: `.pi/skills/verify-deppfellow-page` (built-HTML greps + built CSS greps; this slice adds feature-map candidate "Now line", maintained separately).

## Acceptance Criteria

- L-AC-01 — `global.css` defines `--text-micro: 0.6875rem` and `--text-micro--line-height: 1.4`; `.impeccable/design.json` and `DESIGN.md` front matter carry the matching `micro` entry (REQ-07).
- L-AC-02 — Built home HTML carries the exact sentence with exactly one anchor to `/now/` on the first "now"; the trailing "now" is outside the anchor (SPEC-AC-03, REQ-05).
- L-AC-03 — With `NOW.md` absent, the sentence is absent from `dist/index.html` and the page count stays 49 (SPEC-AC-02, REQ-06).
- L-AC-04 — Built CSS applies `var(--text-micro)` to the line and moves the link to `var(--color-iris)` on hover (REQ-05).
- L-AC-05 — With `ABOUT.md` also absent, the sentence still renders on home (REQ-06).
- L-AC-06 — The Rule band and Menu panel markup in built HTML are unchanged against a pre-epic build (no Now link, no new row) (REQ-08).

## Specification Coverage

REQ-05, REQ-06, REQ-07, REQ-08 - matches the Spec's coverage row for T2 exactly.

## Preserved Invariants

- The band and Menu panel render exactly as before on every page.
- The about section and plate stack markup are unchanged when both files exist.
- No page gains a script; the line is plain server-rendered HTML.
- `/now/` behavior from T1 is unchanged.

## Out of Scope

- `/now/` page internals (T1).
- `DESIGN.md` prose sections - the Now line element entry and Now page section (T3); only the front-matter `typography` mirror belongs here.
- Any band, Menu-panel, footer, or article-page presence for `/now/`.
