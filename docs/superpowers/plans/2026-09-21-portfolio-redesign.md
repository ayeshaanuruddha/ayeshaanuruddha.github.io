# Portfolio Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Port four finished Claude Design pages into the Jekyll site, removing Bootstrap and jQuery entirely and converting the portfolio modals into real case-study pages.

**Architecture:** One hand-written stylesheet (`design.css`) built from design tokens replaces `bootstrap.min.css` + the Freelancer skin `main.css`. Shared sticky-header and footer includes serve four layouts (home, case-study, resume, hub). The `_projects` collection flips to `output: true`, giving each project a page at `/projects/:name/`. A ~70-line compat shim preserves the two hand-built legacy pages unchanged.

**Tech Stack:** Jekyll 4.x, Liquid, kramdown, vanilla CSS/JS. No framework, no build step beyond Jekyll. `html-proofer` via `bundle exec rake test`.

**Spec:** `docs/superpowers/specs/2026-09-21-portfolio-redesign-design.md`

## Global Constraints

- **No JavaScript framework and no jQuery.** The only script on the finished site is the home carousel (~20 lines, vanilla).
- **Palette is fixed.** Use the exact hex values in the spec's token table. Do not invent shades.
- **Fonts:** Manrope (500/600/700/800) + IBM Plex Mono (400/500/600) sitewide; Lora + Noto Sans Sinhala additionally on the Creative Hub only.
- **`warehouse.html` and `businessone.html`: their page CONTENT is frozen.** Their headings, tables, status pills, diagrams, floating home button and inline styling must not be touched or restyled — that is what "out of scope" means, and the compat shim exists to keep that content rendering as it does today. **Shared site chrome is deliberately NOT frozen.** Those two pages route through `_layouts/page.html`, so they receive the new header and the new footer like every other page. This is intended: the new footer removes the home address (a privacy finding the review raised twice), the visitor badge, and the theme's mobile scroll-top button that the review flagged as a Critical double-FAB overlap on `businessone.html`. Keeping the old footer for these two pages would re-publish the address, restore that overlap, and pin `footer.html` alive against its Task 8 deletion.
- **Preserve all project body content.** ~1,000 lines across four files. Styling changes only; no prose is deleted.
- **`text-align: justify` must not survive anywhere.** Review finding.
- **Every case-study table must be horizontally scrollable.** The POS data dictionary is six columns.
- Site is on branch `new-look`. Do not push.

## Spec Corrections

Two errors in the spec, corrected here and authoritative:

1. **`_layouts/page.html` is NOT unchanged.** It includes `nav.html` and `js.html`, both of which get deleted. It must be updated to use the new chrome (Task 2).
2. **`_config.yml` has no `exclude:` key**, so `docs/` would be published into `_site` — publishing internal specs and plans to the live site. An `exclude` entry is required (Task 1).

## File Structure

| File | Responsibility |
|---|---|
| `_includes/css/design.css` | **New.** All site styling: tokens, base, components, hub block, compat shim, media queries |
| `_layouts/style.css` | Includes `design.css` only |
| `_includes/site_header.html` | **New.** Sticky blurred header + pill nav, takes `active` param |
| `_includes/site_footer.html` | **New.** Shared footer |
| `_includes/head.html` | Font links swapped; `feed.xml` link made absolute |
| `_layouts/default.html` | Home page |
| `_layouts/case-study.html` | **New.** Project pages |
| `_layouts/resume.html` | Resume |
| `_layouts/hub.html` | **New.** Creative Hub |
| `_layouts/page.html` | Legacy shell for `warehouse.html` / `businessone.html` |
| `_projects/*.markdown` | Renamed, front matter extended, bodies restyled |
| `_config.yml` | Collection output, `exclude`, dead-key removal |

**Deleted:** `_includes/css/main.css`, `nav.html`, `header.html`, `about.html`, `portfolio_grid.html`, `modals.html`, `contact_static.html`, `contact_disqus.html`, `js_disqus.html`, `js.html`, all of `js/*` except nothing (whole directory), `css/font-awesome/` stays.

---

### Task 1: CSS foundation and build hygiene

**Files:**
- Create: `_includes/css/design.css`
- Create: `docs/superpowers/design-source/*.dc.html` (reference copies)
- Modify: `_layouts/style.css`, `_includes/head.html`, `_config.yml`
- Delete: `_includes/css/main.css`, `_includes/css/bootstrap.min.css`

**Interfaces:**
- Produces: the full token set as `:root` custom properties; utility class names `.wrap`, `.wrap-narrow`, `.eyebrow`, `.pill`, `.btn-solid`, `.btn-ghost`, `.card`, `.status`, consumed by every later task.

- [ ] **Step 1: Save the design files to disk so later tasks can read them**

The designs live only in the Claude Design project, not in this repo. Pull them down first — every later task references them.

```
DesignSync get_file, projectId e4b2ed60-d9f2-4d17-b253-e4a4130e46f6, for each path:
  Home-Final.dc.html, Resume-Final.dc.html, CreativeHub-Final.dc.html, CaseStudy-Final.dc.html
Save each to docs/superpowers/design-source/<name>
```

If DesignSync returns an authorization error, stop and ask the user to run `/design-login`. Do not proceed from memory.

- [ ] **Step 2: Add the assertion that `docs/` is not published**

Run this now to see it fail:

```bash
bundle exec jekyll build >/dev/null 2>&1 && test -d _site/docs && echo "FAIL: docs published" || echo "PASS: docs not published"
```

Expected: `FAIL: docs published`

- [ ] **Step 3: Add `exclude` to `_config.yml`**

Add near the build settings block:

```yaml
exclude:
  - docs
  - Gemfile
  - Gemfile.lock
  - Rakefile
  - freelancer-theme-jekyll.gemspec
  - README.md
  - screenshot.png
  - vendor
```

- [ ] **Step 4: Re-run the assertion**

```bash
bundle exec jekyll build >/dev/null 2>&1 && test -d _site/docs && echo "FAIL: docs published" || echo "PASS: docs not published"
```

Expected: `PASS`

- [ ] **Step 5: Write `_includes/css/design.css` — tokens and base**

