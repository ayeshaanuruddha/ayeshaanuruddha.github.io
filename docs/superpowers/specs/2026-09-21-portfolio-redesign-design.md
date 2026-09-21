# Portfolio redesign — design spec

**Date:** 2026-09-21
**Branch:** `new-look`
**Source designs:** Claude Design project `e4b2ed60-d9f2-4d17-b253-e4a4130e46f6` ("Ayesha's Portfolio Redesign")

## Context

`ayeshaanuruddha.github.io` is a Jekyll site on the 2015 Start Bootstrap "Freelancer"
theme. A design review in the source project found the site has two visual
personalities: an unmodified Bootstrap template on the home/resume/extracurriculars
pages, and two genuinely modern hand-built case-study pages. Three unrelated colour
systems appear across one small site.

Four finished designs now replace the template look. This spec covers porting them
into Jekyll.

## Design → page mapping

| Design file | Target |
|---|---|
| `Home-Final.dc.html` | `index.html` |
| `Resume-Final.dc.html` | `resume.html` |
| `CreativeHub-Final.dc.html` | `extracurriculars.html` |
| `CaseStudy-Final.dc.html` | New per-project template for `_projects/*` |

`CaseStudy-Final` renders one instance (`[FIN-INT-101]`, the POS journal project).
It is a style specimen, not a content spec — it shows roughly 15% of that project's
actual body.

## Decisions

1. **Remove Bootstrap entirely.** Originally the plan was to keep it and layer on
   top, but once modals, scrollspy and the collapsing navbar are gone, nothing uses
   it. Measured dependency in the two legacy pages is ten classes.
2. **Retire `_includes/css/main.css`** (the Freelancer skin). No new design uses it.
   It is also the layer that would fight hardest: it forces `text-transform:
   uppercase` and Montserrat on every heading and `p { font-size: 20px }`.
3. **Four case-study pages**, one per project, replacing the Bootstrap modals.
4. **Full project content preserved**, restyled. The ~1,000 lines of Gherkin
   scenarios, data dictionaries, RBAC matrices and DoD checklists are the strongest
   evidence on the site and the design review said so explicitly.
5. **`warehouse.html` and `businessone.html` stay as-is**, linked as "View full
   spec". They keep their current appearance via a compat shim.

## Design tokens

Lifted verbatim from the four design files. Declared as `:root` custom properties.

### Colour

| Token | Value | Use |
|---|---|---|
| `--bg` | `#FAF8F4` | Page background |
| `--bg-hub` | `#F6F1E7` | Creative Hub only |
| `--surface` | `#FFFFFF` | Cards, panels |
| `--surface-alt` | `#F1EEE7` | Portfolio band, nav pill track |
| `--surface-quiet` | `#F7F5F1` | Mono callout inside cards |
| `--ink` | `#1E1A2E` | Primary text |
| `--ink-hub` | `#241C14` | Creative Hub text |
| `--body` | `#3A3745` | Body copy |
| `--muted` | `#5B5867` | Secondary text |
| `--muted-hub` | `#5B4E3D` | Creative Hub secondary |
| `--line` | `#E8E3DD` | Borders |
| `--line-strong` | `#D9D2C6` | Button borders |
| `--line-hub` / `--line-hub-soft` | `#D9CDB6` / `#E4DAC4` | Hub rules |
| `--purple` | `#5B3A8E` | Primary accent |
| `--purple-dark` | `#3E2860` | Link hover |
| `--purple-deep` | `#241C36` | Contact panel, code blocks |
| `--purple-tint` / `--purple-tint-2` | `#EFE9F8` / `#F0EAF9` | Accent surfaces |
| `--purple-line` | `#E2D6F2` | Tag borders |
| `--purple-code` / `--purple-code-ink` | `#B7A9E0` / `#E4E1F0` | Code block text |
| `--green` / `--green-dark` | `#1F7A56` / `#154A35` | Eyebrow labels, status |
| `--green-tint` / `--green-line` | `#E4F3EC` / `#CDE9DC` | Green surfaces |
| `--gold` / `--gold-ink` | `#A9822E` / `#7A5A11` | Tertiary accent |
| `--gold-tint` / `--gold-line` | `#F6EDD8` / `#EBDBAE` | Gold surfaces |
| `--gold-bright` | `#D9B24A` | Contact panel accent |

