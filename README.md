# General English Academy

A static, dependency-free English learning web app. It is one `index.html` (all CSS and JavaScript inline, vanilla JS, no CDN, no build step) that loads course content from JSON files in `data/`. This repository ships the complete **A1 (Beginner)** course: 4 units, 12 lessons, 4 unit quizzes and 144 questions, all in American English.

Features: hash-router single-page app, spoken audio through the browser's Web Speech API, four question types (multiple choice, true/false, fill-in-the-blank, dictation), immediate feedback in lessons, scored unit quizzes with review, progress saved in the browser, light/dark/auto theme, progress export/import/reset, keyboard-accessible and mobile-first layout. Levels are never locked.

## Curriculum (A1)

| Unit | Title | Lesson 1 (Vocabulary & Grammar) | Lesson 2 (Listening & Speaking) | Lesson 3 (Reading & Writing) |
|---|---|---|---|---|
| 1 | Meeting people | Hello! I am... (the verb *be*, greetings, countries) | Nice to meet you (introductions, thirteen/thirty) | About me (profile, numbers, the alphabet and spelling) |
| 2 | My day and my family | My family and my routine (present simple) | What time is it? (telling time, the *-s* ending) | A day with my family (a/an, plurals, possessives) |
| 3 | Food, shopping and money | Food and drink (countable/uncountable, some/any) | At the diner (ordering, prices, this/these sounds) | Shopping for the week (this/that, like/want, prices) |
| 4 | Around town | Places in my town (there is/are, prepositions) | Excuse me, where is...? (directions, can/can't) | My neighborhood (imperatives, present continuous intro) |

Every unit ends with an 8-question quiz (pass mark 70%) that mixes all four question types. The A2, B1, B2 and C1 slots exist in the app but need their own `data/level-<id>.json` files; until then those levels show "Content not installed" and nothing else breaks.

## File structure

```
index.html            the whole app (HTML + CSS + JS)
data/level-a1.json    A1 course content (included)
data/level-a2.json    optional, same schema (a2, b1, b2, c1 are the other fixed names)
README.md
TESTING.md            exactly what was and was not tested
```

## Run locally

Browsers block `fetch()` on `file://` pages, so serve the folder over HTTP:

```bash
cd general-english-academy
python3 -m http.server 8000
# open http://localhost:8000/
```

## Deploy on GitHub Pages

Repository: <https://github.com/ahmedzaid2004zaid-ops/general-english-academy>

1. Put `index.html`, `README.md`, `TESTING.md` and the `data/` folder (containing `level-a1.json`) in the repository root on the `main` branch. Commit and push.
2. On GitHub open the repository, then **Settings**, then **Pages** (under "Code and automation").
3. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
4. Choose branch **main** and folder **/ (root)**, then click **Save**.
5. Wait one to two minutes. The site appears at `https://ahmedzaid2004zaid-ops.github.io/general-english-academy/`. Refresh the Pages settings screen to see the exact URL once the deployment is green.
6. Open the URL and check that the A1 card shows 0% and the lessons open. (This deployment has not been done by the author of this file; see TESTING.md.)

## Deploy on Netlify or Cloudflare Pages

- Netlify CLI: `npx netlify-cli deploy --dir=. --prod` (or drag the folder onto <https://app.netlify.com/drop>).
- Cloudflare Pages CLI: `npx wrangler pages deploy . --project-name general-english-academy`

No build command is needed. The publish/output directory is the repository root.

## Data format reference (`data/level-<id>.json`)

```
Level    { id:"a1", code:"A1", title, tagline, description, outcomes:[5-6 strings], units:[4 Unit] }
Unit     { id:"a1-u1", number, title, summary, objectives:[3-4], lessons:[3 Lesson],
           quiz:{ id:"a1-u1-quiz", title, passScore:70, questions:[8 Question] } }
Lesson   { id:"a1-u1-l1", number, title, focus:"Vocabulary & Grammar"|"Listening & Speaking"|"Reading & Writing",
           minutes, objectives:[3], sections:[Section...] }      // the last section is always an exercise
Question { id (unique in the file), type:"mcq"|"truefalse"|"fillblank"|"dictation", prompt,
           options:[3-4] (mcq), answer (mcq: 0-based index, truefalse: boolean),
           answers:[accepted strings] (fillblank, dictation), audio:"text read aloud" (required for dictation),
           explanation (required; say why) }
```

Section types (each has `type` and `heading`):

| type | fields |
|---|---|
| vocabulary | `items:[{word,pos,meaning,example,pronunciation}]` (8-10) |
| grammar | `explanation:[paragraphs]`, `table?:{headers,rows}`, `examples:[{text,note}]` (4-6), `commonMistakes?:[{wrong,right,why}]` |
| dialogue | `context`, `lines:[{speaker,text}]` (8-12), `notes:[...]` |
| listening | `instructions`, `transcript:[{speaker,text}]`, `questions:[4-5 Question]` |
| reading | `title`, `passage:[paragraphs]`, `glossary:[{word,meaning}]`, `questions:[4-5 Question]` |
| speaking | `task`, `steps:[]`, `usefulPhrases:[]`, `sampleResponse` (self-assessed) |
| writing | `task`, `wordTarget:"40-60"`, `checklist:[]`, `modelAnswer` (self-assessed) |
| pronunciation | `focus`, `explanation`, `pairs:[{words:[a,b],tip}]`, `practice:[sentences]` |
| american | `items:[{phrase,meaning,example,note?}]` |
| text | `body:[paragraphs]` |
| exercise | `instructions`, `questions:[5-6 Question]` |

Answer checking for `fillblank` and `dictation` ignores capital letters, extra spaces, curly versus straight apostrophes, and trailing punctuation (`. ! ? , ; :`), and accepts any string in `answers`. Punctuation in the middle of an answer still counts, so add a variant to `answers` when a comma is optional.

When a level loads, the app checks it (unique ids, mcq answer in range, fillblank has `___` and answers, explanations present, dictation has audio, lessons end with an exercise). Problems are shown on a red card on the level page and in the browser console; the lessons still open.

## How to add a lesson

1. Open `data/level-a1.json` (or another level). In the unit's `lessons` array add a Lesson object with a new unique `id` (for example `a1-u1-l4`), the next `number`, a `title`, `focus`, `minutes`, `objectives`, and `sections`.
2. Build `sections` from the types above and finish with an `exercise` section.
3. Give every question a unique `id` across the whole file and an `explanation`. For `fillblank` put `___` in the prompt; for `dictation` set `audio`.
4. Reload the page. If something is wrong, the level page shows a red content-check card and the console lists each problem.
5. The progress bars count all lessons, so adding a lesson changes the percentages (completion records for other lessons stay valid).

The app works with any number of lessons per unit, but the schema targets 3 lessons per unit and 4 units per level.

## Limitations

- Spoken audio uses the browser's built-in text-to-speech. Voices, accent and quality differ by browser and operating system, and some browsers have no English voice. If speech is unsupported the app says so and keeps all text readable.
- Progress lives in the browser's localStorage on one device. There are no accounts and no sync; clearing site data deletes it. Use Settings to export a JSON backup and import it elsewhere.
- Writing and speaking are self-assessed with checklists and model answers. The app does not grade free writing or listen to speech.
- The course is compact (12 lessons at A1), not a full-length program.
- Only the A1 content is included. A2, B1, B2 and C1 need their own JSON files.