```css
:root{
  --bg:#FAF8F4; --bg-hub:#F6F1E7;
  --surface:#FFFFFF; --surface-alt:#F1EEE7; --surface-quiet:#F7F5F1;
  --ink:#1E1A2E; --ink-hub:#241C14; --body:#3A3745;
  --muted:#5B5867; --muted-hub:#5B4E3D;
  --line:#E8E3DD; --line-strong:#D9D2C6;
  --line-hub:#D9CDB6; --line-hub-soft:#E4DAC4;
  --purple:#5B3A8E; --purple-dark:#3E2860; --purple-deep:#241C36;
  --purple-tint:#EFE9F8; --purple-tint-2:#F0EAF9; --purple-line:#E2D6F2;
  --purple-code:#B7A9E0; --purple-code-ink:#E4E1F0; --purple-ink:#544685;
  --green:#1F7A56; --green-dark:#154A35; --green-tint:#E4F3EC; --green-line:#CDE9DC;
  --gold:#A9822E; --gold-ink:#7A5A11; --gold-tint:#F6EDD8; --gold-line:#EBDBAE;
  --gold-bright:#D9B24A;
  --sans:'Manrope',system-ui,sans-serif;
  --mono:'IBM Plex Mono',ui-monospace,monospace;
  --serif:'Lora',Georgia,serif;
  --sinhala:'Noto Sans Sinhala',var(--serif);
  --shadow-hero:0 20px 45px -12px rgba(30,26,46,.18);
  --shadow-card:0 20px 40px -14px rgba(30,26,46,.2);
}
*,*::before,*::after{box-sizing:border-box;}
body{margin:0;background:var(--bg);color:var(--ink);font-family:var(--sans);
  font-size:16px;line-height:1.6;-webkit-font-smoothing:antialiased;}
img{max-width:100%;height:auto;}
a{color:var(--purple);text-decoration:none;}
a:hover{color:var(--purple-dark);}
h1,h2,h3,h4{margin:0 0 12px;line-height:1.25;font-weight:800;letter-spacing:-.01em;}
h1{font-size:clamp(30px,5.4vw,58px);line-height:1.08;letter-spacing:-.02em;}
h2{font-size:30px;} h3{font-size:19px;} h4{font-size:16px;}
p{margin:0 0 14px;}
code,pre{font-family:var(--mono);}
.wrap{max-width:1200px;margin:0 auto;padding:0 48px;}
.wrap-narrow{max-width:960px;margin:0 auto;padding:0 48px;}
.wrap-tight{max-width:840px;margin:0 auto;padding:0 48px;}
.eyebrow{font-size:13px;font-weight:700;color:var(--green);
  text-transform:uppercase;letter-spacing:.08em;margin-bottom:10px;}
.sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;
  overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0;}
```

- [ ] **Step 6: Append shared components to `design.css`**

Buttons, pills, status badges, cards, tables, code blocks, Gherkin rows. Read
`docs/superpowers/design-source/Home-Final.dc.html` and `CaseStudy-Final.dc.html`
and translate each repeated inline-style cluster into one class. Required class
names, because later tasks reference them:

```css
.btn-solid{display:inline-block;font-size:14px;font-weight:700;background:var(--ink);
  color:var(--bg);padding:15px 26px;border-radius:12px;}
.btn-ghost{display:inline-block;font-size:14px;font-weight:700;color:var(--ink);
  padding:15px 26px;border-radius:12px;border:1px solid var(--line);}
.pill{display:inline-block;font-size:13px;font-weight:600;color:var(--purple);
  background:var(--surface-alt);padding:10px 18px;border-radius:999px;}
.status{font-size:11px;font-weight:700;padding:4px 12px;border-radius:999px;}
.status--purple{color:var(--purple);background:var(--purple-tint);}
.status--green{color:var(--green);background:var(--green-tint);}
.status--gold{color:var(--gold-ink);background:var(--gold-tint);}
.card{background:var(--surface);border:1px solid var(--line);border-radius:20px;padding:28px;}
.mono-note{background:var(--surface-quiet);border-radius:12px;padding:12px 14px;
  font-family:var(--mono);font-size:12px;line-height:1.6;color:var(--muted);}
.codeblock{background:var(--purple-deep);border-radius:16px;padding:22px 24px;
  margin:0 0 32px;overflow-x:auto;}
.codeblock pre{margin:0;font-size:12.5px;line-height:1.7;color:var(--purple-code-ink);}
.codeblock__label{font-family:var(--mono);font-size:11px;color:var(--purple-code);margin-bottom:10px;}
```

- [ ] **Step 7: Append the legacy compat shim to `design.css`**

These ten classes are all `warehouse.html` and `businessone.html` use. Values
reproduce Bootstrap 3 / Freelancer appearance so those pages do not shift.

```css
/* Legacy compat — keeps warehouse.html and businessone.html unchanged.
   Do not use these classes in new markup. */
.container{width:100%;max-width:1170px;margin:0 auto;padding:0 15px;}
.container-fluid{width:100%;margin:0 auto;padding:0 15px;}
.row{margin:0 -15px;}
.row::after{content:"";display:table;clear:both;}
.col-lg-12{position:relative;width:100%;padding:0 15px;float:left;}
.text-center{text-align:center;}
.table{width:100%;max-width:100%;margin-bottom:20px;border-collapse:collapse;}
.table>tbody>tr>td,.table>thead>tr>th{padding:8px;line-height:1.42857143;
  vertical-align:top;border-top:1px solid #ddd;}
.table>thead>tr>th{vertical-align:bottom;border-bottom:2px solid #ddd;}
.table-bordered,.table-bordered>tbody>tr>td,.table-bordered>thead>tr>th{border:1px solid #ddd;}
.table-responsive{width:100%;margin-bottom:15px;overflow-x:auto;}
.btn{display:inline-block;margin-bottom:0;font-weight:400;text-align:center;
  vertical-align:middle;cursor:pointer;border:1px solid transparent;
  padding:6px 12px;font-size:14px;line-height:1.42857143;border-radius:4px;}
.btn-lg{padding:10px 16px;font-size:18px;line-height:1.33;border-radius:6px;}
.btn-outline{color:#fff;background:transparent;border-color:#fff;}
.btn-outline:hover,.btn-outline:focus{color:#18bc9c;background:#fff;border-color:#fff;}
.img-responsive{display:block;max-width:100%;height:auto;}
.img-centered{margin:0 auto;}

/* Legacy pages keep Bootstrap's base metrics. All their inline sizing is in em
   units (font-size:1.05em etc.), so the base must stay 15px or every table and
   pill on those pages reflows ~7% larger. Typography and background deliberately
   follow the new design — see the ruling in the ledger. */
body.legacy{font-size:15px;line-height:1.42857143;}
```

- [ ] **Step 8: Point `_layouts/style.css` at the new stylesheet**

Replace its entire contents with:

```
{% include css/design.css %}
```

- [ ] **Step 9: Swap the fonts in `_includes/head.html`**

Replace the two Google Fonts `<link>` tags (Montserrat, Lato) with:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Manrope:wght@500;600;700;800&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

