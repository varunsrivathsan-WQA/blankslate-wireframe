---
name: blankslate-wireframe
description: "Build a lo-fi wireframe from a plain-English brief as BOTH a matching Figma frame and a self-contained HTML/CSS file, using only Blankslate's 66 primitives and real Lucide icon instances, and checking both source files for drift before building."
---

# Blankslate wireframe builder

Turn a plain-English brief ("wireframe a checkout flow", "mock up a SaaS pricing page") into **two matching deliverables built from the same brief**: a real Figma frame made of actual component instances, and a self-contained HTML file using the matching CSS classes. Never just one side unless the user explicitly asks for HTML-only or Figma-only.

Blankslate is a deliberately constrained, 66-primitive lo-fi wireframing kit (grayscale + one link blue, system font, one component per Figma page, one `.wf-*` class per primitive in `blankslate.css`) — the original 50 general-purpose primitives, plus a 16-primitive shadcn/ui-parity extension added later (chat bubbles, calendar, carousel, OTP input, skeleton loader, etc.). **The extension is not a separate category anywhere a user browses it** — every one of those 16 lives alphabetically inside its real category (see the vocabulary table below), both in Figma's page list and in the catalog. A 17th extension primitive, Kbd, was built and then removed — it never had a matching Figma page and isn't part of the shipped set; don't reintroduce it. Interactive controls and card-like surfaces use real corner radii matching shadcn/ui (see "Radius" below); pure layout/structural primitives stay flat/boxy on purpose, matching real shadcn. The constraint is the entire point — treat every rule below as a hard constraint, not a suggestion, and never invent a 67th primitive on either side.

**This skill requires a Figma file that already has the Blankslate component library built into it** (all 66 primitives, one per page, plus the Icon component set) — it builds *into* that library, it doesn't create the library from nothing. If the user's Figma file doesn't have it yet, say so and ask them to duplicate/share the reference Blankslate library file rather than trying to build all 66 primitives from scratch mid-task.

## Step 0 — ask which Figma file, every time

Before doing anything else, **ask the user which Figma file/link to build the wireframe into** — don't assume a file used earlier in this conversation or project still applies, and don't guess from a file name. Get either a Figma URL (extract the fileKey from `/design/:fileKey/...`) or an explicit fileKey. Do not proceed to Step 1 without it.

Also locate `blankslate-tokens.css` and `blankslate.css` (in this order for `<head>` links). This skill ships its own canonical copies at `references/blankslate-tokens.css` and `references/blankslate.css` (relative to this SKILL.md) — use those by default. If the current project/repo already has its own copies (for example, a team's customized fork with local changes), prefer those instead so you build against whatever the user is actually maintaining, and flag it to the user if the two versions disagree in a way that matters (a renamed or missing token, a changed primitive). Don't reconstruct the CSS from memory in either case — the vocabulary table below is for mapping a brief to the right primitive, not a substitute for the real, current file.

## Step 1 — parity check both sources before building anything

This is not optional and not a one-time setup step — run it fresh every time this skill is invoked, because either side may have drifted since the last build:

1. **Figma side:** using `use_figma` (load the `figma-use` skill first — mandatory, see Step 4), list the pages in the given file and confirm there's one page per expected primitive (see the vocabulary below — 66 total, one continuous alphabetical sequence, no separate "Extended" block) plus the Icon component set (see "Icons" below), named to match. Note any primitive with no matching page (Kbd should have none — that's correct, not a gap), and any extra page that looks like a primitive not in the vocabulary.
2. **Code side:** read `blankslate.css` and extract the actual `.wf-*` class list (grep for top-level primitive rules, not internal `__` sub-parts). Compare against the vocabulary below. Note that `blankslate.css` still groups the 16 extension primitives together near the end of the file purely to keep its own diff/history readable — that's a stylesheet-organization convenience, not a category boundary; each rule's doc comment names which of the 8 real categories it belongs to.
3. **Cross-check:** the Figma page list and the CSS class list should describe the *same* 66 primitives. If they don't agree with each other or with the table below — a primitive renamed, added, or missing on one side — **stop and flag the specific discrepancy to the user** before building, rather than silently building from whichever side looks more complete or guessing which one is "current." This mirrors how this project already handles component drift: flag it, ask, never auto-resolve.
4. **Structural quality, not just content parity:** a Figma primitive can match the CSS in content (right colors, right text, right children) while still being built wrong structurally — a plain static frame with manually-positioned children where the CSS uses `display:flex`/`grid`, or a set of separately-duplicated components standing in for what should be one component with variant properties. See "Figma structural quality" under Step 4 for what correct looks like and how to check for it; don't treat "looks right in a screenshot" as sufficient on its own for a primitive with internal layout or more than one state/size.

