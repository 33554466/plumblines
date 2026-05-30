# Provenance

**A reading instrument for the open web.**
Paste an article. See which statistics the writer actually sourced, which they hand-waved, how confidently they're claiming things, where the rhetoric is doing the work, and how much of the piece is opinion wearing a lab coat.

**Live at [plumblines.org](https://plumblines.org).**

---

## What it does

Provenance is a single self-contained HTML page that takes an article you paste in and produces a structured audit of it:

- **The Ledger** — every statistical or quantitative claim in the article, tagged green (named source), amber (vague attribution like *"studies show"*), or red (no source given at all). Each claim is shown with the exact sentence it came from and the source as the writer stated it.
- **Fact ↔ Opinion meter** — a rough estimate of how much of the piece is verifiable factual claims versus opinion, interpretation, and rhetoric.
- **Hedged ↔ Assertive meter** — a separate axis showing how confidently the writer states things, from heavily qualified (*"may," "could," "appears to"*) to flat assertion. Comes with example phrases pulled from the text.
- **Rhetorical Cargo** — loaded language flagged with a category (intensifier, dismissive, euphemism, charged framing, etc.) and a short neutral note on what work the phrase is doing.
- **Dig Deeper** — research directions a curious reader could pursue for broader context, with named, authoritative starting-point sources (Pew, BLS, OECD, FRED, etc.) rendered as search links.

It then offers:
- A **shareable link** that encodes the full analysis in the URL (no server, no account, no tracking).
- **Markdown** export for keeping a record or sharing as a document.
- **PDF** export via the browser's print dialog.
- A **second-opinion mode** that runs the same article through two different AI models in parallel and surfaces where they agree and where they disagree.

---

## What it does *not* do

This part is important. Provenance is deliberately scoped to be honest about what it can and can't do.

- It does **not** verify whether any claim is true.
- It does **not** hunt down sources on the open web.
- It reports **only** what the article itself provides.

A named source is not the same as a good source — a statistic from a lobby group is still "sourced." Provenance tells you the writer named someone; the judgment about whether that someone is credible is yours.

The "Dig Deeper" feature suggests starting points for further reading, but it does *not* provide numbers — those should always be checked at the source itself.

---

## How it works (and why it's free)

Provenance is a single static HTML file. It runs entirely in your browser, on **your own** AI API key — there is no backend server. Your key is sent only to the AI provider you choose, and never to me or anyone else.

This means:
- **Free to use forever** — there's no infrastructure to pay for, no monetization, no ads.
- **Your data stays yours** — the article text and your API key never touch any server except the AI provider's own API.
- **Inspectable** — the entire tool is the single `index.html` file in this repo. You can read it before trusting it.

The tradeoff is that you need to bring your own AI API key. See **Getting a key** below.

---

## Getting a key

Provenance supports four providers. You only need one.

| Provider | Free tier? | Where to get a key |
|---|---|---|
| **Google (Gemini)** | ✅ Yes — daily limits, no credit card | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
| **OpenAI** | ❌ No — pay as you go | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |
| **Anthropic (Claude)** | ❌ No — pay as you go | [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys) |
| **xAI (Grok)** | ❌ No — pay as you go | [console.x.ai](https://console.x.ai) |

**For the truly-free path: use Google Gemini.** No payment method required.

For the paid providers, you typically add $5 of prepaid credit and a single article analysis costs a fraction of a cent — that's hundreds of articles before you need to top up.

---

## How to use it

1. Visit [plumblines.org](https://plumblines.org).
2. In the **Setup** panel, pick a provider, paste your API key.
   - Optionally tick "Remember this key on this device" so you don't have to paste it again.
3. Paste the article text into the box. (Select the article body on the original page, copy, paste here.)
4. Optionally:
   - Toggle **Flag loaded language** and **Suggest ways to dig deeper** (both on by default).
   - Tick **Second opinion** to compare two different models on the same article.
5. Click **Run the Audit** (or press Cmd/Ctrl + Enter).

After the audit renders, you can **Copy share link**, **Export Markdown**, or **Export PDF** at the bottom.

---

## Privacy and security

- Your **API key** is stored only in your browser's `localStorage` (and only if you tick "Remember"). It is sent only to the AI provider's API. It never touches a server I control.
- The **article text** you paste is sent only to the AI provider you chose. It is *not* included in shareable links.
- Shareable links contain only the **analysis result**, encoded into the URL fragment (the part after the `#`). URL fragments are never sent to web servers — they exist only in your browser.
- There is **no automatic analytics, no telemetry, no tracking, no account system, no cookies**. Visits are not counted by default.
- There is one **opt-in tally** at the bottom of the page — if you click "I read here," your browser makes a single anonymous request to a free public counter service (Abacus) which increments and returns a running total. No personal data is sent. No cookies are set. The only thing stored locally is a flag so the button doesn't reappear on your next visit.

If you're security-cautious: read the source. The file is human-readable. Search for `fetch(` to find every outbound network call; you'll see the four AI providers' endpoints, Google Fonts, and the opt-in counter — nothing else.

---

## Running locally

You don't need a server. Open the HTML file directly in any modern browser:

```
git clone https://github.com/<your-username>/plumblines.git
cd plumblines
# macOS:
open index.html
# Windows:
start index.html
# Linux:
xdg-open index.html
```

Two notes:

- **CORS:** Some AI providers may block API requests made directly from a `file://` URL. If you hit that, either host the file on a simple static site (GitHub Pages, Cloudflare Pages, Netlify Drop — all free), or just use the live version at [plumblines.org](https://plumblines.org).
- **Browser support:** Provenance uses modern browser APIs (CompressionStream, Clipboard API). It should work in any browser from 2023 onward. Older browsers may degrade gracefully on the share feature.

---

## Features in detail

### Bring-your-own-key architecture
There is no backend. Calls go directly from your browser to the AI provider you choose, authenticated with your own key. This is the only architecture that makes a free, no-account, no-tracking tool sustainable indefinitely.

### Grounding check
For every claim and every flagged phrase, the model is required to provide the exact sentence from the article it came from. Provenance then performs a literal text-search to confirm that sentence actually appears in what you pasted. If it doesn't, the card gets a red **"quote not found verbatim"** badge — protecting against the most common AI hallucination failure mode.

### Second-opinion mode
Tick the checkbox, fill in a second provider's setup panel, and Provenance runs your article through both models in parallel. The results are merged using fuzzy text matching to identify which claims both models flagged, which only one did, and where the two models disagreed on a claim's sourcing.

For meaningful disagreement, **pick two different providers** (e.g. Gemini + Claude). Two models from the same provider mostly agree with themselves.

### Share links
The share button gzips and base64url-encodes the entire analysis (minus the original article) into a URL fragment. Anyone you send the link to gets the full audit, rendered identically, with a banner indicating it's a shared view. No accounts. No expiration. No server. The link works forever, hosted by nothing.

### Markdown export
A clean Markdown rendering of the audit, suitable for archiving, sharing as a document, or pasting into a notebook.

### PDF export
Uses the browser's native print dialog (which on every modern browser includes "Save as PDF"). A purpose-built print stylesheet hides controls and renders the audit in clean black ink on white paper, with page-break protection around each card.

---

## Hosting your own copy

If you want to fork this and host your own version (with a different name, different style, different prompt — whatever):

1. Fork the repo on GitHub.
2. Edit `index.html` however you like.
3. In your fork's **Settings → Pages**, enable Pages from the `main` branch.
4. Optionally point a custom domain at it (see the [GitHub Pages docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)).

Total cost: ~$10/year if you want a custom domain, $0 otherwise.

---

## Why this exists

There's a lot of writing in the world dressed up as fact-driven reporting when it's mostly opinion with statistics scattered through it for credibility. Sometimes the statistics are sourced; often they're not. Sometimes the writing hedges responsibly; often it asserts confidently on flimsy ground. Reading carefully enough to notice all of this — every time, on every article — is work. Most people don't do it.

Provenance is a small tool to make that work cheaper. Not to replace your judgment, not to tell you what's true, just to surface the sourcing and rhetorical structure of a piece at a glance so you can decide for yourself how much weight to give it.

---

## License

MIT. Do whatever you want with it — fork it, modify it, ship your own version. Attribution appreciated but not required.

---

## Acknowledgments

Built collaboratively with [Claude](https://claude.ai) over a single long conversation that started with "is this a money-making business?" (it wasn't) and ended with a working tool that costs nothing to run and helps a little with a real problem. The journey from idea to live URL is itself a small demonstration of what these tools are now capable of when used as collaborators rather than oracles.

If you find Provenance useful, the best thing you can do is share the link with one other person who reads a lot of news.