Also fix the relative feed link, which breaks on `/projects/*/` pages:

```html
<link rel="alternate" type="application/rss+xml" title="RSS" href="{{ '/feed.xml' | absolute_url }}">
```

- [ ] **Step 10: Delete the old stylesheets and verify**

```bash
git rm -q _includes/css/main.css _includes/css/bootstrap.min.css
bundle exec jekyll build 2>&1 | tail -3
grep -c 'Bootswatch\|normalize.css' _site/style.css || echo "PASS: no bootstrap in output"
grep -c 'FAF8F4' _site/style.css && echo "PASS: tokens present"
```

Expected: build succeeds; no Bootswatch; tokens present.

- [ ] **Step 11: Commit**

```bash
git add -A _includes/css _layouts/style.css _includes/head.html _config.yml docs
git commit -m "feat(css): replace bootstrap and freelancer skin with design.css"
```

---

### Task 2: Shared header and footer

**Files:**
- Create: `_includes/site_header.html`, `_includes/site_footer.html`
- Modify: `_layouts/page.html`

**Do NOT delete `nav.html`, `header.html` or `footer.html` in this task.**
`footer.html` is still included by `default.html` and `resume.html`, and `nav.html`
and `header.html` by `default.html` — those layouts are not rebuilt until Tasks 3 and
6. Jekyll errors on a missing include, so deleting them here breaks the very build
this task verifies. Their removal is deferred to Task 8, which asserts no references
remain before deleting.

**Interfaces:**
- Consumes: `.wrap` and token variables from Task 1.
- Produces: `{% include site_header.html active="work" %}` — accepted `active` values are `about`, `work`, `resume`, `hub`, `contact`, or omitted. And `{% include site_footer.html %}`.

- [ ] **Step 1: Write `_includes/site_header.html`**

```html
<header class="site-header">
  <a class="site-header__brand" href="{{ '/' | relative_url }}">
    <span class="site-header__mark">AA</span>{{ site.title }}
  </a>
  <nav class="site-nav" aria-label="Main">
    <a href="{{ '/#about' | relative_url }}"{% if include.active == 'about' %} class="is-active"{% endif %}>About</a>
    <a href="{{ '/#portfolio' | relative_url }}"{% if include.active == 'work' %} class="is-active"{% endif %}>Work</a>
    <a href="{{ '/resume.html' | relative_url }}"{% if include.active == 'resume' %} class="is-active"{% endif %}>Resume</a>
    <a href="{{ '/extracurriculars.html' | relative_url }}"{% if include.active == 'hub' %} class="is-active"{% endif %}>Creative Hub</a>
    <a href="{{ '/#contact' | relative_url }}" class="{% if include.active == 'contact' %}is-active{% else %}is-cta{% endif %}">Contact</a>
  </nav>
</header>
```

Note on the active state: the designs highlight the current page's pill. Home has
no matching item, so Contact keeps a CTA fill there. `.is-active` and `.is-cta`
render identically; two names keep the intent readable.

- [ ] **Step 2: Add header and footer CSS to `design.css`**

```css
.site-header{position:sticky;top:0;z-index:20;background:rgba(250,248,244,.85);
  backdrop-filter:blur(10px);border-bottom:1px solid var(--line);display:flex;
  align-items:center;justify-content:space-between;padding:18px 48px;gap:16px;flex-wrap:wrap;}
.site-header__brand{display:flex;align-items:center;gap:10px;font-size:15px;
  font-weight:800;color:var(--ink);}
.site-header__mark{display:inline-flex;align-items:center;justify-content:center;
  width:32px;height:32px;border-radius:10px;background:var(--purple);
  color:var(--bg);font-size:12px;font-weight:700;}
.site-nav{display:flex;gap:6px;font-size:14px;font-weight:600;
  background:var(--surface-alt);border-radius:999px;padding:5px;}
.site-nav a{color:var(--muted);padding:9px 18px;border-radius:999px;white-space:nowrap;}
.site-nav a.is-active,.site-nav a.is-cta{color:var(--bg);background:var(--purple);}
.site-footer{border-top:1px solid var(--line);padding:24px 48px;display:flex;
  justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px;
  font-size:13px;color:var(--muted);}
.site-footer a{color:var(--muted);}
```

- [ ] **Step 3: Write `_includes/site_footer.html`**

```html
<footer class="site-footer">
  <span>&copy; {{ site.footer.copyright }} {{ site.time | date: '%Y' }}</span>
  <a href="{{ '/extracurriculars.html' | relative_url }}">Creative Hub &mdash; fiction &amp; essays &rarr;</a>
  <a href="#page-top">Back to top &uarr;</a>
</footer>
```

The postal address and visitor badge are deliberately absent (spec, privacy finding).

- [ ] **Step 4: Update `_layouts/page.html` so the legacy pages survive the deletions**

It currently includes `nav.html` and `js.html`, both being deleted. Replace its body with:

```html
<!DOCTYPE html>
<html lang="en">
{% include head.html %}
<body id="page-top" class="legacy">
  {% include site_header.html %}
  {{ content }}
  {% include site_footer.html %}
</body>
</html>
```

`warehouse.html` and `businessone.html` already hide `nav`/`header` in their own
`<style>` blocks, so the new header is invisible there — matching today's behaviour.

**`class="legacy"` is required, not optional.** Every piece of inline sizing on those
two pages is in `em` units, so they need the 15px base that `body.legacy` restores in
`design.css`. Without the class, every table and pill on both pages reflows about 7%
larger.

- [ ] **Step 5: Verify the legacy pages still build and now carry the new chrome**

No deletions here — see the Files note above.

```bash
bundle exec jekyll build 2>&1 | tail -3
grep -c 'site-header' _site/warehouse.html && echo "PASS: new header on legacy page"
grep -c 'table-bordered' _site/businessone.html && echo "PASS: legacy classes intact"
grep -c 'class="legacy"' _site/businessone.html && echo "PASS: legacy body class"
grep -c 'site-footer' _site/index.html || echo "expected 0 — home is rebuilt in Task 3"
```

Expected: build succeeds with no "Could not locate the included file" error; the
first three greps non-zero. The home page still renders old chrome at this point,
which is correct — Task 3 rebuilds it.

- [ ] **Step 6: Commit**

```bash
git add -A _includes _layouts/page.html
git commit -m "feat(chrome): shared sticky header and footer includes"
```

---

### Task 3: Home page

**Files:**
- Modify: `_layouts/default.html`
- Create: `img/profile-headshot.webp`
- Delete: `_includes/about.html`, `portfolio_grid.html`, `modals.html`, `contact_static.html`, `contact_disqus.html`

