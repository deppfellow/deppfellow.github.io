---
document_type: specification
initiative_slug: now-page
contract_status: approved # derived, never hand-edited
created_at: 2026-09-27 # fixed at first derivation
updated_at: 2026-09-27 # last derivation date
---

# now-page — the Now page and the Now line

## Objective

Give the site a dedicated **Now page** at `/now/` rendering `NOW.md` from the vault root - the person's latest condition as one snapshot rewritten in place - and a **Now line** under the home page's Observer's note as its single discovery point. The Rule band stays untouched; the feature lights up only when the wiki actually carries `NOW.md`.

> **Now page** - the standalone page rendering `NOW.md` from the vault root: the person's latest condition, one snapshot rewritten in place as it changes - never a dated feed. The significant counterpart to the Logs category.
> **Now line** - the small serif line under the Observer's note on the home page, linking to the Now page; it renders only when `NOW.md` exists.
> **Logs** - the casual daily category: small, unimportant day-to-day notes written when the mood strikes. It carries no statement of current focus; the Now page owns that.

## Repository Context

Astro 7.3.2 + Tailwind CSS 4.3.3, static output, `unified()` Markdown processor (ADR-0014) with the global `resolveVaultMarkup` remark plugin, `rehypeCallouts`, and `rehypeKatex`. Vault projection lives in `src/lib/vault.ts` (`readNotes`, `readAbout`); `src/content.config.ts` loads `ABOUT.md` into the `meta` collection as entry `about`, and `src/pages/index.astro` renders it as the Observer's note above the plate stack. `src/components/RuleBand.astro` renders the band on every page. Design tokens are custom properties in `src/styles/global.css` (`@theme` + base layer), mirrored in `DESIGN.md` front matter and `.impeccable/design.json`. Pagefind indexes only `data-pagefind-body`-marked regions (note detail pages only); the sitemap is a hand-built path list (48 locs = 49 HTML pages minus 404); `llms.txt`, RSS, `catalog.json`, and the `/raw/` mirror cover published notes only. The fixture vault loads 21 notes (22 found, 1 skipped by design) with `ABOUT.md` at its root. td project state lives at `.todos/`; the project verification skill is `.pi/skills/verify-deppfellow-page`.

## Confirmed Requirements

### The Now page

- REQ-01 — The site serves `/now/` as a dedicated page rendering `NOW.md` from the vault root: h1 from the `# ` heading, prose body at the `65ch` measure in the reading-page treatment - no plate stack, no FAB, and the band without the Search trigger. ({D-02})
- REQ-02 — `NOW.md` renders through the same pipeline as `ABOUT.md`: callouts and math/KaTeX render; vault markup (wikilinks, embeds, inline `#tags`) renders unresolved - the meta loader passes no per-note context, exactly as ABOUT.md behaves today. ({D-02})
- REQ-03 — When `NOW.md` carries an `updated` front-matter date, `/now/` renders a small tabular Scales date line under the h1 ("Updated YYYY-MM-DD"); absent front matter renders no line. The body itself stays a pure rewritten-in-place snapshot. ({D-01, D-04})
- REQ-04 — When `NOW.md` is absent from the vault, the `/now/` route is not built at all - no stub, no redirect. ({D-06})

### The Now line

- REQ-05 — The home page renders the Now line under the Observer's note: the serif sentence "See the now page for the overview of what I'm doing now." with "now" linked to `/now/`, hover to iris, set at the micro size. ({D-03})
- REQ-06 — The Now line renders only when `NOW.md` exists, and its slot follows the about section independent of `ABOUT.md`'s presence. ({D-03, D-06})
- REQ-07 — A new token `--text-micro: 0.6875rem` (11px) with line-height `1.4` joins the token slots in `global.css`, mirrored in `DESIGN.md` front matter and `.impeccable/design.json`; generic for future fine print, first consumer the Now line. ({D-08})

### Band, search, and feeds

