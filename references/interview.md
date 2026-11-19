# Interview question bank

Use this to run the "grill me" interview (round 1), the multiple-choice round (round 2) and the spec playback (round 3). Do not ask every question mechanically: drop the ones the source code or the user's request already answers, and say what you inferred instead ("I can see it's a Blazor WASM PWA, so…").

Every question gets a **suggested default** in italics. Defaults are what make the interview fast: the user answers what they care about and says "default" to the rest.

## Contents
- Interview depth (set by flags: `--auto`, `--quick`, `--simple`, `--standard`, `--advanced`)
- Round 1: grill (themes A–F, plus G for advanced)
- Round 2: multiple choice (AskUserQuestion)
- Round 3: spec playback
- Answer shorthand to expect

---

## Interview depth

The depth level decides which questions go into round 1 and whether round 2 runs. It comes from the flag the user passed (see SKILL.md, "Flags"), or from their words ("quick", "just the essentials", "go deep"). The levels nest: each one asks everything the level below it asks, plus more.

| Depth | Round 1 asks | Round 2 (multiple choice) | Typical count |
|---|---|---|---|
| `--auto` | Nothing | No | 0 |
| `--quick` | ★ essentials: **1, 2, 3, 12, 30** | Only if an answer is missing and blocks the build | about 5 |
| `--simple` | quick + **5, 8, 9, 10, 15, 19, 24, 32** | One call at most | about 13 |
| `--standard` *(default)* | All of A–F: **1–33** | Yes | 33 |
| `--advanced` | All of A–G: **1–40** | Yes, plus an outline review before the spec | 40 |

Rules that apply at every depth:
- **Anything not asked takes its default.** List those defaults in the spec under "Assumed defaults" so the user can change them before building.
- **Skip what you can already answer.** If the source or the request answers a question, don't ask it at any depth; state the inference instead.
- **Answers given up front count.** Answers in the original request (e.g. "for a dev meetup, 10 min") answer those questions whatever the depth.
- **The user can change depth mid-interview** ("just use defaults for the rest", "ask me everything"). Follow them.
- **The spec gate never moves.** Every depth, including `--auto`, ends with the spec and waits for "go".

In round 1, say which depth you're using in the opening line ("Quick mode: 5 questions; everything else uses the defaults below"). At standard and advanced depth, also offer the shortcut: *"Answer the ★ ones (or all of them); say 'defaults' for the rest."*

---

## Round 1: grill (one message, grouped, numbered)

Open with one line on what you found in the repo (stack, size, main areas), then the depth line, then only the questions for that depth. **Keep the numbers below even when you skip some** (e.g. quick mode asks 1, 2, 3, 12, 30), so answers like "3: Teams, 12: default" always mean the same thing.

### A. Audience & goal
1. ★ **Who is watching?** Developers, execs, customers, investors, a meetup? *Default: inferred from the request, else developers.*
2. ★ **What should they believe afterwards?** One sentence. This becomes the deck's thesis and the reveal. *Default: "<product> makes <core job> easy."*
3. ★ **Where and how will it be shown?**
   - In a room on a projector or big screen.
   - Shared over Teams or Zoom.
   - A self-running kiosk loop.
   - A link people open on their own.

   This drives the controls preset (see E), whether speaker notes matter (notes are for live talks, and for Teams or Zoom they live in a separate window), and contrast and text size. Projectors wash out subtle colours and pale text, so favour high contrast and larger type there. *Default: live talk shared over Teams or Zoom.*
4. **What does the audience already know?** About the domain (e.g. padel formats, hotel operations) and about the tech (e.g. Blazor, React). Unfamiliar terms get a one-line explanation on screen or in the notes; familiar ones don't. *Default: knows the tech if they're developers; assume no domain knowledge.*
5. **How long?** *Default: 10–15 minutes live, about 2 minutes self-running.* For live talks this also sets the speaker-notes timer (`CONFIG.talkMinutes`).
6. **Tone?** Serious, friendly, or playful? *Default: friendly.*
7. **Should the deck show how it was made** (the interview, the spec, Claude reading the source)? This fits talks *about* AI-assisted work, not product pitches. *Default: no, unless the audience is developers learning about Claude.*

### B. Presenter & event
8. **Presenter and event details for the title and closing slides:** your name, role and company, the talk title, the event name and date, and how people can reach you. *Default: none of these on slides unless given; the title slide shows the product name. Never invent a name, handle or date.*
9. **Call to action:** the exact URL, repo, app-store link or sign-up page for the "try it" and closing slides, and whether to include a QR code (only if one can be embedded as inline SVG, otherwise show the URL in large type). *Default: no link unless you give one. Never invent a URL.*
10. **Do you already have talking points or a script?** Paste key lines, a rough script, or "say X on slide Y". They go into the speaker notes in your words, and Claude only fills the gaps. *Default: Claude drafts the notes from the audience and takeaway; you can edit them in the notes window afterwards.*

