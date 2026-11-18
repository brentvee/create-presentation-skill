# Deck engine guide

`assets/deck-engine.html` is a working, tested presentation runtime: a fixed 1600×900 stage scaled to fit, a clock-driven timeline per slide, and a presenter dock (play/pause, speed, scrubber with beat ticks, holds, auto-advance, replay, skip-to-end, overview, fullscreen, Clean view, help) above a live storyboard strip. You write only the slides, the deck CSS and one builder per slide.

`assets/example-staydesk-deck.html` is a complete 16-slide deck built on this engine. **Grep it for patterns instead of reading it whole** (it is about 140 KB): `grep -n 'data-id="search"'`, `grep -n '    search(s, tl)'`, and so on.

## Contents
1. Filling the template
2. Timeline API
3. Helpers
4. CSS primitives
5. Slide patterns (with pointers into the example)
6. Rules that prevent the bugs we already hit
7. Verification script
8. Speaker notes (separate synced window)

---

## 1. Filling the template

Copy the engine to the output path, then replace the five placeholders:

| Placeholder | Replace with |
|---|---|
| `{{TITLE}}` | A 2–4 word deck name ("Acme Demo") |
| `{{DESCRIPTION}}` | One sentence |
| `/* {{DECK_CSS}} */` | All deck-specific CSS: recreated app UI, per-slide layouts, brand token overrides |
| `<!-- {{SLIDES}} ... -->` | `<section class="slide" data-id="x" data-title="Shown in storyboard">…</section>` × N |
| `/* {{PRERENDER}} ... */` | Optional JS that **generates repeated markup** into slides (a screen used on three slides, leaderboards, icon rows) before the engine snapshots them. It runs once, before any builder, so builders and thumbnails see the result. Don't put `<script>` tags inside SLIDES. |
| `/* {{HELPERS}} ... */` | Optional deck-specific helper functions shared by builders (macros like `tapAndScore(tl, app, …)`) |
| `/* {{BUILDERS}} ... */` | `x(s, tl) { … },` for each `data-id` |

Also set **`CONFIG`** at the top of the script from the controls preset (live / kiosk / shareable; the comments in the file show the values).

**Brand:** override the tokens in `:root` (`--bg`, `--bg-2`, `--ink`, `--muted`, `--teal`, `--teal-2`, `--sand`, `--coral`, `--sky`, and `--on-accent` for text on `--sand` fills). The names are historical: `--sand` is the primary accent and `--coral` the secondary, so map the brand onto them instead of renaming. Everything in the engine reads these tokens: the background blobs, play button, hold hint, active step, callout icon, ripple and confetti. Rebranding is therefore just the tokens, with no per-rule overrides. These `:root` values are the **dark** theme.

**Light/dark theme:** built in. The `data-theme` attribute on `<html>` is the default (`light` out of the box; set `dark` if the interview says so). A ☀/☾ dock button and the `T` key toggle it, and the viewer's choice is remembered. In light mode the engine swaps the neutrals (`--page`, `--bg`, `--bg-2`, `--ink`, `--ink-2`, `--muted`, `--faint`, `--panel`) and darkens the accents by reading optional per-deck variables:

```css
:root {                          /* in DECK_CSS, next to the brand tokens */
  --sand-light: #4d7c0f;         /* the primary accent, dark enough for text on a light page */
  --coral-light: #0369a1; --teal-2-light: #0284c7; --sky-light: #0369a1;
  --on-accent-light: #ffffff;    /* text on the (now darker) accent: white usually reads best */
  --bg-light: #eef3fb; --page-light: #dde5f2; --ink-light: #0b1d42;   /* optional brand-tinted neutrals */
}
```

Very light brand colours (lime, cyan, yellow) need darker `-light` variants, because they are unreadable as text on a light page. Anything else you hard-code in DECK_CSS (gradients, panel backgrounds, icon tints) needs a `:root[data-theme="light"] .x { … }` twin. Code cards, terminals and the recreated app screens keep their own colours in both themes. To remove the toggle, delete the `#b-theme` button and fix `data-theme`.

