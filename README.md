# create-presentation

A Claude Code skill that interviews you ("grill me" style) and then builds an **animated, self-contained HTML product demo deck from your app's source code**. The deck recreates your real screens, labels, validation messages and states, animates them with a fake cursor, typing, zooms and state changes, and adds presenter controls.

- One HTML file, zero dependencies, works offline
- Light/dark themes (light by default) with a toggle
- Play/pause, 0.25–2× speed, scrubber with beat holds, replay, storyboard strip, overview, fullscreen
- Browser frame or phone frame (for mobile-first apps)
- Speaker notes in a separate, synced window, for presenting over Teams or Zoom

## Install

**For all your projects** (personal skill):

```bash
git clone https://github.com/brentvee/create-presentation-skill ~/.claude/skills/create-presentation
```

On Windows, clone into `%USERPROFILE%\.claude\skills\create-presentation`.

**For one project** (shared with everyone who clones that project):

```bash
git clone https://github.com/brentvee/create-presentation-skill .claude/skills/create-presentation
```

Restart Claude Code (or start a new session) and the skill is available.

## Use

Run it from inside your app's folder. It presents the app in the **current directory**:

```
cd my-app
/create-presentation
/create-presentation --quick
/create-presentation demo for a dev meetup, ~10 min
/create-presentation --help
```

It says which folder it's using in its first line, so you can catch a mistake. In a repo with several projects it picks the user-facing app. If there's no app in the folder, it stops and tells you instead of guessing.

To present an app somewhere else, pass its path: `/create-presentation --quick ../other-app`.

It also triggers on requests like "make a demo deck of this app".

### Flags: how many questions you get

| Flag | Questions | Use it when |
|---|---|---|
| `--auto` | none | You want Claude's best guess and will fix it at the spec |
| `--quick` | about 5 ★ essentials | You know what you want and are short on time |
| `--simple` | about 13 | The essentials plus presenter, call to action, theme, notes |
| `--standard` *(default)* | 33 + multiple choice | The full interview |
| `--advanced` | 40 + outline review | You want to shape every slide, timing and visual detail |

Aliases: `-q`/`--fast`, `--basic`, `--full`/`--deep`, `--defaults`/`--yolo`. Typing "quick" or "ask me everything" works too. **Every level ends with the spec, and nothing is built until you say "go".** Anything not asked uses a sensible default, and the spec lists those defaults so you can change any of them.

1. **Survey.** Claude reads your app: routes, pages, models, validation, brand and seed data.
2. **Grill.** Numbered questions grouped by theme (audience, presenter and event, product, style, controls, sharing), each with a default. How many depends on the flag above. Answer in shorthand, e.g. `3: Teams. 12: your pick, cut the last one. rest default`.
3. **Multiple choice.** Open decisions come back as clickable options.
4. **Spec.** Claude plays back the plan. Nothing is built until you say **go**.
5. **Build and verify.** Claude writes the deck, opens it in a browser and checks every slide in both themes.

### Presenting

| Key | Action |
|---|---|
| `→` / `←` | Next beat or slide / previous slide |
| `Space` | Pause / play (continue from a beat) |
| `+` `−` `0` | Speed / reset |
| `R` / `E` | Replay slide / skip to end |
| `T` | Light / dark |
| `N` | Open speaker notes in a separate window |
| `D` | Hide the dock (clean view) |
| `F` / `O` / `?` | Fullscreen / overview / help |

### Speaker notes (Teams / Zoom)

Press `N` in the deck to open the speaker view in its own window. It shows:
- the talking points for the current slide;
- a cue for each moment, highlighted as the slide plays;
- a talk timer;
- a preview of the next slide;
- big Prev / Play / Next buttons.

The two windows stay in sync, and either one can drive the deck. **Share only the deck window** in Teams or Zoom, not your whole screen, and keep the notes window on another monitor or beside it.

You can edit notes in the notes window while you rehearse. **Copy edited notes** gives you a message to paste back to Claude to make the edits permanent.

## What's inside

| Path | Purpose |
|---|---|
| `SKILL.md` | The workflow Claude follows |
| `references/interview.md` | Question bank, multiple-choice round, spec template |
| `references/engine.md` | Engine API, helpers, slide patterns, rules, verification script |
| `assets/deck-engine.html` | The presentation runtime template |
| `assets/example-staydesk-deck.html` | A complete worked example deck (fictional hotel booking app) |