- REQ-08 — The Rule band is unchanged: no Now link on the band, no Menu-panel row; `/now/` ships the standard band (name, categories, toggle) without Search trigger and stays out of the Pagefind index (no `data-pagefind-body` mark), same as home and tag pages. ({D-03, D-05})
- REQ-09 — The sitemap gains exactly one `/now/` loc when the page builds; `llms.txt`, RSS, `catalog.json`, and the `/raw/` mirror are unchanged. ({D-05})

### Contracts and docs

- REQ-10 — `DESIGN.md` carries the micro size in its typography front matter, a Now line element entry, and a Now-page section; `CONTEXT.md` carries the Now page, Now line, and Logs entries (updated in session). No new ADR: every decision here is feature-local and reversible. ({D-07, D-08})

## Scenarios

- SPEC-AC-01 — Fixture build with `NOW.md` present: `dist/now/index.html` builds (50 HTML pages, 49 sitemap locs) rendering the h1, the prose at the `65ch` measure, and the Updated line when front matter carries it; the home page shows the Now line in micro size under the Observer's note. ({D-02, D-04, D-08})
- SPEC-AC-02 — Fixture build with `NOW.md` removed: no `dist/now/` route, no Now line on home, page and loc counts back to 49/48, everything else byte-identical. ({D-06})
- SPEC-AC-03 — Clicking "now" in the Now line navigates to `/now/`; the band there shows name, categories, and toggle but no Search trigger; a full Pagefind search of the fixture site never returns `/now/`. ({D-03, D-05})
- SPEC-AC-04 — A `NOW.md` containing a callout and a wikilink renders the callout as on any note while the wikilink stays literal text - identical to `ABOUT.md` behavior. ({D-02})

## Constraint

- The band vocabulary holds: categories-with-counts, theme toggle, Search - the Now feature adds nothing to the band or the Menu panel on any page. ({D-03})
- No new scripts anywhere: `/now/` and home ship exactly today's script set (theme pre-paint + band), honoring the ADR-0002 line as narrowed in the D-16 spirit.
- Snapshot discipline: the Now page is one rewritten-in-place snapshot - it never gains dated entries, feed machinery, or per-entry furniture; that shape belongs to Logs. ({D-01})

## Non-goals

- Any band or Menu-panel presence for `/now/`; any new navigation beyond the Now line.
- Search indexing, RSS, catalog, raw-mirror, or `llms.txt` exposure of `NOW.md`.
- Authoring the real `NOW.md` - the site ships the mechanism and the fixture; content lives in `deppfellow-wiki` and gates the production rollout (D-06).
- Date machinery beyond the optional `updated` front matter - no git-derived timestamps, no auto-dating.
- FAB, plate stack, reading-page metadata rows, or any note-detail furniture on `/now/`.

## Implementation Decisions

- Vault read: `readNow` in `src/lib/vault.ts` mirrors `readAbout` - `join(root, "NOW.md")`, `existsSync` guard, gray-matter front matter, title from the `# ` heading (front-matter `title` fallback), returning `{ title, body, updated? }`. ({D-02, D-04})
- Content collection: the `meta` collection loads entry id `now` beside `about`; Astro 7.3.2's one-loader-per-collection rule means the two logical loaders compose through a single runtime loader (D-09); the collection schema becomes `z.object({ updated: z.coerce.date().optional() })` (the about entry simply omits it). ({D-02, D-04})
- Conditional route: `src/pages/now/[...slug].astro` with `getStaticPaths` returning one entry (rest param undefined → `/now/`) when the meta `now` entry exists, else `[]` - Astro builds the route iff the file exists, satisfying REQ-04 without post-build surgery. ({D-06})
- Now line: `src/pages/index.astro` renders the sentence after the about section inside the same `max-w-275` container, sized `var(--text-micro)`, link hover to iris, presence keyed on `getEntry("meta", "now")`. ({D-03})
- Sitemap: the `paths` array prepends `"/now/"` iff the meta `now` entry exists, keeping routes and locs in lockstep. ({D-05})
- Fixture: `fixtures/vault/NOW.md` at the vault root, front matter `description`/`created`/`updated` symmetric with `ABOUT.md`, body with a `# Now` heading and two short paragraphs; root-level files are invisible to `readNotes`, so the 21-note load contract and the exact skip warning are untouched. ({D-02})
- Token: `--text-micro: 0.6875rem` and `--text-micro--line-height: 1.4` follow the existing token naming pattern in `global.css`; `DESIGN.md` typography front matter and `.impeccable/design.json` mirror them. ({D-08})

