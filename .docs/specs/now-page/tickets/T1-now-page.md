---
td: td-56f10a
type: feature
priority: P1
ownership: agent-owned
blocked-by: None
spec: .docs/specs/now-page/SPEC.md §Ticket Decomposition slice T1
---

# T1: the /now/ route end to end

Delivered behavior: building the fixture vault with `NOW.md` at its root serves `/now/` (h1, prose at the `65ch` measure, Updated line when front matter carries `updated`) and lists it in the sitemap; without the file, no route exists.

## Objective

When this ticket closes, the site renders `NOW.md` from the vault root as the Now page: the meta collection carries entry `now` beside `about`, `/now/` builds only when the file exists, the sitemap tracks it, and the page ships no scripts, no FAB, and no Search trigger beyond today's band set. This slice is the foundation; the home Now line (T2) keys its presence on this loader.

## Interface Contract

- `readNow(root: string)` exported from `src/lib/vault.ts` next to `readAbout`. Returns `{ title: string; body: string; updated?: Date } | null`. Returns `null` if and only if `<root>/NOW.md` does not exist. `title` comes from the first `# ` heading in the body, else front-matter `title`, else `"Now"`. `updated` parses from front matter `updated` via gray-matter when present.
- `src/content.config.ts`: the `meta` collection loads both vault-root entries - `about` as before and `now` beside it. Astro 7.3.2 permits one runtime loader per collection, so the two logical loaders (`deppfellow-about`, `deppfellow-now`) compose through a single runtime loader; entry ids and the schema are unchanged from this contract (D-09). The collection schema becomes `z.object({ updated: z.coerce.date().optional() })`; the existing `about` entry omits `updated`.
- New route `src/pages/now/[...slug].astro`: `getStaticPaths` returns exactly one entry producing `/now/` when the meta `now` entry exists, and zero entries when it does not. The page renders `Base` (title from the note title), `RuleBand` without `search`, an h1 with the title, and when `updated` exists a Scales date line "Updated YYYY-MM-DD" (tabular numerals) under the h1. The body renders at the `note measure` width. No `data-pagefind-body`, no FAB, no pagefind markup, no scripts beyond what `Base` and `RuleBand` already ship.
- The composed meta loader clears its collection store before loading - the data store persists across builds, and a stale entry would survive a deleted vault file (D-10).
- `src/pages/sitemap.xml.ts` includes exactly one `/now/` loc when the meta `now` entry exists, none otherwise.

## Examples

- Fixture build (Setup below) → `dist/now/index.html` contains an h1 with "Now", the string "Updated 2026-09-27", and both fixture prose paragraphs; `dist/sitemap.xml` contains exactly one `<loc>` whose URL path is `/now/`.
- Edge: rename `fixtures/vault/NOW.md` to `NOW.md.bak`, rebuild → no `dist/now/` directory exists, the sitemap has 48 locs, and the build still reports "Loaded 21 notes".
- Edge: `NOW.md` without an `updated` field → `/now/` renders with no "Updated" string anywhere in the page.

## Setup

Create `fixtures/vault/NOW.md` with exactly this content:

```markdown
---
description: Current focus of Deppfellow
created: 2026-09-27
updated: 2026-09-27
tags: [now]
---

# Now

I am wiring the observatory: the agents that read this vault and the pages they publish. The current work is the Now page you are reading and the plate it sits on.

Next up is a pass over the memory layers note while the logs stay small and daily.
```

All commands run with `WIKI_PATH=fixtures/vault`. The vault root is only read for `NOW.md`; the file sits outside every category directory, so `readNotes` never sees it and the 21-note load contract (22 found, 1 skipped, exact warning) is untouched by design.

## Gate

```sh
npm run format && npm run format:check && npm run lint && npm run check && npx tsc --noEmit -p tsconfig.json && WIKI_PATH=fixtures/vault npm run build && ls dist/now/index.html
```

Verification skill: `.pi/skills/verify-deppfellow-page` (fixture build gate + built-HTML greps; this slice adds feature-map candidates "Now page", maintained separately).

## Acceptance Criteria

- L-AC-01 — Fixture build exits 0 and writes `dist/now/index.html` containing the h1 "Now", the string "Updated 2026-09-27", and the fixture prose (SPEC-AC-01).
- L-AC-02 — `dist/sitemap.xml` contains exactly one loc ending in `/now/` and 49 locs total (SPEC-AC-01, REQ-09).
- L-AC-03 — With `NOW.md` renamed away, a rebuild produces no `dist/now/`, 49 HTML pages, and a 48-loc sitemap; the build log still reports "Loaded 21 notes" with the exact broken-note skip warning (SPEC-AC-02).
- L-AC-04 — `dist/now/index.html` carries no `data-pagefind-body`, no `pagefind-modal`, and no script tag that home does not also carry (SPEC-AC-03, REQ-08).
- L-AC-05 — With `NOW.md` lacking `updated`, `dist/now/index.html` contains no "Updated" string (REQ-03).
- L-AC-06 — A temporary `NOW.md` containing a callout and `[[memory-layers]]` renders the callout markup while the literal text `[[memory-layers]]` survives into the built HTML (SPEC-AC-04, REQ-02).

## Specification Coverage

REQ-01, REQ-02, REQ-03, REQ-04, REQ-08, REQ-09 - matches the Spec's coverage row for T1 exactly.

## Preserved Invariants

- The Rule band renders unchanged on every existing page; no band or Menu-panel changes anywhere.
- The home page's about section and plate stack are untouched (the Now line belongs to T2).
- No page gains a script; the theme pre-paint and band scripts remain the only ones.
- The fixture load contract stands: 21 notes loaded, 22 found, exactly 1 skip, same warning text.
- Existing routes, RSS (17 items), catalog.json, llms.txt, and the raw mirror are byte-identical apart from the sitemap's single added loc.

## Out of Scope

- The Now line on the home page and the `--text-micro` token (T2).
- `DESIGN.md` prose sections (T3); the design.json mirror is also T2's (token), not T1's.
- Search indexing of `/now/`, RSS/catalog/raw exposure of `NOW.md`, FAB, plate stack.
- Authoring the real `NOW.md` in `deppfellow-wiki`.