**Do NOT delete `js_disqus.html`.** It is included by `_includes/js.html`, which
`_layouts/resume.html` still loads until Task 6. The reference sits inside a
`{% if site.contact == "disqus" %}` branch that is currently false, so it would not
error today — but that is a fragile reason to delete a referenced file. Task 8
removes it together with `js.html` itself. The other five are referenced only by
`_layouts/default.html`, which this task rewrites, so they are safe to remove here.

**Interfaces:**
- Consumes: `site_header.html`, `site_footer.html`, all Task 1 classes. **Task 4 runs before this task**, so the project front matter (`code`, `status`, `status_tone`, `card_summary`, `card_quote`, `client`) already exists and `p.url` resolves.
- Produces: `#page-top`, `#about`, `#portfolio`, `#contact` anchors that the nav and resume deep-links target. Carousel track id `portfolio-track`.
- **Binding class-name contract** — Task 9 writes media queries against these exact names, so use them verbatim: `.hero` (hero grid), `.about-grid` (about 2-col), `.portfolio-track` (carousel), `.pcard` (project card), `.contact-panel` (dark contact grid).

- [ ] **Step 1: Install the headshot**

It was extracted from the design project's `.image-slots.state.json` to
`/private/tmp/claude-501/.../scratchpad/m-home-headshot.webp` (668×890). If that
scratch file is gone, re-extract: read `.image-slots.state.json` via DesignSync,
base64-decode key `m-home-headshot`.

```bash
cp <scratch>/m-home-headshot.webp img/profile-headshot.webp
sips -g pixelWidth -g pixelHeight img/profile-headshot.webp
```

Expected: 668 × 890.

- [ ] **Step 2: Rebuild `_layouts/default.html`**

Port section by section from `docs/superpowers/design-source/Home-Final.dc.html`,
converting inline styles to classes. Section order — note About now precedes
Portfolio, reversing today's order:

1. `{% include site_header.html %}`
2. `<section id="page-top" class="hero">` — 2-col `1.15fr / .85fr`: availability badge, `h1`, lede, `.btn-solid` (CV) + `.btn-ghost` (case studies); right column the headshot in a 24px-radius frame with `--shadow-hero`
3. `<section id="about">` — 2-col `1.4fr / 1fr`: prose `.card` + three stat cards (purple `2+ yrs`, green `4`, gold `BSc Hons`), then the resume deep-link `.pill` row
4. `<section id="portfolio">` — `--surface-alt` band, heading + prev/next buttons, then `.portfolio-track` looping `site.projects`
5. `<section id="contact">` — `--purple-deep` panel, details column + Formspree form
6. `{% include site_footer.html %}`

Cards loop over the collection rather than being hand-written:

```liquid
{% assign projects = site.projects | sort: "date" | reverse %}
{% for p in projects %}
  <article class="pcard">
    <div class="pcard__band pcard__band--{{ p.status_tone }}">{{ p.code }}</div>
    <div class="pcard__body">
      <div class="pcard__meta">
        <span class="pcard__code">{{ p.code }}</span>
        <span class="status status--{{ p.status_tone }}">{{ p.status }}</span>
      </div>
      <h3>{{ p.title }}</h3>
      <p>{{ p.card_summary }}</p>
      <div class="mono-note">{{ p.card_quote }}</div>
      <div class="pcard__foot">
        <span>{{ p.client }}</span>
        <a href="{{ p.url | relative_url }}">Explore &rarr;</a>
      </div>
    </div>
  </article>
{% endfor %}
```

`.pcard__band` is the tinted band standing in for imagery (spec, open item A):
`height:170px; display:flex; align-items:center; justify-content:center;
font-family:var(--mono); font-size:13px; letter-spacing:.08em;` with
`--purple-tint`/`--green-tint`/`--gold-tint` backgrounds per tone.

- [ ] **Step 3: Keep the contact form accessible**

The design is placeholder-only, which the review flagged. Keep the Formspree
action and add visually-hidden labels plus HTML5 validation:

```html
<form action="https://formspree.io/f/xgaerkdz" method="POST" class="contact-form">
  <label class="sr-only" for="cf-name">Name</label>
  <input id="cf-name" type="text" name="name" placeholder="Name" required>
  <label class="sr-only" for="cf-email">Email address</label>
  <input id="cf-email" type="email" name="_replyto" placeholder="Email address" required>
  <input type="hidden" name="_subject" value="New submission!">
  <input type="text" name="_gotcha" style="display:none">
  <label class="sr-only" for="cf-msg">Message</label>
  <textarea id="cf-msg" name="message" rows="4" placeholder="Message" required></textarea>
  <button type="submit">Send message</button>
</form>
```

- [ ] **Step 4: Inline the carousel script at the end of `default.html`**

Ported from the design's own `DCLogic` block — already vanilla.

```html
<script>
(function(){
  var track=document.getElementById('portfolio-track');
  if(!track)return;
  function step(dir){
    var start=track.scrollLeft,target=start+dir*324,t0=performance.now();
    function frame(now){
      var t=Math.min(1,(now-t0)/300),e=1-Math.pow(1-t,3);
      track.scrollLeft=start+(target-start)*e;
      if(t<1)requestAnimationFrame(frame);
    }
    requestAnimationFrame(frame);
  }
  document.querySelectorAll('[data-scroll]').forEach(function(b){
    b.addEventListener('click',function(){step(Number(b.dataset.scroll));});
  });
})();
</script>
```

- [ ] **Step 5: Delete the superseded includes and verify**

```bash
git rm -q _includes/about.html _includes/portfolio_grid.html _includes/modals.html \
  _includes/contact_static.html _includes/contact_disqus.html
bundle exec jekyll build 2>&1 | tail -3
for id in page-top about portfolio contact; do
  grep -q "id=\"$id\"" _site/index.html && echo "PASS: #$id" || echo "FAIL: #$id"; done
grep -c 'portfolioModal' _site/index.html || echo "PASS: no modals"
grep -c 'profile-headshot' _site/index.html && echo "PASS: headshot wired"
grep -c '/projects/' _site/index.html && echo "PASS: home links to project pages"
grep -o 'FIN-INT-101\|WMS-FST-0104\|WMS-BUG-1042' _site/index.html | sort -u
```

Expected: four anchors PASS, no modals, headshot present, project links present,
and all three ticket codes rendered from front matter (proving the cards read real
data, not empty values).

- [ ] **Step 6: Commit**

```bash
git add -A _layouts/default.html _includes img/profile-headshot.webp
git commit -m "feat(home): rebuild home page to new design"
```

---