Only once both sides check out (or the user has confirmed how to proceed despite a flagged gap) move on.

## Step 2 — parse the brief into sections

Break the requested page/flow into the regions it implies (e.g. "pricing page" → nav, hero/intro, pricing cards, FAQ, footer). For a multi-screen flow, plan one HTML file and one Figma frame per screen unless the user asks for a single long scrolling page.

## Step 3 — map every element to a documented primitive

For each piece of UI the brief implies, pick the nearest match from the vocabulary table below — never invent a class or freehand a Figma shape for something a primitive already covers, and never invent an icon that isn't in the Icon component set (see "Icons" below). If something in the brief has **no matching primitive** (e.g. a rich data-viz chart, a drag-and-drop kanban board), stop and flag it back to the user by name instead of improvising new markup/shapes or silently dropping the requirement.

### Token naming — numeric step-of-10, same on both sides

Every scale in `blankslate-tokens.css` — font size, line height, radius, spacing (space/gap/padding-vertical/padding-horizontal) — steps by 10 (10, 20, 30…) and mirrors the Figma variable names 1:1, so a token name is copy-pasteable between the two: `--font-20` in CSS is `Size/Font Size/font-20` in Figma, same pattern for `--line-*`, `--radius-*`, `--space-*` / `--gap-*` / `--padding-*-*`. Two deliberate exceptions to know about, both documented inline in `blankslate-tokens.css`:

- **Radius is numbered by pixel value, not by declaration order.** `--radius-10` = flat/0px, `--radius-20` = soft/2px, `--radius-30` = control/6px, `--radius-40` = md/10px — monotonic with size, which is why "control" (30) sits after "soft" (20) even though the underlying raw token order is flat/soft/control/md. Always use the *numbered* name (`--radius-30`) in a primitive's CSS rule and bind the matching numbered variable in Figma — never the descriptive raw name (`--radius-control`) directly.
- **Opacity's numeric value differs between the two platforms on purpose.** CSS's native `opacity` property takes 0–1, so `--opacity-50` is `0.5` in `blankslate-tokens.css`; Figma internally divides a bound opacity value by 100, so the *same* variable reads as `50` inside the Figma properties panel. This is a real, necessary platform difference — don't "fix" one side to match the other's raw number.

### Color infrastructure — error/warning/success exist but are unused on purpose

