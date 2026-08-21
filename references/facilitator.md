# Facilitator edition and guide

**The facilitator edition IS the course.** It is the classroom build with the
teaching script injected as a sticky rail beside every page, so the
facilitator teaches from the exact pages the room is looking at. It is never
a separate runbook document, a standalone script page, or a re-flowed
"instructor packet" site. This rule was violated once (Leading with AI's
first draft shipped a standalone runbook) and the correction cost a full
rebuild; do not repeat it. Reference implementations:

- Horizontal-deck courses: `courses/emotional-intelligence/facilitator/` in
  `me5231979/Course_Library`.
- Book-mode (pager) courses: `facilitator/` in `me5231979/AI-Managers`
  (Leading with AI), including its `tools/build-facilitator.py`, which is the
  template generator to copy for new book-mode courses.

The facilitator edition is generated, never hand-edited. Regenerate after
every change to the course or to `notes.json`:

```
python3 tools/build.py              # the learner editions first
node    tools/shoot-slides.mjs      # page images for the printable guide
python3 tools/build-facilitator.py  # facilitator/index.html + guide.html
```

## notes.json schema

```json
{
  "meta": {
    "program": "Course Name · Vanderbilt Learning Series",
    "totalMinutes": 90,
    "audience": "…", "roomSetup": "…", "postSession": "…",
    "paths": { "full": 90, "core": 60 },
    "framework": "…",
    "objectives": ["…"],
    "materials": ["…"],
    "prep": { "weekBefore": ["…"], "dayBefore": ["…"], "thirtyMinBefore": ["…"] },
    "contingencies": [{ "if": "…", "then": "…" }],
    "toughQuestions": [{ "q": "…", "a": "…" }],
    "templates": [{ "label": "Pre-work invite", "body": "Subject: …" }]
  },
  "sections": [
    {
      "id": "clarity",               // MUST match a section/page id in the course
      "title": "Module 02: clear communication",
      "minutes": 7, "coreMinutes": 5,
      "coreNote": "Core: load scenario 01 only; protect the live rewrite.",
      "purpose": "Why this section exists, one sentence.",
      "say": "Verbatim-usable facilitator script for the page.",
      "facilitate": ["Do steps, one action each."],
      "ask": "Question to the room, with the expected answers named.",
      "debrief": "Optional: the synthesis line after the activity.",
      "validate": "How you know the learning landed (the L2 check).",
      "watchFor": "Common failure mode and the recovery move.",
      "transition": "One line that lands the section and points to the next."
    }
  ]
}
```

Rules that must hold (the generator refuses to build otherwise):

- **Timing sums exactly.** Sum of `minutes` = the Full path; sum of
  `coreMinutes` = the Core path; `totalMinutes` agrees with the sections.
  The advertised lengths appear on the library card, the syllabus, and every
  rail head, so a drift here lies to three surfaces at once.
- Every section `id` exists in the course (a slide id on deck courses, a
  section id from the pager `PLAN` on book-mode courses).
- Say / Do (facilitate) / Ask / Debrief (validate) / Transition is the ATD
  structure. Write `say` so a nervous first-time facilitator could read it
  aloud verbatim. `debrief` is optional per section; `validate` and
  `watchFor` are not.
- Name the never-cut activity. Every course has one section whose activity
  carries the Level 3 measure (Leading with AI: the Module 05 commitment
  round). Its `coreNote` never cuts it, and the briefing's rule-strip names
  it.
- No em dashes in any string, anywhere in the JSON.

## What the generator injects

`tools/build-facilitator.py` reads the **classroom** build and `notes.json`
and writes `facilitator/index.html`. It must produce all of:

1. **Identity.** Title `(facilitator)`, `<meta name="robots" content="noindex">`,
   `body` gains class `fac`, and the nav lockup gains a gold
   `.fac__badge` reading "Facilitator Edition" (hidden under 700px).
