# Screen-reader accessibility

Every deck must work with a screen reader. These rules come from a screen-reader user and are not optional. The engine already handles many of them; the rest are your job when you write the slides, the builders and the deck CSS.

## Contents
1. The rules
2. What the engine already does
3. Your job when building a deck
4. Testing before you hand it over

---

## 1. The rules

**What the screen reader reads**
- Hidden content must also be hidden from the screen reader. When an element is invisible (for example opacity 0), it gets `aria-hidden="true"`, and that comes off again when it shows.
- Visible text that carries meaning stays readable as plain text. Don't replace it with `role="img"` or `aria-label`, and don't hide it.
- When text changes between views or states, only the current text is read, never an old or hidden version.
- Pure decoration is hidden: icons, moving symbols, arrow markers, visual effects and animation markers.
- Nothing is read twice.

**Interactive elements**
- Everything clickable is a real `<button>` or link. If it can't be a native element, give it `role="button"` and `tabindex="0"`, and make Enter and Space activate it.
- Each element in a diagram or chart has an `aria-label` that gives all the facts about it in full sentences. That includes facts shown only visually (lines, colours, positions, connections).
- Update those labels whenever the view, a setting or the state changes.

**Tooltips and visual repeats**
- A tooltip or visual panel that repeats information already in an `aria-label` gets `aria-hidden="true"`.

**Controls**
- Group related buttons with `role="group"` and `aria-labelledby` pointing to a **visible** caption.
- Mark the selected option in a set of buttons with `aria-pressed="true"`.
- On/off controls are `role="switch"` with `aria-checked`. If the control also shows "On" or "Off" as text, hide that word with `aria-hidden="true"`.
- Change a button's text at the moment of the click, not after an animation ends.

**Announcements**
- A short description of the current view goes into an `aria-live="polite"` region, so each change is announced.
- An area gets `aria-busy="true"` while it animates, and `"false"` after.

**Keyboard**
- Escape closes or resets a temporary state, such as a highlight, a selection or a panel.
- Every control stays usable during animations. Never disable controls while something animates.

---

## 2. What the engine already does

You don't need to write these, but don't undo them:

| Rule | How the engine handles it |
|---|---|
| Hidden content hidden | After every timeline step, `.fx`, `.w`, `.mask`, `.callout` and anything with **`data-reveal`** get `aria-hidden="true"` until they gain `.in`, `.show`, `.cur` or `.open`, and lose it the moment they do. Inactive slides and `.gone` are `display:none`. |
| Decoration hidden | `#bg`, `.cursor`, `.ripple`, `.confetto`, `.token`, `.spark`, `.clap`, `.scan`, anything with **`data-decor`**, the progress bar, scrubber track, toast and hold hint. Every `<svg>` without a `role`/`aria-label` is treated as an icon and hidden. |
| Nothing read twice | Storyboard thumbnails (full copies of each slide) are `aria-hidden`; the visible caption names each button. Toasts are hidden and announced once through the live region. Speaker notes (`<aside class="notes">`) are `display:none` on the stage. |
| Live region | One `#sr-live` (polite, atomic). Announces "Slide 3 of 15: Title." on every slide change, "Step 2 of 3: …" when a `.steps` checklist advances, "Paused…" at a hold, and every toast (speed, theme and so on). Debounced, so scrubbing announces only where you land. The stage itself is **not** a live region, so typing animations aren't read letter by letter. |
| `aria-busy` | `#stage` is `aria-busy="true"` while a slide plays and `"false"` when paused, holding or finished. |
| Slides | Each slide is `role="group"`, `aria-roledescription="slide"`, `aria-label="3 of 15: Title"`. |
| Controls | All dock controls are native buttons with names (their visible text, or `aria-label` for icon-only ones). They're grouped with visible captions ("Slides", "Speed", "Storyboard"). Holds, Auto, Dark and Clean view are `role="switch"` with `aria-checked`. The timeline slider has an `aria-valuetext` ("4.2 of 19.0 seconds"). Text labels hidden on narrow screens stay readable. The Play button's text changes on the click itself. |
| Fake app controls | Buttons, links and inputs inside a recreated screen (`.vp`) do nothing, so they get `tabindex="-1"` and `aria-disabled="true"`. Their text stays readable. |
| Dialogs and Escape | Overview and help are `role="dialog"` with `aria-modal` and a heading. Escape closes them and returns focus to the button that opened them. |
| Never disabled | No control is disabled during animations. |
| Notes window | Real buttons for cue times, labelled editable textboxes, captioned groups, its own polite live region (slide changes, the current cue, holds), and hidden decoration. |
| `step()` | Marks the current checklist item `aria-current="step"`, so "now" and "done" aren't colour-only. |