`--error`, `--warning`, and `--success` (Layer 3, all three theme blocks) are real, correctly-themed tokens sitting in both the CSS and the Figma variable collection, but **no primitive references them today** — urgency in Blankslate is always communicated via a label or icon choice (e.g. Alert/banner's title text saying "Error"), never via color, per the grayscale-plus-one-blue constraint (see Step 6 rule 4). Don't wire these into a primitive's default styling just because they exist; they're there so a future explicitly-requested "make this alert red" kind of ask has a real token to reach for, deliberately conflicting with the grayscale rule until a user asks for exactly that.

### Icons

Every icon in Blankslate is a real inlined Lucide SVG (ISC license) — never an abstract box, never an icon font, never a build-time sprite step. On the HTML side every icon carries `class="wf-icon"` with `fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"`; color always comes from the parent's CSS `color`, never a separate fill/stroke override. Default size is 16px; add `data-size="md"` for 20px or `data-size="lg"` for 24px. On the Figma side, every icon is an instance of Blankslate's **Icon** component set (one variant per icon name, e.g. "chevron-down", "close", "search", "plus", "info", "globe", "mail", "message-circle").

Two ways an icon appears, and they're built differently:
- **Fixed-meaning glyph** (a chevron on Collapsible, a close × on Modal/Toast, a search glyph on Search bar, calendar/nav-arrow glyphs on Calendar/Carousel/Pagination, a chevron-right separator on Breadcrumb, a globe/mail/message-circle row on Footer, etc.) — just a plain Icon instance of the right variant, same on every instance of that primitive. No swap control needed.
- **"Any icon goes here" slot** — **Accordion**'s header chevron, **Icon button**, **Icon placeholder**, **Select**'s dropdown chevron, and **List item**'s leading icon — these expose an explicit `INSTANCE_SWAP` component property named `"Icon"` on the root component (`root.addComponentProperty("Icon", "INSTANCE_SWAP", defaultComponentId)`), so a consumer can swap the icon per-instance from a dropdown in Figma's properties panel, the same way a variant property works. There is no remaining abstract-box exception — every primitive that used to hold a placeholder box (List item included) now holds a real default icon.

If you need an icon that isn't already in the library, don't fall back to an abstract shape: build a new Icon variant in the same style instead (24×24 viewBox, `stroke="#616161"` bound to the ink-600 variable, `stroke-width: 1.5`, round caps/joins, `fill="none"`) via `figma.createNodeFromSvg(svgString)`, restyle every descendant to match, wrap it in `figma.createComponent()` named `icon=<name>`, and append it to the Icon component set at the next open grid slot — then use it like any other icon instance. Flag the addition to the user rather than silently reusing an unrelated existing glyph.

**Gotcha — resizing an icon instance:** `instance.resize(w, h)` only changes the frame's declared width/height; it does **not** rescale children whose constraints aren't set to "Scale," so shrinking/growing an icon instance with `.resize()` alone leaves the internal vector paths at their original size, causing clipping or a tiny icon in an oversized frame. Always do it in this order instead: `inst.resize(24, 24)` first (restores the frame to the icon's native 24×24, matching its un-moved children), then `inst.rescale(target / 24)` (scales frame **and** every descendant vector/stroke together, anchored top-left), then re-set `inst.x`/`inst.y` if an auto-layout parent shifted it mid-script. This is the only correct way to land an icon instance at a non-native size (e.g. 16×16 inside a `data-size` default slot, or 14–20px inside a primitive like Breadcrumb's separator/Map marker/Spinner/Video placeholder). If an icon has *already* been resized incorrectly (e.g. by a prior plain `.resize()` call) and you need to correct it, don't compound the mistake by resizing again then rescaling — instead compute `factor = desiredSize / currentReportedSize` from whatever its current (wrong) size actually is, and call `rescale(factor)` alone, with no accompanying `resize()` call.

**Gotcha — binding a radius (or any) variable by ID:** `figma.variables.getVariableByIdAsync(id)` requires the **full** id string including the `VariableID:` prefix (e.g. `"VariableID:228:14"`). Passing just the bare numeric part (`"228:14"`) returns `null` silently, and a `setBoundVariable` call built from that `null` does nothing — no thrown error, no binding applied. Always re-read `node.boundVariables` after the call to confirm the binding actually landed; a lack of a thrown error is not confirmation. If you don't already have the right ID, look it up by name (`figma.variables.getLocalVariableCollectionsAsync()` → find the "Blankslate Tokens" collection → resolve each `variableId` and filter by `.name`) rather than guessing or reusing an ID remembered from a previous file — variable IDs are per-file and don't carry over.

### Radius

Four radius tokens, used by category — never a literal value on either side. Names are numbered by pixel value, not declaration order (see "Token naming" above):

- **`--radius-10` (flat, 0px):** pure layout/structural primitives — Navbar, Sidebar, Page/Frame, Footer, Table, Spacer, Image placeholder, Icon placeholder's outer dashed box. Real shadcn doesn't round these either.
- **`--radius-20` (soft, 2px):** the smallest controls — Checkbox, Tooltip's simple bubble.
- **`--radius-40` (md, 10px):** card-like surfaces — Accordion, Alert/banner, Card, Modal/dialog shell, Toast/snackbar, Logo placeholder, Tooltip's rich variant, and most of the Media/Forms/Data/Feedback extension primitives (Chat bubble, Calendar, Carousel, Collapsible section, OTP slot, and similar).
- **`--radius-30` (control, 6px):** interactive controls — Button, Badge/tag, Text input, Textarea, Select, Search bar, Icon button, Dropdown/submenu, Pagination's page-number chips, Nav/menu item's active-state highlight (this one cascades: Nav/menu item is reused inside Navbar, Sidebar, and Dropdown/submenu, so fixing it there fixes all three).

Documented literal exceptions (never tokens, but always called out in a comment on both sides): **circular** shapes use `border-radius: 50%` or an explicit `max(width, height)` (Avatar, Map marker, Skeleton's circle variant); **pill** shapes use a literal `9999px` or half-height value (Progress bar + its fill, Toggle/switch's track + knob, Slider's track + thumb); **Button group**'s segmented-control join (first child keeps its left `--radius-30` corners and squares its right corners, last child the mirror, middle children fully squared, 0 gap, and the interior borders thinned by one side to avoid a doubled seam — mirrors shadcn's `-ml-px` overlap trick).

### The 66 primitives (exact markup — copy these patterns, don't paraphrase them)

Each category below is the full blended, alphabetized set — the shadcn/ui-parity extension's members sit inline with the original 50, marked **(ext)** on first mention only as a note for this table; there is no separate section for them anywhere else (not in Figma's page list, not in the catalog, not in `blankslate.css`'s category structure as seen by anyone but the stylesheet's own internal comments).

**Layout & structure (9)**
- Container — `<div class="wf-container">...</div>` (centered max-width column, horizontal padding only)
- Divider — `<hr class="wf-divider">`
- Grid — `<div class="wf-grid"><div class="wf-grid-cell">...</div>...</div>`
- Page/Frame — `<div class="wf-page">...</div>`
- Row — `<div class="wf-row">...</div>` (horizontal rhythm)
- Scroll container (ext) — `<div class="wf-scroll">…long content…</div>` (fixed height, scrolls)
- Section — `<section class="wf-section">...</section>` (full-width band, vertical padding only)
- Spacer — `<div class="wf-spacer" data-size="5"></div>` (data-size: 1–8, default 5; Figma side is a variant set, see "Variants" below)
- Stack — `<div class="wf-stack">...</div>` (vertical rhythm)

**Navigation (8)**
- Breadcrumb — `<nav class="wf-breadcrumb"><a>Home</a><svg class="wf-icon">…chevron-right…</svg><span data-current="true">Current</span></nav>` (real chevron separator, not a "/" character)
- Dropdown/submenu — `<div class="wf-dropdown"><a class="wf-nav-item">…</a>…</div>` (radius-30)
- Footer — `<footer class="wf-footer"><div class="wf-row wf-footer__columns"><nav class="wf-stack"><h3>…</h3><a class="wf-footer__link">…</a>…</nav></div><hr class="wf-divider"><div class="wf-row"><small>© …</small><div class="wf-row"><svg class="wf-icon">…globe…</svg><svg class="wf-icon">…mail…</svg><svg class="wf-icon">…message-circle…</svg></div></div></footer>` (container stays flat — only the social-icon row changed from gray boxes to real icons)
- Nav/menu item — `<a class="wf-nav-item">Menu item</a>` and `<a class="wf-nav-item" data-active="true">Active item</a>` (reused inside Navbar/Sidebar/Dropdown; active state is radius-30, see "Radius" above; Figma side is a variant set, see "Variants" below)
- Navbar — `<nav class="wf-navbar"><div class="wf-navbar__logo"></div><div class="wf-row">…nav items…</div><button class="wf-button">…</button></nav>`
- Pagination — `<nav class="wf-pagination"><button class="wf-page-number" aria-label="Previous"><svg class="wf-icon">…chevron-left…</svg></button><a class="wf-page-number">1</a><a class="wf-page-number" data-active="true">2</a>…<button class="wf-page-number" aria-label="Next"><svg class="wf-icon">…chevron-right…</svg></button></nav>` (chips are radius-30; real Previous/Next chevron controls flank the numbers)
- Sidebar — `<nav class="wf-sidebar"><a class="wf-nav-item">…</a><a class="wf-nav-item" data-active="true">…</a>…</nav>`
- Tab bar — `<div class="wf-tab-bar"><button class="wf-tab" data-active="true">…</button><button class="wf-tab">…</button></div>`

**Typography (6)**
- Blockquote — `<blockquote class="wf-blockquote"><p>"…"</p><cite>— Attribution</cite></blockquote>`
- Body text — `<p class="wf-body">…</p>`
- Eyebrow/label — `<p class="wf-eyebrow">Eyebrow label</p>`
- Heading — `<h2 class="wf-heading" data-level="2">…</h2>` (data-level 1–6, independent of the semantic tag)
- Link text — `<a class="wf-link" href="#">Link text</a>` (the ONLY place blue appears)
- Placeholder text block — `<p class="wf-placeholder-text">Lorem ipsum…</p>`

**Actions (5)**
- Button — `<button class="wf-button" data-variant="primary">Button</button>` (variant: primary | secondary; radius-30)
- Button group — `<div class="wf-button-group"><button class="wf-button wf-button-group__item" data-variant="secondary">One</button>…</div>` (seamless segmented control — every child button also gets the `wf-button-group__item` class; see "Radius" above for the join treatment)
- Icon button — `<button class="wf-icon-button"><svg class="wf-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><path d="M5 12h14"/><path d="M12 5v14"/></svg></button>` (radius-30; holds a real icon — swappable via the `"Icon"` instance-swap property on the Figma side, see "Icons" above; default is a plus glyph)
- Link-styled button — `<button class="wf-link-button">Link-styled button</button>` (an action, not navigation — don't use Link text for this)
- Toggle group (ext) — `<div class="wf-toggle-group"><button data-active="true">Left</button><button>Center</button><button>Right</button></div>` (segmented, multi-option — distinct from Toggle/switch's single on/off)

**Media & branding (8)**
- Aspect ratio box (ext) — `<div class="wf-aspect-ratio" data-ratio="16-9"><div class="wf-image-placeholder" style="width:100%;height:100%">Image</div></div>` (data-ratio: 16-9 | 4-3 | 1-1; Figma side is a variant set, see "Variants" below)
- Avatar — `<span class="wf-avatar">AB</span>` (circular, `border-radius: 50%` exception)
- Carousel (ext) — `<div class="wf-carousel"><div class="wf-carousel__viewport"><div class="wf-carousel__slide">…</div></div><button class="wf-carousel__control" data-direction="prev">…</button><button class="wf-carousel__control" data-direction="next">…</button></div>`
- Icon placeholder — `<span class="wf-icon-placeholder"><svg class="wf-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></svg></span>` (outer box stays flat/radius-10; holds a real icon — swappable via the `"Icon"` instance-swap property on the Figma side; default is an info glyph)
- Image placeholder — `<div class="wf-image-placeholder">Image</div>`
- Logo placeholder — `<div class="wf-logo-placeholder">Logo</div>` (radius-40)
- Map marker (ext) — `<span class="wf-marker"><svg class="wf-icon">…map-pin…</svg></span>`
- Video placeholder — `<div class="wf-video-placeholder"><svg class="wf-icon" data-size="md" ...>…play…</svg>Video</div>`

**Forms (13)**
- Attachment chip (ext) — `<div class="wf-attachment"><svg class="wf-icon">…paperclip…</svg><span class="wf-attachment__name">file.pdf</span><button class="wf-attachment__remove"><svg class="wf-icon">…×…</svg></button></div>`
- Calendar (ext) — `<div class="wf-calendar"><div class="wf-calendar__header"><button><svg class="wf-icon">…prev…</svg></button><span class="wf-calendar__label">September 2026</span><button><svg class="wf-icon">…next…</svg></button></div><div class="wf-calendar__weekdays">…</div><div class="wf-calendar__grid"><span class="wf-calendar__day" data-muted="true">31</span>…<span class="wf-calendar__day" data-selected="true">15</span>…</div></div>` (full month grid — every visible day 1–30 plus muted leading/trailing days from the adjacent months, not a partial/truncated grid; exactly one day shown selected, matching the documented example)
- Checkbox — `<label class="wf-checkbox"><input type="checkbox">Checkbox label</label>` (radius-20)
- Date field (ext) — `<div class="wf-date-field"><input type="text" placeholder="MM/DD/YYYY"><svg class="wf-icon">…calendar…</svg></div>`
- Form group — `<div class="wf-form-group"><label>Label</label><input class="wf-input"><small>Helper text</small></div>` (error state: `<small data-variant="error">This field is required.</small>`; Figma side is a variant set, see "Variants" below)
- OTP input (ext) — `<div class="wf-otp"><input class="wf-otp__slot" maxlength="1">…</div>`
- Radio — `<label class="wf-radio"><input type="radio" name="…">Radio label</label>`
- Search bar — `<div class="wf-search-bar"><svg class="wf-icon" ...>…</svg><input type="search" placeholder="Search"></div>` (radius-30)
- Select — `<div class="wf-select"><select><option>Select an option</option></select><svg class="wf-icon">…chevron-down…</svg></div>` (radius-30; real chevron, swappable via the `"Icon"` instance-swap property — not a CSS-drawn triangle)
- Slider (ext) — `<div class="wf-slider"><div class="wf-slider__track"><div class="wf-slider__fill" style="width:62%"></div><span class="wf-slider__thumb" style="left:62%"></span></div></div>` (pill exception; track/fill/thumb are CSS `position:relative`/`absolute` — stays a plain Figma frame, not auto-layout, see "Auto-layout" below)
- Text input — `<input class="wf-input" type="text" placeholder="Placeholder">` (radius-30)
- Textarea — `<textarea class="wf-textarea" placeholder="Placeholder"></textarea>` (radius-30)
- Toggle/switch — `<label class="wf-toggle" data-checked="true"><span class="wf-toggle__track"><span class="wf-toggle__knob"></span></span>Toggle label</label>` (pill exception; Figma side is a variant set, see "Variants" below — the track/knob stay a plain absolutely-positioned frame inside each variant, same reasoning as Slider)

**Data & content display (9)**
- Accordion item — `<div class="wf-accordion-item"><div class="wf-accordion-item__header"><h4 class="wf-heading" data-level="4">…</h4><svg class="wf-icon" ...>…chevron…</svg></div><hr class="wf-divider"><p class="wf-accordion-item__body">…</p></div>` (radius-40; renders open/static — no JS in v1)
- Badge/tag — `<span class="wf-badge">Badge</span>` (radius-30, matching Button — not a smaller radius)
- Card — `<div class="wf-card"><div class="wf-image-placeholder">…</div><h3 class="wf-heading" data-level="3">Card title</h3><p class="wf-body">…</p><button class="wf-button">Learn more</button></div>` (radius-40)
- Chat bubble (ext) — `<div class="wf-bubble" data-role="assistant">…</div>` (data-role: assistant | user)
- Collapsible section (ext) — `<div class="wf-collapsible"><div class="wf-collapsible__trigger"><span>…</span><svg class="wf-icon">…chevron…</svg></div><div class="wf-collapsible__content">…</div></div>` (renders open/static)
- List item — `<div class="wf-list-item"><div class="wf-row"><svg class="wf-list-item__icon wf-icon" data-size="lg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/></svg><span>Label</span></div><span>Detail</span></div>` (icon slot holds a real icon now — swappable via the `"Icon"` instance-swap property on the Figma side, see "Icons" above; this used to be the one deliberate abstract-box exception, it no longer is)
- Message row (ext) — `<div class="wf-message" data-role="assistant"><span class="wf-avatar">…</span><div class="wf-message__content"><div class="wf-bubble">…</div><span class="wf-message__time">…</span></div></div>`
- Stat/metric block — `<div class="wf-stat"><strong>1,234</strong><span>Metric label</span></div>`
- Table — `<table class="wf-table"><thead><tr><th>Name</th>…</tr></thead><tbody><tr><td>Item</td>…</tr></tbody></table>` (sortable header upgrade: `<th data-sortable="true">Name <svg class="wf-icon">…chevron-down…</svg></th>`; a native table grid — stays a plain Figma frame per row/cell, not auto-layout, see "Auto-layout" below)

**Feedback & overlays (8)**
- Alert/banner — `<div class="wf-alert"><svg class="wf-icon" data-size="md" ...>…</svg><div class="wf-alert__content"><strong class="wf-alert__title">Heads up!</strong><span class="wf-alert__description">Message.</span></div></div>` (icon + title + description anatomy, matching real shadcn Alert; radius-40; urgency via icon/title wording, e.g. "Error" — never via color, see "Color infrastructure" above)
- Empty state (ext) — `<div class="wf-empty"><span class="wf-icon-placeholder"><svg class="wf-icon">…</svg></span><h3 class="wf-empty__title">No results</h3><p class="wf-empty__body">…</p><button class="wf-button" data-variant="primary">Reset filters</button></div>`
- Modal/dialog shell — `<div class="wf-modal"><div class="wf-modal__header"><h2 class="wf-heading" data-level="3">Title</h2><button><svg class="wf-icon">…×…</svg></button></div><hr class="wf-divider"><p class="wf-modal__body">…</p><div class="wf-modal__footer"><button class="wf-button" data-variant="secondary">Cancel</button><button class="wf-button" data-variant="primary">Confirm</button></div></div>` (radius-40; Sheet/Drawer upgrade: add `data-position="right"|"left"|"bottom"` to slide from an edge instead of centering — those edge variants stay flat on purpose)
- Progress bar — `<div class="wf-progress"><div class="wf-progress__fill" style="width:62%"></div></div>` (pill exception; no flex container in the CSS at all — stays a plain Figma frame, see "Auto-layout" below)
- Skeleton loader (ext) — `<span class="wf-skeleton" data-shape="text"></span>` (data-shape: text | circle; each shape is a single leaf shimmer shape with nothing to lay out)
- Spinner (ext) — `<svg class="wf-spinner" viewBox="0 0 24 24">…</svg>` (its own rotating class, not `.wf-icon`; the component is just the icon instance filling the frame — nothing to lay out)
- Toast/snackbar — `<div class="wf-toast"><span>Message.</span><button><svg class="wf-icon">…×…</svg></button></div>` (radius-40)
- Tooltip — `<div class="wf-tooltip"><span class="wf-tooltip__bubble">Tooltip text</span><span class="wf-tooltip__pointer"></span></div>` (simple bubble: radius-20; rich variant/Hover Card upgrade: radius-40, and its bubble is a vertical auto-layout stack of a heading + body paragraph with a real gap token between them — never two text nodes placed side by side; Figma side is a variant set, see "Variants" below)

## Step 4 — build the Figma frame

Load the `figma-use` skill (or read the `skill://figma/figma-use/SKILL.md` MCP resource) before any `use_figma` call — mandatory, not optional.

1. In the given fileKey, create one new frame per screen, named clearly (e.g. "Wireframe — Checkout: Payment step").
2. Build the frame's layout using real instances of Blankslate's own layout primitives — Page/Frame, Section, Container, Stack, Row — as VERTICAL/HORIZONTAL auto-layout frames, exactly mirroring the HTML structure being built in Step 5. Don't hand-draw layout with arbitrary rectangles; the layout is primitives too.
3. For every other element, find its main component by locating the page whose name matches the primitive (e.g. a page named "Button") — pages are one continuous alphabetical list, not grouped by category or by original/extension — get its component/component set, and `createInstance()`. For any icon (see "Icons" above), instantiate from the Icon component set the same way — never draw a freehand vector. Never hardcode component ids from a previous build — the file may differ from the one used last time, and ids differ per file.
4. Apply the brief's specific content via `instance.setProperties({...})` for variant/content component properties (introspect `instance.componentProperties` to find the right key, e.g. one starting with "Content" or "Label") — `figma.loadFontAsync(...)` for every font/style involved BEFORE setting any text.
5. Append children into their parent auto-layout frames (never set manual x/y unless deliberately overlapping two layers, e.g. Avatar-style, or building one of the documented absolute-positioning exceptions in "Auto-layout" below). Remember: for a VERTICAL auto-layout frame, `primaryAxisSizingMode` governs height and `counterAxisSizingMode` governs width — backwards from naive intuition, and the single most common cause of silently truncated frames. For HORIZONTAL frames it's reversed.
6. If any icon instance needs a non-native size, use the resize-then-rescale pattern from "Icons" above — never a plain `.resize()`, which clips the icon instead of scaling it.
7. Take a screenshot of the finished frame and check it actually matches intent before reporting done.

### Figma structural quality — auto-layout and true variants are both required

Two structural rules apply to every primitive built or touched in Figma, on top of the visual/content correctness covered elsewhere in this skill. Both are about *how* a component is built, not what it looks like — a screenshot can look completely correct while either rule is silently broken, so check the underlying node properties (`layoutMode`, `componentPropertyDefinitions`), not just a rendered screenshot.

**Auto-layout, not static frames.** Any Figma frame that corresponds to a CSS container using `display: flex` or `display: grid` must be built as a real Figma auto-layout frame (`layoutMode: "HORIZONTAL"` or `"VERTICAL"`, with `layoutWrap: "WRAP"` for a wrapping grid), never a plain static frame (`layoutMode: "NONE"`) with children manually positioned by x/y. Map the CSS onto Figma properties directly: `gap` → `itemSpacing` (bind to the matching `Spacing/Gap/gap-N` variable), `padding` → `paddingTop/Bottom/Left/Right` (bind to `Spacing/Padding Vertical/…` / `Spacing/Padding Horizontal/…`), `justify-content`/`align-items` → `primaryAxisAlignItems`/`counterAxisAlignItems` (`flex-start`→`MIN`, `center`→`CENTER`, `flex-end`→`MAX`, `space-between`→`SPACE_BETWEEN`), and an explicit CSS px `width`/`height` → `primaryAxisSizingMode`/`counterAxisSizingMode: "FIXED"` vs. `width:auto`/`fit-content` → `"AUTO"` (hug) vs. `width:100%`/`flex:1` on a child → that child's `layoutSizingHorizontal`/`layoutSizingVertical: "FILL"`.

The one real exception: **don't** force auto-layout onto something the CSS itself positions with `position: absolute` or `position: relative` + absolutely-positioned children — converting these would change the actual mechanism, not just tidy up the metadata. The documented cases in this kit are: Slider's fill/thumb over its track, Toggle/switch's knob over its track, Carousel's prev/next controls over its viewport, Table's native row/cell grid, and any primitive with literally no flex/grid declared in its CSS at all (Progress bar, Spinner, Divider, Skeleton loader — these are bare leaf shapes with nothing to lay out in the first place). Leave those as plain frames; don't "fix" them into auto-layout just for the sake of it.

Every icon instance or image-placeholder child sitting inside an auto-layout frame should have its `constraints` property set to `{horizontal: "CENTER", vertical: "CENTER"}`, not left at the Figma default `{horizontal: "MIN", vertical: "MIN"}` ("top-left anchored"). Auto-layout governs the actual rendered position regardless of this setting, so a stale `MIN/MIN` constraint won't cause a visible bug today — but it's wrong metadata that misdescribes the component's intended resize behavior, and it's worth setting correctly at creation time rather than leaving it for a later cleanup pass.

**Variants, not duplicate components.** Any primitive with more than one state, size, or style meant to be browsed as options of *the same thing* — a size scale (Spacer's 8 sizes), a ratio/shape scale (Aspect ratio box's 3 ratios), an on/off or default/alternate state (Toggle/switch, Form group's error state, Nav/menu item's active state), a content-richness variant (Tooltip's simple vs. rich) — must be built as **one Figma component set with real variant properties**, never as several separately-named, separately-maintained duplicate components or instances that merely *look* related. A true variant set is what lets a consumer pick the size/state/ratio from a dropdown in Figma's properties panel the same way `data-*` attributes drive it in HTML, and it's what keeps N near-identical copies from drifting out of sync over time.

To build one: construct each state as its own plain `COMPONENT` first (getting each one visually correct independently, exactly as you would any other primitive), name each one `"<PropertyName>=<value>"` (e.g. `"Ratio=4-3"`, `"Ratio=16-9"`, `"Ratio=1-1"`, or `"State=default"`/`"State=error"`) — Figma infers the variant property and its options directly from this naming convention — then combine them with `figma.combineAsVariants([comp1, comp2, ...], parentPage)` and rename the resulting `COMPONENT_SET` to the primitive's plain name (e.g. `"Aspect ratio box"`). `combineAsVariants` auto-arranges the variants into a tidy grid inside the new set and leaves the set's own `layoutMode` as `"NONE"` — that's correct and expected; a component set's outer wrapper is a canvas arrangement, not a CSS-mapped container, so don't try to auto-layout it. Every existing multi-state primitive in the reference Blankslate library (Aspect ratio box, Spacer, Tooltip, Toggle/switch, Form group, Nav/menu item) is already built this way — use one of those as a reference if unsure what the end state should look like.

## Step 5 — build the matching HTML file

- One self-contained HTML file per screen, `<head>` linking `blankslate-tokens.css` then `blankslate.css` (in that order).
- Structure it with the exact same layout nesting (Page → Section → Container → Stack/Row → primitives) used for the Figma frame in Step 4 — the two should read as the same screen, not two different interpretations of the brief.
- Real, plausible placeholder copy for the brief's domain (not raw "Lorem ipsum" everywhere) — the same copy used for the Figma instances' text overrides, so both sides say the same thing.

## Step 6 — hard rules while composing (both sides)

1. **No invented classes, freehand shapes, or invented icons.** If it isn't in the table above, it isn't a Blankslate primitive; if it isn't in the Icon component set (Figma) / a real inlined Lucide SVG (HTML), it isn't a Blankslate icon (see "Icons" above for how to add a genuinely missing icon the right way, instead of faking one).
2. **Variants only via documented mechanisms** — CSS `data-` attributes (`data-variant`, `data-level`, `data-active`, `data-current`, `data-checked`, `data-size`, `data-role`, `data-shape`, `data-direction`, `data-ratio`, `data-selected`, `data-muted`, `data-position`, `data-sortable`) on the HTML side, real component variant properties built as a component set (and, for Accordion's chevron / Icon button / Icon placeholder / Select's chevron / List item's icon, the `"Icon"` instance-swap property) on the Figma side — see "Variants" under Step 4 for how a variant set is built correctly. Never a modifier class, a one-off manual style override, or a separately-duplicated component for something a variant already covers or should cover.
3. **Nothing outside the token file, on either side** — including radius. No literal hex colors, no arbitrary pixel spacing/radius/font-size in the HTML except the documented circular/pill/structural-join exceptions called out in "Radius" above; every Figma property bound to the Blankslate Tokens variable collection, never a literal value typed into the properties panel (same exceptions apply, and only those; remember the `VariableID:` prefix gotcha above when binding one programmatically). If a value isn't a token in `blankslate-tokens.css` and isn't one of the documented exceptions, it doesn't belong in either output.
4. **Grayscale + one blue, and urgency is a label/icon, not a color** — on both sides (see "Color infrastructure" above for why the error/warning/success tokens exist but stay unused).
5. **Flag, don't guess**, on anything with no matching primitive or icon, on a source-parity gap found in Step 1, or on any mismatch discovered between the two finished outputs — always ask rather than silently improvising or letting one side lead without telling the user.

## Step 7 — report back

Deliver the HTML file(s) (e.g. via SendUserFile) and the Figma frame's URL (`https://figma.com/design/<fileKey>/...?node-id=<id>`), state which primitives (and which icons) were used for anything non-obvious, and surface: any Step 1 parity gaps, anything flagged in Step 3 with no matching primitive, and any difference between the two finished outputs that a straight re-read of both doesn't fully explain.
