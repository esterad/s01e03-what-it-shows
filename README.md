# kb — the support knowledge base

**kb** = knowledge base. What the support desk knows about how our products break, written
down where the code can keep it honest.

One desk takes reports for every product we run: `sched-prod` (calendar invites for about
300k people) and the rest. When something
goes wrong, the reports reach the desk first. This kb is how the desk triages them, what each
incident turned out to be, and why we decided what we decided.

## What's here

```
src/routes/kb/      the pages (.svx: Markdown with components)
  triage/           how the desk handles incoming reports
  incidents/        one page per big ticket
  decisions/        numbered decision records
src/lib/kb/         components and the logic they run
src/lib/layouts/    the frame every page shares
src/lib/data/       exports from the ticket system
```

## Conventions

- Words in the page, logic in `src/lib` (imported as `#lib/…`). A number that can be computed belongs in a component.
- Every page starts with frontmatter: `title`, `owner`, `status`, `reviewed`.
- A decision is never edited after it is accepted. A new record replaces it.
- A page you add is signed: its frontmatter carries your `signature` and it ends with `<RenderStamp {signature} />`, so the desk knows who wrote it.

## Owners

Support desk. Decision records: whoever signed them.

---

Exported 2026-10-04 from the `support` repo, `apps/kb/` → `src/`.

## S01E03: #4471 split

- New page: `src/routes/kb/incidents/4471-split/+page.svx`, signed with the course ticket as `signature` (desk ref DR-NXCV-XM64).
- New component: `src/lib/kb/ProblemTable.svelte` computes the counts with the desk's own view as exported (`causes.ts`, `locales.ts`, `desk-notes.json`). No existing file in `src/lib` was changed.
- The page as the desk sees it: `build/kb/incidents/4471-split/index.html` (whole static build in `build/`; `npm install && npm run build` rebuilds it).
- Tickets, one per problem in #4471 (412 unique reports):
  - [#1 shared parser swaps day and month — 379 reports](https://github.com/esterad/s01e03-what-it-shows/issues/1) — closed, fixed
  - [#2 booking-v3 form hint MM/DD/YYYY — 33 reports](https://github.com/esterad/s01e03-what-it-shows/issues/2) — closed, won't fix (booking-v3 retired)
  - [#3 date moves by exactly one day — 5 reports](https://github.com/esterad/s01e03-what-it-shows/issues/3) — open, no fix yet