### Task 4: Project collection becomes real pages

**Files:**
- Modify: `_config.yml`
- Rename: all four `_projects/*.markdown`
- Create: `_layouts/case-study.html`

**Interfaces:**
- Consumes: Task 1 classes, Task 2 chrome.
- Produces: `/projects/<slug>/` URLs and the front-matter contract (`code`, `status`, `status_tone`, `card_summary`, `card_quote`, `client`, `full_spec`, `full_spec_label`, `ticket`) that Task 3's card loop already reads.

- [ ] **Step 1: Assert the project URLs do not exist yet**

```bash
bundle exec jekyll build >/dev/null 2>&1
test -f _site/projects/pos-journal-functional-spec/index.html \
  && echo "FAIL: already exists" || echo "PASS: not yet built"
```

Expected: `PASS: not yet built`

- [ ] **Step 2: Flip the collection to output in `_config.yml`**

```yaml
collections:
  projects:
    output: true
    permalink: /projects/:name/
```

- [ ] **Step 3: Drop the date prefixes from the filenames**

Collections do not need them, and they would leak into URLs.

```bash
cd _projects
git mv 2025-08-20-e-vote-mpc.markdown e-vote-mpc.markdown
git mv 2025-03-01-pos-journal-functional-spec.markdown pos-journal-functional-spec.markdown
git mv 2025-02-20-wms-functional-spec-template.markdown wms-functional-spec-template.markdown
git mv 2025-02-15-rca-bug-ticket.markdown rca-bug-ticket.markdown
cd ..
```

- [ ] **Step 4: Extend each file's front matter**

Remove the dead `#layout: default` comment, `modal-id`, `img` and `alt`. Add a
`title` (each file currently has none — the card and page heading need it) and the
card/ticket fields. Values come from `CaseStudy-Final.dc.html` and
`Home-Final.dc.html`. Example for `pos-journal-functional-spec.markdown`:

```yaml
---
layout: case-study
title: POS Webhook Ingestion & Balanced Journal Entries
date: 2025-03-01
code: FIN-INT-101
status: UNDER CONSTRUCTION
status_tone: gold
client: Omnichannel Retail Enterprise
project-date: March 2025
category: Financial Systems & ERP Integration
card_summary: >-
  Integration contract and double-entry accounting engine between an edge POS
  gateway and a cloud ERP general ledger, built under strict SOX 404 compliance.
card_quote: >-
  GIVEN a $5.40 sale WHEN the payment.captured webhook fires THEN ΣDebits −
  ΣCredits = $0.00 before ERP commit.
full_spec: /businessone.html
full_spec_label: View full spec
ticket:
  type: Story
  priority: High
  labels: [Functional Requirement, Finance-Integration, POS-Engine, General-Ledger, ERP]
  owner: Business Analyst
  signoff: Product Owner (Finance), Lead Integration Architect, Lead QE, Lead Dev, Principal Controller, SME/Dev
---
```

The other three, from the design's cards:

| File | code | status | tone | full_spec |
|---|---|---|---|---|
| `e-vote-mpc` | `FYP · 2025` | `FEATURED` | `purple` | `https://github.com/ayeshaanuruddha/e-vote-main` (label `View source`) |
| `wms-functional-spec-template` | `WMS-FST-0104` | `UNDER CONSTRUCTION` | `gold` | `/warehouse.html` |
| `rca-bug-ticket` | `WMS-BUG-1042` | `READY FOR REVIEW` | `green` | `/warehouse.html` |

`e-vote-mpc` has no ticket metadata — omit the `ticket:` key entirely.

- [ ] **Step 5: Write `_layouts/case-study.html`**

```html
<!DOCTYPE html>
<html lang="en">
{% include head.html %}
<body id="page-top">
  {% include site_header.html active="work" %}
  <article class="wrap-tight cs">
    <a class="cs__back" href="{{ '/#portfolio' | relative_url }}">&larr; All case studies</a>
    <div class="cs__code">[{{ page.code }}]</div>
    <h1>{{ page.title }}</h1>
    {% if page.ticket %}
    <div class="card cs__ticket">
      <table>
        <tr><th>Type</th><td>{{ page.ticket.type }}</td></tr>
        <tr><th>Priority</th><td>{{ page.ticket.priority }}</td></tr>
        <tr><th>Labels</th><td><span class="cs__labels">{{ page.ticket.labels | join: " · " }}</span></td></tr>
        <tr><th>Ticket Status</th><td><span class="status status--{{ page.status_tone }}">{{ page.status }}</span></td></tr>
        <tr><th>Ticket Owner</th><td>{{ page.ticket.owner }}</td></tr>
        <tr><th>Signed off</th><td>{{ page.ticket.signoff }}</td></tr>
      </table>
    </div>
    {% endif %}
    <div class="cs-body">{{ content }}</div>
    <div class="cs__foot">
      <span>Client: <strong>{{ page.client }}</strong> &middot; Date: <strong>{{ page.project-date }}</strong></span>
      {% if page.full_spec %}
      <a href="{{ page.full_spec | relative_url }}">{{ page.full_spec_label }} &rarr;</a>
      {% endif %}
    </div>
  </article>
  {% include site_footer.html %}
</body>
</html>
```

- [ ] **Step 6: Verify the four pages now build**

```bash
bundle exec jekyll build 2>&1 | tail -3
for s in e-vote-mpc pos-journal-functional-spec wms-functional-spec-template rca-bug-ticket; do
  test -f _site/projects/$s/index.html && echo "PASS: /projects/$s/" || echo "FAIL: /projects/$s/"; done
grep -c 'FIN-INT-101' _site/projects/pos-journal-functional-spec/index.html && echo "PASS: ticket code"
```

Expected: four PASS lines and the ticket code present.

Note: this task runs **before** Task 3. The home page still carries the old
Bootstrap markup at this point, so do not check it here — verifying that the home
page links to `/projects/` belongs to Task 3.

- [ ] **Step 7: Commit**

```bash
git add -A _config.yml _projects _layouts/case-study.html
git commit -m "feat(projects): case-study pages replace portfolio modals"
```

---

### Task 5: Restyle the project bodies

**Files:**
- Modify: all four `_projects/*.markdown` (body only, front matter already done)
- Modify: `_includes/css/design.css` (add `.cs-body` block)

**Interfaces:**
- Consumes: `.cs-body` wrapper from Task 4.
- Produces: nothing downstream.

This is the largest mechanical task: 381 inline styles across four files. **No prose is deleted.**

- [ ] **Step 1: Assert the current state**

```bash
grep -c 'text-align: justify' _projects/*.markdown
grep -c 'class="table' _projects/*.markdown
```

