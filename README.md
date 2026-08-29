# Managerial Economics — TOC Survey

A one-page drag-and-drop survey in which instructors build their preferred table of
contents for *Managerial Economics* (Snowberg, Stevenson & Wolfers). Respondents set
their number of class sessions (default 15, adjustable with ± or "+ Add a session"),
drag the 15 planned chapters into those sessions (or discard them to an "unused" pile),
optionally stretch a chapter across consecutive sessions (the ⤓ button; later sessions
show a dashed "continued" card), answer the survey's own questions — which can sit above
the sessions, below them, or both — and submit. Every response lands as one row in a Google Sheet; multi-session
chapters are recorded in the readable TOC ("runs N sessions") and as `spans` in the JSON
column. Prerequisite enforcement treats a spanned chapter as finishing in its last
session: dependent chapters must start at or after that session.

A **Start from book order** button seeds the sessions with all chapters in the book's
order — one chapter per session, so the session count becomes the chapter count — so
respondents who mostly agree only express deviations; whether a response started from
that prefill is recorded (`prefilled` in the JSON column) so anchored responses can be
segmented in analysis. **Start over** (next to it) empties every session and the unused
pile and returns the count to 15, after a confirmation; the written answers are kept. On phones and tablets — where browsers don't
support drag-and-drop — the chapter list appears first, the instructions switch to
tap-based wording, and everything is doable by buttons alone: **+** places a chapter,
**⇄** moves it between sessions, ▲▼ reorder, ⤓ extends, 🗑 discards.

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
it after the first successful entry.

**`/edit` — survey editor**, in two tabs.

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

Saving either tab updates the Sheet, so changes reach respondents immediately. In the
local preview, `index.html#edit` saves only to that browser.

**`/analyze` — results dashboard.** Every statistic is computed in the page from the raw
responses; chapter positions are normalized to each respondent's course length so a
10-session and a 28-session course are comparable. It shows:

- headline tiles — responses, median sessions, chapters used, adoption likelihood,
  agreement with book order;
- the **consensus ordering** — chapters by median start position, with inclusion rates;
- a **placement heatmap** — where in the course each chapter lands (a tight band is
  consensus, a smear is disagreement);
- **inclusion & class time** per chapter — who schedules it, how many sessions it gets;
- the **most contested pairs** — orderings respondents genuinely disagree about;
- **camps** — respondents clustered by TOC similarity, each with its majority ordering;
- a chart per question, in the shape its answers take — a distribution for scales and
  ratings, counts for choice, dropdown, checkbox and date/time answers, mean/median/range
  for numbers, a shaded matrix for grids — and all free-text answers as quotes linked to
  their authors;
- an **individual browser** — click any respondent to see their course rendered exactly
  as they built it, plus all their answers;
- **Download all responses** for offline analysis, as **CSV** (one row per respondent,
  with a column per extra question and, for every chapter, the session it was placed in
  — or `unused` — and how many sessions it runs) or as **JSON** (the raw structure,
  bundled with the chapter and question definitions).

### AI report & chat

The dashboard's "AI analysis" section asks Claude to write a synthesis report, and offers
a chat box for follow-ups ("which two chapters could we merge with the least
resistance?"). Setup: in the Sheet, **TOC Survey menu → Set Anthropic API key**, pasting
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

## Where the data lives, and getting it out

Every submission is one row in the Sheet's **Responses** tab: the readable TOC, the
unused pile, all answers, and a JSON copy of the full structure for programmatic
analysis. Each extra question also gets its own `Q: <id>` column (created automatically
after the core columns) so answers can be filtered and pivoted directly; a checkbox answer
flattens to `a | b` there and a grid answer to `row: value | row: value`, while the JSON
copy keeps them structured. Free-text cells
are sanitized against formula injection. Export any time with File → Download →
CSV/XLSX, or the dashboard's CSV/JSON downloads. The Sheet itself stays private to your
Google account; only the survey page is public.

**Response notifications** — the sheet owner gets a short email per submission (name,
institution, the TOC, likes/missing). Turn off with `NOTIFY_ON_RESPONSE = false` at the
top of `Code.gs`.

## Preview without deploying

Double-click `index.html`. It runs with the built-in chapter list, shows a "Preview mode"
banner, logs would-be submissions to the browser console, and the preview links open the
survey editor (browser-local saves) and the dashboard with demo data.
