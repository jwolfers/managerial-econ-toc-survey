# Managerial Economics — TOC Survey

A one-page drag-and-drop survey in which instructors build their preferred table of
contents for *Managerial Economics* (Snowberg, Stevenson & Wolfers). Respondents set
their number of class sessions — the count box asks "How many class sessions?" from a ringed
card, because it is the one setting everyone must change and it was easy to skim past (default
15, adjustable with ±, or "+ Add a session", which doubles as a drop target: a chapter dropped
on it gets a new session of its own at the end, and the ⇄ popup offers the same as "New
session at the end"; each session header also carries a 🗑 that deletes that session where it
stands, since the count box can only trim from the end and a week that turns out to belong to
the one before it is rarely the last one) —
then drag the 15 planned chapters into those sessions (or discard them to an "unused" pile),
optionally stretch a chapter across consecutive sessions (the ⤓ button; later sessions
show a dashed "continued" card), add topics of their own to any session, answer the survey's own questions — which can sit above
the sessions, below them, or both — and submit. Every response lands as one row in a Google Sheet; multi-session
chapters are recorded in the readable TOC ("runs N sessions") and as `spans` in the JSON
column. Prerequisite enforcement treats a spanned chapter as finishing in its last
session: dependent chapters must start at or after that session.

**Nothing a respondent does is lost by accident.** A part-built response is kept in their
browser — the board, and every word they have typed — so a closed tab or a crashed browser
costs nothing, and the draft is cleared only once the response is safely submitted. Removing a
session is the one destructive control, so both ways of doing it — cutting the count, and the
per-session 🗑 — ask first whenever the sessions going away still hold chapters or their own
topics, and the count box ignores a value it can't read (an
emptied box means "no change", not "one session"). Deleting a session in the middle shifts
everything after it up a slot, so the relative order of every chapter that stays is preserved;
a chapter that ran through the deleted session simply runs one session fewer. What doesn't stay
is that session's own chapters, which go back to the chapter list — so the bin's rule applies
there too, and a session won't delete while something scheduled later still builds on a chapter
inside it. (Trimming from the end never needs that check: a chapter that builds on another
always sits in a later session, so it is being removed as well.) A chapter refused a session because of its
prerequisites goes back exactly where it came from, unused pile included, and a chapter that
leaves the board forgets how many sessions it ran, so it never returns silently stretched.