Record the numbers. Target for both after this task: zero.

- [ ] **Step 2: Add the `.cs-body` styling block to `design.css`**

```css
.cs-body{font-size:15px;line-height:1.75;color:var(--body);}
.cs-body h3{font-size:20px;font-weight:800;margin:32px 0 16px;color:var(--ink);}
.cs-body h4{font-size:16px;font-weight:700;margin:24px 0 12px;color:var(--ink);}
.cs-body hr{border:0;border-top:1px solid var(--line);margin:32px 0;}
.cs-body ul{padding-left:20px;line-height:1.7;}
.cs-body li{margin-bottom:6px;}
.cs-body code{background:var(--surface-quiet);padding:2px 6px;border-radius:4px;font-size:12.5px;}
.cs-body .table-wrap{overflow-x:auto;border:1px solid var(--line);
  border-radius:16px;margin:0 0 32px;}
.cs-body table{border-collapse:collapse;width:100%;min-width:520px;}
.cs-body th,.cs-body td{border:1px solid var(--line);padding:12px;
  font-size:13.5px;text-align:left;vertical-align:top;}
.cs-body th{background:var(--surface-alt);font-weight:700;}
.cs-body img{border-radius:12px;border:1px solid var(--line);margin:8px 0;}
```

`min-width:520px` on the table plus `overflow-x` on the wrapper is what makes the
six-column data dictionary usable on a phone.

- [ ] **Step 3: Strip inline styles from the bodies**

Per file, remove every `style="…"` attribute on `<p>`, `<td>`, `<th>`, `<ul>`,
`<li>`, `<table>`, `<thead>`, `<h3>`, `<h4>` and `<div>`. Remove `class="table
table-bordered"` and its variants. Wrap each `<table>` in `<div class="table-wrap">`.

```bash
python3 - <<'EOF'
import re,glob
for p in glob.glob('_projects/*.markdown'):
    s=open(p).read()
    head,sep,body=s.partition('---\n')[2].partition('\n---\n')
    body=re.sub(r'\s*style="[^"]*"','',body)
    body=re.sub(r'\s*class="table[^"]*"','',body)
    body=re.sub(r'<table>',r'<div class="table-wrap"><table>',body)
    body=re.sub(r'</table>',r'</table></div>',body)
    open(p,'w').write('---\n'+head+'\n---\n'+body)
EOF
```

**Then read each file and fix what the regex could not**: the POS project's dark
JSON block is a `<div>` with hardcoded `#0f172a`/`#38bdf8`/`#fde047` spans whose
styles the strip removed. Rebuild it as `.codeblock` from Task 1, keeping the JSON
text verbatim but dropping the per-token colour spans.

- [ ] **Step 4: Re-run the assertions**

```bash
grep -c 'text-align: justify' _projects/*.markdown || echo "PASS: no justify"
grep -c 'class="table' _projects/*.markdown || echo "PASS: no bootstrap tables"
grep -c 'table-wrap' _projects/*.markdown
bundle exec jekyll build 2>&1 | tail -3
```

Expected: both PASS lines; `table-wrap` count matches the original table count
(9 + 14 + 11 + 0 = 34).

- [ ] **Step 5: Confirm no content was lost**

```bash
git diff --stat _projects/
git diff _projects/ | grep '^-' | grep -v '^---' | grep -vE 'style=|class="table' | head -20
```

Expected: the second command prints nothing but whitespace-only or tag-boundary
changes. Any removed prose line is a bug — restore it.

- [ ] **Step 6: Commit**

```bash
git add -A _projects _includes/css/design.css
git commit -m "refactor(projects): strip inline styles, wrap tables for mobile"
```

---

### Task 6: Resume page

**Files:**
- Modify: `_layouts/resume.html`, `resume.html`
- Modify: `_includes/css/design.css`

**Interfaces:**
- Consumes: Task 1 and 2.
- Produces: section IDs `experience`, `education`, `skills`, `achievements`, `volunteer`, `publications`, `co-curricular`, `references` — Task 3's home page deep-links target these.
- **Binding class-name contract** — Task 9 writes media queries against these exact names, so use them verbatim: `.matrix` (competency grid), `.timeline` (experience entry), `.skills-grid` (skills 2-col), `.refs` (references 2-col).

- [ ] **Step 1: Rebuild `_layouts/resume.html`**

```html
<!DOCTYPE html>
<html lang="en">
{% include head.html %}
<body id="page-top">
  {% include site_header.html active="resume" %}
  {{ content }}
  {% include site_footer.html %}
</body>
</html>
```

**The `class="legacy"` currently on this layout's `<body>` must be removed** — the
replacement above already omits it, so do not carry it across. Task 5 added it as a
temporary measure so the old Bootstrap-grid resume kept working once the compat shim
was scoped to `body.legacy`. Once this page is rebuilt to the new design it is no
longer a legacy page, and leaving the class would silently pin it to the old 15px
base type scale. Also drop the inline `style="padding-top: 100px;"` — the sticky
header handles its own spacing now.

- [ ] **Step 2: Rebuild `resume.html` from the design**

Port from `docs/superpowers/design-source/Resume-Final.dc.html`. Order: title +
Download PDF → competency matrix (4 cards, **new section**) → Experience →
Education → Skills → Achievements → Volunteer → Publications → Co-Curricular →
References.

Two changes from today's page: the profile photo section is removed (the design has
none), and the competency matrix is added. All eight existing section IDs are
preserved.

- [ ] **Step 3: Add resume CSS to `design.css`**

```css
.matrix{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:14px;}
.matrix__cell{border-radius:16px;padding:20px;}
.matrix__cell h3{font-size:15px;font-weight:800;margin-bottom:6px;}
.matrix__cell p{font-size:12px;color:var(--muted);line-height:1.6;margin:0;}
.timeline{display:grid;grid-template-columns:minmax(130px,160px) minmax(0,1fr);
  gap:20px;background:var(--surface);border:1px solid var(--line);
  border-radius:20px;padding:28px;margin-bottom:28px;}
.timeline__when{font-family:var(--mono);font-size:12px;color:var(--muted);}
.tags{display:flex;gap:8px;flex-wrap:wrap;}
.tags span{font-size:12px;background:var(--surface);padding:6px 12px;
  border-radius:999px;border:1px solid var(--line);}
.refs{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:16px;}
```

- [ ] **Step 4: Verify the deep links resolve**

```bash
bundle exec jekyll build 2>&1 | tail -3
for id in experience education skills achievements volunteer publications co-curricular references; do
  grep -q "id=\"$id\"" _site/resume.html && echo "PASS: #$id" || echo "FAIL: #$id"; done
grep -o 'resume.html#[a-z-]*' _site/index.html | sort -u
```

