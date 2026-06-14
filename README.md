# Coco Coffee — Feedback Signal

A small prototype that turns raw customer feedback into a tracked signal. Each comment is matched to a theme (e.g. *Service Speed*, *Coffee Quality*, *etc*) and a sentiment (positive / neutral / negative), which then roll up into a live dashboard.

The idea behind it is that most organisations have plenty of qualitative feedback — reviews, survey free-text, complaint emails — but it sits unread because nobody has time to read and tag it all by hand. This explores turning that into a tracked metric, with a person able to review, correct, split one comment into multiple issues, or remove it entirely.

## Demo

https://github.com/user-attachments/assets/665ba8f5-b07f-460b-a983-53dcf832a94c

## Try it

Open `index.html` in a browser, or deploy via GitHub Pages. No setup needed for the core tool — it runs entirely client-side using a rule-based categoriser.

An optional field lets you paste your own Anthropic API key to use live Claude categorisation instead. **An API key is required for this** — Anthropic's API needs a billable account to authenticate requests, the same as any third-party API. The key is used only in browser memory for that session; it is never saved, stored, or sent anywhere except directly to Anthropic.

## A note on the recording

The recording demonstrates the interface, the rule-based categorisation, and the review/edit/split/delete workflow. The *live Claude categorisation* path is shown separately below, using real prompts and responses tested directly against Claude — rather than via the in-app key field — since generating a billable API key carries a minimum spend that wasn't justified just for this demo. The prompt and output format are identical either way; only the calling mechanism differs.

## Categorisation prompt — tested examples

The tool sends a prompt of this shape:

> Existing themes: Coffee Quality, Service Speed, Staff Friendliness, Pricing, Ambience & Seating, Cleanliness, Mobile App & Ordering, Food & Pastries
>
> Customer feedback: "[feedback text]"
>
> Respond with ONLY a JSON object: {"theme": "...", "sentiment": "positive|negative|neutral", "isNewTheme": true|false}. Pick the single most prominent theme. Reuse an existing theme if it reasonably fits. Only set isNewTheme true and propose a new short theme if nothing existing fits.

Tested directly against Claude:

| Feedback | Response |
|---|---|
| "queued over 15 minutes for a matcha latte and when I got it, it wasn't mixed properly, all the powder was sitting at the bottom" | `{"theme": "Service Speed", "sentiment": "negative", "isNewTheme": false}` |
| "the loyalty app sent me a notification about a free birthday drink but when I tried to claim it in store the barista said the system doesn't show that" | `{"theme": "Mobile App & Ordering", "sentiment": "negative", "isNewTheme": false}` |
| "there's never anywhere to lock up my bike outside the Camden branch, I always have to leave it round the corner" | `{"theme": "Bike Parking & Accessibility", "sentiment": "negative", "isNewTheme": true}` |

The first example is a useful case: a single AI pass picks one theme (Service Speed) even though the feedback contains a second distinct issue (the drink wasn't made properly — a Coffee Quality problem). This is exactly the scenario the human review and split-issue feature is designed for — no automated single-pass categoriser, rule-based or AI, will always capture every issue in a comment.

## How it works

1. **Input** — a piece of free-text feedback is entered (or one of the example prompts is used).
2. **Categorisation** — by default, a rule-based keyword matcher assigns a theme and sentiment. If an API key is supplied, Claude is called instead using the prompt above. If a live call fails, it falls back to the rule-based matcher.
3. **Dashboard** — a running total of feedback logged, the overall positive percentage, the highest-volume theme, and the most recently added theme. A stacked bar chart shows volume per theme, split by sentiment.
4. **Human review** — every entry can be edited, split, or deleted. A reviewer can change the theme or sentiment, or split one comment into multiple tagged issues. Reviewed entries are marked accordingly.

## Design notes

- **Palette**: a pink and green colour scheme, paired with rounded pill-shaped tags, buttons and cards throughout — chosen to feel like a friendly, modern consumer app rather than an internal reporting tool.
- **Typography**: Poppins for headings, Inter for body text and UI elements.
- **Why a coffee shop**: a relatable, low-stakes domain to demonstrate a pattern (feedback → theme → sentiment → dashboard) that applies just as well to retail, financial services, or public sector feedback.

## Why I built it this way

A single automated pass over a comment is useful but rarely perfect — it picks one theme, and sometimes misses a second issue or the sentiment doesn't quite match the nuance of what's written (see the tested example above). Rather than treating that first pass as final, the design assumes a person will check it: tags can be corrected, split, merged, or removed, and the dashboard reflects whatever the current, reviewed state is.

The rule-based matcher and the Claude-based categoriser share the same input (feedback text + list of known themes) and output (theme + sentiment) shape, so either can be used behind the same interface and review workflow.

## Tech

- Vanilla HTML/CSS/JS, no framework, no build step
- Chart.js for the themes-by-volume chart
- Browser local storage for persistence
- Optional live categorisation via the Anthropic API (user-supplied key, session-only)