**The constraints, said up front.** Chapter 1 is the cost-benefit toolkit every part opener
requires, so the prerequisites can only ever place it first — but "Approach" in its title reads
like a preface, and instructors were meeting the rule as an error message. So a note under the
lede says that Chapter 1 comes first and that a few later chapters follow the ones they build
on, and a fresh board arrives with Chapter 1 already in Session 1. It stays an ordinary chapter
otherwise — free to be moved, stretched or discarded — and **Start over** returns the board to
exactly that state. (The seeding is skipped if the Sheet's first chapter has prerequisites of
its own, i.e. isn't the opener.)

**Topics of their own.** Under every session sits "+ Add a topic of your own", for
something an instructor teaches that isn't in our chapter list. A written-in topic
becomes a dashed card in that session and behaves like a chapter — reorder it, move it
to another session, run it across several — but it has no requirements, never appears in
the chapter bank or the unused pile, and is renamed (✎) or deleted (🗑) rather than
discarded. Own topics take negative ids so they can never collide with a real chapter:
the ids sit in the `sessions` arrays of the JSON column and their titles in a `custom`
map beside them, and the readable TOC column names them "Their own topic — …". Because
they are one respondent's invention, they are excluded from every chapter statistic
(inclusion rates, consensus ordering, contested pairs, camps) and reported on their own
instead — see the dashboard's "Topics they added themselves".

A **Recommended sequence** button seeds the sessions with all chapters in the book's
order — one chapter per session, so the session count becomes the chapter count — so
respondents who mostly agree only express deviations; whether a response started from
that prefill is recorded (`prefilled` in the JSON column) so anchored responses can be
segmented in analysis. **Start over** (next to it) empties every session and the unused
pile and returns the board to how it arrived — 15 sessions, Chapter 1 in the first — after a
confirmation; own topics go with it, and the written answers are kept. (On an untouched board
neither button asks: there is nothing to overwrite.) On phones and tablets — where browsers don't
support drag-and-drop — the chapter list appears first, the instructions switch to
tap-based wording, and everything is doable by buttons alone: **+** places a chapter,
**⇄** moves it between sessions, ▲▼ reorder, ⤓ extends, 🗑 discards — and the
add-a-topic field is a plain text input, so it works the same everywhere.

Styled to match the Stevenson & Wolfers 3e design: Source Sans 3 / Source Serif 4, with
the 3e cover palette (deep red, blue, berry, purple, teal; chartreuse and gold accents).

## Architecture

**GitHub Pages hosts the page; a Google Sheet keeps the data.** The static front-end
talks to a small JSON API served by Google Apps Script, which lives inside the Sheet.
GitHub Pages is the only place the page is served from — opening the Apps Script
`/exec` URL in a browser just redirects to the Pages site, so there is never a second
copy of `index.html` to keep in sync.

| File | What it is | Goes where |
|---|---|---|
| `index.html` | The whole app: survey, plus the password-gated `/edit` (chapters **and** questions) and `/analyze` pages. | GitHub repo |
| `edit.html`, `analyze.html` | Tiny redirects so `yoursite/…/edit` and `…/analyze` work as URLs. | GitHub repo |
| `Code.gs` | Apps Script backend: serves config, stores responses, checks the admin password, emails you each submission, proxies the Claude API for the dashboard's AI features. | Apps Script only — **never** push it to a public repo (it contains the admin password). `.gitignore` enforces this. |

Opened as a plain file (double-click), `index.html` runs in **preview mode**: built-in
chapter list, submissions logged to the browser console, and `#analyze` shows synthetic
demo data.

## Deploying

### 1. The backend (one time, ~10 minutes)

1. Go to **[sheets.new](https://sheets.new)** and name the spreadsheet (e.g., "ME TOC Survey").
2. **Extensions → Apps Script.** Delete the placeholder code in `Code.gs` and paste in
   the contents of this folder's `Code.gs`. (There is no `Index` HTML file to create —
   GitHub Pages serves the page.)
3. In the function dropdown pick **`setUp`** and click **Run**. Approve the authorization
   prompts (Sheet access, plus permission to email you each response — set
   `NOTIFY_ON_RESPONSE = false` at the top of `Code.gs` if you don't want that). This
   creates and seeds three tabs: **Chapters**, **Extra Questions**, **Responses**.
4. **TOC Survey menu → Change admin password.** The shipped default is weak and is the
   only thing gating the results dashboard, which shows respondent names and emails.
5. **Deploy → New deployment → Web app.** Execute as: **Me**. Who has access: **Anyone**.
   Click **Deploy** and copy the web-app URL (ends in `/exec`).

Also set `SITE_URL` at the top of `Code.gs` to your GitHub Pages URL, so the `/exec`
URL redirects to the real survey.

### 2. The GitHub Pages front-end

1. In `index.html`, near the top of the `<script>`, paste the web-app URL into
   `var API_URL = "";`.
2. Put `index.html`, `edit.html`, and `analyze.html` in your GitHub site's repo (say, in
   a `toc-survey/` folder). If Pages isn't enabled yet: repo **Settings → Pages →
   Deploy from a branch**.
3. The survey is live at `https://<you>.github.io/…/toc-survey/`. Send that link to
   respondents — it's cleaner than the Apps Script URL and skips Google's "created by
   another user" banner. The admin pages are `…/toc-survey/edit` and
   `…/toc-survey/analyze`.

Do **not** put `Code.gs` in the repo.

### After changing code later

- `index.html` changes: push to GitHub. That's the whole story — there is no second copy.
- `Code.gs` changes: paste into Apps Script, then **Deploy → Manage deployments → Edit →
  New version** (the `/exec` URL stays the same).
- Sheet edits (chapters, questions) need no redeployment — they take effect immediately.

## The two admin pages

Both are gated by the **admin password** — see or change it in the Sheet under the
**TOC Survey menu**. The password is checked server-side on every
privileged call and never appears in the public front-end files; your browser remembers
it after the first successful entry. Because the web app is public, eight wrong guesses in a
row lock admin calls for a minute — long enough to make guessing hopeless, short enough never
to be a problem for you (and changing the password lifts it at once).

**`/edit` — survey editor**, in three tabs.

*Chapters.* Edit titles, parts, section lists, notes; toggle prerequisite chips; add or
delete chapters. The page blocks impossible saves (missing titles, dangling or circular
dependencies).

*Questions.* Everything respondents are asked, in the order they see it, split into the
questions that come **before** the chapter sort (shown above the class sessions — screening
or context) and those that come **after** it (under "A few final questions"). Add a
question to either side, delete one, drag a card by its handle — or press ▲▼ — to reorder
it, and drag it across to the other list (or use its Placement menu) to move it to the
other side of the sort. Each question has a type, an optional description, an optional
*required* flag, and an answer ID that names its column in the Sheet. The available types
match a Google Form:

| | |
|---|---|
| Short answer, Paragraph | one line, or a box |
| Multiple choice, Checkboxes, Dropdown | one-of and many-of, each with an optional "Other…" write-in |
| Linear scale, Star rating | any range (a scale may start at 0), with optional end labels |
| Multiple-choice grid, Checkbox grid | rows × columns, one-of or many-of per row |
| Date, Time, Date and time, Number, Email address, Link (URL) | validated single fields |
| Section heading | a heading and blurb — not a question, collects no answer |

*Rounds.* A round is a name for a stretch of surveying — *Test* while you are trying
things out, then the real thing, then *2027* when the book is revised. The tab shows which
round responses are being recorded into, how many have arrived in it and when it started,
and a field to name and start a new one. Starting a round changes nothing about the survey
and deletes nothing: responses already collected keep the label they have, the link stays
the same, and only what arrives from that moment on carries the new name. Reusing a name
switches back to that round rather than starting a second one. The dashboard then shows one
round at a time, so a test run never mixes into the real results. (The same thing is on the
Sheet's **TOC Survey menu → Start a new round of surveys**.)

Saving one tab leaves unsaved edits in the other alone, and a save only ever writes its own
tab. Saving the Chapters or Questions tab updates the Sheet, so changes reach respondents
immediately. In the local preview, `index.html#edit` saves only to that browser.

**`/analyze` — results dashboard.** Every statistic is computed in the page from the raw
responses; chapter positions are normalized to each respondent's course length so a
10-session and a 28-session course are comparable. It shows:

- a **round picker** in the toolbar whenever responses span more than one round. A fresh
  load opens on the round the newest response arrived in — so the moment a real round
  starts, the test run stops colouring the results — and "All rounds" is one click away.
  Every number on the page, both downloads, and the AI report all follow the picker;
- headline tiles — responses, median sessions, chapters used, adoption likelihood,
  agreement with book order;
- the **consensus ordering** — chapters by median start position, with inclusion rates;
- a **placement heatmap** — where in the course each chapter lands (a tight band is
  consensus, a smear is disagreement);
- **inclusion & class time** per chapter — who schedules it, how many sessions it gets;
- the **most contested pairs** — orderings respondents genuinely disagree about;
- **camps** — respondents clustered by TOC similarity, each with its majority ordering;
- **topics they added themselves** — every write-in topic with its author and the session
  it was slotted into, which is the most direct read on what the book is missing;
- a chart per question, in the shape its answers take — a distribution for scales and
  ratings, counts for choice, dropdown, checkbox and date/time answers, mean/median/range
  for numbers, a shaded matrix for grids — and all free-text answers as quotes linked to
  their authors;
- an **individual browser** — every response with the date and time it was submitted
  (and its round, when several are on screen); click any respondent to see their course
  rendered exactly as they built it, plus all their answers;
- **Download all responses** for offline analysis — whichever round is on screen — as
  **CSV** (one row per respondent, with its timestamp and round, a column per extra question, an **Own topics added** column, and, for every
  chapter, the session it was placed in — or `unused` — and how many sessions it
  runs) or as **JSON** (the raw structure, bundled with the chapter and question
  definitions).

### AI report & chat

The dashboard's "AI analysis" section asks Claude to write a synthesis report, and offers
a chat box for follow-ups ("which two chapters could we merge with the least
resistance?"). Changing the round picker mid-conversation doesn't end the conversation —
comparing rounds is half the point of asking — but it does mark the seam: a divider appears in
the log, and the next question tells the model that the statistics have changed underneath it
and that earlier answers were about other responses. Setup: in the Sheet, **TOC Survey menu → Set Anthropic API key**, pasting
a key from console.anthropic.com. The key stays in the script's properties — it is never
sent to the browser — and the page sends its *precomputed* statistics plus the verbatim
free text, so the model interprets rather than recalculates.

**Model choice.** `claude-opus-5`, set at the top of `Code.gs` along with two effort
levels. The job is interpretive judgment over a small sample — reading disagreement out
of thirty instructors' orderings and saying which of it is real — which is where the
strongest reasoning model pays off, and the volume is so low that cost barely enters:
about 30k tokens in and 10k out per report, roughly 40 cents at $5/$25 per million.
Claude Fable 5 is the only stronger option, but it costs double and its answers can run
several minutes, which risks Apps Script's hard 6-minute execution ceiling with nothing
to show for it; Sonnet 5 is cheaper and faster but gives up exactly the judgment this
task is asking for. Two knobs sit beside the model: `REPORT_EFFORT` (`high`) and
`CHAT_EFFORT` (`medium`), so the long report thinks hard while follow-up questions stay
quick. The survey-statistics digest is prompt-cached, so chat follow-ups re-read it at a
tenth of the input price instead of reprocessing it every turn.

Server-side refusal fallbacks are enabled, so a declined request retries automatically on
a fallback model. A full report takes a minute or two. Note that generating one sends
response data to the Anthropic API (standard API terms; not used for training) — the one
moment data leaves Google.

## Administering the survey

Two equivalent ways to manage chapters — the `/edit` page above, or the Sheet directly:

- **Chapter facts** — edit the *Sections* and *Summary* columns in the **Chapters** tab.
  The summary appears when a respondent clicks a chapter title.
- **Prerequisites** — put comma-separated chapter IDs in the last column of **Chapters**
  (e.g., `2,3`). The survey then refuses any ordering that puts a chapter before its
  prerequisites, with a message explaining why. Prereqs chain transitively, so only
  direct dependencies need listing. Seeded with the real dependencies: Ch 1 before
  everything (each part opener requires it); each part's opening chapter (2, 5, 9, 12)
  before the rest of its part; and Ch 2 before Ch 12.
- **Extra questions** — the `/edit` page's Questions tab is much the easier way, but the
  **Extra Questions** tab holds the same data, one row per question in the order asked:
  *ID*, *Type* (`text`, `textarea`, `choice`, `checkbox`, `dropdown`, `scale`, `rating`,
  `grid`, `checkgrid`, `date`, `time`, `datetime`, `number`, `email`, `url`, `section`),
  *Prompt*, *Options* (`a|b|c` — for a scale, the two end labels), *Placement* (`before` or
  `after` the chapter sort), *Required?* (`yes`/`no`), *Description*, *Grid rows* (`r1|r2`),
  and *Advanced settings* (JSON: a scale's `min`/`max`, and `other` for a write-in option).
  Sheets from before the richer types keep working — the first four columns are unchanged,
  and the rest default to an optional question after the sort. Re-run **TOC Survey menu →
  Set up / repair survey tabs** once to add the new column headers.

## Rounds of surveying

Responses are grouped into named **rounds** — "Test", "Fall 2026 pilot", "2027 survey".
Whichever round is active when a response arrives is stamped into its **Round** column and
never changes afterwards, so naming a new round is a safe, purely forward-looking act: it
relabels nothing. Rounds are named on the `/edit` page's Rounds tab (or the Sheet's TOC
Survey menu), listed oldest-first in the **Rounds** tab with the date each began, and used
by the dashboard to show one round at a time. Responses collected before any round was
named stay unlabelled, and the dashboard offers them as their own bucket. A sheet that
predates rounds simply gains the Round column, appended at the right-hand end, the next
time a response arrives.

## Where the data lives, and getting it out

Every submission is one row in the Sheet's **Responses** tab: the time it arrived, the
round it belongs to, the readable TOC, the unused pile, all answers, and a JSON copy of
the full structure for programmatic analysis. Each extra question also gets its own `Q: <id>` column (created automatically
after the core columns) so answers can be filtered and pivoted directly; a checkbox answer
flattens to `a | b` there and a grid answer to `row: value | row: value`, while the JSON
copy keeps them structured. Topics respondents added themselves appear in the readable TOC
as "Their own topic — …", and in the JSON column as negative ids in `sessions` with their
titles in `custom`. Free-text cells
are sanitized against formula injection — anything a spreadsheet would evaluate (a leading
`=`, `+`, `-`, `@`, tab or carriage return) is quoted on the way in, in the Sheet and in the
dashboard's CSV alike — and every cell is truncated to stay inside the 50,000-character
ceiling, so one very long answer can't fail the whole row. A response built while the Sheet
was unreachable — the page falls back to its built-in chapter list — carries
`fallbackConfig` in the JSON column, so it can be told apart in analysis. Export any time with File → Download →
CSV/XLSX, or the dashboard's CSV/JSON downloads. The Sheet itself stays private to your
Google account; only the survey page is public.

**Response notifications** — off. Set `NOTIFY_ON_RESPONSE = true` at the top of
`Code.gs` to have the sheet owner emailed a short summary per submission (name,
institution, the TOC, likes/missing); responses land in the Sheet either way.

## Preview without deploying

Double-click `index.html`. It runs with the built-in chapter list, shows a "Preview mode"
banner, logs would-be submissions to the browser console, and the preview links open the
survey editor (browser-local saves) and the dashboard with demo data.