Expected: eight PASS lines, and every anchor the home page emits appears in that list.

- [ ] **Step 5: Commit**

```bash
git add -A _layouts/resume.html resume.html _includes/css/design.css
git commit -m "feat(resume): rebuild to new design with competency matrix"
```

---

### Task 7: Creative Hub

**Files:**
- Create: `_layouts/hub.html`
- Modify: `extracurriculars.html`, `_includes/css/design.css`, `_includes/head.html`

**Interfaces:**
- Consumes: Task 1 and 2.
- Produces: nothing downstream.

- [ ] **Step 1: Create `_layouts/hub.html`**

Same shell as the resume layout but with `active="hub"` and `<body class="hub">`.

- [ ] **Step 2: Load the hub's extra fonts conditionally in `head.html`**

Lora and Noto Sans Sinhala are needed only here, so do not make every page pay for them:

```liquid
{% if page.layout == 'hub' %}
<link href="https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,500;0,600;0,700;1,400&family=Noto+Sans+Sinhala:wght@400;500;600&display=swap" rel="stylesheet">
{% endif %}
```

- [ ] **Step 3: Add the hub block to `design.css`**

```css
body.hub{background:var(--bg-hub);color:var(--ink-hub);font-family:var(--serif);}
body.hub .site-nav{border-radius:12px;font-family:var(--mono);font-size:13px;}
body.hub .site-nav a{border-radius:8px;}
.hub-section{max-width:900px;margin:0 auto;padding:56px 48px;
  border-bottom:1px solid var(--line-hub);}
.hub-tag{display:inline-block;font-family:var(--mono);font-size:12px;
  letter-spacing:.12em;font-weight:600;color:var(--bg-hub);
  padding:5px 12px;border-radius:6px;margin-bottom:20px;}
.hub-tag--purple{background:var(--purple);}
.hub-tag--green{background:var(--green);}
.hub-tag--gold{background:var(--gold);}
.hub-list{display:flex;flex-direction:column;font-family:var(--sinhala);}
.hub-list a{display:flex;gap:16px;padding:12px 0;
  border-bottom:1px solid var(--line-hub-soft);font-size:15px;color:var(--ink-hub);}
.hub-list a:last-child{border-bottom:0;}
.hub-list .n{font-family:var(--mono);color:var(--muted-hub);}
```

- [ ] **Step 4: Rebuild `extracurriculars.html`**

Switch front matter to `layout: hub` and drop the `<style>` block that hid the old
navbar. Port from `docs/superpowers/design-source/CreativeHub-Final.dc.html`:
intro, YouTube, Fiction collection, Article collection.

**The design's link list is defective — do not port it.** Take the design's layout,
typography, section structure and ordering, but take every fiction/article
**title-to-URL pairing from the existing `extracurriculars.html`**, which is
authoritative for which post belongs to which story.

The defect, measured: the design omits the story **"The Confession"**
(`https://web.facebook.com/photo/?fbid=3334540376605193&set=gm.707861366452605`),
and that omission shifted every following entry's URL up by one. Eight stories in
the design therefore point at the wrong post —

| Story | Correct URL (live page) | Design wrongly gives it |
|---|---|---|
| චිත්ත මෝහන | `fbid=3117906518268581` | The Confession's URL |
| මෝහා | `fbid=3540936559298906` | චිත්ත මෝහන's |
| Tinkerbell | `posts/4178568962202326` | මෝහා's |
| ආකූල | `fbid=4208413829217839` | Tinkerbell's |
| වෙළෙන්දාගේ දියණිය | `mythologyworld/…/1315632428887074` | ආකූල's |
| පුනරාවර්තන | `349001202263956/…/1788572084973520` | වෙළෙන්දාගේ දියණිය's |
| කාර්මයින් | `fbid=1298510999138425` | පුනරාවර්තන's |
| ආදරණීය SH | `fbid=144887434500793` | කාර්මයින්'s |

Restore "The Confession" in its live-page position (between උදාන ගීතය and
චිත්ත මෝහන) and renumber. **Expected total: 27 outbound content links**, not 26.
The article collection is unaffected — its nine pairings already match.

- [ ] **Step 5: Verify**

```bash
bundle exec jekyll build 2>&1 | tail -3
grep -o 'href="http[^"]*"' _site/extracurriculars.html | wc -l
grep -c 'Noto+Sans+Sinhala' _site/extracurriculars.html && echo "PASS: sinhala font"
grep -c 'Noto+Sans+Sinhala' _site/index.html || echo "PASS: not loaded on home"
```

Expected: exactly **27** outbound content links. Also run this pairing check, which
is the real gate — it confirms every title still points at the post it points at
today:

```bash
python3 -c "
import re
def clean(s):
    s=re.sub(r'<[^>]+>','',s).replace(chr(0x1F534),'').replace(chr(0x1F535),'')
    return re.sub(r'\s+',' ',s).strip()
def pairs(p):
    return {clean(re.sub(r'^\s*\d\d\s*','',t)):u for u,t in
            re.findall(r'href=\"(https?://[^\"]+)\"[^>]*>(.*?)</a>',open(p).read(),re.S)
            if 'fonts.googleapis' not in u}
old=pairs('/dev/stdin')  # replace with the pre-task file from git show
new=pairs('extracurriculars.html')
bad=[t for t,u in old.items() if t in new and new[t]!=u]
print('mismatched pairs:',bad if bad else 'none')
print('titles lost:',[t for t in old if t not in new])
"
```

Compare against the pre-task file via `git show HEAD:extracurriculars.html`. Both
lines must report nothing. Sinhala font present on the hub and absent from home.

- [ ] **Step 6: Commit**

```bash
git add -A _layouts/hub.html extracurriculars.html _includes
git commit -m "feat(hub): rebuild creative hub with serif identity"
```

---

### Task 8: Remove the dead JavaScript

**Files:**
- Delete: `js/` (entire directory), `_includes/js.html`, `_includes/js_disqus.html`
  (deferred from Task 3, since `js.html` references it), and the three old chrome
  includes deferred from Task 2: `_includes/nav.html`, `_includes/header.html`,
  `_includes/footer.html`
- Modify: `_config.yml`

- [ ] **Step 1: Assert nothing references any of it any more**

By now Tasks 2, 3, 6 and 7 have rebuilt every layout, so all six files should be
orphaned.

```bash
grep -rn 'js/\|js\.html\|include nav\.html\|include header\.html\|include footer\.html' \
  _layouts _includes *.html | grep -v 'design-source'
```

