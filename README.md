# Brand Channel Wizard

Lead magnet for course creators: a step by step wizard that prepares a YouTube Brand Channel in 5 phases and ends in a downloadable "YouTube Evergreen Organic Traffic Card" (PDF and .md).

Live: https://yogizoli.github.io/yt-band-channel-wizard/

## How it works

"Prompt pack" wizard. It asks, prepares and packages. It makes no AI calls and no API calls. Anything that needs an outside tool is delivered as a copy-ready prompt that the visitor runs in their own Claude or ChatGPT with their own vidIQ account.

1. Entry gate: vidIQ account, AI app (Claude or ChatGPT), vidIQ connected in that app.
2. 13 questions, one per screen, grouped by the 5 phases.
3. Email gate (name and email), then the kit: ready text, Research Prompt Packet, image prompts, phase checklist.
4. Paste back: the research result, plus optional logo and banner preview with the banner safe area overlay.
5. The card: PDF and .md download, and the program CTA.

## Stack

- One static file: `index.html` (HTML, CSS, vanilla JavaScript). No build step, no framework, no backend.
- PDF: jsPDF 4.2.1 and html2canvas 1.4.1, generated in the browser. Both ship in `vendor/`.
- Lead capture: optional POST to a webhook (GoHighLevel inbound webhook).
- State: `localStorage` in the visitor's browser. Uploaded images stay in memory only.

## Settings

Both are at the top of the last `<script>` block in `index.html`:

- `WEBHOOK_URL`: empty means no POST, the kit is still delivered. Payload: `{name, email, source:"brand-channel-wizard", ai_app, answers}`.
- `PROGRAM_URL`: the link behind the cohort program CTA (placeholder for now).

## The .md card format

Fixed H2 headings, in this order: Channel, Pillars, Playlists, Keywords, Competitors, Outliers, Gaps, Video List, Checklist, Notes, Next Step. Front matter carries `card`, `version`, `generated`, `source`, `ai_app`.

## Things that drift

- The vidIQ connect steps for Claude and ChatGPT (`gate3` in `index.html`).
- The YouTube Studio click paths (`checklist` in `index.html`). Last checked: October 2026.

## Reliability (v1.1.0)

The rule: nothing fails silently, no raw error reaches the visitor, and the page always says what is happening.

- **Status line** under the progress bar: what is running, whether to wait, and when the page can be closed.
- **Saved progress**: answers are saved in the browser as the visitor types. If the browser refuses to save (private mode, full storage), the page says so and asks to keep the tab open. Research that is too large to save is kept on the page and the visitor is told.
- **Saved state is checked on load**: an old, half written or broken state is repaired. If a required answer is missing, the visitor is taken to that question instead of a broken kit.
- **PDF**: tries the designed PDF, then a lighter one, then a plain text PDF. The two PDF tools load from the local `vendor/` copy, then cdnjs, then jsDelivr. A part that hangs is cut off after 25 seconds.
- **Downloads**: after each file the page shows an "open in a new tab" link, because some browsers ignore a started download. The .md text can also be copied.
- **Copy buttons**: if the browser blocks copying, the text is selected and the visitor is told how to copy it by hand.
- **Images**: wrong type, over 30 MB or an unreadable format gives a clear message. Large images are scaled down in memory so the PDF step does not run out of memory.
- **Lead webhook**: never blocks the visitor. The result is recorded and a failed send is retried on the next visit, up to 5 times.
- **Back button**: the browser's Back moves one screen back inside the wizard.
- **If a screen fails to draw**: a recovery card with Reload, Copy the diagnostic report and Go back to the first question.

### Finding out what happened (v1.2.0)

The app keeps a detailed event log. There is no server and no monitoring: the log lives in the visitor's browser, survives reloads and a restart, and holds the last 600 entries across visits.

Every entry has: a running number, a visit id, the time, milliseconds since the page opened, a level (INFO, WARN, ERROR), the screen, the event and a detail. Example:

    #55 2026-10-10T16:55:08.519Z +3631ms WARN [card] pdf-retry: drawing part 4 of 29 (content): TypeError: forced canvas failure | stack: ...

What is logged:

| Event | What it records |
|---|---|
| `open`, `env` | version, starting screen, size of the saved state, browser, viewport, memory, network type, timezone, in-app browser |
| `screen`, `choice` | every move between screens (also browser back and forward) and every option picked |
| `answer`, `validation` | each accepted answer and each refused one, with the lengths of the fields, never their content |
| `lead-form`, `lead` | the email form, and each webhook attempt with result, try number and duration |
| `kit-shown`, `copy`, `copy-failed`, `checklist`, `link` | what was shown, which block was copied and how, ticked items, outside links opened |
| `research` | length of the pasted text, which sections were found and which are missing |
| `image-read`, `image-ok`, `image-rejected`, `image-failed` | file type, size, dimensions, scaling, duration, and the reason when refused |
| `lib-load`, `lib-loaded`, `lib-load-failed`, `lib-missing` | each attempt to load a PDF tool, with URL and duration |
| `pdf-start`, `pdf-layout`, `pdf-part-slow`, `pdf-retry`, `pdf-fallback`, `pdf-done`, `pdf-failed` | every stage of the PDF, the exact part that failed and why, pages and seconds |
| `download`, `md-copied`, `md-failed` | each file made, with name and size |
| `storage`, `save-failed`, `load-failed`, `guard` | saving problems with the browser's own error, broken saved state, forced redirects |
| `shown-to-visitor` | every status message the visitor saw, word for word |
| `page`, `network` | tab hidden or visible, page left (also during a PDF build), online and offline |
| `error`, `promise-error`, `resource-failed`, `render-failed` | every unexpected error with file, line, column and stack |

The log never contains an answer, a name, an email or the pasted research.

How to read it:

- **Footer, "Copy a diagnostic report"**: state summary plus the whole log, on the clipboard.
- **Footer, "Download the full log"**: the same as a .txt file.
- **Add `?debug=1` to the URL**: a live log panel at the bottom of the page.
- **Browser console**: every entry is printed with the prefix `[BCW]`. `BCW.report()` returns the report, `BCW.log()` the raw entries.
- **Storage**: `localStorage` key `bcw_state_log`.

Optional: set `LOG_URL` (next to `WEBHOOK_URL`) and every ERROR also posts the report to that address, at most 5 times per visit. It is empty by default, so nothing leaves the browser.
