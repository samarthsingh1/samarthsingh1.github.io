# Full Spectrum Audit: Portfolio Redesign (dark, monospace, numbered sections)

**Date**: 2026-11-22
**Mode**: Branch vs Main
**Source**: `claude/fervent-goldberg-gm6lr7`, which is `main` (`d733801`) plus 4 commits (`9b461ed` … `416cbb5`) carried over from `claude/serene-pasteur-t642d6`. Changed: `index.html` (−1,246 / +718), `profile.jpg` deleted, `profile.webp` and `profile-480.jpg` added.
**Lenses Run**: 7/10
**Lenses Skipped**: Backend Engineer (static site with no server, Rust or Tauri code) · Structural Engineer (single-file change, no new modules or boundaries) · DX Engineer (the product is a portfolio for recruiters, not a developer tool, so there's no CLI, config or developer-facing error surface)

**How this was verified:** every finding below was checked against the running page, not just read from source. Methods: Chromium via Playwright at 320, 390, 768, 844×390, 1280, 1440@2x and 2560@2x; keyboard walkthroughs; axe-core 4.10.2 in both themes; contrast calculated by hand for gradient text; Chrome DevTools Protocol CPU and heap metrics; print-media emulation and PDF output; JS disabled; reduced motion; the GitHub search API for project links. Evidence images are in `reviews/evidence/`.

---

## Status update (2026-11-22, after fixes)

All six critical issues are resolved. Each fix was re-verified in Chromium:

| # | Critical issue | Status | Verification |
|---|---|---|---|
| 1 | Print / PDF blank | ✅ Fixed | `@media print` block; 0 of 6 sections hidden under print media; 6-page readable PDF |
| 2 | Mobile menu focus | ✅ Fixed | Opening focuses the first link; Tab runs through all 6 links, the theme toggle and the menu button, and leaving the nav closes it; Escape returns focus to `#menu` |
| 3 | Contact email | ✅ Fixed | Now `samarth.singh199@gmail.com` (owner's choice) |
| 4 | Contrast | ✅ Fixed | axe-core: 0 violations in dark and light |
| 5 | Infinite blink | ✅ Fixed | Blinks 4 times (4.4s) then stays solid; the footer cursor starts only when scrolled into view |
| 6 | Focus reshapes controls | ✅ Fixed | Focused controls keep 999px and 50% radii |

Also fixed along the way: the reveal failsafe (P-R1; verified by injecting a script error, after which all 6 sections appear at 3s), no-JS controls and nav (F-R5/U-R7/A-R6), the short-viewport histogram collapse (D-R1/F-R3, which now draws only the code field), U-R1 (Intuit details), U-R4 (stat framing), and an inaccurate project claim found while sourcing copy (the Loan Application card said SVM+PCA was most accurate; the repo README shows LDA at 91%). The remaining recommendations are still open.

## Master Verdict: APPROVE WITH CHANGES

The redesign is a clear improvement over `main`. First-view transfer drops from more than 1.1MB to 139KB, a contact form that could never submit (`YOUR_FORMSPREE_ID`) is gone, and semantics, landmarks, reduced motion and reflow are solid. No page errors at any viewport. No lens found a reason to rework it. But six must-fix issues stand between this branch and a merge:
- One output is broken: print and "Save as PDF" come out blank after the hero.
- The mobile and zoomed menu fails keyboard users.
- The light theme fails AA contrast.
- The blinking cursor fails WCAG 2.2.2.
- The focus style reshapes every pill-shaped control.
- The only contact channel is an unconfirmed university address.

Five of the six are small CSS or JS fixes; the sixth needs your answer.

## Verdict by Lens

| # | Lens | Verdict | Critical | Recommendations |
|---|------|---------|----------|-----------------|
| 1 | Design Engineer | APPROVE WITH CHANGES | 2 | 6 |
| 2 | UX Engineer | APPROVE WITH CHANGES | 2 | 7 |
| 3 | Frontend Engineer | APPROVE WITH CHANGES | 1 | 5 |
| 4 | Backend Engineer | Skipped | – | – |
| 5 | Structural Engineer | Skipped | – | – |
| 6 | Repo Maintainer | APPROVE WITH CHANGES | 0 | 4 |
| 7 | Production Readiness | APPROVE WITH CHANGES | 1 | 6 |
| 8 | DX Engineer | Skipped | – | – |
| 9 | Performance Engineer | APPROVE WITH CHANGES | 0 | 4 |
| 10 | Accessibility Engineer | APPROVE WITH CHANGES | 3 | 6 |

## All Critical Issues (Must Fix)

Deduplicated across lenses, most severe first. IDs point to the full write-up in each lens section.

| # | Issue | Lens(es) | File(s) | Impact | Fix |
|---|-------|----------|---------|--------|-----|
| 1 | **Print and "Save as PDF" are blank after the hero.** `.js .reveal { opacity: 0 }` hides every section until it's scrolled into view, and there's no `@media print`. Measured: all 6 sections at `opacity: 0` under print media; PDF pages 2+ empty (`reviews/evidence/print-pages-1-3.png`). | Frontend (F-C1), Prod (P-R5) | `index.html:177-178` | The PDF a recruiter saves for a hiring packet has no experience, projects or contact details. | Add a print block: force `.reveal` visible, hide the nav, canvas and motion button, switch to dark-on-white tokens, unstick `.sh`. Full CSS in F-C1. |
| 2 | **The mobile and zoomed menu breaks keyboard focus.** The links come before the menu button in the DOM, so Tab after opening the menu skips all 6 links (measured), and Escape inside the menu drops focus to `<body>` (measured). | A11y (A-C1), UX (U-C2), Frontend (F-R1) | `index.html:277-289, 603-606` | The nav is unusable from a keyboard on phones, and for desktop users at 200–400% zoom. Fails WCAG 2.4.3 (A). | Put `#menu` before `#nav-links` in the markup; focus the first link on open; on Escape, close and return focus to `#menu`. |
| 3 | **The only contact channel is unconfirmed.** Every CTA is `mailto:samarth1@umd.edu`; the degree ended May 2026, and there's no other channel on the page. | UX (U-C1), Prod (P-C1) | `index.html:560, 577` | If the mailbox is retired, recruiter emails disappear silently. That's the one conversion the site exists for. | **Your input:** confirm the address delivers, or switch to a personal one. |
| 4 | **Text contrast fails, mainly in the light theme.** axe-core: 5 nodes in dark, 11 in light. Gradient stops measured by hand: light cyan 3.23:1, light pink 4.04:1, on "INTUIT" and "Projects". | Design (D-C2), A11y (A-C2) | `index.html:30-31, 52, 60-62, 171, 203` | Fails WCAG 1.4.3 (AA) on section numbers, project numbers and the employer name. | Verified tokens: light `--holo-1 #08778e`, `--holo-3 #c32f83`, `--faint #726c61`; dark `--faint #80786b` (all 4.6:1 or better). |
| 5 | **The blinking cursors never stop, and "Pause motion" doesn't cover them.** Measured: the nav cursor's animation is still `blink` after pausing. | A11y (A-C3), UX (U-N1) | `index.html:103-104` | Fails WCAG 2.2.2 (A) for users with attention or vestibular sensitivity; the nav cursor is on screen everywhere. | `animation: blink 1.1s steps(1) 4` (stops before 5s), or tie it to a root `motion-paused` class. |
| 6 | **The focus style reshapes every pill and circle control.** `a:focus-visible, button:focus-visible { border-radius: 6px }` overrides the 999px and 50% radii. Measured: 6px on all 13 nav and hero controls while focused. | Design (D-C1) | `index.html:79` | Keyboard users see the CTA, nav pills and icon buttons change shape, and the gradient border corners get cut. | Delete `border-radius: 6px` from that rule; outlines already follow the element's radius. |

## Cross-Cutting Concerns

| Concern | Flagged By | Root Cause | Unified Fix |
|---------|-----------|------------|-------------|
| **Mobile menu focus handling** | A11y, UX, Frontend | The DOM order puts the links before their toggle, and `setMenu(false)` hides the element that has focus. | One change to the markup order plus a `setMenu(open, {restore})` that manages focus (F-R1). It fixes critical #2 and the Escape bug together. |
| **Progressive enhancement assumes JS finishes and the output is a screen** | Frontend, Prod, UX, A11y | `.js` is set in `<head>` before the code that reveals content has run, and the CSS hides content and controls on that assumption. Nothing covers print, a runtime error, or JS being off. | Three CSS-only guards: (a) a `@media print` block (critical #1); (b) a failsafe animation that reveals `.reveal` after 2.5s whether or not JS ran (P-R1); (c) hide JS-only controls with `html:not(.js)` and scope the mobile collapse to `.js .links` (F-R5 / U-R7 / A-R6). |
| **The light theme is second-class** | Design, A11y, UX | Light tokens were never contrast-checked, and `theme-color` is hard-coded to dark. | Adopt the verified token values (critical #4) and update `<meta name="theme-color">` in both theme code paths (D-R4). |
| **Hero canvas is sized to the viewport at native DPR and rebuilt unconditionally** | Performance, Frontend, Design, Prod | `layout()` allocates two full-viewport canvases at up to 2× DPR, repaints all of it at 30fps, re-runs on every `resize`, and uses one geometry for every narrow screen, including short ones. | Cap the DPR by area; split into a static and an animated layer and clear only the dirty rectangle; skip layout when the size is unchanged; drop the histogram when there's too little vertical room (PF-R1, PF-R2/F-R2, D-R1/F-R3). |
| **"Motion" isn't one state** | A11y, UX, Performance | The pause button controls only the canvas, uses label + `aria-pressed` together, isn't remembered, and there's no idle stop. | One `html.motion-paused` state: it stops the canvas and the cursor, has one ARIA pattern, is saved in `localStorage`, and is set automatically after ~45s idle (A-C3, A-R1, U-N1, PF-R3). |
| **Arrows mean different things and are read aloud** | UX, A11y | `→` is used for both in-page and external links, and `.arr` spans aren't `aria-hidden`. | `↓` for in-page, `↗` for external, and `aria-hidden="true"` on all `.arr` (U-R5, A-R4). |
| **Gradient text is declared in three places** | Repo, Design | `.holo`, `.l.featured` and `.project a:hover .ptitle` each repeat the gradient declarations. | Combine them into one selector before applying the contrast fix, so the fix lands everywhere (R-R2). |

## Decisions Only You Can Make

| # | Question | Why it matters | Flagged by |
|---|----------|----------------|------------|
| 1 | Does `samarth1@umd.edu` still receive mail, or should the site use a personal address? | It's the only contact path (critical #3). | UX, Prod |
| 2 | Intuit: start month, team or product area, and 1–3 bullets? | The hero, `<title>` and About all lead with Intuit, but its row is empty (U-R1). | UX |
| 3 | Should a résumé PDF be linked? If so, add the file. | Recruiters forward PDFs (U-R2). | UX |
| 4 | One result line per project (placement, users, a metric), plus demo links if any? | The cards say what each project is, not how it went (U-R6). | UX |
| 5 | Keep dark as the default, or follow the visitor's OS setting? | This was a deliberate choice to match the reference site (D-N3). | Design |
| 6 | Does the new headline copy sound like you? ("Data in. Decisions out." / "Data person, turned software engineer.") | Still open from the last session (U-N4). | UX |

## All Recommendations

| # | Recommendation | Lens | File(s) | Fix |
|---|---------------|------|---------|-----|
| 1 | Every frame repaints the full viewport; 113MB of canvas backing store at 2560×1440@2x | Performance (PF-R1) | `index.html:670-748` | DPR cap by area, two-layer canvas, dirty-rect clears |
| 2 | Hero histogram collapses into one strip under the nav on short viewports (844×390) | Design (D-R1), Frontend (F-R3) | `index.html:695-700` | Skip bars when fewer than 6 rows are free, or use the wide layout when `W > H` |
| 3 | Every `resize` rebuilds the canvas, including the mobile URL-bar toggle | Frontend (F-R2), Performance (PF-R2) | `index.html:666, 778` | Return early when size and DPR are unchanged |
| 4 | One JS exception before the reveal setup hides the entire page | Prod (P-R1) | `index.html:20-23, 583-631` | CSS failsafe reveal animation |
| 5 | Without JS: dead buttons, and the mobile nav can't be reached | UX (U-R7), Frontend (F-R5), A11y (A-R6), Prod (P-R6) | `index.html:120, 158, 242-243` | `html:not(.js)` hides JS-only controls; `.js .links` collapse |
| 6 | Hero animates indefinitely while visible | Performance (PF-R3) | `index.html:750-766` | Auto-pause after ~45s idle |
| 7 | Motion toggle announces "Play motion, pressed" | A11y (A-R1) | `index.html:319, 767-773` | Fixed label with `aria-pressed`, or changing label without it |
| 8 | Project links are ~200-character card text and open new tabs unannounced | A11y (A-R2) | `index.html:428-487` | Title-only link stretched over the card; hidden "(opens in new tab)" |
| 9 | Section `h2`s don't name their section | A11y (A-R3) | `index.html:328, 355, 426, 500, 521, 557` | Visually hidden "About. " etc. prefix |
| 10 | Arrows inconsistent and read aloud | UX (U-R5), A11y (A-R4) | `index.html:314-315, 561-562` | `↓`/`↗` convention plus `aria-hidden` |
| 11 | `color-mix()` without fallback removes the hero scrim on older Safari and Chrome | Design (D-R5), Prod (P-R6) | `index.html:133, 153, 255` | Add an `rgba()` declaration before each one |
| 12 | `theme-color` stays dark in the light theme | Design (D-R4) | `index.html:13, 22, 592-598` | Update the meta tag in the head script and the toggle |
| 13 | Link preview image is a 480px square; no `twitter:card` | Prod (P-R2) | `index.html:8-12` | 1200×630 card plus width, height, alt and `summary_large_image` |
| 14 | No canonical link or JSON-LD `Person` | Prod (P-R3) | `index.html <head>` | Add both |
| 15 | Render-blocking Google Fonts stylesheet | Prod (P-R4) | `index.html:16-18` | Self-host 2 woff2 files, or preload-swap |
| 16 | 35KB WebP for a 36px avatar (25% of page weight) | Performance (PF-R4) | `index.html:301-302` | 120px WebP (~4KB) |
| 17 | Intuit row is empty | UX (U-R1) | `index.html:357-363` | **Your input** |
| 18 | No résumé download | UX (U-R2) | — | **Your input** |
| 19 | Education missing from the nav | UX (U-R3) | `index.html:277-284` | Add the link |
| 20 | Stats "-50%" and "-40%" read as declines | UX (U-R4) | `index.html:341-342` | "50% faster store audits" framing |
| 21 | Project cards show no outcomes | UX (U-R6) | `index.html:428-487` | **Your input** |
| 22 | "Projects" nav link has no hover; GitHub chip hover is the same as rest | Design (D-R2) | `index.html:107-109` | Brightness or underline on hover |
| 23 | No `:active` or pressed states | Design (D-R3) | `index.html:106-211` | `scale(.97)` on press |
| 24 | `▶` renders as a colour emoji on iOS | Design (D-R6) | `index.html:319, 770` | Inline SVGs or U+FE0E |
| 25 | `.links` is `position: fixed` inside a transformed parent | Frontend (F-R4) | `index.html:243-249` | `position: absolute; top: calc(100% + 8px)` |
| 26 | `aria-label` on a generic `div` | A11y (A-R5) | `index.html:572` | Remove it |
| 27 | README is empty | Repo (R-R1) | `README.md` | Add a short README |
| 28 | Gradient-text and label styles duplicated | Repo (R-R2) | `index.html:84, 109, 184, 204, 206` | Combine selectors |
| 29 | A superseded redesign commit is in history | Repo (R-R3) | git history | Squash-merge |
| 30 | Branch bypasses the usual `dev` → `main` flow | Repo (R-R4) | — | PR into `dev`, or fast-forward `dev` after merging |

---

# Lens Reports

---

## 1. Design Engineer

**Verdict**: APPROVE WITH CHANGES

### Plan Summary
The branch replaces the old multi-section card layout with a dark-first, monospace, numbered-section design: a floating pill nav, a live canvas "code histogram" hero, sticky section labels, count-up stats, a command-table skills list and a big footer wordmark. Light theme is an opt-in toggle.

### Files Reviewed
| File | Visual Role | Risk |
|------|-------------|------|
| `index.html` lines 25–270 (`<style>`) | Entire design system: tokens on `:root`, light override on `:root[data-theme="light"]`, all components, breakpoints at 900px and 760px, reduced-motion block | High: every visual decision lives here |
| `index.html` lines 633–782 (hero canvas script) | Draws the background code field and the gradient histogram from CSS tokens | Medium: geometry is computed per viewport |
| `profile.webp`, `profile-480.jpg` | 480×480 portrait shown at 36px in the hero eyebrow; JPG is also the `og:image` | Low |

### Critical Issues (Must Fix)
| # | Issue | File(s) | Impact | Fix |
|---|-------|---------|--------|-----|
| D-C1 | **The keyboard focus style changes the shape of every pill and circle control.** `a:focus-visible, button:focus-visible { …; border-radius: 6px; }` (specificity 0,1,1) beats the single-class radius on `.btn`, `.l`, `.icon-btn`, `.menu`, `.motion`, `.brand` and `.skip`. Measured in Chromium: all 13 controls from the skip link through "Pause motion" report `border-radius: 6px` while focused, versus 999px or 50% at rest. | `index.html:79` | A keyboard user sees "See my work" turn from a pill into a rounded rectangle, and its gradient border corners get cut. The circular theme toggle becomes a squircle. The outline no longer traces the control it belongs to. | Delete `border-radius: 6px` from line 79. Current Chrome, Firefox and Safari draw outlines that follow the element's own radius. |
| D-C2 | **Light-theme accent and faint tokens fail WCAG AA for the text they colour.** Measured on `#f5f2eb`: `--holo-1 #0a93b0` = 3.23:1, `--holo-3 #cf3a8f` = 4.04:1, `--faint #8f887c` = 3.14:1. In the dark theme `--faint #6a6358` = 3.39:1. These colour real text: section numbers `.sh .no` (14px), the gradient "INTUIT" in the hero eyebrow (12px), the "Projects" nav link (14px), project titles on hover, and `.pno` "p/01" (13px). | `index.html:30-31, 52, 60-62, 171, 203` | Text in the selectable light theme is under the 4.5:1 minimum. The gradient's cyan start is the weakest stop, and it lands on the first letters of "INTUIT" and "Projects". | Verified replacements that keep the hue: light `--holo-1: #08778e` (4.65:1), light `--holo-3: #c32f83` (4.63:1), light `--faint: #726c61` (4.66:1), dark `--faint: #80786b` (4.61:1). Optionally light `--holo-2: #6f4dee` (4.73:1); it is exactly 4.50:1 today. Alternatively keep `--faint` for decorative glyphs only and render `.pno` in `--muted`. |

### Recommendations
| # | Issue | File(s) | Why | Fix |
|---|-------|---------|-----|-----|
| D-R1 | **The hero histogram collapses on short, narrow viewports.** At 844×390 (a phone in landscape), the narrow-layout branch places the bars between the nav and the text, but there are only about 5 rows of room. `maxRows` is clamped to 3, so the chart becomes one strip of code chips jammed under the nav pill (screenshot `reviews/evidence/landscape-844x390.jpg`). | `index.html:695-700` | It looks broken on a common orientation. | When `textTop` leaves fewer than ~6 rows, skip the histogram (draw the field only), or use the wide right-side layout whenever `W > H`. |
| D-R2 | **The "Projects" nav link has no hover feedback.** `.l.featured` is declared after `.l:hover` at equal specificity, so `color: transparent` wins and nothing changes on hover. `.l.gh` already has ink colour plus a chip, so its hover is also identical to its rest state. | `index.html:107-109` | It is the one link the design promotes, and it is the one that feels dead. | Add `.l.featured:hover { filter: brightness(1.2); }` (or an underline), and `.l.gh:hover { background: var(--field); }`. |
| D-R3 | **No `:active` or pressed states anywhere.** `.btn`, `.icon-btn`, `.menu`, `.motion` and the project cards have hover styles but no press feedback. On touch screens, where hover doesn't exist, taps get no visual response. | `index.html:106-211` | The controls feel unresponsive on phones, where much of the traffic will come from. | `.btn:active, .icon-btn:active, .menu:active, .motion:active { transform: scale(.97); }` with `transition: transform .1s`. |
| D-R4 | **The browser bar colour ignores the theme.** `<meta name="theme-color" content="#080707">` stays near-black in the light theme (measured), so mobile Safari and Chrome show a black bar above a cream page. | `index.html:13, 22, 592-598` | A visible seam on mobile in light mode. | Set the meta's `content` to `#f5f2eb` in the head script when `theme === 'light'`, and again in the toggle handler. |
| D-R5 | **`color-mix()` has no fallback.** It is used for the hero scrim (`.hero::after`) and the `.btn:hover` glow. Browsers without `color-mix` (Safari < 16.2, Chrome < 111) drop the whole `background` declaration, which leaves hero text sitting directly on the animated code field. | `index.html:133, 153, 255` | Older iOS devices get unreadable hero text. | Put an `rgba(8,7,7,.8)` gradient declaration before each `color-mix` one; the browser keeps the first one it understands. |
| D-R6 | **The play/pause glyphs are text characters.** `▶` (U+25B6) renders as a colour emoji on iOS unless followed by U+FE0E, and `❚❚` depends on font coverage. The theme toggle next to it uses inline SVG. | `index.html:319, 770` | On iPhone the motion button can show a blue emoji play icon that doesn't match the design. | Use two small inline SVGs, as the theme toggle does, or append `︎`. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| D-N1 | Collapse the 8 small monospace sizes (11, 11.5, 12, 13, 14, 14.5, 15 and 17px) into a 3-step scale such as `--fs-xs/--fs-sm/--fs-md`. | The section spacing values (22/26/30/34/36/40/48px) are also ad hoc. A scale keeps future edits consistent. |
| D-N2 | Use a typographic minus (U+2212) in "-50%" and "-40%", in both the HTML and the count-up `data-prefix`. | The hyphen-minus sits low and short next to tabular numerals in JetBrains Mono. |
| D-N3 | When there is no stored theme choice, follow `prefers-color-scheme` instead of always starting dark. | Dark-by-default was a deliberate choice in the previous session to match the reference site, so this is your call. |
| D-N4 | Remove `backdrop-filter` from the mobile `.links` panel. | It sits on a solid `var(--bg)` background, so the blur never shows. Nested inside the nav's own backdrop-filter, it costs compositing for nothing. |

### Design System Compliance
The token system is clean and well used. Every colour in the stylesheet resolves to a custom property on `:root`, with a complete light override (16 tokens re-declared). The canvas reads its colours from the same tokens through `getComputedStyle`, and it re-reads them on theme change (`relayoutField`), so the hero can't drift from the palette. The problems are in token *values* (D-C2), not token *use*. There is no fragmentation: no raw hex outside the token block except inside the data-URI favicon, which is acceptable. The radius family is coherent and concentric: the 20px menu panel with 8px padding holds 12px items, cards are 14px, chips 6px, controls are pills or circles. The one rule that breaks that family is the focus rule (D-C1). Typography uses two families with clear roles: mono for display text and labels, system sans for reading text. That's a good call for legibility.

### Visual State Coverage
| Component/Element | Hover | Focus | Active | Disabled | Loading | Empty | Error | Dark Mode |
|-------------------|-------|-------|--------|----------|---------|-------|-------|-----------|
| Nav link `.l` | ✅ colour | ⚠️ outline, but pill → 6px | ❌ | n/a | n/a | n/a | n/a | ✅ both themes |
| "Projects" `.l.featured` | ❌ no change | ⚠️ pill → 6px | ❌ | n/a | n/a | n/a | n/a | ⚠️ light gradient 3.2–4.0:1 |
| Theme toggle `.icon-btn` | ✅ chip bg | ⚠️ circle → 6px | ❌ | n/a | n/a | n/a | n/a | ✅ icon swaps |
| Menu button `.menu` | ❌ | ⚠️ circle → 6px | ❌ | n/a | n/a | n/a | n/a | ✅ open → X morph |
| Primary CTA `.btn` | ✅ glow | ⚠️ pill → 6px | ❌ | n/a | n/a | n/a | n/a | ✅ |
| Ghost links `.ghost` | ✅ colour + arrow nudge | ✅ | ❌ | n/a | n/a | n/a | n/a | ✅ |
| Motion button `.motion` | ❌ | ⚠️ pill → 6px | ❌ | n/a | n/a | n/a | ✅ hidden if no canvas | ✅ |
| Project card `.project a` | ✅ bg + gradient title + arrow | ✅ keeps 14px | ❌ | n/a | n/a | n/a | n/a | ⚠️ light hover title contrast |
| Footer links | ✅ | ✅ | ❌ | n/a | n/a | n/a | n/a | ✅ |
| Stats `.num` | n/a | n/a | n/a | n/a | ✅ final values rendered in HTML, animated after reveal | n/a | n/a | ✅ |
| Hero canvas | n/a | n/a | n/a | n/a | ✅ plain bg until the font loads | ✅ blank bg without JS | ✅ `getContext` guard | ✅ re-themes live |

### Verdict Details
- Design Token Fidelity: **CONCERNS**. Tokens are used everywhere, but three light-theme values and one dark value fail contrast (D-C2), and `color-mix` has no fallback (D-R5).
- Visual State Coverage: **CONCERNS**. The focus style reshapes controls (D-C1); there are no active states and no hover on the featured link.
- Dark Mode Parity: **CONCERNS**. Dark is the polished theme; light has contrast failures and a static `theme-color`.
- Animation Quality: **PASS**. Reveal uses opacity and transform only; the cursor blink uses `steps(1)`; `prefers-reduced-motion` disables smooth scroll, the reveal, the blink and the canvas. The 700ms reveal is longer than UI-transition guidance, but it suits a one-time scroll entrance.
- Visual Consistency: **PASS**. One coherent visual language with a concentric radius family and consistent mono labelling.

---

## 2. UX Engineer

**Verdict**: APPROVE WITH CHANGES

### Plan Summary
For a visitor, the site goes from a long card layout with a broken contact form to a single scrolling story: a hero with one clear CTA, then six numbered sections, with email as the only way to make contact. The primary visitors are recruiters and hiring managers, often arriving on a phone from LinkedIn or a résumé.

### User Journey Map

**Journey A: a recruiter on a phone, coming from LinkedIn**
1. Lands on the hero and sees the name, "Software engineer · Intuit" and a two-sentence summary. **OK**
2. Wants to know what the current job involves, taps "Experience ↓", and reaches the first row: *Intuit, Software Engineer, 2026 — now*. No month, no team, no bullets. **FAIL** (content gap on the role the whole page leads with)
3. Scans the earlier roles, which have clear bullets and numbers. **OK**
4. Looks for a résumé PDF to forward to the hiring manager. There isn't one, and there wasn't one on `main` either. **FAIL**
5. Opens the menu, taps Contact, then taps the `samarth1@umd.edu` button. The mail app opens. **WARN**: it's a university address and the degree ended May 2026, so it's unconfirmed that the mailbox still receives mail.

**Journey B: a hiring manager on a laptop**
1. Clicks the highlighted "Projects" link in the nav. **OK**
2. Scans five cards (tag, title, one-line description, stack). **WARN**: no outcome or result on any card ("placed X", "N users", "demo"), and nothing to look at without leaving the site.
3. Clicks a card and GitHub opens in a new tab; the ↗ signals that. **OK**
4. Comes back to the tab and the scroll position is intact. **OK**

**Journey C: someone looking for education**
1. Scans the nav: About, Experience, Projects, Skills, Contact. **WARN**: the page has six numbered sections, but section 05 Education is missing from the nav, so the numbering promises a stop that the menu doesn't offer.
2. Scrolls to it. **OK**

**Journey D: a visitor who prefers light mode**
1. Taps the sun icon, and the page switches and remembers the choice. **OK**
2. On mobile, the browser bar stays near-black above a cream page. **WARN**

**Journey E: a motion-sensitive visitor**
1. With the OS set to reduce motion, the canvas is still, and the reveals, smooth scroll and blinking are all off. **OK**
2. Without that setting, they press "Pause motion". The canvas stops, but the blinking cursors in the nav and footer keep going, and the choice is forgotten on the next visit. **WARN**

**Journey F: a keyboard user with the browser zoomed in (400% zoom makes a desktop show the 320px mobile layout)**
1. Tabs to the menu button and presses Enter, and the menu opens. **OK**
2. Presses Tab and lands on "See my work" in the hero. The menu links come *before* the button in the DOM, so they are skipped. **FAIL**
3. Presses Escape while inside the menu, and focus drops to `<body>`. **FAIL**

### Critical Issues (Must Fix)
| # | Issue | Step in Journey | Impact on User | Fix |
|---|-------|----------------|----------------|-----|
| U-C1 | **The only contact route may go nowhere.** The page offers exactly one way to reach you: `mailto:samarth1@umd.edu`, in the contact section and the footer. The removed form never worked (it posted to the placeholder `formspree.io/f/YOUR_FORMSPREE_ID`), so email is now the whole funnel. It's a university address, and the M.S. ended May 2026. | A5 | If that mailbox has been retired or forwards nowhere, recruiter emails disappear silently. That is the single conversion this site exists for. | **Needs your input** (open since the last session): confirm the address still receives mail, or switch to a personal address. A visible "copy address" button would also help desktop visitors without a mail app. |
| U-C2 | **The mobile menu can't be used from a keyboard** (also reported by Accessibility, A-C1). | F2, F3 | Keyboard and switch users, including desktop users at high zoom, can't reach the nav links, and pressing Escape loses their place. | Move `#menu` before `#nav-links` in the DOM, or move focus to the first link when the menu opens. On Escape, return focus to `#menu`. |

### Recommendations
| # | Issue | Impact | Fix |
|---|-------|--------|-----|
| U-R1 | **The Intuit role is empty** (A2). The hero, the About copy, the `<title>` and the meta description all lead with Intuit, but its row has no month and no description. | It's the first thing a recruiter checks, and it reads as unfinished. | **Needs your input:** start month, team or product area, and 1–3 bullets. Even a single "Working on …" line closes the gap. |
| U-R2 | **No résumé download** (A4). It is missing on `main` too, so this isn't a regression. | Recruiters forward PDFs, not URLs, and that's an extra request they have to email you about. | Add `resume.pdf` to the repo and a "Résumé ↓" link next to the email button and in the footer. |
| U-R3 | **Education isn't in the nav** (C1). | The 01–06 numbering implies six destinations, but the nav shows five. | Add `<a class="l" href="#education">Education</a>` (the nav measures 607px at 768px wide, so there's room), or fold Education into About. |
| U-R4 | **The stat labels read as negatives.** "-50% Store audit time" and "-40% Manual reporting" sit next to "4+" and "20+". Out of context, a minus sign scans as a decline. | Your two strongest results are the easiest to misread. | Reframe as gains, for example "50% / faster store audits" and "40% / less manual reporting", and drop the minus prefix in `data-prefix`. |
| U-R5 | **The arrows mean different things in different places.** `→` is used for an in-page jump ("Get in touch →") and for external links ("LinkedIn →", "GitHub →"), while project cards use `↗` for external links. | Visitors can't predict whether a link leaves the site. | `↓` for in-page jumps, `↗` for anything that opens a new tab. |
| U-R6 | **Project cards show no outcomes** (B2). | A hiring manager learns what each project *is*, but not how it went. | One result line per card, such as the competition placement or a measurable result, plus a demo link where one exists. **Needs your input.** |
| U-R7 | **Without JavaScript, the mobile nav is a dead end.** The menu button renders but does nothing, and the links stay `display:none` (measured with JS disabled). | It's rare, but it's a hard dead end when it happens. | Show the links by default and collapse them only under `.js`, for example by scoping the 760px `.links { display:none }` rule to `.js .links`. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| U-N1 | Remember "Pause motion" in `localStorage`, as the theme toggle does, and make it stop the cursor blink too. | A visitor who paused motion once shouldn't have to do it again. |
| U-N2 | Observe `.stats` separately from its section. | The reveal fires when 8% of the About block is visible, so on a laptop the numbers can finish counting before they scroll into view. |
| U-N3 | Highlight the current section in the nav. | The sticky `.sh` labels already provide some of this, so it's low priority. |
| U-N4 | Read the new headline copy aloud once ("Data in. Decisions out.", "Data person, turned software engineer."). | It was flagged for your review last session. Copy should sound like you. |

### Interaction Consistency Check
The page is internally consistent. Every section uses the same number + label + two-line heading pattern; all in-page CTAs scroll smoothly (or jump, under reduced motion); all external links open a new tab with `rel="noopener noreferrer"`. Two deviations: arrow meanings (U-R5), and a nav that skips one numbered section (U-R3). Compared to `main`, the redesign removes a contact form that could never submit (placeholder Formspree ID). That's a net UX win, because a form that silently fails is worse than no form. It also drops the separate "Technical Expertise" cards in favour of the denser skills table, which is faster to scan. Nothing a visitor could do on `main` is lost except the form, which never worked.

### Verdict Details
- Discoverability: **CONCERNS**. Education is missing from the nav; there's no résumé.
- Flow Completeness: **CONCERNS**. The contact flow rests on one unconfirmed mailbox; the current role has no content.
- Error Recovery: **PASS**. There are no destructive actions; theme and motion are one-tap reversible; Escape and outside-click close the menu.
- Feedback & Status: **PASS**. Toggles update their label and icon immediately; the menu icon morphs open and closed.
- Consistency: **CONCERNS**. Arrow semantics (U-R5).

---

## 3. Frontend Engineer

**Verdict**: APPROVE WITH CHANGES

> Scope note: there's no React, TypeScript, Zustand or Tailwind in this repo. The lens is applied to what exists: one static HTML file with inline CSS and roughly 200 lines of vanilla JS in two scripts. React-specific gotchas (Immer, selectors, barrel imports, dynamic Tailwind classes) don't apply. Their vanilla equivalents (listener cleanup, stale state in long-lived callbacks, render triggers, CSS architecture) are covered below.

### Plan Summary
The JS layer is a head script (sets the `.js` class and restores a saved light theme before first paint), plus a body script with four parts: theme toggle, mobile menu, reveal-on-scroll with stat count-up, and an IIFE that runs the hero canvas loop.

### Files Reviewed
| File | Frontend Role | Risk |
|------|--------------|------|
| `index.html:20-23` | Pre-paint script: `.js` class and theme restore, no flash | Low |
| `index.html:25-270` | All CSS: tokens, layout grid, two breakpoints, reduced motion | Medium (no print rules, see F-C1) |
| `index.html:583-631` | Theme, menu, reveal and count-up | Medium (menu focus handling) |
| `index.html:633-782` | Canvas renderer: offscreen background, histogram geometry, rAF loop, visibility pause | Medium (resize and geometry edge cases) |

### Critical Issues (Must Fix)
| # | Issue | File(s) | Why It Breaks | Code Fix |
|---|-------|---------|---------------|----------|
| F-C1 | **Print and "Save as PDF" produce blank pages.** With JS on, `.js .reveal { opacity: 0 }` hides every section until an IntersectionObserver reveals it, and there are no `@media print` rules. Measured with print media emulated right after load: all six sections (`about`, `experience`, `projects`, `skills`, `education`, `contact`) have `opacity: 0`, and `body` colour is cream `rgb(239,233,221)`. Chromium's PDF output (`reviews/evidence/print-pages-1-3.png`) is a faded hero followed by blank pages, with the fixed nav stamped on every page. | `index.html:177-178`, no print block | Recruiters and hiring panels save candidate sites as PDF for review packets. Today that PDF has no experience, projects or contact details in it. | Add at the end of `<style>`: `@media print { .js .reveal { opacity: 1 !important; transform: none !important; } .nav, .motion, .skip, .hero canvas, .hero::after { display: none !important; } :root { --bg: #fff; --ink: #111; --soft: #333; --muted: #555; --faint: #777; } .hero { min-height: 0; padding: 0 0 24px; } .sec { padding-top: 32px; break-inside: avoid-page; } .sh { position: static; } a[href^="http"]::after { content: " (" attr(href) ")"; font-size: .8em; } }` |

### Recommendations
| # | Issue | File(s) | Why | Fix |
|---|-------|---------|-----|-----|
| F-R1 | **Escape and link-tap hide the element that has focus.** `setMenu(false)` removes `.open`, which sets `.links` to `display:none` while focus may be on one of its links. Measured: focus on the "GitHub" menu link, press Escape, and `document.activeElement` becomes `BODY`. The DOM order (links before `#menu`) also means Tab from an opened menu skips it. (Same root cause as A-C1 / U-C2.) | `index.html:603-606`, `:277-289` | Focus is lost and the next Tab starts from the top of the document. | `const setMenu = (open, {restore} = {}) => { const hadFocus = nav.querySelector('#nav-links')?.contains(document.activeElement); nav.classList.toggle('open', open); menuBtn.setAttribute('aria-expanded', String(open)); if (open) nav.querySelector('#nav-links a')?.focus(); else if (hadFocus && restore) menuBtn.focus(); };` On Escape call `setMenu(false, {restore: true})`, and only when the menu is open. |
| F-R2 | **Every `resize` event rebuilds the whole canvas, even if the size didn't change.** Measured: 3 synthetic no-op resizes created 3 offscreen canvases. On iOS and Android, showing or hiding the URL bar fires `resize` while scrolling, but the hero (`100svh`, so its height is stable) doesn't change size. Each rebuild reallocates the background bitmap (about 5MB on a phone), re-renders every row of text and resets the sparks. | `index.html:666-678, 778` | Wasted CPU and memory churn, and a visible "reset" of the animation whenever the user changes scroll direction near the top. | At the top of `layout()`: `if (r.width === W && r.height === H && (window.devicePixelRatio || 1) === lastDpr && !force) return;`, and pass `force` from `relayoutField` (theme change). |
| F-R3 | **The histogram geometry breaks on short viewports.** For `W < 900`, `baseRow = floor((textTop - 28) / ch)`. When text starts at 120px (landscape phone, 844×390), `baseRow` is 5 and `maxRows` is clamped to `max(3, 5 - 7)` = 3, so the bars draw in rows 2–5, which is behind the nav pill (bottom edge measured at 68px). See screenshot `reviews/evidence/landscape-844x390.jpg` (also D-R1). | `index.html:695-700` | Visual defect on landscape phones and small tablets. | `if (baseRow - Math.ceil(100 / ch) < 4) { bars = []; }`, plus a guard in `draw()` (`if (!bars.length) return;` before the axis) so only the field renders. Alternatively use the wide layout when `W > H`. |
| F-R4 | **`.links` is `position: fixed` inside a transformed parent.** `.nav` has `transform: translateX(-50%)` and `backdrop-filter`, and both make it the containing block for fixed descendants. So `.links { position: fixed; top: 60px; left: 50% }` is actually positioned relative to the nav, not the viewport. It works today by coincidence. | `index.html:90, 95, 243-249` | If anyone later removes the transform, for example to center the nav with `margin: auto`, the menu jumps to the top of the viewport. | Say what it is: `.links { position: absolute; top: calc(100% + 8px); left: 50%; transform: translateX(-50%); }` |
| F-R5 | **No-JS state shows dead controls.** `#motion`, `#menu` and `#theme-toggle` render without JS but do nothing, and on mobile the links are `display:none` with no way to open them. The page already sets a `.js` class, so this is easy to scope. | `index.html:120, 158, 242-243` | The mobile nav is unreachable without JS, and two buttons do nothing. | `html:not(.js) .motion, html:not(.js) .menu, html:not(.js) .icon-btn { display: none; }` and scope the mobile `.links { display: none }` to `.js .links`. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| F-N1 | Replace the global `window.relayoutField` with `document.dispatchEvent(new CustomEvent('themechange'))` and listen for it inside the IIFE. | Keeps the hero module self-contained; there's one fewer global. |
| F-N2 | Subscribe to `matchMedia('(prefers-reduced-motion: reduce)').addEventListener('change', …)`. | Currently the setting is read once; changing it in the OS mid-visit is ignored until reload. |
| F-N3 | Add `// @ts-check` plus a few JSDoc types to the inline script, or move it to `main.js` with `@ts-check`. | Gives editor type-checking for the `dataset` coercions (`+el.dataset.to`) and the `querySelector` non-null assumptions at no build cost. |
| F-N4 | Watch `matchMedia('(resolution: …dppx)')` for DPR changes. | Dragging the window to a monitor with a different DPR leaves the canvas blurry until the next resize. |

### Render Path Analysis
- **Before first paint:** the head script adds `.js` and applies `data-theme="light"` if one was stored, so there is no theme flash. The Google Fonts stylesheet is render-blocking (`display=swap` keeps text visible once CSS arrives). The `<img>` has `width`/`height`, so no layout shift.
- **After parse (end-of-body script):** sets the year, syncs the theme label, and attaches listeners: 1 theme click, 1 menu click, 6 nav-link clicks, 1 document keydown, 1 document click, 1 window resize, 2 IntersectionObservers. All are page-lifetime on a single page that never unmounts, so the lack of cleanup is correct here, not a leak.
- **Reveal path:** IO at `threshold: 0.08` adds `.shown` (CSS transition on opacity and transform, compositor-only), starts `countUp` (a 1-second rAF loop per stat), then unobserves. Each element runs once.
- **Hero loop:** `document.fonts.load()` → `layout()` → `draw()` → `setPlaying()`. `tick` runs on every rAF (60–120Hz) but does work only every 33ms or more, so drawing runs at about 30fps. Each drawing frame does: full `clearRect`, `drawImage` of the full-size offscreen background, up to 24 spark `fillText`s, then for each bar one bg-coloured `fillRect`, one gradient `fillRect`, and one `fillText` per row. The loop stops when the hero leaves the viewport (IO sets `visible=false`, and the next tick returns without requesting another frame), when the tab is hidden (rAF stops), and when "Pause motion" is pressed. Measured main-thread task time: about 112 ms/s playing versus 0.7 ms/s paused at 1280×800 @1x. (Scale effects are in the Performance lens.)
- **Re-render triggers:** theme toggle calls `layout()` (full rebuild, appropriate); resize, debounced 150ms, calls `layout()` (full rebuild even when the size is unchanged, F-R2).
- **Stale state check:** the long-lived closures (`tick`, IO callbacks, resize handler) read mutable `let` variables at call time (`playing`, `visible`, `colors`, `bars`), not captured snapshots. `colors` is refreshed on every `layout()`, so a theme change propagates correctly. No stale-closure bugs.

### Type Safety Assessment
There is no type system. The risk points are few and currently safe because the markup is static: `+el.dataset.to` is guarded by the `.num[data-to]` selector; `btn.querySelector('.ic')`, `.querySelector('.mlbl')` and `document.querySelector('.hero-inner')` assume elements that are hard-coded in the same file; `getContext` is feature-checked; `localStorage` reads and writes are wrapped in `try/catch`. The weak spot is coupling by class name between CSS, HTML and JS. A future markup edit could break the JS silently. F-N3 adds cheap checking.

### Verdict Details
- Component Architecture: **PASS**. Clear separation into theme, menu, reveal and hero; the hero is a self-contained IIFE.
- State Management: **CONCERNS**. Menu open state and focus aren't kept in sync (F-R1).
- Type Safety: **PASS** (for vanilla JS). Guards are in place where the DOM could vary.
- Rendering Performance: **CONCERNS**. Rebuilds on every resize (F-R2), plus the scale issues in the Performance lens.
- Bundle Impact: **PASS**. About 43KB HTML with everything inlined, no JS dependencies, 2 font files loaded on first view (139KB total transfer measured).

---

## 4. Backend Engineer

**Skipped**: No Rust, Tauri or server code. This is a static GitHub Pages site with no backend, IPC or persistence beyond one `localStorage` key (covered under Production Readiness).

---

## 5. Structural Engineer

**Skipped**: Single-file change with no new modules, layers or dependency edges. The one structural observation (the hero renderer could live in its own `hero.js`) is recorded as R-N3 under Repo Maintainer.

---

## 6. Repo Maintainer

**Verdict**: APPROVE WITH CHANGES

> Scope note: this is a GitHub Pages user site with no build, no package manager and no lint config. The lens's TypeScript and Rust checks (barrels, `import type`, path aliases, structured logger, Clippy) have nothing to apply to and are marked n/a. What does apply: file hygiene, dead code, duplication, history and repo docs.

### Plan Summary
Four commits, all on top of `main` (`d733801`), replace `index.html` wholesale (−1,246 / +718 lines), delete the 1.06MB `profile.jpg` and add two 480px images (`profile.webp` 35KB, `profile-480.jpg` 51KB). `dev` and `main` are identical, so merging is a clean fast-forward with no conflicts.

### Files Reviewed
| File | Convention Status | Risk |
|------|------------------|------|
| `index.html` | One self-contained page, same as `main`'s structure (inline `<style>` and `<script>`) | Low |
| `profile.jpg` (deleted) | Correctly removed with its only reference (searched the tree: no remaining references) | Low; old cached social previews may point at it, see P-R3 |
| `profile.webp`, `profile-480.jpg` (added) | Clear names; both 480×480; the JPG doubles as `og:image` | Low |
| `README.md` | **A single newline (1 byte)** on both `main` and the branch | Medium for a portfolio repo, see R-R1 |

### Critical Issues (Must Fix)
None.

### Recommendations
| # | Issue | Why | Fix |
|---|-------|-----|-----|
| R-R1 | **The README is empty.** | This repo is pinned public evidence of your engineering. A visitor who lands on `github.com/samarthsingh1/samarthsingh1.github.io` (from your profile, or by clicking "Source on GitHub") sees an empty page. | A short README: live URL, one screenshot, what it's built with ("one static HTML file, no build"), how to run it locally (`python3 -m http.server`), and how theming and the hero canvas work. |
| R-R2 | **The same gradient-text declaration is copied three times.** `background: var(--holo); -webkit-background-clip: text; background-clip: text; color: transparent;` appears in `.holo`, `.l.featured` and `.project a:hover .ptitle`. `.lbl` and `.ptag` are also byte-for-byte the same (mono, 11.5px, .14em tracking, uppercase, muted). | Edits will drift; for example, the light-theme contrast fix (D-C2) only works if it lands everywhere. | Keep `.holo` as the single utility and add the class in markup (`<a class="l holo">`), or combine selectors: `.holo, .l.featured, .project a:hover .ptitle { … }` and `.lbl, .ptag { … }`. |
| R-R3 | **The branch history includes a superseded design.** Commit `9b461ed` ("clean, minimal single-column layout") was fully replaced by `416cbb5` ("dark, monospace, numbered-section"). | On `main`, `git log` would show two redesigns a day apart, and the first never shipped. | Squash-merge the PR, or keep the commits but describe both in the PR body. This is your preference; nothing breaks either way. |
| R-R4 | **The branch doesn't follow the repo's established flow.** Every previous change went `dev` → PR → `main` (PRs 3–8). This work is on `claude/*` branches. | The `dev` branch will fall behind `main` after the merge. | Either open the PR into `dev` and then do `dev` → `main` as usual, or merge into `main` and fast-forward `dev` afterwards. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| R-N1 | Add a 404 page (`404.html`) that matches the design. | GitHub Pages otherwise shows its generic 404 for mistyped paths. |
| R-N2 | Add `.nojekyll`. | The site doesn't use Jekyll; skipping it makes Pages builds faster and avoids Jekyll ignoring files that start with `_` in the future. |
| R-N3 | Move the ~150-line hero renderer to `hero.js` (with `defer`). | It's the only part of the page with real logic. A separate file gets syntax highlighting, `@ts-check` (F-N3), and readable diffs. Keeping the CSS inline is fine. |

### Convention Compliance Checklist
| Convention | Status | Notes |
|-----------|--------|-------|
| File placement | ✅ | Root-level assets, the expected layout for a Pages user site |
| Naming conventions | ✅ | kebab-case for files; short, consistent BEM-free class names (`.sh`, `.lbl`, `.pno`) applied consistently |
| Barrel exports | n/a | No modules |
| Import ordering | n/a | No imports |
| Type imports separated | n/a | No TypeScript |
| Path aliases used | n/a | No bundler |
| No dead code left | ✅ | Old photo removed; no CSS class without markup (scripted check). `.mlbl` has no CSS, but it's a JS hook, which is fine. |
| Structured logging | ✅ | No `console.*` calls |
| No lint suppressions | ✅ | None |
| CLAUDE.md present | ⚠️ | None. Optional for a one-file site, but a README is overdue (R-R1). |

### Reuse Opportunities
- `.holo` already exists as a gradient-text utility; two other rules re-declare it instead of using it (R-R2).
- `.lbl` and `.ptag` could be one class.
- The `.btn` gradient-border technique (`linear-gradient(var(--x), var(--x)) padding-box, var(--holo) border-box`) is reused correctly by `.badge`. Good.

### Verdict Details
- File Organization: **PASS**
- Naming Consistency: **PASS**
- Import Hygiene: **PASS** (n/a)
- Code Reuse: **CONCERNS**. Three copies of the gradient-text block (R-R2).
- Lint Compliance: **PASS**. No tooling, and no obvious HTML validity problems. Block content inside `<a>` is valid HTML5 because it contains no interactive descendants.

---

## 7. Production Readiness

**Verdict**: APPROVE WITH CHANGES

### Plan Summary
What ships: one static `index.html` plus two images, served by GitHub Pages from `samarthsingh1.github.io`. There is no server, database or build step. The production risks are graceful degradation (fonts, JS, old browsers, print), a contact channel that might not deliver, and how the page appears when the link is shared.

### Files Reviewed
| File | Production Role | Risk |
|------|----------------|------|
| `index.html` | The whole site: markup, CSS and JS inline, so every deploy is atomic | Medium (see failure modes) |
| `profile.webp`, `profile-480.jpg` | Avatar, and the `og:image` for link previews | Low |
| External: `fonts.googleapis.com` / `fonts.gstatic.com` | Render-blocking web-font stylesheet plus 2 woff2 files on first view | Low–Medium |
| External: 5 GitHub repos, the GitHub profile, LinkedIn, `mailto:` | Every outbound conversion path | Checked: all 5 project repos are **public** (GitHub search API); `f1-commentary` is a public fork; `Loan-Application-Classification` differs only in case, which GitHub resolves. LinkedIn could not be checked from this network. |

### Critical Issues (Must Fix)
| # | Issue | File(s) | Production Failure Mode | Fix |
|---|-------|---------|------------------------|-----|
| P-C1 | **The only contact channel is unconfirmed** (same as U-C1). Both CTAs and the footer use `mailto:samarth1@umd.edu`; the degree ended May 2026, and there's no fallback channel on the page. | `index.html:560, 577` | Silent failure: recruiter mail bounces, or is accepted and never read. No error reaches you or the sender in the "accepted, unread" case. | **Needs your input.** Confirm the address delivers (send yourself a test), or switch to a personal address before merging. |

### Recommendations
| # | Issue | Why | Fix |
|---|-------|-----|-----|
| P-R1 | **A single JS error before line 620 hides the entire page.** The head script adds `.js` immediately, and `.js .reveal { opacity: 0 }` then hides every section until the reveal code runs. That code sits *after* the theme and menu setup in the same script, so any exception above it (a future markup edit that removes `#year`, or an extension altering the DOM) leaves all six sections permanently invisible. | It turns a minor bug into a blank site. Today's code doesn't throw (no page errors measured at any viewport), but the failure mode is total. | Add a CSS failsafe that needs no JS: `.js .reveal { animation: reveal-failsafe 0s 2.5s forwards; } @keyframes reveal-failsafe { to { opacity: 1; transform: none; } }`. Or move the reveal block to the top of the script and wrap the others in `try { … } catch {}`. |
| P-R2 | **Link previews will be small and cropped.** `og:image` is a 480×480 square; LinkedIn, Slack, iMessage and X render large previews at 1.91:1 (1200×630). There's no `twitter:card`, `og:image:width/height` or `og:image:alt`. | This link mostly travels as a preview in recruiter DMs and Slack threads, so the preview is effectively the first impression. | Add a 1200×630 `og-card.png` (the dark hero with your name works), plus `<meta property="og:image:width" content="1200">`, `…height 630`, `og:image:alt`, and `<meta name="twitter:card" content="summary_large_image">`. After deploying, refresh LinkedIn's cache with its Post Inspector. |
| P-R3 | **No canonical URL or structured data.** "Samarth Singh" is a common name. | Without `rel=canonical` and JSON-LD `Person`, search engines have less to tie this page to you, your Intuit role, UMD, GitHub and LinkedIn. | Add `<link rel="canonical" href="https://samarthsingh1.github.io/">` and a `<script type="application/ld+json">` Person block (`name`, `jobTitle`, `worksFor: Intuit`, `alumniOf`, `sameAs: [GitHub, LinkedIn]`, `url`, `image`). |
| P-R4 | **The Google Fonts stylesheet is render-blocking.** If `fonts.googleapis.com` is slow rather than refused (some corporate proxies hang instead of rejecting), the first paint waits for the request to time out. | Recruiters often browse from locked-down corporate networks. | Self-host the two woff2 files actually used (53KB) with `@font-face { font-display: swap }`, or load the stylesheet non-blocking (`rel="preload" as="style" onload="this.rel='stylesheet'"` with a `<noscript>` fallback). The fallback stack (`ui-monospace, SF Mono, Menlo, Consolas`) and the canvas's `fonts.load().catch()` already degrade correctly. |
| P-R5 | **Print and PDF output are blank after the hero** (F-C1). | A real production output that recruiters use. | See F-C1. |
| P-R6 | **No-JS and old-browser degradation gaps** (F-R5, D-R5). | The mobile nav is unreachable without JS; the hero scrim disappears on browsers without `color-mix`. | See F-R5 and D-R5. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| P-N1 | Privacy-friendly analytics (GoatCounter, Cloudflare Web Analytics or Plausible: one script tag, no cookies). | Right now you can't tell whether anyone visits, which sections they reach, or whether the email button gets clicked. |
| P-N2 | Namespace the storage key: `localStorage['samarth-theme']` instead of `'theme'`. | Every GitHub Pages project site you publish under `samarthsingh1.github.io/<repo>` shares this origin and its `localStorage`, so a generic `theme` key can collide with another project's. |
| P-N3 | Add `robots.txt` and `sitemap.xml`, plus `404.html` (R-N1). | Standard for an indexed site; negligible effort. |
| P-N4 | After merging, check Settings → Pages shows the source as `main` / root. | Inferred from history (every previous change merged to `main`, with `index.html` at the root); this session has no access to Pages settings. |

### Failure Mode Analysis
| Scenario | What Fails | User Impact | Recovery | Plan Handles It? |
|----------|-----------|-------------|----------|-------------------|
| Google Fonts refused | Web font | Fallback monospace font; canvas measures the fallback font | Automatic | ✅ |
| Google Fonts hangs | Render-blocking CSS | Blank screen until the request times out | Automatic, but slow | ⚠️ P-R4 |
| JS disabled | Reveal, canvas, toggles | Content visible (`.js` never set) ✅; mobile nav unreachable and dead buttons ❌ | None | ⚠️ F-R5 |
| Runtime JS error before reveal init | Reveal never runs | **Every section stays invisible** | Only a code fix | ❌ P-R1 |
| No IntersectionObserver | Reveal / visibility pause | Everything shown immediately | Automatic | ✅ (feature-checked) |
| `localStorage` blocked (Safari private, storage disabled) | Theme persistence | Theme isn't remembered | Automatic | ✅ (`try/catch`) |
| No `color-mix()` (Safari < 16.2, Chrome < 111) | Hero scrim, button glow | Hero text sits directly on the animated code field | None | ⚠️ D-R5 |
| Print / Save as PDF | Hidden reveals, light text | Blank pages after the hero | None | ❌ F-C1 |
| Mailbox retired | Contact | Messages bounce or vanish | None on the page | ❓ P-C1 (needs your input) |
| A linked repo is renamed or made private | Project card | The card leads to a GitHub 404 | Edit the link | ✅ all 5 verified public today |
| 5K / 4K hi-DPI monitor | Canvas cost | ~112MB of canvas backing stores; main thread near saturated (headless measurement) | Pause motion / scroll past the hero | ⚠️ Performance lens |
| Bad deploy | Whole site | Wrong page live | `git revert` the merge; Pages redeploys in about a minute; single-file deploys are atomic, so there's never a half-updated mix of old and new | ✅ |
| CDN cache after deploy | Freshness | Old page shown for up to ~10 minutes (Pages' default `max-age=600`) | Automatic | ✅ acceptable |

### Environment Checklist
| Concern | Dev | Prod | Gap? |
|---------|-----|------|------|
| File paths | Relative (`profile.webp`, `profile-480.jpg`) | Same files at the site root | No. Relative paths also work under a sub-path. |
| Network | Local server, fonts reachable | Pages CDN plus Google Fonts | No (a hanging font host is the one risk, P-R4) |
| Permissions | None | None | No |
| Config | None (no env vars, no build) | Pages default (Jekyll passes through `index.html`, which has no front matter) | No; `.nojekyll` is optional (R-N2) |
| Build output | The source file *is* the output | Same | No |
| Absolute URLs | `og:url` and `og:image` are hard-coded to `https://samarthsingh1.github.io/` | Correct for production | No (previews from branch deploys would point at production, which is fine) |

### Verdict Details
- E2E Flow: **CONCERNS**. Visit → read → contact depends on an unconfirmed mailbox (P-C1).
- Error Recovery: **CONCERNS**. One early JS exception blanks the page (P-R1).
- Data Integrity: **PASS**. The only persisted state is the theme preference, and its access is guarded.
- Environment Parity: **PASS**. No build, relative assets, correct absolute meta URLs.
- Observability: **CONCERNS**. No analytics and no error reporting. That's acceptable for a portfolio, but you're blind to traffic and conversions (P-N1).

---

## 8. DX Engineer

**Skipped**: The product is a personal portfolio for recruiters and hiring managers, not a developer tool. There's no CLI, configuration, API surface or developer-facing error message to audit. Maintainer ergonomics (README, file layout) are under Repo Maintainer.

---

## 9. Performance Engineer

**Verdict**: APPROVE WITH CHANGES

> Scope note: there's no IPC, streaming or file I/O here. The lens is applied to its web equivalents: network weight and the critical path, the per-frame cost of the canvas loop, memory, listener lifecycle, and a tab left open for hours. All numbers come from Chromium via Playwright and the Chrome DevTools Protocol (`Performance.getMetrics`). **Caveat:** headless Chromium rasterizes in software, so paint-heavy numbers (TaskDuration) overstate what a GPU-backed browser spends. Use them to compare configurations, not as absolute device timings. Memory sizes and frame counts are exact.

### Plan Summary
Page weight drops sharply compared with `main`. The new cost is a continuously animating full-viewport canvas in the hero. Its CPU and memory cost grows with screen pixel area, and it runs for as long as the hero is visible.

### Files Reviewed
| File | Performance Role | Risk |
|------|-----------------|------|
| `index.html:633-782` | Hero renderer: offscreen background, rAF loop throttled to ~30fps, IntersectionObserver pause | Medium |
| `index.html:16-18` | Google Fonts: 2 preconnects plus a render-blocking stylesheet, 4 faces requested | Low |
| `index.html:300-303` | `<picture>`: 35KB WebP / 51KB JPG for a 36px avatar | Low |
| `index.html:609-631` | Reveal observer and count-up (bounded, one-shot) | Low |

### Critical Issues (Must Fix)
None. The loop is correctly bounded: it does zero work off-screen, it stops in background tabs, and memory doesn't grow.

### Recommendations
| # | Issue | File(s) | Why | Fix |
|---|-------|---------|-----|-----|
| PF-R1 | **Every frame repaints the whole viewport, so cost grows with screen pixels.** `draw()` runs `clearRect` + `drawImage` over the full backing store 30 times a second, although only the histogram (about 19% of the area on desktop) and up to 24 spark cells change. The backing stores are `W×H×dpr²` and there are two of them (visible plus offscreen). Measured: **8MB** (iPhone 13), **40MB** (1440×900 @2x MacBook), **113MB** (2560×1440 @2x, e.g. a 27" 5K display). Main-thread task time while the hero is visible: 112 ms/s at 1280×800 @1x, 418 ms/s at 1440×900 @2x, 452 ms/s on the phone at 4× CPU throttle, 986 ms/s at 2560×1440 @2x (software raster; see caveat). Paused: 0.7 ms/s. | `index.html:670-671, 680, 716-748` | Large hi-DPI displays get the heaviest load: possible scroll and hover jank in the hero, fan noise, laptop battery drain. 113MB of backing store for a decorative effect is a lot on any machine. | (1) Cap the DPR the canvas uses: `dpr = Math.min(window.devicePixelRatio || 1, W * H > 1.5e6 ? 1 : 1.5)`; the field is faint text, and 1× is indistinguishable at 10% opacity. (2) Split into two stacked canvases: a static bottom layer drawn once in `layout()` (no per-frame `drawImage`), and a transparent top layer where each frame clears only the histogram's bounding box and the previous spark cells. That cuts per-frame raster area by about 80% on desktop. |
| PF-R2 | **Each resize event rebuilds the full canvas** (F-R2): 3 no-op resizes created 3 new offscreen canvases. On phones, the URL bar showing or hiding fires `resize` during ordinary scrolling. | `index.html:666-680, 778` | Repeated 5–8MB allocations and full re-renders of every row of text on mobile, plus GC churn, exactly where CPU is scarcest. | Skip `layout()` when `width`, `height` and DPR are unchanged; force it only on theme change. |
| PF-R3 | **The hero animates indefinitely while it's visible.** A tab left open at the top of the page (second monitor, reference tab during an interview loop) runs 30fps forever. Backgrounding the tab and scrolling away both stop it correctly (measured 0 frames over 2s off-screen). | `index.html:750-766` | Continuous battery and CPU use with nobody watching. | Auto-pause after about 45s without `pointermove`, `scroll` or `keydown`, and resume on the next interaction. Freeze on the current frame, don't clear it. Optionally drop to 15fps after the first 10s. |
| PF-R4 | **The 36px avatar downloads a 480px image.** `profile.webp` is 35KB, **25% of the whole page's transfer** (139KB measured), for a 36×36 CSS-pixel render (108 device pixels at 3×). | `index.html:301-302` | The single largest asset, and the easiest to shrink. | Add a 120×120 WebP (~4KB) as the `<source>`. Keep `profile-480.jpg` only as the `og:image`, or replace it with a 1200×630 card (P-R2). |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| PF-N1 | Drop the 700 weight from the Google Fonts request; use 800 for `.l.featured`, `.badge` and `.table dt`, or self-host (P-R4). | Requests 4 faces where 3 would do. Self-hosting also removes two cross-origin connection setups from the critical path. |
| PF-N2 | Use a more opaque `--nav-bg` (e.g. .85) and drop `backdrop-filter` from `.motion` and the mobile `.links`. | A backdrop blur above a canvas that changes 30 times a second makes the compositor re-blur that region every frame. That's cheap on desktop GPUs and measurable on low-end Android. |
| PF-N3 | Drive the loop with a `setTimeout(…, 33)` plus rAF pair instead of waking rAF at 60–120Hz to early-return. | On 120Hz ProMotion displays the callback fires 120 times a second to draw 30 frames. Each wake is cheap, but there's no reason for it. |

### Resource Budget
| Resource | Current (`main`) | After Plan (branch) | Concern? |
|----------|---------|-----------|----------|
| First-view transfer | > 1.1MB: `profile.jpg` alone is 1,087KB, plus Font Awesome (CSS and webfonts), Prism (1 CSS, 2 JS) and 9 Google font weights | **139KB measured** (HTML 43.6KB, WebP 35.6KB, fonts CSS 9.9KB, 2 woff2 at 53.7KB) | ✅ roughly 8× lighter |
| Render-blocking requests | 3 stylesheets (Prism theme, Font Awesome, Google Fonts) | 1 (Google Fonts) | ✅ |
| Third-party JS | Prism core plus a language pack | None | ✅ |
| Event listeners added | n/a | 11 page-lifetime listeners (theme 1, menu 1, nav links 6, document keydown 1, document click 1, window resize 1) plus 2 IntersectionObservers (the reveal one unobserves each target after one use) | ✅ fixed count, nothing accumulates |
| Memory growth per hour | n/a | JS heap after GC: 0.66MB → 0.69MB over 30s (flat); sparks capped at 24 | ✅ |
| Canvas memory (fixed) | none | 8MB (phone) / 40MB (MacBook @2x) / 113MB (5K @2x) | ⚠️ PF-R1 |
| Main-thread cost, hero visible | n/a | 112–986 ms/s depending on screen (software raster, overstated) | ⚠️ PF-R1, PF-R3 |
| Main-thread cost, hero off-screen or paused | n/a | ~0.7 ms/s; 0 frames drawn | ✅ |
| Startup | n/a | DOMContentLoaded 177ms, FCP 204ms (local server). The canvas waits for `document.fonts.load()` and paints after first paint, so it never blocks content. | ✅ |
| Bundle (HTML) | 46.5KB | 43.6KB | ✅ |

### Hot Path Analysis
- **Per-frame path (~30fps while the hero is visible and playing):** `tick` → `draw()`:
  1. Full `clearRect` of the backing store.
  2. Full-size `drawImage` of the offscreen background. This is the dominant raster cost and it scales with `W×H×dpr²`.
  3. Up to 24 spark `fillText` calls with `globalAlpha`.
  4. For each bar (5–23 depending on width): two `fillRect` calls, plus one `fillText` of 3 characters per row (up to ~42 rows on a 1440px-tall screen), so roughly 290 calls at 1440×900 and ~870 at 2560×1440.
  5. Axis: one `fillRect` and one `fillText`.

  Allocation per frame: one 3-character string per bar row, plus a filtered `sparks` array. That's small and short-lived, and the GC numbers confirm it (heap flat).
- **Per-rAF path (60–120Hz):** `tick` early-returns until 33ms have passed. This is negligible, but it is a wake-up (PF-N3).
- **Scroll path:** no scroll listeners. IntersectionObservers handle reveal and visibility, which is the right primitive. `position: sticky` labels and the fixed nav are compositor-handled.
- **Resize path:** debounced 150ms → full `layout()` regardless of whether the size changed (PF-R2).
- **One-shot paths:** count-up (1s rAF per stat, 4 stats, once); reveal transitions (opacity and transform only, compositor-friendly).

### Verdict Details
- Memory Management: **CONCERNS**. No leaks or growth, but a large fixed canvas footprint on hi-DPI screens (PF-R1).
- IPC Efficiency: **PASS** (n/a). Network requests went from more than 10 to 5.
- I/O Patterns: **PASS**. One `localStorage` read at startup, one write per theme toggle.
- Startup Impact: **PASS**. The canvas is deferred until fonts are ready; only one render-blocking stylesheet.
- Long-Session Stability: **CONCERNS**. Correct pausing when off-screen or hidden, but it animates indefinitely while visible (PF-R3).

---

## 10. Accessibility Engineer

**Verdict**: APPROVE WITH CHANGES

### Plan Summary
The redesign adds a skip link, a landmarked floating nav with a disclosure-style mobile menu, an icon-only theme toggle, a canvas animation with a pause control, card-style project links and count-up stats. The foundations are good: landmarks, heading order, labels and reduced motion. The gaps are in mobile-menu focus handling, contrast in the light theme, the motion control's coverage and semantics, and how verbose links are for screen readers.

Evidence: axe-core 4.10.2 (WCAG 2.0/2.1/2.2 A and AA plus best practices) in both themes; keyboard walkthroughs in Chromium at 1280px and iPhone 13 sizes; Playwright ARIA snapshots; reflow at 320px; hand-calculated contrast for the gradient text, which axe can't measure.

### Files Reviewed
| File | A11y Role | Risk |
|------|----------|------|
| `index.html:273-290` | Skip link, `<header>`, `<nav aria-label="Primary">`, theme toggle, mobile menu button | High (menu focus) |
| `index.html:294-320` | Hero: avatar `alt`, `h1`, CTAs, `aria-hidden` canvas, motion toggle | Medium |
| `index.html:322-567` | Sections: `h2`/`h3` structure, lists, `dl`, card links | Medium |
| `index.html:25-270` | Focus styles, contrast tokens, reduced motion | Medium |
| `index.html:583-782` | Menu, theme and motion state handling | High |

### Critical Issues (Must Fix)
| # | Issue | WCAG Criterion | File(s) | Impact | Fix |
|---|-------|---------------|---------|--------|-----|
| A-C1 | **The mobile menu breaks keyboard focus order and loses focus on close.** `#nav-links` comes *before* `#menu` in the DOM. Measured: open the menu with Enter, press Tab, and focus goes to "See my work" in the hero, skipping all 6 menu links. Shift+Tab reaches them only in reverse. Press Escape while on a menu link and `document.activeElement` becomes `<body>`. | 2.4.3 Focus Order (A); 2.4.7 Focus Visible (AA) when focus is lost | `index.html:277-289, 603-606` | The mobile layout also applies to **desktop users at 200–400% zoom**, a core low-vision group (reflow at 320px is otherwise perfect). For them the nav is effectively unusable. | Put `#menu` before `#nav-links` in the markup; CSS `order` can keep the visual layout. On open, focus the first link. On Escape (only when the menu is open), close it and focus `#menu`. Don't move focus on outside clicks. |
| A-C2 | **Text contrast fails, mostly in the light theme.** axe-core reports 5 nodes in dark (`.pno`, 3.39:1) and 11 in light (`.sh .no` section numbers at 3.23:1, `.pno` at 3.14:1). Gradient text, which axe can't measure: the light cyan stop `#0a93b0` is 3.23:1 and the pink stop `#cf3a8f` is 4.04:1. Those stops land on "INTUIT" (12px) and "Projects" (14px). | 1.4.3 Contrast (Minimum) (AA) | `index.html:30-31, 52, 60-62, 171, 203` | Low-vision users can't reliably read the section numbers, project numbers or employer name in light mode. | Verified replacements: light `--holo-1 #08778e` (4.65:1), light `--holo-3 #c32f83` (4.63:1), light `--faint #726c61` (4.66:1), dark `--faint #80786b` (4.61:1). Same as D-C2. |
| A-C3 | **Blinking content keeps blinking with no way to stop it.** The `.cur` cursors in the fixed nav and the footer blink forever (`animation: blink 1.1s steps(1) infinite`). "Pause motion" stops only the canvas; measured `animationName` on the nav cursor after pausing is still `blink`. Only the OS reduced-motion setting stops it. | 2.2.2 Pause, Stop, Hide (A) | `index.html:103-104, 774` | Blinking that runs alongside reading content distracts users with attention or vestibular disorders. The nav cursor is in view on every screen. | Simplest: blink for under 5 seconds and then stay solid: `animation: blink 1.1s steps(1) 4`. Alternatively, have the motion toggle set `html.motion-paused` with `.motion-paused .cur { animation: none; }`, and persist that choice. |

### Recommendations
| # | Issue | WCAG Criterion | Fix |
|---|-------|---------------|-----|
| A-R1 | **The motion toggle contradicts itself.** It changes both its label and `aria-pressed`. After pausing, screen readers announce "Play motion, toggle button, **pressed**", which suggests play is on. Under reduced motion it starts in that state. | 4.1.2 Name, Role, Value (A) | Pick one pattern. Either keep the label fixed at "Pause motion" and toggle `aria-pressed` (pressed = paused), or keep the changing label and remove `aria-pressed`. |
| A-R2 | **Project links are announced as the whole card**, about 200 characters each: "p/01 Capital One competition Deep-Agent AI for Food Bank Operations Real-time analytics pipeline … Kafka ClickHouse FastAPI Docker". They also open a new tab without saying so. | 2.4.4 Link Purpose (A) passes, but verbose; G201 (new-window warning) advisory | Make only the `<h3>` text the link, and stretch it over the card with `.ptitle a::after { content: ""; position: absolute; inset: 0; }` (card `position: relative`). Add `<span class="vh"> (opens in new tab)</span>` to external links. |
| A-R3 | **Several section headings don't name their section.** Heading navigation lists "Data person, turned software engineer." (About) and "The toolbox. What I reach for." (Skills). The "01 About" labels are `div`s, outside the headings. | 2.4.6 Headings and Labels (AA) | Prefix each `h2` with a visually hidden section name: `<h2><span class="vh">About. </span>Data person,…`. Optionally add `aria-labelledby` on each `<section>` so they also show up as named regions. |
| A-R4 | **Decorative arrows are read aloud.** `.ghost .arr` (↓ →) isn't `aria-hidden`, so screen readers say "Experience, down arrow" and "LinkedIn, right arrow". The project ↗ already is hidden. | Best practice | Add `aria-hidden="true"` to every `.arr` span. |
| A-R5 | **`aria-label` on a plain `div`.** `<div class="mark" aria-label="samarth">` uses an attribute that ARIA 1.2 prohibits on generic elements. The text "samarth" is already there. | 4.1.2 (best practice) | Remove `aria-label` from `.mark`. |
| A-R6 | **Without JS, the mobile nav is unreachable** (F-R5): the menu button is visible but inert. | 2.1.1 Keyboard (A) when JS is off | Scope the mobile collapse and the JS-only buttons to `.js`. |

### Nice-to-Haves
| # | Suggestion | Rationale |
|---|------------|-----------|
| A-N1 | Give the count-up a stable accessible value: put the final value in a visually hidden span and `aria-hidden` the animating one. | A screen reader that reads a stat mid-animation hears "0+" or "13+". Rare, but free to prevent. |
| A-N2 | Make the footer links at least 24px tall (`padding: 2px 0` on `.flinks a`). | They measure 22px. They pass 2.5.8 through the spacing exception (24px gaps), but a larger target is friendlier on touch. |
| A-N3 | Use `alt=""` on the 36px avatar. | The name is in the `h1` right after it, so "Portrait of Samarth Singh" is redundant. Keeping it is also acceptable. |

### Keyboard Navigation Walkthrough
| Step | Action | Expected Focus | Expected Announcement | Plan Handles? |
|------|--------|---------------|----------------------|---------------|
| 1 | Tab from page load | Skip link becomes visible top-left | "Skip to content, link" | ✅ |
| 2 | Enter | `<main>` becomes the sequential focus start | (moves to main content) | ✅ |
| 3 | Tab (desktop, from top) | Brand | "Samarth Singh, back to top, link" (contains the visible "samarth", so 2.5.3 passes) | ✅ |
| 4 | Tab ×6 | About … GitHub, in visual order | "About, link" … "GitHub, link" | ✅ |
| 5 | Tab → Enter | Theme toggle | "Switch to light theme, button"; after activation the label reads "Switch to dark theme" | ✅ |
| 6 | Tab ×3 | "See my work", "Experience ↓", "Get in touch →" | "Experience **down arrow**, link" | ⚠️ A-R4 |
| 7 | Tab → Enter | Motion toggle | "Pause motion, toggle button, not pressed" → "Play motion, toggle button, **pressed**" | ⚠️ A-R1 |
| 8 | Tab ×5 | Project cards | Each is ~200 characters of card text; opens a new tab without warning | ⚠️ A-R2 |
| 9 | Tab ×3 | Email, LinkedIn, GitHub | "samarth1@umd.edu, link", "LinkedIn right arrow, link" | ⚠️ A-R4 |
| 10 | Tab ×3 | Footer links | "GitHub, link", "LinkedIn, link", "Email, link" | ✅ |
| 11 | Any focus | Outline visible: 2px `--holo-2` (8.8:1 on dark, 4.5:1 on light), not hidden by the fixed nav (`scroll-padding-top: 96px`) | n/a | ✅ (the shape change is design, see D-C1) |
| M1 | Mobile or zoomed: Tab to the menu button, then Enter | Menu button | "Menu, button, collapsed" → "expanded" | ✅ |
| M2 | Tab | Should be the first menu link | Actually goes to "See my work"; menu skipped | ❌ A-C1 |
| M3 | Shift+Tab ×2 | Should be the last menu link | Reaches "GitHub", in reverse order | ⚠️ A-C1 |
| M4 | Escape while on a menu link | Should return to the menu button | Focus falls to `<body>` | ❌ A-C1 |
| M5 | Enter on a menu link | Menu closes; page jumps to the section | (section content) | ✅ |

### ARIA Checklist
| Element | Role | Label | States | Keyboard | Notes |
|---------|------|-------|--------|----------|-------|
| Skip link | link | "Skip to content" | — | ✅ | Visible on focus |
| `<header>` / `<nav>` | banner / navigation | "Primary" | — | ✅ | Correct landmarks |
| Brand | link | "Samarth Singh, back to top" | — | ✅ | |
| Theme toggle | button | "Switch to light/dark theme" (describes the action) | — | ✅ | SVGs are `aria-hidden` |
| Menu button | button | "Menu" | `aria-expanded`, `aria-controls` | ⚠️ | Focus order and return (A-C1) |
| Hero canvas | (hidden) | — | `aria-hidden="true"` | n/a | Decorative, correctly hidden |
| Avatar | img | "Portrait of Samarth Singh" | — | n/a | `width`/`height` set |
| Motion toggle | button | changes label | `aria-pressed` (conflicts) | ✅ | A-R1 |
| `h1` | heading 1 | "Samarth Singh. Data in. Decisions out." | — | n/a | The `<em>` break is announced with a space |
| Section `h2` ×6 | heading 2 | Taglines | — | n/a | A-R3 |
| Experience / education | list (`ol`) → listitem | — | — | n/a | Correct |
| Project cards | link containing heading 3 and a list | Entire card text | — | ✅ | A-R2 |
| Skills | description list (`dl`/`dt`/`dd`) | — | — | n/a | Good semantic choice |
| Stats | list | — | — | n/a | A-N1 |
| Footer mark | generic | `aria-label` (prohibited) | — | n/a | A-R5 |

### Verdict Details
- Screen Reader Compatibility: **CONCERNS**. Good landmarks and structure, but verbose card links, decorative arrows read aloud, the contradictory toggle, and headings that don't name their sections.
- Keyboard Navigation: **CONCERNS**. The desktop order is perfect; the mobile and zoomed menu is broken (A-C1).
- Focus Management: **CONCERNS**. Focus is lost on Escape; there's no focus move when the menu opens.
- Color Contrast: **CONCERNS**. Dark theme: only `.pno` fails. Light theme: 11 nodes plus the gradient stops (A-C2).
- Motion Sensitivity: **CONCERNS**. Excellent `prefers-reduced-motion` coverage and a pause control for the canvas, but the infinite cursor blink isn't covered (A-C3).