Expected: no output. If anything appears, fix the referencing layout before deleting
— do not delete a file that is still included, because Jekyll errors on a missing
include and the build will fail.

- [ ] **Step 2: Delete**

```bash
git rm -q -r js _includes/js.html _includes/js_disqus.html \
  _includes/nav.html _includes/header.html _includes/footer.html
```

- [ ] **Step 3: Remove dead config keys from `_config.yml`**

Delete `color:` (all four), `skills:`, `disqus_shortname:`, `contact:`, the
`address:` block, and the `credits:` visitor-badge line.

- [ ] **Step 4: Verify the output is clean**

```bash
bundle exec jekyll build 2>&1 | tail -3
grep -rc 'jquery\|bootstrap.min.js\|freelancer.js\|jqBootstrapValidation' _site/ | grep -v ':0' || echo "PASS: no dead JS referenced"
grep -rc 'visitorbadge' _site/ | grep -v ':0' || echo "PASS: badge gone"
test -d _site/js && echo "FAIL: js dir published" || echo "PASS: js dir gone"
```

Expected: three PASS lines.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "chore: remove jquery, bootstrap js and dead config"
```

---

### Task 9: Responsive breakpoints

**Files:**
- Modify: `_includes/css/design.css`

The designs ship no media queries. This fills the gap.

- [ ] **Step 1: Append the media queries to `design.css`**

```css
@media (max-width:900px){
  .hero,.about-grid,.contact-panel,.refs,.skills-grid{grid-template-columns:1fr;}
  .matrix{grid-template-columns:repeat(2,minmax(0,1fr));}
  .timeline{grid-template-columns:1fr;gap:8px;}
  h1{font-size:34px;} h2{font-size:25px;}
}
@media (max-width:700px){
  .wrap,.wrap-narrow,.wrap-tight,.hub-section{padding-left:20px;padding-right:20px;}
  .site-header,.site-footer{padding:14px 20px;}
  .site-header{flex-direction:column;align-items:flex-start;}
  .site-nav{width:100%;overflow-x:auto;justify-content:flex-start;
    -webkit-overflow-scrolling:touch;
    mask-image:linear-gradient(to right,#000 88%,transparent);}
  .contact-panel{padding:28px;}
  .cs__foot,.site-footer{flex-direction:column;align-items:flex-start;}
}
@media (max-width:560px){
  .matrix{grid-template-columns:1fr;}
  .pcard{flex:0 0 260px;width:260px;}
}
```

The `mask-image` gradient on `.site-nav` is the scroll affordance the review asked
for — it hints that the pill row continues past the edge.

- [ ] **Step 2: Verify no horizontal page overflow is introduced**

```bash
bundle exec jekyll build 2>&1 | tail -3
grep -c 'max-width:900px' _site/style.css && echo "PASS: breakpoints in output"
grep -c 'overflow-x:auto' _site/style.css
```

Expected: breakpoints present; `overflow-x:auto` appears at least twice
(`.table-wrap`, `.site-nav`).

- [ ] **Step 3: Commit**

```bash
git add _includes/css/design.css
git commit -m "feat(css): responsive breakpoints the designs omitted"
```

---

### Task 10: Full-site verification

**Files:** none modified unless a check fails.

- [ ] **Step 1: Clean build**

```bash
rm -rf _site && bundle exec jekyll build 2>&1 | tail -6
```

Expected: no "Conflict" warnings, no errors.

- [ ] **Step 2: Link and image integrity**

```bash
bundle exec rake test 2>&1 | tail -20
```

Expected: html-proofer passes. It checks every internal link and image reference.
If it flags external links, re-run with `only_4xx` as the Rakefile already sets.

- [ ] **Step 3: Every page serves**

```bash
(bundle exec jekyll serve --port 4123 >/tmp/serve.log 2>&1 &); sleep 6
for u in / /resume.html /extracurriculars.html /warehouse.html /businessone.html \
  /projects/e-vote-mpc/ /projects/pos-journal-functional-spec/ \
  /projects/wms-functional-spec-template/ /projects/rca-bug-ticket/ /style.css; do
  echo "$u -> $(curl -s -o /dev/null -w '%{http_code}' localhost:4123$u)"; done
pkill -f 'jekyll serve --port 4123'
```

Expected: 200 for all ten.

- [ ] **Step 4: Regression sweep**

```bash
grep -rn 'text-align: justify' _site/ | wc -l          # expect 0
grep -rn 'Montserrat\|Lato' _site/*.html | wc -l       # expect 0
grep -c 'table-bordered' _site/businessone.html        # expect >0 (legacy intact)
du -sh _site
```

- [ ] **Step 5: Hand off the visual check**

The rendered appearance cannot be verified here — there is no browser in this
session. Report to the user, naming exactly what to look at:

1. Home at phone width — hero stacks, nav pill row scrolls with a fading right edge.
2. `/projects/pos-journal-functional-spec/` — the six-column data dictionary scrolls
   horizontally inside its own box rather than stretching the page.
3. `warehouse.html` and `businessone.html` — unchanged from before the redesign.
4. Home hero headshot renders (WebP; fine in all current browsers).

- [ ] **Step 6: Final commit**

```bash
git add -A
git commit -m "chore: verify redesign build, links and routes"
```

---

## Self-Review

**Spec coverage.** Every spec section maps to a task: tokens and CSS architecture →
Task 1; template architecture → Tasks 2, 3, 6, 7; collection and URLs → Task 4; front
matter schema → Task 4 Step 4; case-study content preservation → Task 5; responsive →
Task 9; content/config/assets → Tasks 3, 7, 8; JS removal → Task 8; verification →
Task 10. The spec's "out of scope" list is untouched by design.

**Gaps found and closed during review:**
- The spec said `page.html` was unchanged. It is not — it includes two deleted files.
  Corrected in the Spec Corrections section and handled in Task 2 Step 4.
- `_config.yml` had no `exclude`, so `docs/` would publish to the live site. Added in
  Task 1 Step 3.
- The project files have no `title` in front matter, but the cards and case-study
  pages both need one. Added in Task 4 Step 4.
- `head.html` links `feed.xml` relatively, which breaks on the new nested
  `/projects/*/` URLs. Fixed in Task 1 Step 9.
- The design files exist only in the Claude Design project, so a fresh executor would
  have nothing to port from. Task 1 Step 1 saves them into the repo first.

**Naming consistency.** `.pcard`, `.cs-body`, `.table-wrap`, `.status--{tone}`,
`.matrix`, `.hub-list`, `site_header.html`/`site_footer.html` and the `active` values
are used identically everywhere they appear.