### C. The product
11. *(Not asked.)* **Which app:** Claude uses the path from the arguments, or else the current directory, and names it in the first line of round 1 (SKILL.md Step 1). In a multi-project repo it picks the user-facing app. The user corrects it at the spec if needed. The number is kept so the other numbers don't shift.
12. ★ **Which flows should the demo walk through, and in what priority?** Propose 3–5 from the source (pages, routes, key components) and recommend 3. Ask which are **must-have** and which to **cut first** if the talk runs long. *Default: the 3 flows that show the most state change; the last one is cut first.*
13. **Which states or moments matter most?** Validation errors, empty states, success screens, status transitions, live updates, charts. *Default: one "before → after" per flow.*
14. **Sample data:** realistic names and numbers, or the app's own seed data? Anything to keep off screen (internal names, real customers, secrets)? *Default: the app's seed data if it exists, else plausible fictional data. Never real personal data.*
15. **Can real source code appear on slides?** Code cards and "what Claude read" slides put your actual code on screen. That's fine at an internal or developer talk, but riskier for customers or anything shared publicly. *Default: short excerpts for developer audiences at a private talk; no code (describe the behaviour instead) for customers, execs, or a deck that will be shared publicly.*
16. **Product name, logo and brand colours:** take them from the app (CSS variables, logo files, manifest), or something else? *Default: from the app.* If the repo or namespace name differs from the user-facing name (manifest `name`, page titles, an `AppName` constant), say so and ask which one goes on the slides.
17. **Locale and formats:** currency, date and number formats, and the language on screen. Recreated screens must show them exactly as the app renders them, not as Claude's machine would. *Default: the app's configured culture (look for it in the source); ask if there isn't one.*

