<div align="center">

# MonoFlash

**A zero-build, single-file spaced-repetition study app — with real LaTeX rendering, a from-scratch SM-2 engine, and a design system that's just black.**

![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla%20ES5-F7DF1E?logo=javascript&logoColor=black)
![No Framework](https://img.shields.io/badge/framework-none-success)
![Single File](https://img.shields.io/badge/build%20step-none-blue)
![Dependencies](https://img.shields.io/badge/runtime%20deps-1%20(KaTeX)-informational)
![Lines](https://img.shields.io/badge/lines%20of%20code-3%2C600%2B-orange)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

[Features](#-features) · [The SRS Engine](#-the-spaced-repetition-engine) · [Architecture](#%EF%B8%8F-architecture) · [Hard Problems Solved](#-hard-problems-i-actually-had-to-solve) · [Getting Started](#-getting-started) · [Import Format](#-content-import-format) · [About](#-about-the-developer)

</div>

---

## What is this

MonoFlash is a flashcard / MCQ / short-answer study app I built for myself, because every existing SRS tool I tried either couldn't render math cleanly, looked like a spreadsheet, or wanted me to pay a subscription to study my own notes.

It's one HTML file. No React, no build pipeline, no `npm install`. Open it in a browser and it runs — locally, offline, with your data in `localStorage` and an optional cloud sync layer I wrote on top of Cloudflare Workers so I can study on my phone and pick up on my laptop without losing state.

I built the whole thing — data model, spaced-repetition algorithm, sync backend, animations, the works — while simultaneously prepping for my own HSC-level college admission exam, which is either a good argument for how useful spaced repetition is, or a great argument for how bad my time management is. Probably both.

---

## ✨ Features

**Three card types, one system**
| Type | What it does |
|---|---|
| **Flashcard** | Classic front/back. Tap or press `W` to flip. |
| **MCQ** | Four options, shuffled per-render, keyboard-selectable with `1`–`4`. |
| **SAQ** | Question shown, answer hidden behind a "tap to reveal" pill with a smooth height/opacity transition — forces actual recall before you get the answer, instead of your eyes jumping straight to it. |

**Real math, not a screenshot of math**
Every card surface — front, back, MCQ question, MCQ options, SAQ question and answer — runs through [KaTeX](https://katex.org/), so `$\dfrac{-b\pm\sqrt{b^2-4ac}}{2a}$` renders as an actual equation, not a wall of backslashes. Supports `$...$`, `$$...$$`, `\(...\)`, and `\[...\]`.

**Spaced repetition that's actually spaced repetition**
A real SM-2-family algorithm (the same lineage Anki uses) tracks interval, easiness, and repetition count per card, per device. See [below](#-the-spaced-repetition-engine) for the actual math.

**Study-set selector that doesn't waste your time**
- **SRS** — the default. A priority queue: never-studied cards first, then your weakest cards worst-first, then whatever's due. No manual mode-picking required.
- **All** — every card in the lesson, no filtering, when you want a full deliberate pass.
- **S1 / S2 / ...** — fixed 20-card chunks for predictable, bounded study sessions.
- **Bookmarks** — cards you've manually starred.

**Cloud sync that doesn't ask you to remember to sync**
Every mutation — new card, edited card, bookmark toggle, even individual SRS updates as you study — schedules a debounced push to a Cloudflare Worker. Rapid changes coalesce into one request after ~2.5s of quiet instead of hammering the network; backgrounding the tab force-flushes immediately so nothing's lost if you close the app mid-session.

**Built for how I actually study**
- Full keyboard control: `W`/`A`/`S`/`D` (or arrow keys) to flip/grade/advance, `1`–`4` for MCQ options, `B` to bookmark, `/` to go back, `Shift+,`/`Shift+.` to jump between sets — no mouse required once you're in a groove.
- GitHub-style contribution calendar + streak tracking, computed against a configurable daily card-quota.
- A weak-card detector that isn't just "you got it wrong once" — it weighs lifetime accuracy, recent miss count, and whether you've ever actually nailed it, into a single score that drives the SRS queue.
- Bangla font fallback (`Hind Siliguri`) baked into card and option rendering, because half my content is in Bangla and English fonts render Bangla glyphs like garbage.
- A compact plain-text bulk-import format (see [below](#-content-import-format)) so I can paste 50 cards in ten seconds instead of clicking "add card" fifty times.

---

## 🧠 The Spaced Repetition Engine

Every card tracks `interval`, `easiness`, `repetitions`, `correctCount`, `wrongCount`, and `nextReview`. On each answer:

```js
function updateSRS(s, q) {
    s.lastReviewed = Date.now();
    if (q === 0) {
        // wrong answer — reset the ladder, but don't nuke easiness as hard as SM-2 default
        s.repetitions = 0;
        s.interval = 1;
    } else {
        s.repetitions++;
        s.interval = s.repetitions === 1 ? 1
                    : s.repetitions === 2 ? 6
                    : Math.round(s.interval * s.easiness);
        s.easiness = Math.max(1.3, s.easiness + 0.1 - (1 - q) * 0.2);
    }
    s.nextReview = Date.now() + s.interval * 86400000;
    return s;
}
```

That's the standard SM-2 interval ladder (1 day → 6 days → previous interval × easiness factor, floored at 1.3). On top of it I layered a custom **weakness score** that the SRS queue actually sorts by — not just "is this due," but "how shaky is this, really":

```js
function weaknessScore(s) {
    var acc = correctCount / totalAttempts;
    return (1 - acc) * 10 + wrongCount + (repetitions === 0 ? 5 : 0);
}
```

So a card you've bombed three times in a row outranks a card that's merely "due" — the queue front-loads what you actually don't know, not just what the calendar says to review.

---

## 🏗️ Architecture

```
index.html   ← everything. HTML, CSS, JS, all of it. ~3,600 lines, ~220KB.
```

- **Frontend:** Vanilla JavaScript, written in an ES5-compatible style on purpose — no transpiler, no bundler, no `node_modules`. It runs the same way it'll run in ten years.
- **Styling:** Hand-written CSS with custom properties, a strict pure-black (`#000`/`#0a0a0a`/`#121212`) monochrome palette, and zero UI framework.
- **Math rendering:** [KaTeX](https://katex.org/) loaded from CDN — the one external dependency in the whole project.
- **Persistence:** `localStorage` first (the app is fully usable offline), with an opt-in sync layer against a Cloudflare Worker, keyed by an auto-generated per-device ID.
- **No backend code in this repo** — the sync worker is a separate ~30-line Cloudflare Worker that does exactly one thing: store and return a JSON blob per device ID. It's intentionally dumb; all the logic lives client-side.

---

## 🔧 Hard problems I actually had to solve

Anyone can flip a card with CSS. These are the ones that took actual debugging, not tutorial-following:

**KaTeX inside a flex container silently breaks text flow.**
Any element that's a *direct child* of a `display:flex` container becomes its own flex item — full stop, regardless of whether it's `inline` or `inline-block`. So the moment KaTeX injected a `<span class="katex">` next to plain text inside a flex-centered card face, the browser split "text before," "the equation," and "text after" into three separate flex items laid out in a *row* instead of flowing as one paragraph. Question marks and trailing text would float off to one side, vertically centered independently. Fix: wrap all card content in a plain block child, and let the flex container center *that single wrapper* instead of raw mixed content.

**3D CSS flips make text blurry, and it's not a bug you can nudge away.**
The classic `rotateY(180deg)` + `backface-visibility:hidden` flip forces the browser to rasterize that face onto a GPU compositing layer, and text on that layer gets resampled — permanently soft at rest, worst on KaTeX's thin fraction bars and radicals. `translateZ` hacks don't fix it; the 3D transform itself is the cause. Replaced it with a 2D squash-swap: scale the card to near-zero on one axis, swap which face is `display:flex` at the midpoint, scale back — same "flip" read, but the visible face always sits at `transform: none` at rest, so there's zero blur source. Cleanup is driven by `transitionend`, not a padded `setTimeout`, so there's no lingering soft-text window after the animation visually lands.

**Copy/paste of rendered math returns garbage by default.**
KaTeX renders each glyph as its own positioned `<span>` purely for layout — select and copy that DOM and you get scrambled fragments, not the LaTeX you typed. Fixed by stashing the original raw source in a `data-raw` attribute *before* KaTeX touches the DOM, then intercepting the `copy` event: if the selection lands inside a tagged element, the clipboard gets handed the exact original string instead of whatever the browser scraped from the rendered spans.

**Unicode subject names silently collided in the sidebar.**
`sub-list` DOM IDs were generated by sanitizing subject names down to `a-z0-9_-` only. Any Bangla (or other non-Latin) subject name collapsed to an identical or empty ID — meaning two different subjects could share a sidebar section, breaking expand/collapse state. Fixed by matching on an exact `data-subject` attribute instead of round-tripping through a sanitized ID.

**MCQ correctness checking broke — but only on cards with math in them.**
The "which button is correct" logic re-scanned live DOM `textContent` after render. Fine for plain text; broken for LaTeX options, because KaTeX rewrites that span's internal DOM (visible glyphs + a MathML mirror), changing what `textContent` reports. Fixed by tagging each option with its raw value in a `data-opt-value` attribute *before* rendering, so correctness checking never depends on what the DOM looks like after KaTeX runs.

---

## 🚀 Getting Started

There's no build step. That's the whole pitch.

```bash
git clone https://github.com/<your-username>/monoflash.git
cd monoflash
open index.html   # or just double-click it
```

Want cloud sync across devices? Point `WORKER_URL` in `index.html` at your own Cloudflare Worker (a minimal KV-backed one is all it needs — store a JSON blob per `deviceId` query param, return it on GET). Without it, the app runs fully offline with zero changes.

---

## 📥 Content Import Format

Paste this straight into a lesson's Data tab. Format is `Question -!- Answer`, one card per line, with `##Lesson Name` headers to route into multiple lessons at once.

```
##Physics — Electrostatics
State Coulomb's law. -!- $F = k\dfrac{q_1 q_2}{r^2}$
What is electric flux? -!- The measure of electric field lines passing through a surface

##Math — Quadratics
What is the solution to $ax^2+bx+c=0$? -!- $x=\dfrac{-b\pm\sqrt{b^2-4ac}}{2a}$
```

MCQ lines use six segments instead of two:
```
Which converges? -!- $\sum 1/n$ -!- $\sum 1/n^2$ -!- $\sum n$ -!- $\sum 2^n$ -!- $\sum 1/n^2$
```
*(Question -!- Option 1 -!- Option 2 -!- Option 3 -!- Option 4 -!- Correct Answer)*

SAQ lessons use the exact same two-segment shape as flashcards — the app knows to hide the answer behind a tap because it checks the *lesson's* type, not the line content.

---

## ⌨️ Keyboard Shortcuts

| Key | Action |
|---|---|
| `W` / `↑` / `Space` / `Enter` | Flip flashcard / reveal SAQ answer |
| `A` / `←` | Grade: wrong |
| `D` / `→` | Grade: got it |
| `S` / `↓` | Skip / next |
| `1`–`4` | Select MCQ option |
| `B` | Toggle bookmark |
| `/` | Previous card / toggle sidebar |
| `Shift` + `,` / `.` | Jump to previous / next set |
| `Esc` | Back to lessons |

---

## 🗺️ Roadmap

Things I'd build next if I had infinite time and zero exams:
- Per-lesson analytics (accuracy trend over time, not just current-state)
- Export to Anki's `.apkg` format for people who want to jump ship
- A proper offline-first conflict resolution strategy for sync (currently last-write-wins, which is fine for one person on two devices, not fine for anything more ambitious)

---

## 🙋 About the developer

I'm Wasif Akand, 17, based in Dhaka, Bangladesh. I built MonoFlash from scratch — no template, no starter kit — while prepping for admission to Notre Dame College, Dhaka, because I needed a study tool that actually fit how I study and didn't exist yet.

I like building things end-to-end: front-end, algorithms, backend, the design system, all of it. This project is the clearest single artifact of that — a spaced-repetition engine, a sync backend, real math rendering, and a handful of genuinely gnarly browser-rendering bugs, all solved from a blank HTML file.


---

## 📄 License

MIT — do whatever you want with it. If you build something cool on top of it, I'd like to hear about it.
