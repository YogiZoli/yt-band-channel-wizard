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
- PDF: jsPDF 2.5.1 and html2canvas 1.4.1 from cdnjs, generated in the browser.
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
- **PDF**: tries the designed PDF, then a lighter one, then a plain text PDF. The two PDF tools load from cdnjs, with jsDelivr as the second try. A part that hangs is cut off after 25 seconds.
- **Downloads**: after each file the page shows an "open in a new tab" link, because some browsers ignore a started download. The .md text can also be copied.
- **Copy buttons**: if the browser blocks copying, the text is selected and the visitor is told how to copy it by hand.
- **Images**: wrong type, over 30 MB or an unreadable format gives a clear message. Large images are scaled down in memory so the PDF step does not run out of memory.
- **Lead webhook**: never blocks the visitor. The result is recorded and a failed send is retried on the next visit, up to 5 times.
- **Back button**: the browser's Back moves one screen back inside the wizard.
- **If a screen fails to draw**: a recovery card with Reload, Copy the diagnostic report and Go back to the first question.

### Finding out what happened

The footer has "Copy a diagnostic report". It copies the version, browser, current screen, whether progress is being saved, the lead send status, which PDF tools loaded, and the last 150 events in order (for example `pdf-start`, `pdf-retry`, `pdf-fallback`, `download`, `copy-failed`, `image-failed`, `lead`, `error`). It contains no answers, no name and no email. The same log is in the browser under the `localStorage` key `bcw_state_log`.