2. **The briefing page**, prepended before the learner welcome, id
   `briefing` / `s-briefing`: you-are-here line ("You are here: the
   facilitator prep page. Learners start on the next page."), eyebrow
   "Run this course / 00 · Facilitator only", headline
   `Before the room <em>fills</em>.`, a lead naming both paths and stating
   that learners never see this edition, then the `brief__grid` boxes:
   `01 · A week before / 02 · The day before / 03 · 30 minutes before`
   (gold ✓ check lists), `04 · Materials`, `05 · When things go sideways`
   (contingencies as If/Then rows), `06 · Tough questions, ready answers`
   (`<details>` per question), `07 · Copy-paste templates`. Buttons: print
   the facilitator guide, open the learner edition. The pre-work invite sits
   in the briefing's own rail inside a bordered `.facnote__invite` block.
3. **One `.facnote` rail per page**, injected inside each section, carrying:
   head (`Facilitator script` · `Full X min · Core Y` or `Core skip` · Hide
   button), `purpose`, then tagged blocks in order: `t-say` Say, `t-do` Do,
   `t-ask` Ask, `t-deb` Debrief (when present), **Validate:** and
   **Watch for:** as `.facnote__meta` lines, `t-tra` Transition, and the
   Core cut last, above a dashed rule (`.facnote__core`). Say / Ask /
   Transition render in the serif italic `.facnote__script`, wrapped in
   curly quotes: they are lines to speak.
4. **Hide / Show.** The Hide button adds `fac-hide` to `body` and moves
   focus to a fixed `.facshow` pill ("Show facilitator script"); the pill
   restores it. Persist the choice in localStorage under the course's own
   key prefix (e.g. `leading-facScriptHidden`) so it survives page turns and
   reloads: a facilitator who hides the rail to project a clean page does
   not want it back on every turn.
5. **The QR repoint.** The welcome QR encodes the **learner** classroom URL
   (`data-url` on the QR container), never the facilitator URL. A room that
   scans it lands on its own edition. Test this; it is the easiest thing to
   ship wrong.

## Layout: deck courses vs book-mode courses

The rail is the same; only the grid host differs.

**Horizontal deck** (legacy courses): at ≥1200px,
`body.fac .slide { display:grid; grid-template-columns:minmax(0,1fr) 350px; }`
with the rail sticky in column 2 and the dot rail hidden. Keep
`overflow-y:auto; overflow-x:hidden` on facilitator slides (never
`visible`).

**Book mode** (Working Smarter pattern, all new courses): at ≥1200px,

```css
body.fac .page > section{ display:grid; grid-template-columns:minmax(0,1fr) 360px; align-items:start; }
body.fac .page > section > .wrap{ grid-column:1; }
body.fac .facnote{ grid-column:2; grid-row:1; position:sticky; top:16px; }
body.fac.fac-hide .page > section{ grid-template-columns:1fr; }
```

plus these book-mode-specific splices, all shown working in the AI-Managers
generator:

- The briefing joins the pager: add it to `PLAN` (first entry), `TRACKED`
  (`['briefing','00','Facilitator briefing']`), `PROG_SECTIONS`, and
  `AUTO_ON_LEAVE`, so it is page 00 with its own dot and completion.
- Move the "Scroll for more" pill (`#pgMore`) left of the rail column when
  the rail is shown, or the two overlap.
- Below 1200px the rail drops under the page content with the course's
  standard side padding, not sticky.

**Rail surface and contrast.** The rail is its own fixed dark surface
(`#141414`, gold border, gold left rule) whatever section theme it sits
beside, so its pairs are constant and measurable: white purpose text,
`#D6D6D6` body, `#BDBDBD` meta, gold head. Tag chips: `t-say` gold,
`t-do` sky, `t-ask` sage, `t-deb` highlight (all with `#1C1C1C` text),
`t-tra` `#4F4F4F` with white text. Add these pairs to the course's
contrast gate, exempt `.facnote` / `.brief__*` / `.facshow` /
`.fac__badge` from the raw-token component check (they are permanently
dark, like the book bar), and extend the axe walk to every facilitator
page plus the guide. Extend the interaction tests to cover: exactly one
rail per page, the Hide/Show cycle with focus handling and persistence,
the QR target, and that **neither learner edition carries any facilitator
content** (`.facnote` count 0, no briefing).

## Printable guide (`guide.html`)

Generated by the same script, from the same notes. Front matter (paths,
audience, room, framework, post-session, the three prep lists, materials),
then the run-of-show table with Full and Core columns totalled in the
footer, the contingency and tough-question banks, the templates, and then
one block per section: **page image left (~290px), notes right**, the notes
carrying the same Say / Do / Ask / Validate / Watch for / Transition in
print-friendly type, with the Core cut last. Light-ground page: Oak
`#946E24` accents on white (gold does not survive a white ground), letter
`@page`, `break-inside:avoid` per section, a screen-only Print button.

Images live at `facilitator/img/<section-id>.jpg`, captured by
`tools/shoot-slides.mjs` with Playwright: shoot the **classroom** edition
(so the guide pictures the page without the rail over it), viewport
1440x900, `reducedMotion`, hide the book bar and `#pgMore`, jpeg quality
~68, one shot per pager section. Re-shoot after any visual change to the
course. A missing image degrades via an `onerror` class; do not ship
missing images anyway.

Escape trap: the ✓ comes from CSS `content: "\2713"`. In a non-raw Python
string that reads as an octal escape and renders garbage; write it
`"\\2713"` inside the generator.

## Learning model

- Objectives on the hero use Bloom's verbs (identify, apply, reframe,
  build), five per course, echoed on the syllabus and in `meta.objectives`.
- Kirkpatrick L1: fist-to-five close, counted and recorded. L2: the scored
  knowledge check and each section's `validate`. L3: the committed practice
  (habit, commitment card) plus the 7-day pulse in `postSession`; the count
  of completed commitments is the program metric. The recap section's
  `validate` should tell the facilitator to record the L1 number and the L3
  baseline count.
