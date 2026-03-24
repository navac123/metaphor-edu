# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Deployment

```bash
cp metaphor-edu.html index.html
git add index.html metaphor-edu.html
git commit -m "..."
git push origin main
```

GitHub Pages deploys automatically via `.github/workflows/pages.yml` on push to `main`.
Live URL: `https://navac123.github.io/metaphor-edu/`

## Architecture

Two standalone single-file HTML apps — no build step, no dependencies, no frameworks.

### metaphor-edu.html (= index.html)
The student-facing app. `index.html` is always a verbatim copy of `metaphor-edu.html`.

**8-step flow:** רקע תיאורטי → בחירת המטפורה וחשיפתה → בניית פרומפט → תמונה ראשונית → המטפורה בעידן ה-AI → משוב ושיפור → רפלקציה → סיכום

**State:**
```js
let cs=1, ts=8, id={image1:null, image2:null, imageAI:null, ex:[], articles:[]};
const sn=['רקע תיאורטי','בחירת המטפורה וחשיפתה','בניית פרומפט','תמונה ראשונית','המטפורה בעידן ה-AI','משוב ושיפור','רפלקציה','סיכום'];
```

**Navigation:** `go(n)` switches steps, calls `sf(n)`. `sf(8)` renders the summary. `printSummary()` calls `sf(8)` then `window.print()`.

**Data flow:** `gf(n)` collects all textarea/input values from step n by element ID. `cd()` calls `gf(1..8)` + adds images + mappings. `rd(d)` restores from a data object. `saveL()` persists to localStorage key `edu_meta_data`.

**Topic system:** 5 topics with data objects:
- `topicSummaries` — 4-paragraph narrative + central article (gold box)
- `topicQuizData` — 10 questions per topic (3 require reading the article), max 3 attempts (`edu_meta_tqa_0..4`), pass threshold 7/10
- `topicAIChallenge` — unique provocation question per topic for step 5
- `topicExamples` — example metaphors per topic
- `topicArticles` — supplementary articles shown in library modal only

**Step 5 (AI era):** Fields `aiWhatChanges`, `aiWhatRemains`, `aiPrompt`, `aiComparison`, image stored in `id.imageAI`. Challenge question rendered in `#aiChallengeQ`, refreshed by `showTopicArticles()` and `sf(5)`.

**Library modal:** `showLibraryModal()` / `closeLibraryModal()` — global button, renders `topicArticles`.

**Exports:** HTML (`exH()`), PDF (`printSummary()`), JSON backup (`exB()`), JSON import (`imB()`).

**Analytics:** GoatCounter at `https://metaphor-edu.goatcounter.com/count` via `track(evt)`.

**Palette:** `--p:#2e6b5e --pl:#3d8b7a --a:#c9a84c --bg:#f5f7f4`

### grader.html
Professor-facing grading tool. Not deployed — opened locally in browser.

**Workflow:** Load student JSON files → grade per rubric → export CSV.

**Rubric:** ניתוח מטפורה (30) + תמונה ותהליך (20) + עידן AI (20) + רפלקציה (20) + שלמות אוטומטית (10) = 100.

**Completeness (criterion 5)** is auto-calculated from `computeCompleteness(data)` — counts filled text fields, no manual input.

**Persistence:** Grades saved to localStorage key `metaphor_grades` as `{studentId: grade}`. Student ID = `studentName + '|||' + topicSelect`.

**CSV export:** UTF-8 BOM included for Hebrew Excel compatibility.

## Key conventions

- **Edit tool fails on Hebrew strings** — use Python scripts with `str.replace()` for any complex edits touching Hebrew text.
- `metaphor-edu.html` is the source of truth. Always `cp metaphor-edu.html index.html` before committing.
- `grader.html` is local only — never pushed to GitHub Pages (only the root files are served).
- Color palette is fixed across the app — do not introduce new colors.
