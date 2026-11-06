# Clutch — scroll-scrubbed robotic claw holding a card (deep blue fintech page) — rebuild command

Build ONE file, `index.html`, no framework, no build step, no libraries. The page is a three-chapter fintech story told over a single 8.5-second video that is scrubbed by the scroll position: a black robotic claw hangs from two cables in a deep-blue studio with a bright floor light; a white chip card swings below it, turns flat, settles between the fingers and is held while the camera slowly pushes in and a light sweep crosses the card. The video ships as **204 WebP frames** (a 1920 × 1080 set for desktop and a 960 × 540 set for phones) drawn to one fixed full-viewport `<canvas>`. Three sections scroll over it, each with a sticky (pinned) layer whose content sits **only in the left and right columns**, never in front of the claw. All content is limited to a centred **1440 px** frame; only the video stage is full-bleed. Everything is exact: use the numbers below, never adjectives. Page language: English (`<html lang="en">`).

**Assets (Path A, hosted on Cloudflare R2).** All frames live at `https://assets.layerweb.com.br/clutch/frames-hd/` and `https://assets.layerweb.com.br/clutch/frames-sm/`. Do not download, re-encode or rename them; load them straight from these URLs. Integration logos are inline SVG paths (Simple Icons, CC0) already contained in Appendix A. The only other external resource is Google Fonts (Inter).

**Authoritative code.** Appendix A at the end of this document is the exact production file. If prose and appendix ever disagree, the appendix wins. Reproduce it verbatim, then run the self-check (section 13).

---

## 0. Global shell
- `<!doctype html><html lang="en">`, `<title>Clutch — Smart Grip on Every Cent</title>`, viewport `width=device-width, initial-scale=1`, a meta description, and an inline SVG favicon (section 5).
- Font: Google Fonts **Inter** weights **400, 500, 600** only: `https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap`, with the two `preconnect` links. Fallback `system-ui, sans-serif`. `-webkit-font-smoothing: antialiased`.
- Reset `*{margin:0;padding:0;box-sizing:border-box}`. `html{background:#000c16;color:#fff}`; `body{background:transparent;overflow-x:hidden}`. The page scrolls vertically (total document height 580vh).
- A one-line inline script right after `<body>` adds the class `js` to `<html>`. Then a hidden SVG sprite (`width="0" height="0" style="position:absolute"`, `aria-hidden`) with 27 `<symbol id="i-<name>" viewBox="0 0 24 24">` logo paths. One classic `<script>` (an IIFE) at the end of `<body>`.

## 1. Fixed design canvas and the 1440 px frame
- Design canvas **1440 × 1015 px**. Scale `k = min(min(innerWidth, 1440) / 1440, innerHeight / 1015)`, written by JS to `--k` (5 decimals). `--u: calc(var(--k) * 1px)`; every desktop length is `calc(var(--u) * N)`.
- **1440 px limit:** every `.pin` is `max-width:1440px; margin:0 auto`; the fixed `.nav` is `left:0; right:0; margin:0 auto; max-width:1440px`. On screens wider than 1440 px the content stays centred and only the video stage keeps filling the screen.
- Inside the frame each element is anchored to its own edge (left column → `left`, right column → `right`, top items → `top`, bottom items → `bottom`).
- Layers (z-index): `.stage` canvas `0` (position fixed, inset 0, `pointer-events:none`), `main` `10`, fixed `.nav` `20`.
- Recompute `k`, the stage geometry and the scroll anchors on `resize`, from a `ResizeObserver` on `<body>`, and from a 500 ms interval that compares `innerWidth/innerHeight` with the last values.

## 1.5 Absolute coordinate table (canvas px at k = 1, inside the 1440 frame; desktop / landscape ≥ 900 px)
| Element | Anchor | x | y | w × h | Type |
|---|---|---|---|---|---|
| Logo `.logo` (fixed) | left/top | 41 | 44 | 46 × 40 | SVG icon |
| Nav "Features" / "Journal" | left/top | 175 / 312 | 55 | auto | 16/20, 400, rgba(255,255,255,.78) |
| Nav "Security" / "Talk to Us" | right/top | right 384 / right 233 | 55 | auto | same |
| "Join Now" pill | right/top | right 40 | 40 | 122 × 48 | 16, 600, bg rgba(255,255,255,.14), blur 10px |
| Hero `h1` | left/top | 43 | 202 | nowrap | 87 / 85, 400, tracking −0.035em |
| Hero paragraph | right/top | right 34 | 210 | 280 wide | 20 / 28.5, rgba(255,255,255,.9) |
| Hero CTA (pill + circle) | right/top | right 41 | 393 | 215 + 4 + 54, h 54 | 16, 600, #04121d on #fff |
| "Budgets that flex / by themselves" | left/bottom | 41 | bottom 294 | auto | 15 / 19 |
| Social row | right/bottom | right 41 | bottom 295 | label + 3 × 54 circles, gap 4 | label margin-right 41 |
| Wordmark "CLUTCH PAY" | left 0, right 0 | centred | bottom 48 | nowrap | 226, 500, line-height .8, tracking −0.02em |
| S2 label "02 — By the Numbers" | left/top | 43 | 168 | auto | 13/16, 500, +0.14em, uppercase, #8fc6dd |
| S2 `h2` (2 lines) | left/top | 43 | 200 | nowrap | 66 / 68 |
| S2 paragraph | right/top | right 34 | 210 | 280 wide | 18 / 27 |
| S2 card 1 `.c1` | left/bottom | 43 | bottom 64 | 252 × 222 | glass card |
| S2 card 2 `.c2` | right/bottom | right 40 | bottom 298 | 274 × 222 | glass card |
| S2 card 3 `.c3` | right/bottom | right 40 | bottom 64 | 274 × 222 | glass card |
| S3 label "03 — Fits Your Stack" | left/top | 43 | 168 | auto | label style |
| S3 `h2` (2 lines) | left/top | 43 | 200 | nowrap | 66 / 68 |
| S3 paragraph | left/top | 43 | 362 | 300 wide | 18 / 27 |
| S3 CTA | left/top | 43 | 540 | 215 + 4 + 54, h 54 | as hero CTA |
| S3 note | left/top | 43 | 616 | auto | 15 / 19 |
| S3 logo columns `.logos` | right/top/bottom | right 48 | top 150, bottom 96 | 258 wide | 3 columns, tiles 78, gap 12 |
| S3 footer row | left 43, right 40 | — | bottom 40 | flex space-between | 13, rgba(255,255,255,.6) |