### Type

- **Manrope** 500/600/700/800 — all pages except the hub
- **IBM Plex Mono** 400/500/600 — code, ticket codes, dates, hub nav
- **Lora** 400/500/600/700 + italic — Creative Hub body
- **Noto Sans Sinhala** 400/500/600 — Sinhala titles on the hub

Replaces Montserrat + Lato. Hero uses `clamp(38px, 5.4vw, 58px)`.

### Radius and shadow

Radii: 8, 10, 12, 16, 20, 24, 28, 999px.
Shadows: hero `0 20px 45px -12px rgba(30,26,46,.18)`; stat cards `0 6px 16px -10px`
in the matching accent; card hover `0 20px 40px -14px rgba(30,26,46,.2)`.

## CSS architecture

`_layouts/style.css` currently concatenates `bootstrap.min.css` + `main.css`. It
becomes `design.css` alone.

New `_includes/css/design.css`, in order:

1. `:root` tokens
2. Reset and base (`box-sizing`, body background/type, link colours)
3. Layout primitives (page shells, section padding, max-widths: 1200/960/900/840px)
4. Components — header/nav, hero, stat cards, project cards, tables, callouts,
   code blocks, Gherkin rows, tag pills, status pills, forms, footer
5. Page-scoped blocks — `.hub-*` for the Creative Hub's serif identity
6. **Legacy compat shim** (see below)
7. Media queries

Tokens are hardcoded in the CSS rather than driven from `_config.yml`. The palette is
fixed by the designs; the existing `site.color.*` indirection buys nothing and
currently holds an invalid value (`primary-rgb: "24,288,156"` — 288 exceeds 255).

### Legacy compat shim

`warehouse.html` and `businessone.html` use exactly these:

`container`, `container-fluid`, `row`, `col-lg-12`, `table`, `table-bordered`,
`table-responsive`, `btn`, `btn-lg`, `btn-outline`, `text-center`

`btn-outline` is a Freelancer skin class, not Bootstrap, so it needs porting
regardless. The shim reproduces current appearance so those two pages do not change
visually. Roughly 70 lines.

## Template architecture

| File | Change |
|---|---|
| `_layouts/default.html` | Rebuilt — home page |
| `_layouts/case-study.html` | **New** — project pages |
| `_layouts/hub.html` | **New** — Creative Hub |
| `_layouts/resume.html` | Rebuilt |
| `_layouts/page.html` | Unchanged — serves the two legacy pages |
| `_layouts/style.css` | Includes `design.css` only |
| `_includes/site_header.html` | **New** — sticky blurred nav |
| `_includes/site_footer.html` | **New** |
| `_includes/modals.html` | Deleted |
| `_includes/nav.html`, `header.html`, `about.html`, `portfolio_grid.html`, `contact_static.html` | Folded into the new layouts |
| `_includes/contact_disqus.html`, `js_disqus.html` | Deleted (unused, placeholder shortname) |
| `_includes/css/main.css` | Deleted |
| `_includes/js.html` | Deleted — carousel script inlines into the home layout |

### Navigation

Shared header takes an `active` parameter. The active page's pill gets the purple
fill. Home has no matching nav item, so on home the Contact pill keeps its CTA fill,
matching the design.

## Collection and URLs

```yaml
collections:
  projects:
    output: true
    permalink: /projects/:name/
```

Project files are renamed with `git mv` to drop the date prefixes, which are
meaningless for a collection and would leak into URLs. `date:` stays in front matter
for ordering.