### D. Deck style
18. **Format:** slide deck with animated slides, a self-playing walkthrough (one continuous fake-cursor tour), or a scroll-driven landing page? *Default: slide deck. The engine is built for it.*
19. **Theme:** light or dark by default, and should viewers get a light/dark toggle? *Default: light by default, toggle on (☀/☾ button and the `T` key; the viewer's choice is remembered).* Accents come from the brand. Very light brand colours (lime, cyan, yellow) get darker light-mode variants so text stays readable.
20. **How should product screens look?** Recreated UI in HTML/CSS inside a fake browser or phone frame, or abstract diagrams? *Default: recreated UI, in a phone frame if the app is mobile-first.*
21. **Animation intensity:** subtle (fades, slides), rich (fake cursor, typing, zooms, confetti), or cinematic (more of everything, longer)? *Default: rich.*
22. **Opening:** straight title, a hook slide, or problem-first ("Here's what hurts today")? *Default: problem-first for customers and execs, a hook for developers.*
23. **Ending:** reveal plus call to action, "try it yourself" steps, Q&A, or contact details? *Default: reveal, CTA (if a link was given in 9), Q&A.*

### E. Controls & display
Present this as a preset with the option to customise. Every control below is already built into the engine and can be switched on or off.

24. **Controls preset:**
    - **Live presenter** *(default for talks)*: holds at beats on, auto-advance off, dock visible with a Clean view toggle (`D`).
    - **Kiosk loop:** holds off, auto-advance on, dock hidden, restarts at the end.
    - **Shareable link:** holds off, auto-advance off, dock visible, storyboard prominent.
25. **Which controls should be visible?** Play/pause, speed (0.25–2×), scrubber with beat ticks, replay, skip-to-end, storyboard strip, overview grid, fullscreen, light/dark toggle, Clean view, speaker-notes window, help. *Default: all.*
26. **Replay behaviour:** explicit only (R, ↺, storyboard click), never on Space. *Default: explicit only. Users dislike accidental restarts.*
27. **Navigation extras:** progress bar, `#slide-N` deep links, touch swipe, keyboard shortcuts overlay. *Default: all on.*
28. **Accessibility:** screen-reader support is always built in and checked (`references/accessibility.md`). Ask only about extras: will anyone present or follow along with a screen reader (then keep speaker-notes cues short and test the notes window with one), the minimum text size for the room, and reduced-motion (shows end states). *Default: screen-reader rules on (not optional), nothing below 16px on the 1600×900 stage (larger for a projector), reduced-motion respected.*

### F. Output, sharing & logistics
29. **Where should the file go?** *Default: `demo/index.html` in the app's repo, one self-contained file, no dependencies, works offline.*
30. ★ **Will the deck be shared after the talk?** (Emailed, posted in a channel, uploaded.) Speaker notes live inside the HTML file: anyone who gets the file can press `N` or view the source and read them. If it will be shared, Claude also produces a **share copy** (e.g. `demo/index.share.html`) with the notes and cues stripped out and the Notes button removed. *Default: yes for live talks, so make the share copy.*
31. **Publish it?** Local file only, or also as a shareable link (if an artifact or publishing tool is available)? Publishing always uses the share copy. *Default: local only.*
32. **Speaker notes?** For live talks, a separate synced notes window (press `N`) shows a short summary per slide plus a cue at every beat, a talk timer and a next-slide preview. The presenter can edit them live, and the audience never sees them as long as only the deck window is shared. Includes your own lines from question 10. *Default: drafted notes for live talks, none for kiosk and shareable-link decks.*
33. **Deadline, and git?** Commit when done? *Default: no commit unless asked.*

### G. Advanced only (`--advanced`)
For users who want to shape the deck in detail. Ask these after A–F are answered, in the same numbering.

34. **Outline review:** before the spec, Claude proposes the deck slide by slide (title, purpose, key moment, rough seconds), and you reorder, cut or add slides. *Default: yes at this depth. This is a separate step, before the spec.*
35. **Per-flow detail:** for each flow, the exact screens and steps to show, the data values to use (names, amounts, dates), and which moment gets a zoom, callout or confetti. *Default: Claude picks the clearest path through the real UI, with one zoom or callout per flow.*
36. **Timing and holds:** target seconds per slide, and where the deck should pause (beats) for you to talk. *Default: 15–30 s per flow slide, with a beat before each major moment.*
37. **Visual details:** device frame (generic phone, iPhone-style or Android-style, browser chrome), background style (soft blobs, grid, plain), icon style, and the brand font (falls back to system fonts offline). *Default: generic frames, soft blobs, inline SVG icons matching the app, system fonts.*
38. **Engine settings:** speed steps, auto-advance delay, whether the dock starts visible or in Clean view, and whether the storyboard is shown. *Default: the controls preset's values.*
39. **Speaker-notes style:** a full script to read, bullet prompts, or a few keywords. How many cues per slide, and whether to include timing hints ("you should be at 4:00 here"). *Default: 1–3 sentence summaries, one short cue per beat, no timing hints.*
40. **Verification depth:** which browsers to check, whether to test a projector or Teams share, and whether to test the notes window from a local file. *Default: one Chromium browser, both themes, notes window synced.*

---

## Round 2: multiple choice

After the user answers round 1, collect what is still open or contested and ask it with **AskUserQuestion**: at most 4 questions per call, 2–4 options each, recommended option first with "(Recommended)" in the label. Each question must stand on its own, so name the thing being decided.

Good candidates:
- **Flows** (multiSelect): the flows you found in the source, each described in one line. Follow up on which to cut first if that wasn't answered.
- **Where it's shown:** Teams/Zoom share / projector in a room / kiosk / link.
- **Shared afterwards:** make a notes-free share copy / it stays with me.
- **Code on slides:** short excerpts / no code, describe the behaviour.
- **Controls preset:** Live presenter / Kiosk loop / Shareable link.
- **Theme:** light default with toggle / dark default with toggle / fixed (no toggle).
- **Opening:** straight title / hook / problem-first.
- **Deck shape:** one deck versus talk deck plus separate product demo.

If the user overrides a recommendation, take it without argument. If the deck shows its own making, that override is a nice beat to include.

---

## Round 3: spec playback

Play everything back as one compact spec and end with exactly one question: **"Any changes, or should I start building?"** Do not build until the user says go (or an equivalent). At every depth below standard, and for any question skipped at standard, add a last line listing the defaults you assumed. At `--advanced`, the spec comes after the outline review (34) and includes the per-flow detail and timing.

```
**Brand:** <name>, <palette>, <logo>; <light|dark> by default, toggle <on|off>
**Audience / thesis:** <who> (knows <domain/tech>) should leave believing "<sentence>"
**Shown:** <Teams/Zoom share | projector | kiosk | link>, <length> min
**Presenter / event:** <name, role · talk title · event, date | none on slides>
**Call to action:** <exact URL / repo / QR | none>
**Deck:** <N> slides, single self-contained HTML at <path>, offline, zero dependencies
**Story:** <opening> → <product intro> → <flows> → <reveal> → <ending>
**Flows:** 1. <flow> (<key moments>) · 2. … · 3. … — cut first if long: <flow>
**Screens:** recreated UI in a <browser|phone> frame, fake cursor, <intensity> animation; formats: <locale>
**Code on slides:** <short excerpts | none>
**Controls:** <preset>; <visible controls>; replay explicit only
**Speaker notes:** <drafted + your lines, summary + beat cues in a synced window | none>; timer target <N> min
**Sharing:** <share copy without notes at <path> | not shared>
**Build order:** read source → write deck → verify every slide in a browser → report
**Depth:** <auto | quick | simple | standard | advanced>
**Assumed defaults:** <every unasked question's default, one line each>; say a number to change it
```

---

## Answer shorthand to expect

Users answer tersely: "default", "d", "3: execs", "skip E", "same as you said", "quick", "★ only", or they paste the questions back with answers inline. Map each answer back to its question number. For anything ambiguous, choose the sensible reading, record it in the spec, and let the user correct it at the spec step instead of asking again.
