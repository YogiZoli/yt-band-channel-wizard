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