| From | To | URL |
|---|---|---|
| `2025-08-20-e-vote-mpc.markdown` | `e-vote-mpc.markdown` | `/projects/e-vote-mpc/` |
| `2025-03-01-pos-journal-functional-spec.markdown` | `pos-journal-functional-spec.markdown` | `/projects/pos-journal-functional-spec/` |
| `2025-02-20-wms-functional-spec-template.markdown` | `wms-functional-spec-template.markdown` | `/projects/wms-functional-spec-template/` |
| `2025-02-15-rca-bug-ticket.markdown` | `rca-bug-ticket.markdown` | `/projects/rca-bug-ticket/` |

Sorted by `date` descending, which reproduces the design's card order exactly:
e-vote, POS, WMS, RCA.

## Front matter schema

New fields per project, so layouts render from data rather than hand-written markup:

| Field | Purpose | Values |
|---|---|---|
| `code` | Mono ticket code on card and page | `FYP · 2025`, `FIN-INT-101`, `WMS-FST-0104`, `WMS-BUG-1042` |
| `status` | Status pill text | `FEATURED`, `UNDER CONSTRUCTION`, `READY FOR REVIEW` |
| `status_tone` | Pill colour | `purple`, `gold`, `green` |
| `card_summary` | Card description paragraph | prose |
| `card_quote` | Mono callout on card | prose |
| `client` | Card and page footer | existing value |
| `full_spec` / `full_spec_label` | "View full spec →" target | `businessone.html`, `warehouse.html`, GitHub URL |
| `ticket` | Meta table rows | nested map, see below |

`ticket` is a nested map so the meta table renders as a loop rather than six
hardcoded rows:

```yaml
ticket:
  type: Story
  priority: High
  labels: [Functional Requirement, Finance-Integration, POS-Engine, General-Ledger, ERP]
  owner: Business Analyst
  signoff: Product Owner (Finance), Lead Integration Architect, Lead QE, ...
```

Projects with no ticket metadata (e-vote) omit the key and the layout skips the
table.

Existing `modal-id`, `img` and `alt` are dropped with the modals.

## Page specs

### Home

Sticky header → hero (2-col: copy + headshot, `1.15fr / .85fr`) → About (2-col:
prose card + three stat cards, `1.4fr / 1fr`) → resume deep-link pills → Portfolio
(horizontal snap-scroll carousel, 300px cards, prev/next buttons) → Contact (dark
`#241C36` panel, details + Formspree form) → footer.

**About moves above Portfolio**, reversing the current order. The review found the
existing order asks visitors to judge the work before knowing who she is.

### Case study

Sticky header → "← All case studies" → mono ticket badge → title → meta table →
full project body, restyled → footer meta strip with client, date and "View full
spec →".

The body keeps its existing HTML. The work is stripping 381 inline styles and the
Bootstrap table classes, then styling through `.cs-body` descendant selectors.
`text-align: justify` is removed throughout (review finding: produces visible rivers
at this column width with no hyphenation).

The hardcoded dark JSON block in the POS project (`#0f172a`, `#38bdf8`, `#fde047`)
is retargeted to `--purple-deep` / `--purple-code` / `--purple-code-ink`.

### Resume

Header → title + Download PDF → competency matrix (4 cards, **new**) → Experience →
Education → Skills → Achievements → Volunteer → Publications → Co-Curricular →
References → footer. Section IDs preserved so the home page's deep links keep
working.

The current profile photo section is dropped; the design has none.

### Creative Hub

Warmer `#F6F1E7` background, Lora serif, squared nav (12px/8px radius, mono labels)
rather than the 999px pills. Intro → YouTube → Fiction collection (16 numbered
Sinhala entries) → Article collection (9 entries) → footer.

## Responsive

The designs specify no media queries — every grid is fixed-column, because the design
canvas renders desktop-width. This is filled in, not invented:

| Breakpoint | Behaviour |
|---|---|
| ≤ 900px | All 2-col grids collapse to 1 col (hero, about, contact, references) |
| ≤ 900px | Competency matrix 4 → 2 col; ≤ 560px → 1 col |
| ≤ 900px | Resume skills grid 2 → 1 col |
| ≤ 700px | Section padding 48px → 20px |
| ≤ 700px | Header stacks; nav becomes a horizontal scroll row with a gradient affordance |
| All | Case-study tables wrapped in `overflow-x: auto` |

The table wrapper is critical — the POS data dictionary is six columns and cannot fit
a phone. The review also flagged that the existing pill nav scrolls with no visual
hint, so the gradient affordance addresses that.

## Content, config and assets

- **Headshot.** A 668×890 professional headshot is recovered from
  `.image-slots.state.json` in the design project and becomes the hero image. This is
  the review's single highest-impact fix: the current hero is a stage-performance
  photo carrying a third-party photographer credit burned into the image.
- **Card images.** All four design slots are empty. Cards get a tinted band carrying
  the mono ticket code in the card's status colour. Real screenshots deferred.
- **Postal address** removed from the footer (review flagged it twice: a privacy
  issue, and rendered at the same weight as section headings).
- **Visitor badge** removed — not in the designs.
- **Phone** `(+94) 71 632 5463` added to Contact, per the design.
- **Contact form** keeps Formspree. HTML5 `required` replaces `jqBootstrapValidation`.
  Visually-hidden `<label>`s are added: the design is placeholder-only, which the
  review flagged, and placeholders vanish once typing starts.
- **Fonts** swap to Manrope + IBM Plex Mono (+ Lora, Noto Sans Sinhala on the hub).
- **Font Awesome** stays. The new designs use no icons, but the legacy pages and
  resume body do.
- **Dead config** removed: `site.color.*`, `site.skills`, `disqus_shortname`,
  footer `address`.

### JavaScript

Removed: `bootstrap.min.js`, `bootstrap.js`, `jquery-1.11.0.js`, `jquery.easing.min.js`,
`jqBootstrapValidation.js`, `classie.js`, `cbpAnimatedHeader.js`,
`cbpAnimatedHeader.min.js`, `freelancer.js`, `contact_me_static.js`.

Added: ~20 lines of vanilla JS for the portfolio carousel, ported from the design's
own `DCLogic` block (already vanilla — eased `scrollLeft` animation over 300ms).
It is the only script on the site, and the carousel only exists on the home page, so
it is inlined in `_layouts/default.html` rather than kept as a shared include.
`_includes/js.html` is deleted outright.

Net removal is roughly 236KB of JS (ten files) and 124KB of CSS
(`bootstrap.min.css` 114KB + `main.css` 9KB).

## Out of scope

- Retheming `warehouse.html` / `businessone.html` to the new palette.
- Replacing card placeholder bands with real screenshots.
- The live "UNDER CONSTRUCTION" status on two projects. This is a content decision
  for the site owner, flagged by the review, not a design change.
- `feed.xml`, which currently emits zero entries because it reads `site.posts`.
  Tracked separately.

## Verification

1. `bundle exec jekyll build` completes with no destination conflicts.
2. `bundle exec rake test` — `html-proofer` over `_site`, confirming no broken
   internal link or missing image.
3. `jekyll serve` and confirm 200 for `/`, `/resume.html`, `/extracurriculars.html`,
   `/warehouse.html`, `/businessone.html` and all four `/projects/*/` URLs.
4. Confirm every `resume.html#…` deep link from the home page resolves to a real
   section ID.
5. Confirm `_site` contains no reference to the removed JS or `main.css`.

**Not verifiable here:** rendered appearance. There is no browser in this session, so
the visual check at desktop and phone width is the site owner's. The specific things
to look at are the hero at phone width, the case-study tables' horizontal scroll, and
the two legacy pages confirming the compat shim held.