## 2. Video stage (contract — the key effect)
- Frames `clutch-000.webp` … `clutch-203.webp` (3-digit padding) — the source clip starts 1.5 s in, so the first card of the original clip is never shown. Desktop and landscape load `frames-hd/` (1920 × 1080); when the flow media query matches at load, `frames-sm/` (960 × 540). Geometry is always expressed in **1920 × 1080 source px**; drawing reads `naturalWidth/naturalHeight`, so both sets use the same code.
- Canvas backing store = viewport × `dpr`, `dpr = min(devicePixelRatio, 1.5)`; `imageSmoothingQuality = 'high'`.
- **Claw box (source px):** centre x **960**, width **830**, height **960** from the top (cables at y 0, hanging card at frame 0 reaches ~y 960).
- **Desktop geometry:** the claw is centred on the 1440 px content frame. `cw = min(innerWidth, 1440)`, `fl = (innerWidth − cw)/2`, `c = fl + cw/2`, `gl = fl + 300·k` (right edge of the lower-left content), `gr = fl + cw − 314·k` (left edge of the right column). `s = min(innerHeight / 1080, 2·(gr − c − 12·k) / 830, 2·(c − gl − 12·k) / 830)`, `ox = c − 960·s`, `oy = 0` (cables always hang from the top edge). The wide titles (up to x 565) stay clear because the claw is narrow at their height and its fingers spread below them.
- **Flow geometry (phones, portrait):** `s = min(innerWidth·0.94 / 830, max(band, 120) / 960)` where `band` runs from 10 px under the hero's `.top` to 10 px above its `.bot`; claw centred in that band.
- **Sideways-phone geometry:** `gl` = hero `.top` right + 12, `gr` = hero `.bot` left − 12, `s = min(innerHeight / 1080, (gr − gl) / 830)`, centred between them, `oy = 0`.
- **Edge extension (no letterbox, no seams — "elastic edges"):** where the frame does not reach the screen edge, the picture itself is stretched there; nothing is filled with a flat colour and nothing is faded (a linear fade or a flat strip leaves a visible vertical/horizontal line where the video's own vignette stops).
  1. **Sides.** Let `s` = frame scale, `L = max(0, ox)`, `R = max(0, vw − (ox + 1920·s))`, `A = 480` source px, `N = 40` slices per side. If `L > 0.5` or `R > 0.5`: left join `xl = ox + A·s`, right join `xr = ox + (1920 − A)·s`, `cl = L / A²`, `cr = R / A²`. The centre source range `[A, 1920 − A]` is drawn 1:1 into `[xl, xr]`. The left range `[0, A]` is cut into `N` source slices; with `r = A − u` (distance from the join, source px) slice edges land at `x = xl − (s·r + cl·r²)` (the first at exactly 0). The right range `[1920 − A, 1920]` likewise with `r = u − (1920 − A)` and `x = xr + (s·r + cr·r²)` (the last at exactly `vw`). The slope at each join equals `s`, so there is no kink; at the screen edge the stretch is `s + 2·L/A`. Every destination edge is rounded to a whole device pixel (`round(x·dpr)/dpr`) so neighbouring slices leave no hairline. If neither side is visible, the frame is a single slice `[0, 1920] → [ox, 1920·s]`. The claw never enters the stretched ranges (it spans source x 550 … 1370).
  2. **Bottom.** If `B = vh − (oy + 1080·s) > 0.5`: take the band of the already painted picture (both cross-faded frames included) from `y0 = oy + 1080·s − hb` to the frame bottom, `hb = 140·s`, copy it to a scratch canvas, and redraw it in `N = 36` horizontal slices stretched down to the screen bottom with `y = y0 + r + c·r²`, `c = B / hb²` (slope 1 at the join), at full alpha. This runs once per drawn position, after the last frame pass.
  3. **Floor dimmer (CSS, not canvas):** a `div.dim` inside `.stage`, `position:absolute; left:0; right:0; bottom:0`, height `120·s + B` px when `B > 0.5` (else 0), background `linear-gradient(180deg, rgba(0,12,22,0) 0, rgba(0,12,22,.016) 15%, rgba(0,12,22,.06) 30%, rgba(0,12,22,.14) 45%, rgba(0,12,22,.25) 60%, rgba(0,12,22,.38) 75%, rgba(0,12,22,.5) 88%, rgba(0,12,22,.58) 100%)` (an eased ramp: a straight ramp shows a bright ridge where it starts). It keeps the bottom texts readable and is not part of the frame blend.
  4. **Flow layout only** (frame starts below the top): flat `rgb(0,9,18)` above the frame and, drawn on the canvas, a gradient `rgb(0,9,18)` → transparent over the first `160·s` px of the frame so the cables fade in. DO NOT stretch the top rows (the cables turn into two long bars).
  5. DO NOT feather the frame edges with a mask, DO NOT paint flat side strips, DO NOT mirror the frame.
- Stage fades in (`opacity 0 → 1`, 0.9 s) when frame 0 has loaded. Loading: progressive, 8 parallel requests, order every 8th, 4th, 2nd, then the rest; a missing frame is replaced by the nearest loaded index.
- **Scroll → frame map** (piecewise linear; `y(sec)` = `offsetTop`, `h(sec)` = `offsetHeight`, `v` = `innerHeight`):
  | Scroll position | Frame | Story beat |
  |---|---|---|
  | 0 | 0 | card hangs below the open claw |
  | hero top + hero height − v | 16 | card mid-swing |
  | s2 top | 40 | card turning flat |
  | s2 top + s2 height − v | 64 | card settled in the grip |
  | s3 top | 140 | slow push-in during the hold |
  | document height − v | 203 | light sweep done, final frame |
- **Smoothing (image-sequence playback):** `cur` follows the scroll target with a critically damped spring driven by real elapsed time (no overshoot, same feel on 60 Hz and 144 Hz screens). `smoothDamp(x, goal, v, smoothTime = 0.17, dt)`: `ω = 2 / smoothTime`, `e = ω·dt`, `k = 1 / (1 + e + 0.48·e² + 0.235·e³)`, `d = x − goal`, `tmp = (v + ω·d)·dt`, new `x = goal + (d + tmp)·k`, new `v = (v − ω·tmp)·k`. `dt` = seconds since the previous animation frame, clamped to 0.001 … 0.064. Snap (`cur = target`, `v = 0`) when `|target − cur| < 0.004` and `|v| < 0.02`. Reduced motion → `cur = round(target)`, no spring, no blend.
- **Frame blending:** `cur` is fractional. With `a = floor(cur)` and `t = cur − a`, paint frame `a` at `globalAlpha = 1` and, when `t > 0.01`, frame `a + 1` on top at `globalAlpha = t` (skip the second pass if the two available frames are more than 2 indexes apart). Each pass paints the whole canvas opaquely (strips and dimmers included), so every pixel is exactly `(1 − t)·A + t·B`. Redraw only when `|cur − drawn| > 0.004` or the geometry changed. Between two frames the blend reads as a soft motion blur. DO NOT replace it with a jump to the nearest frame (slow scrolling would then play at 24 steps per second).
- **Decode ahead:** after each drawn position call `img.decode()` once per frame for the indexes `a − 2` … `a + 10` in the scroll direction (direction = sign of the spring velocity, or of `target − cur` when the velocity is 0), so drawing never waits for a decode. Keep the `Image` elements. DO NOT convert every frame to an `ImageBitmap` (about 1.7 GB of decoded pixels).
- DO NOT use a `<video>` element with `currentTime`. DO NOT use `object-fit: cover`. DO NOT centre the claw in the gap between the columns (it drifts right, because the left column is wider). DO NOT let any content leave the 1440 px frame.

## 3. Sections and pinning
- `main` holds `section.hero` **180vh**, `section.s2#results` **220vh**, `section.s3#stack` **180vh**. Each holds one `.pin`: `position:sticky; top:0; height:100vh` (then `100svh`), `overflow:hidden`, max-width 1440 centred. Desktop children of `.pin`, `.top`, `.bot`, `.stats` are `position:absolute`; `.top`, `.bot`, `.stats` are `display:contents` on desktop.
- **Pin fade-out:** every tick, `t = pin.getBoundingClientRect().top`; if `t < 0`, `opacity = max(0, 1 + t / (0.55·v))`, else cleared.
- **Reveal (one-time):** a section gets `on` when its top is above `0.45 × innerHeight`; the hero gets `on` at boot. S2 turning `on` starts the count-up.

## 4. Typography and copy (literal)
- Titles (`.title`): each line `span.ln` (`overflow:hidden; padding-bottom:.06em; margin-bottom:-.06em`) wraps a `span` filled with `linear-gradient(180deg,#ffffff 0%,#ffffff 30%,rgba(196,229,243,.66) 100%)` clipped to text. Three-line titles: `background-size:100% 300%`, positions `0 0 / 0 50% / 0 100%`. Two-line titles (`.title.two`): `100% 200%`, positions `0 0 / 0 100%`. The lines read as one vertical gradient.
- Hero `h1`: "Smart Grip on" / "Every Cent" / "You Earn" (ink widths 522 / 416 / 338 at 87 px). Paragraph: "Live spending alerts and automatic saving rules keep a steady hand on every card you own". CTA "Explore Now" → `#results`. Note "Budgets that flex" `<br>` "by themselves". Social label "Join the" `<br>` "Community". Wordmark "CLUTCH PAY".
- Nav: "Features", "Journal", "Security", "Talk to Us", pill "Join Now".
- S2: label "02 — By the Numbers"; `h2` "Results That" / "Keep Their Grip" (ink 354 / 449 at 66 px); paragraph "A few figures from the last twelve months, pulled straight from the dashboard our members open every day."; cards:
  - "120K+" · "Active Members" · "People in more than thirty countries use Clutch to track every purchase, from a morning coffee to the monthly rent."
  - "99.9%" · "Uptime All Year" · "Payments, alerts and card controls stayed online through every payday rush and holiday sale."
  - "4.9/5" · "Average App Rating" · "Scored by over eighteen thousand reviewers who moved their everyday spending to Clutch."
- S3: label "03 — Fits Your Stack"; `h2` "Plugs Right Into" / "Every Tool" (ink 444 / 298); paragraph "Link your bank, payroll, accounting and team apps in a couple of clicks. Data moves both ways in real time, so every budget stays current."; CTA "Browse Integrations" → `#top`; note "120+ apps · two-way sync · set up in minutes"; footer "© 2026 Clutch. All rights reserved." + "Privacy", "Terms", "Status".

## 5. Brand
- Icon only, `aria-label="Clutch"`, 46 × 40 at (41, 44). Idea: two hooks closing around a single point.
```html
<svg viewBox="0 0 46 40" fill="none" aria-hidden="true">
  <path d="M9 6C4 15 6 27 17 32" stroke="#fff" stroke-width="4.5" stroke-linecap="round"/>
  <path d="M37 6c5 9 3 21-8 26" stroke="#fff" stroke-width="4.5" stroke-linecap="round"/>
  <circle cx="23" cy="27" r="4" fill="#fff"/>
</svg>
```
- Favicon: the same shapes on a `#021a2b` rounded rect (rx 9), SVG data URI. DO NOT redraw or restyle the icon; DO NOT add a text wordmark next to it.

## 6. Pixel contracts — fragile components
**Wordmark "CLUTCH PAY".** White, `opacity:.24`, `mix-blend-mode:overlay`, centred in the 1440 frame, `bottom: 48u`. Overlay keeps it invisible on the black claw and the white card. DO NOT use plain translucent white or `soft-light`.

**CTA pair.** White pill 215 × 54 + white circle 54, gap 4; arrow 20 × 20, stroke 1.6. Hover: pill `#dff1fa`, circle rotates −45° (0.45 s `cubic-bezier(.16,1,.3,1)`). DO NOT merge into one button; no glow.

**Stat cards (S2).** Three `article.card`, `height:222u`, padding `26u 24u 24u`, radius 20u, background `rgba(1,14,24,.62)`, inset 1 px `rgba(255,255,255,.12)`, `backdrop-filter: blur(16px)`. Number 56u / 1, 400, −0.04em, tabular; title 16/22, 500, `margin-top:30u`; text 13/19, rgba(255,255,255,.62), `margin-top:8u`. Card 1 sits alone at the bottom of the left column; cards 2 and 3 are stacked at the bottom of the right column (12u apart). The claw hangs between the two groups. Count-up when S2 turns `on`: 1600 ms, ease-out cubic `1 − (1 − p)^3`, from 0 to `data-count` with `data-dec` decimals and `data-suffix` (`120` + "K+", `99.9` + "%", `4.9` + "/5"). Reduced motion: no count-up (final text stays). DO NOT place a card under the claw.

**Logo columns (S3).** `.logos` (role `img`, `aria-label` listing the brands) holds 3 `.col`, each with a `ul.track` (`aria-hidden`) of 9 logos written twice (18 tiles) so `translateY(−50%)` loops seamlessly. Tracks: `animation: up var(--dur) linear infinite` (`@keyframes up { to { transform: translateY(-50%) } }`), durations 34 s / 28 s / 38 s, the middle column `animation-direction: reverse`, hover on a column pauses it. Tile 78 × 78u, radius 20u, `rgba(4,24,38,.62)` + inset 1 px `rgba(255,255,255,.12)`, hover `rgba(30,70,95,.75)`; icon 30u, white, opacity .86. Container mask `linear-gradient(180deg, transparent 0, #000 16%, #000 84%, transparent 100%)`. Columns: (visa, stripe, slack, quickbooks, apple, figma, klarna, dropbox, hubspot), (mastercard, applepay, notion, xero, google, discord, revolut, trello, zapier), (paypal, googlepay, github, shopify, wise, gitlab, square, asana, airtable). DO NOT use `backdrop-filter` on the tiles (54 moving blurred layers stutter). DO NOT use coloured logos: monochrome white only.

## 7. Entrance timeline
- Nav items: opacity 0 → 1 and `translate: 0 −10u → 0` (0.7 s / 0.8 s `cubic-bezier(.16,1,.3,1)`), delays 0–300 ms in 60 ms steps, when fonts are ready (`html.loaded`).
- Title lines: inner span `translate: 0 108% → 0`, 1.1 s; hero delays 150 / 240 / 330 ms, section titles 80 / 170 ms.
- Blocks `.rv`: opacity 0 → 1 and `translate: 0 18u → 0` (0.9 s / 1 s), per-element `--d` delays in the markup.
- Wordmark: `translate: 0 30% → 0`, opacity 0 → .24, 1.4 s, delay 300 ms. Use the individual `translate` property, not `transform`.

## 8. Responsive (exact)
- **Desktop / landscape** (`min-width: 900px` and `min-aspect-ratio: 1/1`): canvas layout above.
- **Flow layout** (`max-width: 899px` or `max-aspect-ratio: 1/1`): `k = min(max(innerWidth / 430, .8), 1)`. Nav 72 px; logo 36 × 32 at (20, 20); links hidden; pill auto width, 40 tall, padding 0 18, 14 px, right 20 / top 16. `.pin` flex column, `space-between`, padding `84px 20px 22px`; `.top`/`.bot` flex columns gap 12; all children static. Hero title `clamp(34px, 10.6vw, 52px)` / .98; S2 and S3 titles `clamp(30px, 9vw, 46px)` / 1.02. Paragraphs 15 / 21 with `text-shadow: 0 1px 12px rgba(0,12,22,.45)`. Labels 11 / 14. CTA 46 tall. Social circles 40. Hidden: hero note, wordmark, S2 and S3 paragraphs, S3 note, card texts, footer. Cards: `.stats` flex row gap 8, each `flex:1`, padding 14 12, radius 14, number `clamp(20px, 6vw, 28px)`, title 12 / 15 with margin-top 10. Logos: two horizontal rows (third column hidden), tiles 56 (radius 15, icon 24), tracks `flex-direction:row; width:max-content`, `animation-name: side` (`translateX(−50%)`), horizontal mask `linear-gradient(90deg, transparent 0, #000 12%, #000 88%, transparent 100%)`.
- **Phone held sideways** (`max-width: 899px` and `min-aspect-ratio: 1/1`, on top of the flow rules): `.pin` flex row, `align-items:center`, padding `64px 20px 14px`; `.top`/`.bot` 31 % wide, gap 10; hero title `clamp(24px, 4.4vw, 36px)`, S2/S3 `clamp(22px, 3.8vw, 32px)`; paragraphs 13 / 18; CTA 40; social circles 34; cards stacked (column, gap 6), each a baseline row (number 20 px, title beside it); logo tiles 44 (radius 12, icon 20).

## 9. Accessibility
- `prefers-reduced-motion: reduce`: no entrance transitions, no frame easing, no count-up, logo tracks paused, no stage fade.
- Stage `aria-hidden`. Logo `aria-label="Clutch"`. Wordmark `aria-hidden`. Logo wall `role="img"` + descriptive `aria-label`; the duplicated tracks are `aria-hidden`.
- Frame errors are ignored (nearest loaded frame drawn); the stage is a fixed canvas, so there is no layout shift.

## 10. Assets
Two folders, 204 WebP files each, quality 80: `frames-hd/` 1920 × 1080 and `frames-sm/` 960 × 540.

| Frame | Desktop URL (1920 × 1080) | Phone URL (960 × 540) | Used at |
|---|---|---|---|
| 0 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-000.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-000.webp | scroll 0 |
| 16 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-016.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-016.webp | hero pin releases |
| 40 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-040.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-040.webp | s2 pins |
| 64 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-064.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-064.webp | s2 releases |
| 140 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-140.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-140.webp | s3 pins |
| 203 | https://assets.layerweb.com.br/clutch/frames-hd/clutch-203.webp | https://assets.layerweb.com.br/clutch/frames-sm/clutch-203.webp | end of page |

All files follow the pattern `https://assets.layerweb.com.br/clutch/frames-hd/clutch-NNN.webp` and `https://assets.layerweb.com.br/clutch/frames-sm/clutch-NNN.webp` with NNN = 000 … 203 (10.0 MB and 3.5 MB). Every URL was checked with `curl -I` and returns 200.

## 11. Do-not list (global)
- DO NOT put any content over the claw's column; titles may reach 565u at the top; everything below them on the left stays within 300u, and the right column starts at 1126u.
- DO NOT swap the frames for a `<video>`, a GIF, shapes or placeholders. DO NOT reuse frames from another clip.
- DO NOT add scroll libraries (no GSAP, no Lenis); native scroll only.
- DO NOT change the section heights (180vh / 220vh / 180vh) — the frame anchors depend on them.
- DO NOT add orange or red: the palette is deep blue `#000c16`, ice `#8fc6dd`, white.

## 12. Interaction model
- Native scroll drives the frame. Hero CTA jumps to `#results`, S3 CTA to `#top`. Hovering a logo column pauses it. No other interactive controls.

## 13. Final self-check (run before answering)
1. `document.documentElement.scrollHeight === 5.8 × innerHeight` (± 2 px).
2. Scrolling 3 px per animation frame changes the picture on at least 95 % of the animation frames (no 24-steps-per-second stair-stepping), and at the scroll position of frame 40.5 the canvas is the 50/50 blend of frames 40 and 41. Scroll 0 shows frame 0 (card hanging below the claw, no second card); the bottom of the page shows frame 203.
3. All frame requests return 200; console has no errors.
4. `document.fonts.check('400 87px Inter')` is true.
5. At 2560 × 1080 all text sits between x 560 and 2000; the stage shows no vertical line at the frame edges (the largest brightness step between two pixels 2 px apart, along rows 120 / 350 / 600 / 690 with the page text hidden, stays below 4) and no horizontal line near the bottom.
6. At 1440 × 900 no card or text overlaps the claw.
7. At 390 × 844 the claw sits between the title block and the bottom block; nav links and wordmark are hidden; two logo rows scroll sideways.
8. No orange colours, no source-brand names, no `{{` placeholders.

---

# Appendix A — `index.html` (verbatim, the whole file)

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Clutch — Smart Grip on Every Cent</title>
<meta name="description" content="Clutch keeps a steady hold on your spending with live alerts, automatic saving rules and integrations with the tools you already use.">
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 46 40'%3E%3Crect width='46' height='40' rx='9' fill='%23021a2b'/%3E%3Cg fill='none' stroke='%23fff' stroke-width='4.5' stroke-linecap='round'%3E%3Cpath d='M9 6C4 15 6 27 17 32'/%3E%3Cpath d='M37 6c5 9 3 21-8 26'/%3E%3C/g%3E%3Ccircle cx='23' cy='27' r='4' fill='%23fff'/%3E%3C/svg%3E">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap" rel="stylesheet">
<style>
/* ---------- reset + tokens ---------- */
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --k:1;                       /* design-canvas scale, set by JS: min(min(vw,1440)/1440, vh/1015) */
  --u:calc(var(--k) * 1px);    /* one canvas pixel */
  --ink:#ffffff;
  --ink-2:rgba(255,255,255,.78);
  --ice:#8fc6dd;               /* accent text (labels) */
  --line:rgba(255,255,255,.12);
  --ease-out:cubic-bezier(.16,1,.3,1);
}
html{background:#000c16;color:var(--ink);font-family:'Inter',system-ui,sans-serif;-webkit-font-smoothing:antialiased}
body{background:transparent;overflow-x:hidden}
a{color:inherit;text-decoration:none}
ul{list-style:none}

/* ---------- video stage (fixed, full-bleed, behind everything) ---------- */
.stage{position:fixed;inset:0;z-index:0;pointer-events:none}
.stage canvas{display:block;width:100%;height:100%;opacity:0;transition:opacity .9s ease}
.stage.ready canvas{opacity:1}
.stage .dim{position:absolute;left:0;right:0;bottom:0;height:0;background:linear-gradient(180deg,rgba(0,12,22,0) 0,rgba(0,12,22,.016) 15%,rgba(0,12,22,.06) 30%,rgba(0,12,22,.14) 45%,rgba(0,12,22,.25) 60%,rgba(0,12,22,.38) 75%,rgba(0,12,22,.5) 88%,rgba(0,12,22,.58) 100%)}

/* ---------- fixed navigation (content limited to 1440 px) ---------- */
.nav{position:fixed;top:0;left:0;right:0;margin:0 auto;max-width:1440px;height:calc(var(--u) * 128);z-index:20;pointer-events:none}
.nav > *{position:absolute;pointer-events:auto}
.logo{left:calc(var(--u) * 41);top:calc(var(--u) * 44);width:calc(var(--u) * 46);height:calc(var(--u) * 40);display:block}
.logo svg{display:block;width:100%;height:100%}
.nav .lk{top:calc(var(--u) * 55);font-size:calc(var(--u) * 16);line-height:calc(var(--u) * 20);font-weight:400;color:var(--ink-2);transition:color .25s ease}
.nav .lk:hover{color:#fff}
.lk-1{left:calc(var(--u) * 175)}
.lk-2{left:calc(var(--u) * 312)}
.lk-3{right:calc(var(--u) * 384)}
.lk-4{right:calc(var(--u) * 233)}
.join{right:calc(var(--u) * 40);top:calc(var(--u) * 40);width:calc(var(--u) * 122);height:calc(var(--u) * 48);border-radius:999px;
  display:flex;align-items:center;justify-content:center;font-size:calc(var(--u) * 16);font-weight:600;
  background:rgba(255,255,255,.14);-webkit-backdrop-filter:blur(10px);backdrop-filter:blur(10px);transition:background .3s ease}
.join:hover{background:rgba(255,255,255,.26)}

/* ---------- scroll sections ---------- */
main{position:relative;z-index:10}
.sec{position:relative}
.hero{height:180vh}
.s2{height:220vh}
.s3{height:180vh}
.pin{position:sticky;top:0;height:100vh;height:100svh;max-width:1440px;margin:0 auto;overflow:hidden}
.pin > *,.top > *,.bot > *,.stats > *{position:absolute}

/* shared type */
.label{font-size:calc(var(--u) * 13);line-height:calc(var(--u) * 16);font-weight:500;letter-spacing:.14em;text-transform:uppercase;color:var(--ice)}
.title{font-weight:400;letter-spacing:-.035em;white-space:nowrap}
.title .ln{display:block;overflow:hidden;padding-bottom:.06em;margin-bottom:-.06em}
.title .ln > span{display:block;background:linear-gradient(180deg,#ffffff 0%,#ffffff 30%,rgba(196,229,243,.66) 100%);
  background-size:100% 300%;-webkit-background-clip:text;background-clip:text;color:transparent}
.title .ln:nth-child(1) > span{background-position:0 0}
.title .ln:nth-child(2) > span{background-position:0 50%}
.title .ln:nth-child(3) > span{background-position:0 100%}
.title.two .ln > span{background-size:100% 200%}
.title.two .ln:nth-child(2) > span{background-position:0 100%}
.copy{font-size:calc(var(--u) * 20);line-height:calc(var(--u) * 28.5);color:rgba(255,255,255,.9);letter-spacing:-.01em}
.small{font-size:calc(var(--u) * 15);line-height:calc(var(--u) * 19);color:rgba(255,255,255,.86)}

/* pill + arrow button */
.cta{display:flex;gap:calc(var(--u) * 4);height:calc(var(--u) * 54)}
.cta .pill{width:calc(var(--u) * 215);height:100%;border-radius:999px;background:#fff;color:#04121d;display:flex;align-items:center;justify-content:center;
  font-size:calc(var(--u) * 16);font-weight:600;transition:background .3s ease}
.cta .arw{width:calc(var(--u) * 54);height:100%;border-radius:50%;background:#fff;color:#04121d;display:flex;align-items:center;justify-content:center;transition:transform .45s var(--ease-out)}
.cta .arw svg{width:calc(var(--u) * 20);height:calc(var(--u) * 20)}
.cta:hover .pill{background:#dff1fa}
.cta:hover .arw{transform:rotate(-45deg)}

/* ---------- hero (positions from the reference canvas 1440 x 1015) ---------- */
.hero .title{left:calc(var(--u) * 43);top:calc(var(--u) * 202);font-size:calc(var(--u) * 87);line-height:calc(var(--u) * 85)}
.hero .copy{right:calc(var(--u) * 34);top:calc(var(--u) * 210);width:calc(var(--u) * 280)}
.hero .cta{right:calc(var(--u) * 41);top:calc(var(--u) * 393)}
.hero .note{left:calc(var(--u) * 41);bottom:calc(var(--u) * 294)}
.social{right:calc(var(--u) * 41);bottom:calc(var(--u) * 295);display:flex;align-items:center;gap:calc(var(--u) * 4)}
.social p{margin-right:calc(var(--u) * 41)}
.social a{width:calc(var(--u) * 54);height:calc(var(--u) * 54);border-radius:50%;background:rgba(255,255,255,.16);display:flex;align-items:center;justify-content:center;transition:background .3s ease}
.social a:hover{background:rgba(255,255,255,.3)}
.social svg{width:calc(var(--u) * 17);height:calc(var(--u) * 17)}
.mark{left:0;right:0;bottom:calc(var(--u) * 48);text-align:center;font-weight:500;font-size:calc(var(--u) * 226);line-height:.8;letter-spacing:-.02em;
  color:#fff;opacity:.24;mix-blend-mode:overlay;white-space:nowrap;pointer-events:none}

/* ---------- section 2 : results (title top-left, three glass cards split by the claw) ---------- */
.s2 .label{left:calc(var(--u) * 43);top:calc(var(--u) * 168)}
.s2 .title{left:calc(var(--u) * 43);top:calc(var(--u) * 200);font-size:calc(var(--u) * 66);line-height:calc(var(--u) * 68)}
.s2 .copy{right:calc(var(--u) * 34);top:calc(var(--u) * 210);width:calc(var(--u) * 280);font-size:calc(var(--u) * 18);line-height:calc(var(--u) * 27)}
.card{bottom:calc(var(--u) * 64);width:calc(var(--u) * 252);height:calc(var(--u) * 222);padding:calc(var(--u) * 26) calc(var(--u) * 24) calc(var(--u) * 24);border-radius:calc(var(--u) * 20);
  background:rgba(1,14,24,.62);box-shadow:inset 0 0 0 1px var(--line);-webkit-backdrop-filter:blur(16px);backdrop-filter:blur(16px)}
.card.c1{left:calc(var(--u) * 43)}
.card.c2{right:calc(var(--u) * 40);bottom:calc(var(--u) * 298);width:calc(var(--u) * 274)}
.card.c3{right:calc(var(--u) * 40);width:calc(var(--u) * 274)}
.card b{display:block;font-weight:400;font-size:calc(var(--u) * 56);line-height:1;letter-spacing:-.04em;font-variant-numeric:tabular-nums}
.card h3{margin-top:calc(var(--u) * 30);font-weight:500;font-size:calc(var(--u) * 16);line-height:calc(var(--u) * 22)}
.card p{margin-top:calc(var(--u) * 8);font-size:calc(var(--u) * 13);line-height:calc(var(--u) * 19);color:rgba(255,255,255,.62)}

/* ---------- section 3 : integrations (copy left, looping logo columns right) ---------- */
.s3 .label{left:calc(var(--u) * 43);top:calc(var(--u) * 168)}
.s3 .title{left:calc(var(--u) * 43);top:calc(var(--u) * 200);font-size:calc(var(--u) * 66);line-height:calc(var(--u) * 68)}
.s3 .copy{left:calc(var(--u) * 43);top:calc(var(--u) * 362);width:calc(var(--u) * 300);font-size:calc(var(--u) * 18);line-height:calc(var(--u) * 27)}
.s3 .cta{left:calc(var(--u) * 43);top:calc(var(--u) * 540)}
.s3 .note{left:calc(var(--u) * 43);top:calc(var(--u) * 616)}
.logos{right:calc(var(--u) * 48);top:calc(var(--u) * 150);bottom:calc(var(--u) * 96);width:calc(var(--u) * 258);display:flex;gap:calc(var(--u) * 12);overflow:hidden;
  -webkit-mask-image:linear-gradient(180deg,transparent 0,#000 16%,#000 84%,transparent 100%);mask-image:linear-gradient(180deg,transparent 0,#000 16%,#000 84%,transparent 100%)}
.logos .col{flex:1;min-width:0}
.logos .track{display:flex;flex-direction:column;gap:calc(var(--u) * 12);padding-bottom:calc(var(--u) * 12);animation:up var(--dur,30s) linear infinite}
.logos .col:nth-child(2) .track{animation-direction:reverse}
.logos .col:hover .track{animation-play-state:paused}
.logos li{flex:none;width:calc(var(--u) * 78);height:calc(var(--u) * 78);border-radius:calc(var(--u) * 20);display:flex;align-items:center;justify-content:center;
  background:rgba(4,24,38,.62);box-shadow:inset 0 0 0 1px var(--line);transition:background .3s ease}
.logos li:hover{background:rgba(30,70,95,.75)}
.logos svg{width:calc(var(--u) * 30);height:calc(var(--u) * 30);fill:#fff;opacity:.86}
@keyframes up{to{transform:translateY(-50%)}}
@keyframes side{to{transform:translateX(-50%)}}
.foot{left:calc(var(--u) * 43);right:calc(var(--u) * 40);bottom:calc(var(--u) * 40);display:flex;justify-content:space-between;font-size:calc(var(--u) * 13);color:rgba(255,255,255,.6)}
.foot nav{display:flex;gap:calc(var(--u) * 24)}
.foot a:hover{color:#fff}

/* ---------- entrances ---------- */
.js .rv{opacity:0;translate:0 calc(var(--u) * 18);transition:opacity .9s ease,translate 1s var(--ease-out);transition-delay:var(--d,0ms)}
.js .title .ln > span{translate:0 108%;transition:translate 1.1s var(--ease-out);transition-delay:var(--d,0ms)}
.js .mark{translate:0 30%;opacity:0;transition:translate 1.4s var(--ease-out),opacity 1.4s ease;transition-delay:300ms}
.js .nav > *{opacity:0;translate:0 calc(var(--u) * -10);transition:opacity .7s ease,translate .8s var(--ease-out);transition-delay:var(--d,0ms)}
.js .on .rv,.js.loaded .nav > *{opacity:1;translate:0 0}
.js .on .title .ln > span{translate:0 0}
.js .on .mark{translate:0 0;opacity:.24}

/* ---------- flow layout: phones, portrait tablets ---------- */
@media (max-width:899px),(max-aspect-ratio:1/1){
  .nav{height:72px}
  .logo{left:20px;top:20px;width:36px;height:32px}
  .nav .lk{display:none}
  .join{right:20px;top:16px;width:auto;height:40px;padding:0 18px;font-size:14px}
  .pin{display:flex;flex-direction:column;justify-content:space-between;padding:84px 20px 22px}
  .pin > *,.top > *,.bot > *,.stats > *{position:static}
  .top,.bot{display:flex;flex-direction:column;gap:12px}
  .hero .title{font-size:clamp(34px,10.6vw,52px);line-height:.98}
  .s2 .title,.s3 .title{font-size:clamp(30px,9vw,46px);line-height:1.02}
  .copy{width:auto;font-size:15px;line-height:21px;text-shadow:0 1px 12px rgba(0,12,22,.45)}
  .small{font-size:13px;line-height:17px}
  .label{font-size:11px;line-height:14px}
  .cta{height:46px;gap:4px}
  .cta .pill{width:auto;flex:1;font-size:15px}
  .cta .arw{width:46px}
  .cta .arw svg{width:18px;height:18px}
  .social{gap:4px}
  .social p{margin-right:auto}
  .social a{width:40px;height:40px}
  .social svg{width:15px;height:15px}
  .hero .note,.mark,.s2 .copy,.s3 .copy,.s3 .note,.card p,.foot{display:none}
  .stats{display:flex;gap:8px}
  .card,.card.c3{flex:1;min-width:0;width:auto;height:auto;padding:14px 12px;border-radius:14px}
  .card b{font-size:clamp(20px,6vw,28px)}
  .card h3{margin-top:10px;font-size:12px;line-height:15px}
  .logos{display:flex;flex-direction:column;width:auto;gap:8px;-webkit-mask-image:linear-gradient(90deg,transparent 0,#000 12%,#000 88%,transparent 100%);mask-image:linear-gradient(90deg,transparent 0,#000 12%,#000 88%,transparent 100%)}
  .logos .col:nth-child(3){display:none}
  .logos .track{flex-direction:row;width:max-content;gap:8px;padding:0 8px 0 0;animation-name:side}
  .logos li{width:56px;height:56px;border-radius:15px}
  .logos svg{width:24px;height:24px}
  .js .rv{translate:0 14px}
}
/* phone held sideways: texts in two side columns, claw in the middle */
@media (max-width:899px) and (min-aspect-ratio:1/1){
  .pin{flex-direction:row;align-items:center;padding:64px 20px 14px}
  .top,.bot{width:31%;gap:10px}
  .hero .title{font-size:clamp(24px,4.4vw,36px)}
  .s2 .title,.s3 .title{font-size:clamp(22px,3.8vw,32px)}
  .copy{font-size:13px;line-height:18px}
  .cta{height:40px}
  .cta .arw{width:40px}
  .social a{width:34px;height:34px}
  .stats{flex-direction:column;gap:6px}
  .card{display:flex;align-items:baseline;gap:10px;padding:10px 12px}
  .card b{font-size:20px}
  .card h3{margin-top:0}
  .logos li{width:44px;height:44px;border-radius:12px}
  .logos svg{width:20px;height:20px}
}
@media (min-width:900px) and (min-aspect-ratio:1/1){
  .top,.bot,.stats{display:contents}
}

@media (prefers-reduced-motion:reduce){
  .js .rv,.js .title .ln > span,.js .mark,.js .nav > *{transition:none!important;translate:0 0!important;opacity:1!important}
  .js .mark{opacity:.24!important}
  .logos .track{animation-play-state:paused}
  .stage canvas{transition:none}
}
</style>
</head>
<body>
<script>document.documentElement.classList.add('js')</script>
<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <!-- integration logos: Simple Icons (CC0), monochrome -->
  <symbol id="i-visa" viewBox="0 0 24 24"><path d="M9.112 8.262L5.97 15.758H3.92L2.374 9.775c-.094-.368-.175-.503-.461-.658C1.447 8.864.677 8.627 0 8.479l.046-.217h3.3a.904.904 0 01.894.764l.817 4.338 2.018-5.102zm8.033 5.049c.008-1.979-2.736-2.088-2.717-2.972.006-.269.262-.555.822-.628a3.66 3.66 0 011.913.336l.34-1.59a5.207 5.207 0 00-1.814-.333c-1.917 0-3.266 1.02-3.278 2.479-.012 1.079.963 1.68 1.698 2.04.756.367 1.01.603 1.006.931-.005.504-.602.725-1.16.734-.975.015-1.54-.263-1.992-.473l-.351 1.642c.453.208 1.289.39 2.156.398 2.037 0 3.37-1.006 3.377-2.564m5.061 2.447H24l-1.565-7.496h-1.656a.883.883 0 00-.826.55l-2.909 6.946h2.036l.405-1.12h2.488zm-2.163-2.656l1.02-2.815.588 2.815zm-8.16-4.84l-1.603 7.496H8.34l1.605-7.496z"/></symbol>
  <symbol id="i-stripe" viewBox="0 0 24 24"><path d="M13.976 9.15c-2.172-.806-3.356-1.426-3.356-2.409 0-.831.683-1.305 1.901-1.305 2.227 0 4.515.858 6.09 1.631l.89-5.494C18.252.975 15.697 0 12.165 0 9.667 0 7.589.654 6.104 1.872 4.56 3.147 3.757 4.992 3.757 7.218c0 4.039 2.467 5.76 6.476 7.219 2.585.92 3.445 1.574 3.445 2.583 0 .98-.84 1.545-2.354 1.545-1.875 0-4.965-.921-6.99-2.109l-.9 5.555C5.175 22.99 8.385 24 11.714 24c2.641 0 4.843-.624 6.328-1.813 1.664-1.305 2.525-3.236 2.525-5.732 0-4.128-2.524-5.851-6.594-7.305h.003z"/></symbol>
  <symbol id="i-slack" viewBox="0 0 24 24"><path d="M5.042 15.165a2.528 2.528 0 0 1-2.52 2.523A2.528 2.528 0 0 1 0 15.165a2.527 2.527 0 0 1 2.522-2.52h2.52v2.52zM6.313 15.165a2.527 2.527 0 0 1 2.521-2.52 2.527 2.527 0 0 1 2.521 2.52v6.313A2.528 2.528 0 0 1 8.834 24a2.528 2.528 0 0 1-2.521-2.522v-6.313zM8.834 5.042a2.528 2.528 0 0 1-2.521-2.52A2.528 2.528 0 0 1 8.834 0a2.528 2.528 0 0 1 2.521 2.522v2.52H8.834zM8.834 6.313a2.528 2.528 0 0 1 2.521 2.521 2.528 2.528 0 0 1-2.521 2.521H2.522A2.528 2.528 0 0 1 0 8.834a2.528 2.528 0 0 1 2.522-2.521h6.312zM18.956 8.834a2.528 2.528 0 0 1 2.522-2.521A2.528 2.528 0 0 1 24 8.834a2.528 2.528 0 0 1-2.522 2.521h-2.522V8.834zM17.688 8.834a2.528 2.528 0 0 1-2.523 2.521 2.527 2.527 0 0 1-2.52-2.521V2.522A2.527 2.527 0 0 1 15.165 0a2.528 2.528 0 0 1 2.523 2.522v6.312zM15.165 18.956a2.528 2.528 0 0 1 2.523 2.522A2.528 2.528 0 0 1 15.165 24a2.527 2.527 0 0 1-2.52-2.522v-2.522h2.52zM15.165 17.688a2.527 2.527 0 0 1-2.52-2.523 2.526 2.526 0 0 1 2.52-2.52h6.313A2.527 2.527 0 0 1 24 15.165a2.528 2.528 0 0 1-2.522 2.523h-6.313z"/></symbol>
  <symbol id="i-quickbooks" viewBox="0 0 24 24"><path d="M12 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0zm.642 4.1335c.9554 0 1.7296.776 1.7296 1.7332v9.0667h1.6c1.614 0 2.9275-1.3156 2.9275-2.933 0-1.6173-1.3136-2.9333-2.9276-2.9333h-.6654V7.3334h.6654c2.5722 0 4.6577 2.0897 4.6577 4.667 0 2.5774-2.0855 4.6666-4.6577 4.6666H12.642zM7.9837 7.333h3.3291v12.533c-.9555 0-1.73-.7759-1.73-1.7332V9.0662H7.9837c-1.6146 0-2.9277 1.316-2.9277 2.9334 0 1.6175 1.3131 2.9333 2.9277 2.9333h.6654v1.7332h-.6654c-2.5725 0-4.6577-2.0892-4.6577-4.6665 0-2.5771 2.0852-4.6666 4.6577-4.6666Z"/></symbol>
  <symbol id="i-apple" viewBox="0 0 24 24"><path d="M12.152 6.896c-.948 0-2.415-1.078-3.96-1.04-2.04.027-3.91 1.183-4.961 3.014-2.117 3.675-.546 9.103 1.519 12.09 1.013 1.454 2.208 3.09 3.792 3.039 1.52-.065 2.09-.987 3.935-.987 1.831 0 2.35.987 3.96.948 1.637-.026 2.676-1.48 3.676-2.948 1.156-1.688 1.636-3.325 1.662-3.415-.039-.013-3.182-1.221-3.22-4.857-.026-3.04 2.48-4.494 2.597-4.559-1.429-2.09-3.623-2.324-4.39-2.376-2-.156-3.675 1.09-4.61 1.09zM15.53 3.83c.843-1.012 1.4-2.427 1.245-3.83-1.207.052-2.662.805-3.532 1.818-.78.896-1.454 2.338-1.273 3.714 1.338.104 2.715-.688 3.559-1.701"/></symbol>
  <symbol id="i-figma" viewBox="0 0 24 24"><path d="M15.852 8.981h-4.588V0h4.588c2.476 0 4.49 2.014 4.49 4.49s-2.014 4.491-4.49 4.491zM12.735 7.51h3.117c1.665 0 3.019-1.355 3.019-3.019s-1.355-3.019-3.019-3.019h-3.117V7.51zm0 1.471H8.148c-2.476 0-4.49-2.014-4.49-4.49S5.672 0 8.148 0h4.588v8.981zm-4.587-7.51c-1.665 0-3.019 1.355-3.019 3.019s1.354 3.02 3.019 3.02h3.117V1.471H8.148zm4.587 15.019H8.148c-2.476 0-4.49-2.014-4.49-4.49s2.014-4.49 4.49-4.49h4.588v8.98zM8.148 8.981c-1.665 0-3.019 1.355-3.019 3.019s1.355 3.019 3.019 3.019h3.117V8.981H8.148zM8.172 24c-2.489 0-4.515-2.014-4.515-4.49s2.014-4.49 4.49-4.49h4.588v4.441c0 2.503-2.047 4.539-4.563 4.539zm-.024-7.51a3.023 3.023 0 0 0-3.019 3.019c0 1.665 1.365 3.019 3.044 3.019 1.705 0 3.093-1.376 3.093-3.068v-2.97H8.148zm7.704 0h-.098c-2.476 0-4.49-2.014-4.49-4.49s2.014-4.49 4.49-4.49h.098c2.476 0 4.49 2.014 4.49 4.49s-2.014 4.49-4.49 4.49zm-.097-7.509c-1.665 0-3.019 1.355-3.019 3.019s1.355 3.019 3.019 3.019h.098c1.665 0 3.019-1.355 3.019-3.019s-1.355-3.019-3.019-3.019h-.098z"/></symbol>
  <symbol id="i-klarna" viewBox="0 0 24 24"><path d="M4.592 2v20H0V2h4.592zm11.46 0c0 4.194-1.583 8.105-4.415 11.068l-.278.283L17.702 22h-5.668l-6.893-9.4 1.779-1.332c2.858-2.14 4.535-5.378 4.637-8.924L11.562 2h4.49zM21.5 17a2.5 2.5 0 110 5 2.5 2.5 0 010-5z"/></symbol>
  <symbol id="i-dropbox" viewBox="0 0 24 24"><path d="M6 1.807L0 5.629l6 3.822 6.001-3.822L6 1.807zM18 1.807l-6 3.822 6 3.822 6-3.822-6-3.822zM0 13.274l6 3.822 6.001-3.822L6 9.452l-6 3.822zM18 9.452l-6 3.822 6 3.822 6-3.822-6-3.822zM6 18.371l6.001 3.822 6-3.822-6-3.822L6 18.371z"/></symbol>
  <symbol id="i-hubspot" viewBox="0 0 24 24"><path d="M18.164 7.93V5.084a2.198 2.198 0 001.267-1.978v-.067A2.2 2.2 0 0017.238.845h-.067a2.2 2.2 0 00-2.193 2.193v.067a2.196 2.196 0 001.252 1.973l.013.006v2.852a6.22 6.22 0 00-2.969 1.31l.012-.01-7.828-6.095A2.497 2.497 0 104.3 4.656l-.012.006 7.697 5.991a6.176 6.176 0 00-1.038 3.446c0 1.343.425 2.588 1.147 3.607l-.013-.02-2.342 2.343a1.968 1.968 0 00-.58-.095h-.002a2.033 2.033 0 102.033 2.033 1.978 1.978 0 00-.1-.595l.005.014 2.317-2.317a6.247 6.247 0 104.782-11.134l-.036-.005zm-.964 9.378a3.206 3.206 0 113.215-3.207v.002a3.206 3.206 0 01-3.207 3.207z"/></symbol>
  <symbol id="i-mastercard" viewBox="0 0 24 24"><path d="M11.343 18.031c.058.049.12.098.181.146-1.177.783-2.59 1.238-4.107 1.238C3.32 19.416 0 16.096 0 12c0-4.095 3.32-7.416 7.416-7.416 1.518 0 2.931.456 4.105 1.238-.06.051-.12.098-.165.15C9.6 7.489 8.595 9.688 8.595 12c0 2.311 1.001 4.51 2.748 6.031zm5.241-13.447c-1.52 0-2.931.456-4.105 1.238.06.051.12.098.165.15C14.4 7.489 15.405 9.688 15.405 12c0 2.31-1.001 4.507-2.748 6.031-.058.049-.12.098-.181.146 1.177.783 2.588 1.238 4.107 1.238C20.68 19.416 24 16.096 24 12c0-4.094-3.32-7.416-7.416-7.416zM12 6.174c-.096.075-.189.15-.28.231C10.156 7.764 9.169 9.765 9.169 12c0 2.236.987 4.236 2.551 5.595.09.08.185.158.28.232.096-.074.189-.152.28-.232 1.563-1.359 2.551-3.359 2.551-5.595 0-2.235-.987-4.236-2.551-5.595-.09-.08-.184-.156-.28-.231z"/></symbol>
  <symbol id="i-applepay" viewBox="0 0 24 24"><path d="M2.15 4.318a42.16 42.16 0 0 0-.454.003c-.15.005-.303.013-.452.04a1.44 1.44 0 0 0-1.06.772c-.07.138-.114.278-.14.43-.028.148-.037.3-.04.45A10.2 10.2 0 0 0 0 6.222v11.557c0 .07.002.138.003.207.004.15.013.303.04.452.027.15.072.291.142.429a1.436 1.436 0 0 0 .63.63c.138.07.278.115.43.142.148.027.3.036.45.04l.208.003h20.194l.207-.003c.15-.004.303-.013.452-.04.15-.027.291-.071.428-.141a1.432 1.432 0 0 0 .631-.631c.07-.138.115-.278.141-.43.027-.148.036-.3.04-.45.002-.07.003-.138.003-.208l.001-.246V6.221c0-.07-.002-.138-.004-.207a2.995 2.995 0 0 0-.04-.452 1.446 1.446 0 0 0-1.2-1.201 3.022 3.022 0 0 0-.452-.04 10.448 10.448 0 0 0-.453-.003zm0 .512h19.942c.066 0 .131.002.197.003.115.004.25.01.375.032.109.02.2.05.287.094a.927.927 0 0 1 .407.407.997.997 0 0 1 .094.288c.022.123.028.258.031.374.002.065.003.13.003.197v11.552c0 .065 0 .13-.003.196-.003.115-.009.25-.032.375a.927.927 0 0 1-.5.693 1.002 1.002 0 0 1-.286.094 2.598 2.598 0 0 1-.373.032l-.2.003H1.906c-.066 0-.133-.002-.196-.003a2.61 2.61 0 0 1-.375-.032c-.109-.02-.2-.05-.288-.094a.918.918 0 0 1-.406-.407 1.006 1.006 0 0 1-.094-.288 2.531 2.531 0 0 1-.032-.373 9.588 9.588 0 0 1-.002-.197V6.224c0-.065 0-.131.002-.197.004-.114.01-.248.032-.375.02-.108.05-.199.094-.287a.925.925 0 0 1 .407-.406 1.03 1.03 0 0 1 .287-.094c.125-.022.26-.029.375-.032.065-.002.131-.002.196-.003zm4.71 3.7c-.3.016-.668.199-.88.456-.191.22-.36.58-.316.918.338.03.675-.169.888-.418.205-.258.345-.603.308-.955zm2.207.42v5.493h.852v-1.877h1.18c1.078 0 1.835-.739 1.835-1.812 0-1.07-.742-1.805-1.808-1.805zm.852.719h.982c.739 0 1.161.396 1.161 1.089 0 .692-.422 1.092-1.164 1.092h-.979zm-3.154.3c-.45.01-.83.28-1.05.28-.235 0-.593-.264-.981-.257a1.446 1.446 0 0 0-1.23.747c-.527.908-.139 2.255.374 2.995.249.366.549.769.944.754.373-.014.52-.242.973-.242.454 0 .586.242.98.235.41-.007.667-.366.915-.733.286-.417.403-.82.41-.841-.007-.008-.79-.308-.797-1.209-.008-.754.615-1.113.644-1.135-.352-.52-.9-.578-1.09-.593a1.123 1.123 0 0 0-.092-.002zm8.204.397c-.99 0-1.606.533-1.652 1.256h.777c.072-.358.369-.586.845-.586.502 0 .803.266.803.711v.309l-1.097.064c-.951.054-1.488.484-1.488 1.184 0 .72.548 1.207 1.332 1.207.526 0 1.032-.281 1.264-.727h.019v.659h.788v-2.76c0-.803-.62-1.317-1.591-1.317zm1.94.072l1.446 4.009c0 .003-.073.24-.073.247-.125.41-.33.571-.711.571-.069 0-.206 0-.267-.015v.666c.06.011.267.019.335.019.83 0 1.226-.312 1.568-1.283l1.5-4.214h-.868l-1.012 3.259h-.015l-1.013-3.26zm-1.167 2.189v.316c0 .521-.45.917-1.024.917-.442 0-.731-.228-.731-.579 0-.342.278-.56.769-.593z"/></symbol>
  <symbol id="i-notion" viewBox="0 0 24 24"><path d="M4.459 4.208c.746.606 1.026.56 2.428.466l13.215-.793c.28 0 .047-.28-.046-.326L17.86 1.968c-.42-.326-.981-.7-2.055-.607L3.01 2.295c-.466.046-.56.28-.374.466zm.793 3.08v13.904c0 .747.373 1.027 1.214.98l14.523-.84c.841-.046.935-.56.935-1.167V6.354c0-.606-.233-.933-.748-.887l-15.177.887c-.56.047-.747.327-.747.933zm14.337.745c.093.42 0 .84-.42.888l-.7.14v10.264c-.608.327-1.168.514-1.635.514-.748 0-.935-.234-1.495-.933l-4.577-7.186v6.952L12.21 19s0 .84-1.168.84l-3.222.186c-.093-.186 0-.653.327-.746l.84-.233V9.854L7.822 9.76c-.094-.42.14-1.026.793-1.073l3.456-.233 4.764 7.279v-6.44l-1.215-.139c-.093-.514.28-.887.747-.933zM1.936 1.035l13.31-.98c1.634-.14 2.055-.047 3.082.7l4.249 2.986c.7.513.934.653.934 1.213v16.378c0 1.026-.373 1.634-1.68 1.726l-15.458.934c-.98.047-1.448-.093-1.962-.747l-3.129-4.06c-.56-.747-.793-1.306-.793-1.96V2.667c0-.839.374-1.54 1.447-1.632z"/></symbol>
  <symbol id="i-xero" viewBox="0 0 24 24"><path d="M12 0C5.373 0 0 5.373 0 12s5.373 12 12 12 12-5.373 12-12S18.627 0 12 0zm6.585 14.655c-1.485 0-2.69-1.206-2.69-2.689 0-1.485 1.207-2.691 2.69-2.691 1.485 0 2.69 1.207 2.69 2.691s-1.207 2.689-2.69 2.689zM7.53 14.644c-.099 0-.192-.041-.267-.116l-2.043-2.04-2.052 2.047c-.069.068-.16.108-.258.108-.202 0-.368-.166-.368-.368 0-.099.04-.191.111-.263l2.04-2.05-2.038-2.047c-.075-.069-.113-.162-.113-.261 0-.203.166-.366.368-.366.098 0 .188.037.258.105l2.055 2.048 2.048-2.045c.069-.071.162-.108.26-.108.211 0 .375.165.375.366 0 .098-.029.188-.104.258l-2.056 2.055 2.055 2.051c.068.069.104.16.104.258 0 .202-.165.368-.365.368h-.01zm8.017-4.591c-.796.101-.882.476-.882 1.404v2.787c0 .202-.165.366-.366.366-.203 0-.367-.165-.368-.366v-4.53c0-.204.16-.366.362-.366.166 0 .316.125.346.289.27-.209.6-.317.93-.317h.105c.195 0 .359.165.359.368 0 .201-.164.352-.375.359 0 0-.09 0-.164.008l.053-.002zm-3.091 2.205H8.625c0 .019.003.037.006.057.02.105.045.211.083.31.194.531.765 1.275 1.829 1.29.33-.003.631-.086.9-.229.21-.12.391-.271.525-.428.045-.058.09-.112.12-.168.18-.229.405-.186.54-.083.164.135.18.391.045.57l-.016.016c-.21.27-.435.495-.689.66-.255.164-.525.284-.811.345-.33.09-.645.104-.975.06-1.095-.135-2.01-.93-2.28-2.01-.06-.21-.09-.42-.09-.645 0-.855.421-1.695 1.125-2.205.885-.615 2.085-.66 3-.075.63.405 1.035 1.021 1.185 1.771.075.419-.21.794-.734.81l.068-.046zm6.129-2.223c-1.064 0-1.931.865-1.931 1.931 0 1.064.866 1.931 1.931 1.931s1.931-.867 1.931-1.931c0-1.065-.866-1.933-1.931-1.933v.002zm0 2.595c-.367 0-.666-.297-.666-.666 0-.367.3-.665.666-.665.367 0 .667.299.667.665 0 .369-.3.667-.667.666zm-8.04-2.603c-.91 0-1.672.623-1.886 1.466v.03h3.776c-.203-.855-.973-1.494-1.891-1.494v-.002z"/></symbol>
  <symbol id="i-google" viewBox="0 0 24 24"><path d="M12.48 10.92v3.28h7.84c-.24 1.84-.853 3.187-1.787 4.133-1.147 1.147-2.933 2.4-6.053 2.4-4.827 0-8.6-3.893-8.6-8.72s3.773-8.72 8.6-8.72c2.6 0 4.507 1.027 5.907 2.347l2.307-2.307C18.747 1.44 16.133 0 12.48 0 5.867 0 .307 5.387.307 12s5.56 12 12.173 12c3.573 0 6.267-1.173 8.373-3.36 2.16-2.16 2.84-5.213 2.84-7.667 0-.76-.053-1.467-.173-2.053H12.48z"/></symbol>
  <symbol id="i-discord" viewBox="0 0 24 24"><path d="M20.317 4.3698a19.7913 19.7913 0 00-4.8851-1.5152.0741.0741 0 00-.0785.0371c-.211.3753-.4447.8648-.6083 1.2495-1.8447-.2762-3.68-.2762-5.4868 0-.1636-.3933-.4058-.8742-.6177-1.2495a.077.077 0 00-.0785-.037 19.7363 19.7363 0 00-4.8852 1.515.0699.0699 0 00-.0321.0277C.5334 9.0458-.319 13.5799.0992 18.0578a.0824.0824 0 00.0312.0561c2.0528 1.5076 4.0413 2.4228 5.9929 3.0294a.0777.0777 0 00.0842-.0276c.4616-.6304.8731-1.2952 1.226-1.9942a.076.076 0 00-.0416-.1057c-.6528-.2476-1.2743-.5495-1.8722-.8923a.077.077 0 01-.0076-.1277c.1258-.0943.2517-.1923.3718-.2914a.0743.0743 0 01.0776-.0105c3.9278 1.7933 8.18 1.7933 12.0614 0a.0739.0739 0 01.0785.0095c.1202.099.246.1981.3728.2924a.077.077 0 01-.0066.1276 12.2986 12.2986 0 01-1.873.8914.0766.0766 0 00-.0407.1067c.3604.698.7719 1.3628 1.225 1.9932a.076.076 0 00.0842.0286c1.961-.6067 3.9495-1.5219 6.0023-3.0294a.077.077 0 00.0313-.0552c.5004-5.177-.8382-9.6739-3.5485-13.6604a.061.061 0 00-.0312-.0286zM8.02 15.3312c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9555-2.4189 2.157-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.9555 2.4189-2.1569 2.4189zm7.9748 0c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9554-2.4189 2.1569-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.946 2.4189-2.1568 2.4189Z"/></symbol>
  <symbol id="i-revolut" viewBox="0 0 24 24"><path d="M20.9133 6.9566C20.9133 3.1208 17.7898 0 13.9503 0H2.424v3.8605h10.9782c1.7376 0 3.177 1.3651 3.2087 3.043.016.84-.2994 1.633-.8878 2.2324-.5886.5998-1.375.9303-2.2144.9303H9.2322a.2756.2756 0 0 0-.2755.2752v3.431c0 .0585.018.1142.052.1612L16.2646 24h5.3114l-7.2727-10.094c3.6625-.1838 6.61-3.2612 6.61-6.9494zM6.8943 5.9229H2.424V24h4.4704z"/></symbol>
  <symbol id="i-trello" viewBox="0 0 24 24"><path d="M21.147 0H2.853A2.86 2.86 0 000 2.853v18.294A2.86 2.86 0 002.853 24h18.294A2.86 2.86 0 0024 21.147V2.853A2.86 2.86 0 0021.147 0zM10.34 17.287a.953.953 0 01-.953.953h-4a.954.954 0 01-.954-.953V5.38a.953.953 0 01.954-.953h4a.954.954 0 01.953.953zm9.233-5.467a.944.944 0 01-.953.947h-4a.947.947 0 01-.953-.947V5.38a.953.953 0 01.953-.953h4a.954.954 0 01.953.953z"/></symbol>
  <symbol id="i-zapier" viewBox="0 0 24 24"><path d="M4.157 0A4.151 4.151 0 0 0 0 4.161v15.678A4.151 4.151 0 0 0 4.157 24h15.682A4.152 4.152 0 0 0 24 19.839V4.161A4.152 4.152 0 0 0 19.839 0H4.157Zm10.61 8.761h.03a.577.577 0 0 1 .23.038.585.585 0 0 1 .201.124.63.63 0 0 1 .162.431.612.612 0 0 1-.162.435.58.58 0 0 1-.201.128.58.58 0 0 1-.23.042.529.529 0 0 1-.235-.042.585.585 0 0 1-.332-.328.559.559 0 0 1-.038-.235.613.613 0 0 1 .17-.431.59.59 0 0 1 .405-.162Zm2.853 1.572c.03.004.061.004.095.004.325-.011.646.064.937.219.238.144.431.355.552.609.128.279.189.582.185.888v.193a2 2 0 0 1 0 .219h-2.498c.003.227.075.45.204.642a.78.78 0 0 0 .646.265.714.714 0 0 0 .484-.136.642.642 0 0 0 .23-.318l.915.257a1.398 1.398 0 0 1-.28.537c-.14.159-.321.284-.521.355a2.234 2.234 0 0 1-.836.136 1.923 1.923 0 0 1-1.001-.245 1.618 1.618 0 0 1-.665-.703 2.221 2.221 0 0 1-.227-1.036 1.95 1.95 0 0 1 .48-1.398 1.9 1.9 0 0 1 1.3-.488Zm-9.607.023c.162.004.325.026.48.079.207.065.4.174.563.314.26.302.393.692.366 1.088v2.276H8.53l-.109-.711h-.065c-.064.163-.155.31-.272.439a1.122 1.122 0 0 1-.374.264 1.023 1.023 0 0 1-.453.083 1.334 1.334 0 0 1-.866-.264.965.965 0 0 1-.329-.801.993.993 0 0 1 .076-.431 1.02 1.02 0 0 1 .242-.363 1.478 1.478 0 0 1 1.043-.303h.952v-.181a.696.696 0 0 0-.136-.454.553.553 0 0 0-.438-.154.695.695 0 0 0-.378.086.48.48 0 0 0-.193.254l-.99-.144a1.26 1.26 0 0 1 .257-.563c.14-.174.321-.302.533-.378.261-.091.54-.136.82-.129.053-.003.106-.007.163-.007Zm4.384.007c.174 0 .347.038.506.114.182.083.34.211.458.374.257.423.377.911.351 1.406a2.53 2.53 0 0 1-.355 1.448 1.148 1.148 0 0 1-1.009.517c-.204 0-.401-.045-.582-.136a1.052 1.052 0 0 1-.48-.457 1.298 1.298 0 0 1-.114-.234h-.045l.004 1.784h-1.059v-4.713h.904l.117.805h.057c.068-.208.177-.401.328-.56a1.129 1.129 0 0 1 .843-.344h.076v-.004Zm7.559.084h.903l.113.805h.053a1.37 1.37 0 0 1 .235-.484.813.813 0 0 1 .313-.242.82.82 0 0 1 .39-.076h.234v1.051h-.401a.662.662 0 0 0-.313.008.623.623 0 0 0-.272.155.663.663 0 0 0-.174.26.683.683 0 0 0-.027.314v1.875h-1.054v-3.666Zm-17.515.003h3.262v.896L3.73 13.104l.034.113h1.973l.042.9H2.4v-.9l1.931-1.754-.045-.117H2.441v-.896Zm11.815 0h1.055v3.659h-1.055V10.45Zm3.443.684.019.016a.69.69 0 0 0-.351.045.756.756 0 0 0-.287.204c-.11.155-.174.336-.189.522h1.545c-.034-.526-.257-.787-.74-.787h.003Zm-5.718.163c-.026 0-.057 0-.083.004a.78.78 0 0 0-.31.053.746.746 0 0 0-.257.189 1.016 1.016 0 0 0-.204.695v.064c-.015.257.057.507.204.711a.634.634 0 0 0 .253.196.638.638 0 0 0 .314.061.644.644 0 0 0 .578-.265c.14-.223.204-.48.189-.74a1.216 1.216 0 0 0-.181-.711.677.677 0 0 0-.503-.257Zm-4.509 1.266a.464.464 0 0 0-.268.102.373.373 0 0 0-.114.276c0 .053.008.106.027.155a.375.375 0 0 0 .087.132.576.576 0 0 0 .397.11v.004a.863.863 0 0 0 .563-.182.573.573 0 0 0 .211-.457v-.14h-.903Z"/></symbol>
  <symbol id="i-paypal" viewBox="0 0 24 24"><path d="M7.016 19.198h-4.2a.562.562 0 0 1-.555-.65L5.093.584A.692.692 0 0 1 5.776 0h7.222c3.417 0 5.904 2.488 5.846 5.5-.006.25-.027.5-.066.747A6.794 6.794 0 0 1 12.071 12H8.743a.69.69 0 0 0-.682.583l-.325 2.056-.013.083-.692 4.39-.015.087zM19.79 6.142c-.01.087-.01.175-.023.261a7.76 7.76 0 0 1-7.695 6.598H9.007l-.283 1.795-.013.083-.692 4.39-.134.843-.014.088H6.86l-.497 3.15a.562.562 0 0 0 .555.65h3.612c.34 0 .63-.249.683-.585l.952-6.031a.692.692 0 0 1 .683-.584h2.126a6.793 6.793 0 0 0 6.707-5.752c.306-1.95-.466-3.744-1.89-4.906z"/></symbol>
  <symbol id="i-googlepay" viewBox="0 0 24 24"><path d="M3.963 7.235A3.963 3.963 0 00.422 9.419a3.963 3.963 0 000 3.559 3.963 3.963 0 003.541 2.184c1.07 0 1.97-.352 2.627-.957.748-.69 1.18-1.71 1.18-2.916a4.722 4.722 0 00-.07-.806H3.964v1.526h2.14a1.835 1.835 0 01-.79 1.205c-.356.241-.814.379-1.35.379-1.034 0-1.911-.697-2.225-1.636a2.375 2.375 0 010-1.517c.314-.94 1.191-1.636 2.225-1.636a2.152 2.152 0 011.52.594l1.132-1.13a3.808 3.808 0 00-2.652-1.033zm6.501.55v6.9h.886V11.89h1.465c.603 0 1.11-.196 1.522-.588a1.911 1.911 0 00.635-1.464 1.92 1.92 0 00-.635-1.456 2.125 2.125 0 00-1.522-.598zm2.427.85a1.156 1.156 0 01.823.365 1.176 1.176 0 010 1.686 1.171 1.171 0 01-.877.357H11.35V8.635h1.487a1.156 1.156 0 01.054 0zm4.124 1.175c-.842 0-1.477.308-1.907.925l.781.491c.288-.417.68-.626 1.175-.626a1.255 1.255 0 01.856.323 1.009 1.009 0 01.366.785v.202c-.34-.193-.774-.289-1.3-.289-.617 0-1.11.145-1.479.434-.37.288-.554.677-.554 1.165a1.476 1.476 0 00.525 1.156c.35.308.785.463 1.305.463.61 0 1.098-.27 1.465-.81h.038v.655h.848v-2.909c0-.61-.19-1.09-.568-1.44-.38-.35-.896-.525-1.551-.525zm2.263.154l1.946 4.422-1.098 2.38h.915L24 9.963h-.965l-1.368 3.391h-.02l-1.406-3.39zm-2.146 2.368c.494 0 .88.11 1.156.33 0 .372-.147.696-.44.973a1.413 1.413 0 01-.997.414 1.081 1.081 0 01-.69-.232.708.708 0 01-.293-.578c0-.257.12-.47.363-.647.24-.173.54-.26.9-.26Z"/></symbol>
  <symbol id="i-github" viewBox="0 0 24 24"><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/></symbol>
  <symbol id="i-shopify" viewBox="0 0 24 24"><path d="M15.337 23.979l7.216-1.561s-2.604-17.613-2.625-17.73c-.018-.116-.114-.192-.211-.192s-1.929-.136-1.929-.136-1.275-1.274-1.439-1.411c-.045-.037-.075-.057-.121-.074l-.914 21.104h.023zM11.71 11.305s-.81-.424-1.774-.424c-1.447 0-1.504.906-1.504 1.141 0 1.232 3.24 1.715 3.24 4.629 0 2.295-1.44 3.76-3.406 3.76-2.354 0-3.54-1.465-3.54-1.465l.646-2.086s1.245 1.066 2.28 1.066c.675 0 .975-.545.975-.932 0-1.619-2.654-1.694-2.654-4.359-.034-2.237 1.571-4.416 4.827-4.416 1.257 0 1.875.361 1.875.361l-.945 2.715-.02.01zM11.17.83c.136 0 .271.038.405.135-.984.465-2.064 1.639-2.508 3.992-.656.213-1.293.405-1.889.578C7.697 3.75 8.951.84 11.17.84V.83zm1.235 2.949v.135c-.754.232-1.583.484-2.394.736.466-1.777 1.333-2.645 2.085-2.971.193.501.309 1.176.309 2.1zm.539-2.234c.694.074 1.141.867 1.429 1.755-.349.114-.735.231-1.158.366v-.252c0-.752-.096-1.371-.271-1.871v.002zm2.992 1.289c-.02 0-.06.021-.078.021s-.289.075-.714.21c-.423-1.233-1.176-2.37-2.508-2.37h-.115C12.135.209 11.669 0 11.265 0 8.159 0 6.675 3.877 6.21 5.846c-1.194.365-2.063.636-2.16.674-.675.213-.694.232-.772.87-.075.462-1.83 14.063-1.83 14.063L15.009 24l.927-21.166z"/></symbol>
  <symbol id="i-wise" viewBox="0 0 24 24"><path d="M6.488 7.469 0 15.05h11.585l1.301-3.576H7.922l3.033-3.507.01-.092L8.993 4.48h8.873l-6.878 18.925h4.706L24 .595H2.543l3.945 6.874Z"/></symbol>
  <symbol id="i-gitlab" viewBox="0 0 24 24"><path d="m23.6004 9.5927-.0337-.0862L20.3.9814a.851.851 0 0 0-.3362-.405.8748.8748 0 0 0-.9997.0539.8748.8748 0 0 0-.29.4399l-2.2055 6.748H7.5375l-2.2057-6.748a.8573.8573 0 0 0-.29-.4412.8748.8748 0 0 0-.9997-.0537.8585.8585 0 0 0-.3362.4049L.4332 9.5015l-.0325.0862a6.0657 6.0657 0 0 0 2.0119 7.0105l.0113.0087.03.0213 4.976 3.7264 2.462 1.8633 1.4995 1.1321a1.0085 1.0085 0 0 0 1.2197 0l1.4995-1.1321 2.4619-1.8633 5.006-3.7489.0125-.01a6.0682 6.0682 0 0 0 2.0094-7.003z"/></symbol>
  <symbol id="i-square" viewBox="0 0 24 24"><path d="M4.01 0A4.01 4.01 0 000 4.01v15.98c0 2.21 1.8 4 4.01 4.01h15.98C22.2 24 24 22.2 24 19.99V4A4.01 4.01 0 0019.99 0H4zm1.62 4.36h12.74c.7 0 1.26.57 1.26 1.27v12.74c0 .7-.56 1.27-1.26 1.27H5.63c-.7 0-1.26-.57-1.26-1.27V5.63a1.27 1.27 0 011.26-1.27zm3.83 4.35a.73.73 0 00-.73.73v5.09c0 .4.32.72.72.72h5.1a.73.73 0 00.73-.72V9.44a.73.73 0 00-.73-.73h-5.1Z"/></symbol>
  <symbol id="i-asana" viewBox="0 0 24 24"><path d="M18.78 12.653c-2.882 0-5.22 2.336-5.22 5.22s2.338 5.22 5.22 5.22 5.22-2.34 5.22-5.22-2.336-5.22-5.22-5.22zm-13.56 0c-2.88 0-5.22 2.337-5.22 5.22s2.338 5.22 5.22 5.22 5.22-2.338 5.22-5.22-2.336-5.22-5.22-5.22zm12-6.525c0 2.883-2.337 5.22-5.22 5.22-2.882 0-5.22-2.337-5.22-5.22 0-2.88 2.338-5.22 5.22-5.22 2.883 0 5.22 2.34 5.22 5.22z"/></symbol>
  <symbol id="i-airtable" viewBox="0 0 24 24"><path d="M11.992 1.966c-.434 0-.87.086-1.28.257L1.779 5.917c-.503.208-.49.908.012 1.116l8.982 3.558a3.266 3.266 0 0 0 2.454 0l8.982-3.558c.503-.196.503-.908.012-1.116l-8.957-3.694a3.255 3.255 0 0 0-1.272-.257zM23.4 8.056a.589.589 0 0 0-.222.045l-10.012 3.877a.612.612 0 0 0-.38.564v8.896a.6.6 0 0 0 .821.552L23.62 18.1a.583.583 0 0 0 .38-.551V8.653a.6.6 0 0 0-.6-.596zM.676 8.095a.644.644 0 0 0-.48.19C.086 8.396 0 8.53 0 8.69v8.355c0 .442.515.737.908.54l6.27-3.006.307-.147 2.969-1.436c.466-.22.43-.908-.061-1.092L.883 8.138a.57.57 0 0 0-.207-.044z"/></symbol>
</svg>

<div class="stage" aria-hidden="true"><canvas id="stage"></canvas><div class="dim"></div></div>

<header class="nav">
  <a class="logo" href="#" aria-label="Clutch" style="--d:0ms">
    <!-- Clutch icon: two hooks closing around a single point -->
    <svg viewBox="0 0 46 40" fill="none" aria-hidden="true">
      <path d="M9 6C4 15 6 27 17 32" stroke="#fff" stroke-width="4.5" stroke-linecap="round"/>
      <path d="M37 6c5 9 3 21-8 26" stroke="#fff" stroke-width="4.5" stroke-linecap="round"/>
      <circle cx="23" cy="27" r="4" fill="#fff"/>
    </svg>
  </a>
  <a class="lk lk-1" href="#" style="--d:60ms">Features</a>
  <a class="lk lk-2" href="#" style="--d:120ms">Journal</a>
  <a class="lk lk-3" href="#" style="--d:180ms">Security</a>
  <a class="lk lk-4" href="#" style="--d:240ms">Talk to Us</a>
  <a class="join" href="#" style="--d:300ms">Join Now</a>
</header>

<main>
  <!-- 01 · hero : frames 0 → 16 while pinned (card swings into the open claw) -->
  <section class="sec hero" id="top">
    <div class="pin">
      <div class="top">
        <h1 class="title">
          <span class="ln"><span style="--d:150ms">Smart Grip on</span></span>
          <span class="ln"><span style="--d:240ms">Every Cent</span></span>
          <span class="ln"><span style="--d:330ms">You Earn</span></span>
        </h1>
      </div>
      <div class="bot">
        <p class="copy rv" style="--d:450ms">Live spending alerts and automatic saving rules keep a steady hand on every card you own</p>
        <a class="cta rv" href="#results" style="--d:550ms"><span class="pill">Explore Now</span><span class="arw"><svg viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><path d="M4 10h12M11 5l5 5-5 5"/></svg></span></a>
        <p class="note small rv" style="--d:650ms">Budgets that flex<br>by themselves</p>
        <div class="social rv" style="--d:700ms">
          <p class="small">Join the<br>Community</p>
          <a href="#" aria-label="Clutch on X"><svg viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2.2" stroke-linecap="round"><path d="M5 4l14 16M19 4L5 20"/></svg></a>
          <a href="#" aria-label="Clutch on Instagram"><svg viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2"><rect x="3.5" y="3.5" width="17" height="17" rx="5"/><circle cx="12" cy="12" r="4"/><circle cx="17.2" cy="6.8" r="1" fill="#fff" stroke="none"/></svg></a>
          <a href="#" aria-label="Clutch on LinkedIn"><svg viewBox="0 0 24 24" fill="#fff"><rect x="4" y="9" width="3.6" height="11" rx=".6"/><circle cx="5.8" cy="5.4" r="2.1"/><path d="M10 9h3.4v1.6c.6-1 1.9-1.9 3.7-1.9 3 0 3.9 1.9 3.9 4.9V20h-3.6v-5.6c0-1.5-.4-2.6-1.9-2.6s-2 1.1-2 2.6V20H10z"/></svg></a>
        </div>
      </div>
      <div class="mark" aria-hidden="true">CLUTCH PAY</div>
    </div>
  </section>

  <!-- 02 · results : frames 40 → 64 while pinned (card settles between the fingers) -->
  <section class="sec s2" id="results">
    <div class="pin">
      <div class="top">
        <p class="label rv">02 — By the Numbers</p>
        <h2 class="title two">
          <span class="ln"><span style="--d:80ms">Results That</span></span>
          <span class="ln"><span style="--d:170ms">Keep Their Grip</span></span>
        </h2>
        <p class="copy rv" style="--d:300ms">A few figures from the last twelve months, pulled straight from the dashboard our members open every day.</p>
      </div>
      <div class="bot">
        <div class="stats">
          <article class="card c1 rv" style="--d:380ms">
            <b data-count="120" data-suffix="K+">120K+</b>
            <h3>Active Members</h3>
            <p>People in more than thirty countries use Clutch to track every purchase, from a morning coffee to the monthly rent.</p>
          </article>
          <article class="card c2 rv" style="--d:460ms">
            <b data-count="99.9" data-dec="1" data-suffix="%">99.9%</b>
            <h3>Uptime All Year</h3>
            <p>Payments, alerts and card controls stayed online through every payday rush and holiday sale.</p>
          </article>
          <article class="card c3 rv" style="--d:540ms">
            <b data-count="4.9" data-dec="1" data-suffix="/5">4.9/5</b>
            <h3>Average App Rating</h3>
            <p>Scored by over eighteen thousand reviewers who moved their everyday spending to Clutch.</p>
          </article>
        </div>
      </div>
    </div>
  </section>

  <!-- 03 · integrations : frames 140 → 203 while pinned (claw holds the card, light sweeps across it) -->
  <section class="sec s3" id="stack">
    <div class="pin">
      <div class="top">
        <p class="label rv">03 — Fits Your Stack</p>
        <h2 class="title two">
          <span class="ln"><span style="--d:80ms">Plugs Right Into</span></span>
          <span class="ln"><span style="--d:170ms">Every Tool</span></span>
        </h2>
        <p class="copy rv" style="--d:300ms">Link your bank, payroll, accounting and team apps in a couple of clicks. Data moves both ways in real time, so every budget stays current.</p>
        <a class="cta rv" href="#top" style="--d:380ms"><span class="pill">Browse Integrations</span><span class="arw"><svg viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"><path d="M4 10h12M11 5l5 5-5 5"/></svg></span></a>
        <p class="note small rv" style="--d:440ms">120+ apps · two-way sync · set up in minutes</p>
      </div>
      <div class="bot">
        <div class="logos rv" style="--d:300ms" role="img" aria-label="Integrations including Visa, Mastercard, PayPal, Stripe, Apple Pay, Google Pay, Shopify, Slack, Notion, GitHub and more"></div>
      </div>
      <div class="foot rv" style="--d:520ms"><span>© 2026 Clutch. All rights reserved.</span><nav><a href="#">Privacy</a><a href="#">Terms</a><a href="#">Status</a></nav></div>
    </div>
  </section>
</main>

<script>
(() => {
  /* ---------- config ---------- */
  const FRAME_BASE = 'https://assets.layerweb.com.br/clutch/frames-hd/';        // 1920x1080 frames: clutch-000.webp … clutch-203.webp
  const FRAME_BASE_SM = 'https://assets.layerweb.com.br/clutch/frames-sm/';  // 960x540 copies for phones
  const COUNT = 204;                    // 24 fps source, starts 1.5 s into the clip
  const SRC_W = 1920, SRC_H = 1080;     // geometry is expressed in 1920x1080 source px
  const TOP = 'rgb(0,9,18)';            // average colour of the video's top row
  const EDGE_V = 140;                   // bottom band (source px) that is stretched downward
  const EDGE_SRC = 480;                 // outer source px (each side) that may be stretched sideways; the claw starts at x 550
  const CLAW_X = 960, CLAW_W = 830, CLAW_H = 960; // claw + hanging card: centre x, width, height from the top
  const W = 1440, H = 1015;             // design canvas (content never wider than 1440 px)
  const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
  const flowMQ = matchMedia('(max-width: 899px), (max-aspect-ratio: 1/1)');
  const landMQ = matchMedia('(max-width: 899px) and (min-aspect-ratio: 1/1)');
  const root = document.documentElement;
  const base = flowMQ.matches ? FRAME_BASE_SM : FRAME_BASE;

  /* ---------- canvas scale ---------- */
  let k = 1;
  function setScale() {
    // flow layout (phones): --u only sizes small details, so keep it near 1px
    k = flowMQ.matches ? Math.min(Math.max(innerWidth / 430, 0.8), 1) : Math.min(Math.min(innerWidth, W) / W, innerHeight / H);
    root.style.setProperty('--k', k.toFixed(5));
  }

  /* ---------- frames: progressive load (every 8th, 4th, 2nd, then the rest) ---------- */
  const imgs = new Array(COUNT), ok = new Uint8Array(COUNT);
  const order = [];
  for (const step of [8, 4, 2, 1]) for (let i = 0; i < COUNT; i += step) if (!order.includes(i)) order.push(i);
  let next = 0, active = 0;
  function pump() {
    while (active < 8 && next < order.length) {
      const i = order[next++], im = new Image();
      active++;
      im.decoding = 'async';
      im.onload = () => { ok[i] = 1; active--; if (i === 0) start(); need = true; pump(); };
      im.onerror = () => { active--; pump(); };
      im.src = base + 'clutch-' + String(i).padStart(3, '0') + '.webp';
      imgs[i] = im;
    }
  }
  function nearest(f) {
    for (let d = 0; d < COUNT; d++) {
      if (f - d >= 0 && ok[f - d]) return f - d;
      if (f + d < COUNT && ok[f + d]) return f + d;
    }
    return -1;
  }

  /* ---------- stage geometry ---------- */
  const cv = document.getElementById('stage'), cx = cv.getContext('2d');
  let G = { s: 1, ox: 0, oy: 0 }, vw = 0, vh = 0, dpr = 1;
  function geometry() {
    vw = innerWidth; vh = innerHeight;
    dpr = Math.min(devicePixelRatio || 1, 1.5);
    cv.width = Math.round(vw * dpr); cv.height = Math.round(vh * dpr);
    let s, ox, oy;
    if (landMQ.matches) {
      // phone held sideways: claw between the two side blocks, cables from the top edge
      const t = document.querySelector('.hero .top').getBoundingClientRect();
      const b = document.querySelector('.hero .bot').getBoundingClientRect();
      const gl = t.right + 12, gr = b.left - 12;
      s = Math.min(vh / SRC_H, (gr - gl) / CLAW_W);
      ox = (gl + gr) / 2 - CLAW_X * s;
      oy = 0;
    } else if (flowMQ.matches) {
      // phones: fit the claw in the band between the hero's top block and bottom block
      const pin = document.querySelector('.hero .pin');
      const t = pin.querySelector('.top').getBoundingClientRect();
      const b = pin.querySelector('.bot').getBoundingClientRect();
      const p = pin.getBoundingClientRect();
      const top = t.bottom - p.top + 10, bot = b.top - p.top - 10;
      s = Math.min(vw * 0.94 / CLAW_W, Math.max(bot - top, 120) / CLAW_H);
      ox = vw / 2 - CLAW_X * s;
      oy = top + (bot - top - CLAW_H * s) / 2;
    } else {
      // desktop: claw centred on the 1440 px content frame; it may not grow past the right column (starts at 1126)
      // or the lower-left content (ends at 300). The wide titles stay clear because the fingers spread below them.
      const cw = Math.min(vw, W), fl = (vw - cw) / 2, c = fl + cw / 2;
      const gl = fl + 300 * k, gr = fl + cw - (W - 1126) * k;
      s = Math.min(vh / SRC_H, 2 * (gr - c - 12 * k) / CLAW_W, 2 * (c - gl - 12 * k) / CLAW_W);
      ox = c - CLAW_X * s;
      oy = 0;
    }
    G = { s, ox, oy };
    // "elastic" side edges: where the frame is narrower than the viewport, its outer part is stretched sideways with a
    // quadratic mapping (slope at the join = the frame scale s), so the video's own vignette simply keeps going.
    const dw = SRC_W * s, L = Math.max(0, ox), R = Math.max(0, vw - (ox + dw));
    const snap = x => Math.round(x * dpr) / dpr;      // whole device pixels: neighbouring slices never leave a hairline
    slices = [];
    if (L > 0.5 || R > 0.5) {
      const A = EDGE_SRC, N = 40, xl = ox + A * s, xr = ox + (SRC_W - A) * s, cl = L / (A * A), cr = R / (A * A);
      for (let i = 0; i < N; i++) {                   // left side, from the screen edge to the join
        const r0 = A * (1 - i / N), r1 = A * (1 - (i + 1) / N);
        const d0 = i === 0 ? 0 : snap(xl - (s * r0 + cl * r0 * r0)), d1 = i === N - 1 ? snap(xl) : snap(xl - (s * r1 + cl * r1 * r1));
        if (d1 > d0) slices.push([A * i / N, A / N, d0, d1 - d0]);
      }
      slices.push([A, SRC_W - 2 * A, snap(xl), snap(xr) - snap(xl)]);
      for (let i = 0; i < N; i++) {                   // right side, from the join to the screen edge
        const r0 = A * i / N, r1 = A * (i + 1) / N;
        const d0 = i === 0 ? snap(xr) : snap(xr + (s * r0 + cr * r0 * r0)), d1 = i === N - 1 ? vw : snap(xr + (s * r1 + cr * r1 * r1));
        if (d1 > d0) slices.push([SRC_W - A + A * i / N, A / N, d0, d1 - d0]);
      }
    } else slices = [[0, SRC_W, ox, dw]];
    const Bg = Math.max(0, vh - (oy + SRC_H * s));
    dim.style.height = Bg > 0.5 ? (120 * s + Bg) + 'px' : '0px';   // floor dimmer: from 120 source px above the frame bottom to the screen bottom
    need = true;
  }
  const dim = document.querySelector('.stage .dim'), band = document.createElement('canvas'), bx = band.getContext('2d');
  let slices = [];

  /* draw one frame (as slices) plus the extensions above and below, so the video blends into any viewport */
  function paint(i, alpha) {
    const im = imgs[i], iw = im.naturalWidth, ih = im.naturalHeight, q = iw / SRC_W;
    const { s, ox, oy } = G;
    const dh = SRC_H * s;
    cx.setTransform(dpr, 0, 0, dpr, 0, 0);
    cx.imageSmoothingEnabled = true; cx.imageSmoothingQuality = 'high';
    cx.globalAlpha = alpha;
    const T = Math.max(0, oy), B = Math.max(0, vh - (oy + dh));
    if (T > 0) { cx.fillStyle = TOP; cx.fillRect(0, 0, vw, oy + 1); }
    for (const [sx, sw, dx, dwid] of slices) {
      cx.drawImage(im, sx * q, 0, sw * q, ih, dx, oy, dwid, dh);
    }
    // flow layout: the cable fades out of a flat night colour above the video instead of being stretched
    if (T > 0) {
      const g = cx.createLinearGradient(0, oy, 0, oy + 160 * s);
      g.addColorStop(0, TOP); g.addColorStop(1, 'rgba(0,9,18,0)');
      cx.fillStyle = g; cx.fillRect(0, oy - 1, vw, 160 * s + 1);
    }
    cx.globalAlpha = 1;
  }

  /* draw a fractional position: the two neighbouring frames, the second one cross-faded in by the fraction.
     Every pass paints the whole canvas opaquely, so (1 - t)·A + t·B holds per pixel, strips and dimmers included. */
  function draw(pos) {
    const a = Math.floor(pos), t = pos - a;
    const fa = nearest(a);
    if (fa < 0) return false;
    paint(fa, 1);
    const fb = nearest(Math.min(a + 1, COUNT - 1));
    if (t > 0.01 && fb >= 0 && fb !== fa && Math.abs(fb - fa) <= 2) paint(fb, t);
    extendBottom();
    return true;
  }

  /* elastic bottom: the band of what is painted (both frames already blended) is stretched down to the screen edge
     with y = y0 + r + c·r² (slope 1 at the join, c = B / band²). Runs once per drawn position, at full alpha. */
  function extendBottom() {
    const { s, oy } = G, dh = SRC_H * s, B = Math.max(0, vh - (oy + dh));
    if (B <= 0.5) return;
    const hb = Math.min(EDGE_V * s, dh), y0 = oy + dh - hb, c = B / (hb * hb), N = 36;
    const hd = Math.round(hb * dpr), yd = Math.round(y0 * dpr);
    if (band.width !== cv.width || band.height !== hd) { band.width = cv.width; band.height = hd; }
    bx.setTransform(1, 0, 0, 1, 0, 0);
    bx.drawImage(cv, 0, yd, cv.width, hd, 0, 0, cv.width, hd);
    cx.setTransform(dpr, 0, 0, dpr, 0, 0);
    cx.globalAlpha = 1;
    for (let i = 0; i < N; i++) {
      const r0 = hb * i / N, r1 = hb * (i + 1) / N;
      const d0 = i === 0 ? y0 : Math.round((y0 + r0 + c * r0 * r0) * dpr) / dpr;
      const d1 = i === N - 1 ? vh : Math.round((y0 + r1 + c * r1 * r1) * dpr) / dpr;
      if (d1 > d0) cx.drawImage(band, 0, i * hd / N, cv.width, hd / N, 0, d0, vw, d1 - d0);
    }
  }

  /* ---------- scroll → frame ---------- */
  const secs = [...document.querySelectorAll('.sec')];
  let anchors = [];
  function measure() {
    const y = s => s.offsetTop, h = s => s.offsetHeight, v = innerHeight;
    const [a, b, c] = secs;
    const max = document.documentElement.scrollHeight - v;
    anchors = [
      [0, 0],                    // card hangs from the open claw
      [y(a) + h(a) - v, 16],     // hero released: card mid-swing
      [y(b), 40],                // section 2 pinned: card turning flat
      [y(b) + h(b) - v, 64],     // section 2 released: card settled in the grip
      [y(c), 140],               // section 3 pinned: slow push-in during the hold
      [max, COUNT - 1]           // end of page: light sweep across the card, final frame
    ];
  }
  function target() {
    if (!anchors.length) return 0;
    const y = Math.min(Math.max(scrollY, 0), anchors[anchors.length - 1][0]);
    for (let i = 1; i < anchors.length; i++) {
      const [y0, f0] = anchors[i - 1], [y1, f1] = anchors[i];
      if (y <= y1) return f0 + (f1 - f0) * (y1 > y0 ? (y - y0) / (y1 - y0) : 1);
    }
    return COUNT - 1;
  }

  /* critically damped spring (no overshoot), driven by real elapsed time so 60 Hz and 144 Hz screens move alike */
  function smoothDamp(x, goal, v, smoothTime, dt) {
    const w = 2 / smoothTime, e = w * dt, k = 1 / (1 + e + 0.48 * e * e + 0.235 * e * e * e);
    const d = x - goal, tmp = (v + w * d) * dt;
    return [goal + (d + tmp) * k, (v - w * tmp) * k];
  }
  /* decode the next frames in the scroll direction ahead of time, so drawing them never stalls */
  const decoding = new Set();
  function prefetch(pos, v) {
    const dir = v < 0 ? -1 : 1, a = Math.floor(pos);
    for (let d = -2; d <= 10; d++) {
      const i = a + d * dir;
      if (i < 0 || i >= COUNT || !ok[i] || decoding.has(i)) continue;
      decoding.add(i); imgs[i].decode().catch(() => {});
    }
  }

  let cur = 0, vel = 0, drawn = -1, need = true, started = false, last = performance.now();
  function tick(now) {
    const dt = Math.min(Math.max((now - last) / 1000, 0.001), 0.064); last = now;
    const t = target();
    if (reduce) { cur = Math.round(t); vel = 0; }
    else {
      [cur, vel] = smoothDamp(cur, t, vel, 0.17, dt);
      if (Math.abs(t - cur) < 0.004 && Math.abs(vel) < 0.02) { cur = t; vel = 0; }
    }
    cur = Math.min(Math.max(cur, 0), COUNT - 1);
    if ((need || Math.abs(cur - drawn) > 0.004) && draw(cur)) { drawn = cur; need = false; prefetch(cur, vel || t - cur); }
    reveal();
    requestAnimationFrame(tick);
  }
  function start() {
    if (started) return;
    started = true;
    document.querySelector('.stage').classList.add('ready');
  }

  /* ---------- section reveals (one-time) + pin fade-out ---------- */
  const pins = secs.map(s => s.querySelector('.pin'));
  function reveal() {
    const v = innerHeight;
    pins.forEach(p => { const t = p.getBoundingClientRect().top; p.style.opacity = t < 0 ? Math.max(0, 1 + t / (v * 0.55)).toFixed(3) : ''; });
    for (const s of secs) if (!s.classList.contains('on') && s.getBoundingClientRect().top < v * 0.45) { s.classList.add('on'); if (s.id === 'results') countUp(); }
  }

  /* ---------- section 2: numbers count up once ---------- */
  function countUp() {
    document.querySelectorAll('[data-count]').forEach(el => {
      const to = +el.dataset.count, dec = +(el.dataset.dec || 0), suf = el.dataset.suffix || '';
      if (reduce) return;
      const t0 = performance.now(), dur = 1600;
      const step = now => {
        const p = Math.min(1, (now - t0) / dur), e = 1 - Math.pow(1 - p, 3);
        el.textContent = (to * e).toFixed(dec) + suf;
        if (p < 1) requestAnimationFrame(step);
      };
      requestAnimationFrame(step);
    });
  }

  /* ---------- section 3: three looping logo columns (rows on phones) ---------- */
  const cols = [
    ['visa', 'stripe', 'slack', 'quickbooks', 'apple', 'figma', 'klarna', 'dropbox', 'hubspot'],
    ['mastercard', 'applepay', 'notion', 'xero', 'google', 'discord', 'revolut', 'trello', 'zapier'],
    ['paypal', 'googlepay', 'github', 'shopify', 'wise', 'gitlab', 'square', 'asana', 'airtable']
  ];
  const durs = [34, 28, 38];
  const logos = document.querySelector('.logos');
  cols.forEach((list, c) => {
    const col = document.createElement('div'); col.className = 'col';
    const tr = document.createElement('ul'); tr.className = 'track'; tr.style.setProperty('--dur', durs[c] + 's');
    tr.setAttribute('aria-hidden', 'true');
    for (const name of list.concat(list)) tr.insertAdjacentHTML('beforeend', `<li><svg viewBox="0 0 24 24"><use href="#i-${name}"/></svg></li>`);
    col.append(tr); logos.append(col);
  });

  /* ---------- resize (three triggers) ---------- */
  let lastW = 0, lastH = 0;
  function refresh() {
    setScale();
    requestAnimationFrame(() => { geometry(); measure(); });
    lastW = innerWidth; lastH = innerHeight;
  }
  addEventListener('resize', refresh);
  new ResizeObserver(refresh).observe(document.body);
  setInterval(() => { if (innerWidth !== lastW || innerHeight !== lastH) refresh(); }, 500);

  setScale();
  const boot = () => { geometry(); measure(); root.classList.add('loaded'); secs[0].classList.add('on'); };
  (document.fonts && document.fonts.ready ? document.fonts.ready : Promise.resolve()).catch(() => {}).then(boot);
  pump();
  requestAnimationFrame(tick);
})();
</script>
</body>
</html>
```