**Fonts:** system stacks only (`--sans`, `--mono`). Do not use web fonts, because the deck must work offline. If the brand font isn't installed it falls back silently, so name it first in the stack and accept the fallback. Code ligatures are switched off (`=>` stays `=>`).

**Icons:** icon fonts (Bootstrap Icons, Font Awesome, Material Symbols) are web fonts and won't load offline. Draw the handful you need as inline SVG. Put them **once** in a hidden sprite at the top of `#stage` (`<svg width="0" height="0" style="position:absolute"><symbol id="i-play" viewBox="0 0 24 24">…</symbol></svg>`) and use `<svg class="ic"><use href="#i-play"/></svg>` everywhere. Match the app's icon names so the source mapping stays obvious.

**QR codes and images:** no network. Use inline SVG, or a data URI for small rasters such as a logo. Only add a QR code if you can embed it as inline SVG (for example one the user provides). Otherwise show the URL in large type.

## 2. Timeline API

Each builder gets `s` (the slide's `<section>`, freshly restored from its original HTML) and `tl` (a Timeline). Calls are sequential: `tl.cur` advances as you add waits and tweens.

```js
tl.wait(ms)                         // advance the cursor
tl.do(() => { … })                  // one-shot cue at the current time
tl.tw(dur, () => {                  // tween: setup runs once at start…
  const from = …;                   //   (measure DOM here, not at build time)
  return p => { … };                //   …update(p) runs every frame, p = eased 0→1
}, ease.inOut, /*advance=*/true)    // advance=false → runs in parallel with what follows
tl.beat('cue for the presenter')    // a hold point (with Holds on, playback pauses here);
                                    //   the label shows as a cue in the speaker-notes window
tl.cue('talking point')             // a cue at this moment, without pausing
```

Builders run again on every replay or backward seek, from pristine HTML, so they must be pure: query elements inside `s` and never keep state outside the closure.

`ease.linear`, `ease.inOut`, `ease.out` are available.

## 3. Helpers

| Helper | Use |
|---|---|
| `show(tl, els, gap)` | Adds `.in` to each element (reveals `.fx`), optionally staggered by `gap` ms |
| `type(tl, el, text, cps, cls)` | Types text. `cls='caret'` for the deck caret, `'typing'` (default) for a thin caret inside recreated UI |
| `countUp(tl, el, from, to, dur, fmt)` | Number tween (parallel; add `tl.wait(dur)` if needed) |
| `placeCursor(root, x, y)` | Set the fake cursor's start position (call at build time) |
| `moveTo(tl, root, target, dur, fx, fy)` | Glide the `.cursor` inside `root` to an element (fx/fy = point within it) |
| `click(tl, root, pressEl?)` | Click ripple, plus an optional `.press` scale on a button |
| `zoom(tl, app, target or null, scale, dur)` | Zoom the recreated app to an element (`null` zooms back out). `app` must sit inside a positioned, overflow-hidden viewport |
| `callout(tl, s, idx)` | Show the idx-th `.callout` in the slide (`-1` hides all) |
| `step(tl, s, n)` | Advance a `.steps` checklist (n-th = now, earlier = done) |
| `confetti(container, n, ox, oy)` | Burst inside a positioned container |
| `bump(el)` | Quick scale pop (counts, badges) |
| `offsetIn(el, root)`, `$`, `$$` | Layout measurement and query helpers |

`root` for cursor helpers is any `position: relative` element that contains a `.cursor` (copy the cursor SVG from the example).

## 4. CSS primitives

- `.fx` plus a direction (`up`, `down`, `left`, `right`, `pop`, `blur`) is hidden until `.in` is added. Use these for nearly every reveal.
- `.w` is for words in a headline that cascade in (`show(tl, $$('.w', s), 110)`).
- `.kicker`, `.huge`, `.big`, `.h2`, `.lede` and `.grad` (gradient text) are the type scale.
- `.chip`, `.caret`, `.gone` (display none), `.bump`.
- `.msg` / `.msg.claude` with `.av` and `.bubble` make chat bubbles. `.dlabel` is a small caps label.
- **Product frame:** `.slide.product` (row layout), `.side` (left column with `.stepno`, `h2`, `p`, `.steps`, `.src`), `.browser > .chrome + .vp > .app` (fake browser 1130×760), `.cursor`, `.ripple`, `.callout` (`.right`, `.top`), `.btn-p`/`.btn-o`, `.inp`, `.val[data-ph]`, `.press`, `.loading`. The `.a-side`/`.a-main`/`.a-top`/`.a-content` app-shell classes are optional defaults (a sidebar layout). Restyle them in DECK_CSS to match the real app.
- **Phone frame (built in):** `<div class="phone"><div class="vp"><div class="status">…</div><div class="app">…<div class="cursor touch fx"></div></div></div></div>`. The viewport is 380×788.
  - `.cursor.touch` is a fingertip dot. `moveTo` and `click` work the same, and the ripple reads as a tap.
  - **Scrolling:** wrap the content in `.scroller` and tween its `translateY`. The cursor helpers account for `.scroller` transforms, so after a scroll `moveTo` still lands on the element.
  - **Zooming:** keep the fixed chrome (status bar, header, tab bar) as siblings of the zoom layer, not inside it, and pass the zoom layer as `app` to `zoom()`. Its parent must be the clipping viewport. Zoom only into targets narrower than the viewport, and use a highlight ring for wide ones.
  - **Layout:** phone slides are `.side | .phone | .aside`. The `.aside` column needs a job: a synced code panel, static callouts (`.aside .callout`), a second phone (organiser and spectator), or a big result. Don't leave 700px empty.
  - **Text size:** text inside a recreated phone screen can be 12–15px (that's what phones use). Deck text outside the frame stays ≥16px.
- **Built-in extras:** `.mask` (left-to-right wipe on `.in`), `.confetto` (used by `confetti()`).
- **Storyboard thumbnails** render every slide's *original* HTML (after PRERENDER) at 11%, with the slide's `.active` layout and all `.fx` forced visible. Mark overlays and end states with `data-mini-hide`, and content that is `display:none` until revealed with `data-mini-show`, so thumbnails look like the slide.

## 5. Slide patterns

Grep the example for each one.

| Pattern | Example `data-id` | Notes |
|---|---|---|
| Title with cascading words | `title` | |
| Product intro with feature cards and stack chips | `meet` | `countUp` for a lines-of-code stat |
| Process pipeline | `process` | Steps light up as a fill line tweens |
| Interview / Q&A | `grill`, `answers` | Chat bubble typing, question lists, typed answers, "accepted defaults" chips |
| Multiple-choice replay | `choices` | Stacked `.qpanel`s, a cursor picks options, a log builds on the side |
| Spec checklist plus "go" | `spec` | Lines tick on, then a big bubble pops |
| Terminal prompt typing | `prompt` | Multi-span typing with highlighted keywords |
| Source → Claude → output flow | `inputs` | Tokens fly along a bezier (`tl.tw` with `advance=false`) into an orb |
| Section divider with mask wipe | `divider` | |
| **Product flow in a fake browser** | `search`, `booking`, `status` | Cursor, typing into inputs, dropdowns, cards cascading, stamps and collapse, zoom and callout, validation shake, confirmation with confetti, table row state changes and filters, a state-machine SVG lit in sync |
| Counters and mapping rows | `recap` | |
| Closing | `end` | |

**Story shape that works:** hook → meet the product → (optional: how it was made) → divider → 2–4 product flows → explicit reveal ("…and this whole deck was generated from the source") → call to action or "try it yourself" steps → Q&A. Each product flow slide has a left column with a 3-step `.steps` checklist that advances in sync with the animation. This is what keeps the audience oriented.

## 6. Rules that prevent the bugs we already hit

- **Anything long must be clock-driven.** Use `tl.tw` for scrolling text, progress fills and movement, not CSS keyframes. Seeking finishes CSS animations instantly, so a scrub into the middle of a long CSS animation jumps straight to its end state. Short CSS transitions on `.fx` are fine.
- **Measure in tween setup, not at build time.** Elements revealed later (`display:none` until a cue) have zero size at build time.
- **Aim the cursor at HTML elements** (the button, not the `<svg>` icon inside it). SVG targets are supported, but a target that can't be measured makes `moveTo` log an error and skip the move rather than freezing silently. The verification script reports these.
- **Space never restarts.** The engine already makes play/pause a no-op at the end of a slide. Keep replay explicit (R, ↺, storyboard click).
- **Keep the dock visible by default.** Auto-hiding controls read as "there are no controls". Clean view (`D`) is for the live moment.
- **Use real strings from the source:** validation messages, status names, button labels, routes, seed data. That is the whole point of "generated from the source", and developers notice.
- **Pin formats.** Write currencies and dates exactly as the app renders them in its configured culture. Don't let the machine's locale leak in.
- **Stay on the stage.** Everything lives inside 1600×900. Test long names in tables and grids (wrap or truncate), and keep body text ≥16px.
- **Unique `data-id`s** that match builder names. `data-title` feeds the storyboard and overview.
- **No external requests.** No CDNs, fonts or images by URL. Inline SVG for icons and logos (or a data URI if the logo is a small raster).
- **Be honest on slides about the process.** If the deck shows how it was made, don't present a condensed summary as "the one prompt I typed".

## 7. Verification script

Open the deck in a browser (built-in browser pane, or Chrome via `file:///…`). The engine exposes a small API for this:

- `deck.at(n, ms?)` jumps to slide `n` (1-based) at `ms`, or to the end state if `ms` is omitted, and pauses there.
- `deck.state` reports the current slide, time and theme.
- `deck.theme('light' | 'dark')` sets the theme without saving it.

Use these instead of simulating keys. Run this in the page. It walks every slide to its end state and reports timings, errors, overflow and stuck cursors:

```js
window.__errs = []; addEventListener('error', e => __errs.push(e.message));
const wait = ms => new Promise(r => setTimeout(r, ms));
const n = document.querySelectorAll('#stage .slide').length, out = [];
for (let i = 1; i <= n; i++) {
  deck.at(i); await wait(150);
  const act = document.querySelector('#stage .slide.active');
  const st = document.querySelector('#stage').getBoundingClientRect();
  const outside = (r, b) => r.width > 0 && (r.right > b.right + 2 || r.left < b.left - 2 || r.bottom > b.bottom + 2 || r.top < b.top - 2);
  // 1. Deck elements poking outside the 1600x900 stage (app screens and particles excluded)
  const over = [...act.querySelectorAll('*')].filter(e => !e.closest('.vp, .confetto, .token, .clap, .spark') && outside(e.getBoundingClientRect(), st)).length;
  // 2. Text overflowing an app viewport sideways (vertical overflow is normal: it scrolls)
  const side = [...act.querySelectorAll('.vp')].reduce((k, vp) => { const b = vp.getBoundingClientRect();
    return k + [...vp.querySelectorAll('*')].filter(e => !e.closest('.confetto') && e.children.length === 0 && e.textContent.trim() && (e.getBoundingClientRect().right > b.right + 2 || e.getBoundingClientRect().left < b.left - 2)).length; }, 0);
  // 3. Cursors stuck at NaN
  const nan = [...act.querySelectorAll('.cursor')].some(c => /NaN/.test(c.style.transform));
  out.push(`${i}: ${deck.state.dur}ms${over ? ' OVERFLOW:' + over : ''}${side ? ' VP-SIDE:' + side : ''}${nan ? ' CURSOR-NaN' : ''}`);
}
out.join(' | ') + ' || errors: ' + __errs.join('; ') + ' || warnings: ' + (window.__deckWarnings || []).join('; ')
```

Then screenshot every product-flow slide **at its end state and at 2–3 mid points** (`deck.at(5, 8000)`). Check **both themes**: run `deck.theme('dark')` and re-screenshot the title, one product slide and the reveal.

Browser quirks we hit:
- Screenshots in a background pane can lag one step, so re-take one if it shows the previous state.
- Navigating to a `file://` deck may open a new tab, so list the tabs and use the new tab's id.
- Custom viewport sizes reset per tab, and `#slide-N` links don't work in local-file snapshots. Use `deck.at()` instead. After editing the file, reload in the **same** tab (`location.reload()` via JavaScript) instead of navigating again, which keeps the tab id and viewport.

**Checking the speaker notes:** open the same deck in a second tab with `#notes` on the end (use an `http://localhost` URL, because the built-in browser pane can't open a `file://` deck as a second window). In that tab:
- `deckNotes.data` lists each slide's title, summary and cues. Check that every content slide has a summary and every `beat()` has a label.
- `deckNotes.connected` should be `true`.
- `deckNotes.cmd('goto', { i: 3 })` should move the deck.

## 8. Speaker notes (separate synced window)

For live talks shared over Teams or Zoom, the presenter keeps talking points in a **separate window** that the audience never sees. Pressing `N` (or **Notes** in the dock) opens the same HTML file with `#notes`, which renders a speaker view instead of the deck.

**Authoring:** both parts live in the deck file.

```html
<section class="slide" data-id="search" data-title="Search & results">
  <aside class="notes">Search flow: dates and guests in, available rooms out.
    Watch two rooms get stamped <b>Booked</b> by the overlap check.</aside>
  …
</section>
```

```js
search(s, tl) {
  …
  tl.wait(300).beat('Inputs filled in. Now run the search');      // shown as a cue; pauses with Holds on
  tl.cue('Mention the price × nights total');                     // cue only, no pause
  …
}
```

- **Summary** (`<aside class="notes">`): 1–3 sentences per slide about *what to say and why it matters to this audience*, not a description of what's on screen. HTML is allowed. It's never shown on the stage or in thumbnails.
- **Cues**: label every `tl.beat()` with what the presenter says or points at as that moment lands. Add `tl.cue()` for talking points that shouldn't pause. Keep each cue short enough to read at a glance (about 12 words).
- **Talk length:** set `CONFIG.talkMinutes` from the interview, because the notes timer counts against it.

**What the notes window shows:** the slide number and title, the editable summary, and the cue list with the current cue highlighted as the deck plays (click a cue's time to jump the deck there). Plus a "Holding at a beat" banner, "on screen now" and "next up" previews, a talk timer against `talkMinutes` (it starts when the deck leaves slide 1), the clock, and big Prev / Play / Next buttons. The keys and presentation clickers (PageUp/PageDown) work in either window.

**Sync:** the deck sends its state to the window it opened (`postMessage`) and over a `BroadcastChannel` named after the file, on every change plus a heartbeat about once a second. The notes window sends commands back by exactly one route (its opener if it has one, otherwise the channel), so nothing fires twice. If either window reloads, the channel reconnects them, and a notes tab opened by hand with `#notes` also connects. Pop-ups opened by `N` or a click are allowed by browsers; if one is blocked, the deck shows a hint.

**Edits:** the presenter can edit the summary and cues in the notes window. Edits are kept in that browser (`localStorage`) and marked ✎. **Copy edited notes** produces an "Apply these speaker-note edits to the deck" message to paste back to Claude, which then updates the `<aside>`s and `beat()` labels in the file.

**Presenting over Teams or Zoom:** share **only the deck window**, never the whole screen. Put the notes window on a second monitor, or beside the deck on a single monitor (window sharing doesn't capture it).