## Testing/Seam Decisions

- `.pi/skills/verify-deppfellow-page` is the verifier's driving instrument: its command map gates the fixture build, unit suites, visual capture, and live browser drives. This initiative's scenarios land as new feature-map rows ("Now page", "Now line") added through `maintain-verification-skill`, never per-ticket. The sitemap map row's numbers shift with this initiative (48 → 49 locs when `NOW.md` is present).
- Machine checks: the full CI suite (format, lint, check, tsc, fixture build); fixture-build greps - `dist/now/index.html` carries the h1 and Updated line, `dist/sitemap.xml` carries exactly one `/now/` loc; absence drive - build with `NOW.md` moved aside yields no `dist/now/`, no Now line in `dist/index.html`, 49 pages / 48 locs; grep `global.css` for `--text-micro` and its mirror in `DESIGN.md` front matter and `.impeccable/design.json`; the script whitelist is unchanged (no page gains a script).
- Session-visible checks (live drive): home → click "now" → `/now/` renders; hover moves the link to iris; the Updated line appears/disappears with front matter; Pagefind never surfaces `/now/`.

## Governing References

- ADR-0002 Performance contract (no-new-scripts line holds as narrowed in the D-16 spirit; this initiative adds none)
- ADR-0014 unified() Markdown processor (meta entries share the global pipeline, context-free)
- `DESIGN.md` (amended by T3); `CONTEXT.md` (Now page, Now line, Logs entries, updated in session)
- `.pi/skills/verify-deppfellow-page` (verifier instrument)
- Gate events E-01 (slug confirmed), E-02 (frontier empty, read-back issued), E-03 (council skipped by human instruction), and E-04 (derivation gate: frontier empty, human confirmed via /draft-spec) map through this spec's existence and the decision log.

## Ticket Decomposition

| Slice          | Delivered behavior                                                                                                                                   | Ownership   | Blocked-by |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------- |
| T1 (td-56f10a) | `/now/` end to end: `readNow` + meta `now` loader with `updated` schema, fixture `NOW.md`, conditional `/now/` route with Updated line, sitemap loc. | agent-owned | None       |
| T2 (td-c4470c) | The Now line on home: `--text-micro` token, sentence under the Observer's note linked to `/now/`, presence keyed on `NOW.md`.                        | agent-owned | T1         |
| T3 (td-6a3f94) | Design-contract docs: `DESIGN.md` micro size, Now line element entry, Now-page section.                                                              | agent-owned | T1, T2     |

REQ coverage: T1 → REQ-01/02/03/04/08/09; T2 → REQ-05/06/07/08; T3 → REQ-10.

## Open Questions & Accepted Risks

- Accepted risk — home-only discovery (D-03): direct landings on articles, projects, logs, and tag pages have no path to `/now/`; the home page is the hub, per the nownownow pattern. Revisit only if the human asks for a band presence.
- Accepted risk — NOW.md vault markup renders unresolved (wikilinks, embeds, inline `#tags` stay literal), exactly like `ABOUT.md` today; resolving them would mean passing a `fileURL` context and registering a now context in the loader. Surfaced at the quiz gate; the settled D-02 keeps the shared pipeline as-is unless the human amends.
- Accepted risk — the conditional route leans on Astro's optional rest param (`params` with undefined slug → `/now/`); if Astro semantics change, the fallback is a post-build route guard. Revisit at any Astro major bump.