---

## 3. Your job when building a deck

1. **Hidden-until-revealed content that isn't `.fx`, `.w`, `.mask` or `.callout`** (for example cards with their own reveal class, validation messages, panels that slide in) gets **`data-reveal`**. Reveal it by adding `.in`, `.show`, `.cur` or `.open`, and the engine handles `aria-hidden`. If you use another visibility class, toggle `aria-hidden` yourself in the same cue.
2. **Decoration you add** (particles, flying tokens, stamps that repeat a callout, arrows, glows) gets **`data-decor`** or `aria-hidden="true"`. Elements you create in a builder get the attribute when you create them.
3. **Diagrams and charts:** label the `<svg>` (`role="group"` or `role="img"` plus `aria-label` describing the whole). Give each meaningful part (`<g>` per node or bar) `role="img"` and an `aria-label` in full sentences that includes what is only visual: "Confirmed. Leads to Checked in through CheckIn, and to Cancelled through Cancel." Hide the purely visual parts (edges, arrowheads, axis ticks) with `aria-hidden="true"` *only* if their facts are in a label. **When a builder changes the diagram's state** (highlights an edge, moves a bar), update the labels in the same cue: "…This is the transition that just happened."
4. **State shown only by colour** (a badge turning green, a row dimming, a selected filter) needs the same fact in text. Prefer visible text, as the recreated UI usually already has it ("Checked in"). In a set of options, mark the selected one with `aria-pressed="true"` and update it in the same cue.
5. **Changing text:** when a cue replaces text (a status badge, a counter, a heading), replace the text itself. Don't keep the old version in the DOM but hidden behind CSS.
6. **Typing animations** are fine: the stage isn't live, so they're not read letter by letter.
7. **Keep visible text as text.** No `role="img"` or `aria-label` over readable words, and no text in images.
8. **Anything clickable that you add** to a slide (rare in a demo) is a `<button>`. If it can't be, use `role="button"`, `tabindex="0"`, and activation on Enter and Space.
The StayDesk example shows all of this: `data-reveal` on room cards, question panels, the dropdown and validation messages; `data-decor` on the BOOKED stamps; a labelled state-machine diagram whose labels update as bookings move; `aria-pressed` on the filter buttons; and filtered-out rows hidden with `aria-hidden`. Grep for `data-reveal`, `relabel` and `aria-pressed`.

9. **Escape:** if a slide adds a temporary state of its own (a highlight the presenter can toggle, a selection), Escape clears it.

---

## 4. Testing before you hand it over

Check **each slide, at its start, at each beat and at its end, in both themes**, plus each setting state (Holds on/off, Auto on/off, Dark on/off, Clean view on/off, overview and help open).

1. **`deck.audit()`** in the page returns a list of problems for the current view: invisible text that's still readable, decoration not hidden, controls without names, switches without `aria-checked`, unlabelled SVGs, groups without captions, text read twice, and a missing live region or `aria-busy`. Loop over every slide and several times within it (`deck.at(n, ms)`), and fix everything it reports. A clean deck returns `[]` everywhere. The script in `references/engine.md` §7 does this.
2. **Accessibility tree:** if Playwright is available, take an aria snapshot (`await expect(page.locator('#stage')).toMatchAriaSnapshot()` or `page.locator('body').ariaSnapshot()`) for each view and state. Check that only current information is read, nothing hidden appears, and nothing appears twice. The built-in browser's `read_page` lists hidden and `aria-hidden` nodes too, so it **can't** be used to confirm hiding. Use it only for names and structure, and rely on `deck.audit()` for hiding.
3. **Live region:** after `deck.at(n)`, wait about 400ms, then read `#sr-live`. It should name the slide (and the step, if any).
4. **Notes window:** open `#notes`. Cue times must be buttons, the summary and cues labelled textboxes, and `#nv-live` must announce the slide and the current cue.
5. **Keyboard only:** Tab through the dock. Every control should be reachable and show a focus ring. Escape closes the overview and help and returns focus to the opener.
