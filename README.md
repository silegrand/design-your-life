# Design Your Life

A voice-first, unhurried AI conversation that helps older adults reflect on what matters to them, on their own terms. Powered by Claude Opus 4.7. Entry stage of the Assistiv platform for ageing-in-place in the UK.

Built for the Anthropic Hackathon, April 2026.

---

## Try it

**Live demo:** [silegrand.github.io/design-your-life](https://silegrand.github.io/design-your-life)

You will be asked to paste your own Anthropic API key at the start. The key is held in browser memory only. Nothing is saved, logged, or transmitted anywhere except directly to the Anthropic API. Close the tab and everything is gone.

The conversation takes about twenty minutes. It works best in Chrome (voice input uses the Web Speech API).

## What it is

Design Your Life is a single-file HTML application. Approximately 3,350 lines. No backend, no database, no build step. Everything runs in the browser, with Anthropic's API called directly.

The conversation follows a Motivational Interviewing structure: eighteen questions across four sections, with warm reflections after each answer, ending in a personalised **Staying Well Plan** and — if the person chooses — a **Summary for their GP**.

Four parallel Claude Opus 4.7 calls run for every answered question: a warm reflection, plus three safety classifiers (hostility, crisis, safeguarding). This is the architecture that lets the platform listen well while holding firm clinical boundaries.

For the reasoning behind every design decision, read **[DESIGN_NOTES.md](./DESIGN_NOTES.md)**.

## Running locally

No installation. No dependencies.

1. Clone the repo: `git clone https://github.com/silegrand/design-your-life.git`
2. Open `index.html` in Chrome (double-click, or `File → Open File`)
3. Paste your Anthropic API key when prompted
4. Begin

Alternatively, serve it locally with any static server:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000` in Chrome.

## Repository contents

| File | Purpose |
|------|---------|
| `index.html` | The entire application. Single file. |
| `DESIGN_NOTES.md` | Clinical reasoning, safety architecture, design principles |
| `Claude_MD_V11.docx` | Full working specification (for those who want the complete account) |
| `README.md` | This file |

## Technical architecture

**Model:** `claude-opus-4-7` across all calls.

**API calls per session:**
- Five to seven concurrent calls per answered question (reflection, hostility check, crisis check, safeguarding check, optional voice summariser)
- One call for the Staying Well Plan
- One call for the GP Summary (if requested)
- Approximately $1–$2 of Opus 4.7 usage per complete session

**Voice input:** Chrome Web Speech API. Verbatim or tidied (tidied uses a dedicated Opus 4.7 call).

**Browser support:** Chrome recommended for voice. The typing path works in any modern browser.

**Data:** No persistence. No database. No backend. API key held in memory for the session and discarded on tab close.

## Safety architecture

Every answered question runs four concurrent Opus 4.7 classifiers. Results are evaluated in strict precedence: crisis > safeguarding > hostility > reflection. Only one outcome is presented to the person.

Each classifier fails closed: on API error, the safer classification is assumed.

- **Crisis detection** → Samaritans, Shout text line, 999/111
- **Adult safeguarding detection** (UK Care Act 2014 categories) → Hourglass, local council safeguarding team, 999
- **Hostility detection** → Sovereignty Pause with two equal-weight buttons (carry on, end here)
- **Reflection** → warm MI reflection if no classifier fires

The safety screens signpost and stop. The software does not attempt to hold anyone in acute distress or process safeguarding disclosures. That belongs to humans who are trained and present.

## Licence

Source provided for hackathon review. All rights reserved. Please contact for licensing.

## Credits

Design Your Life is part of the Assistiv platform.

Built by Simon Legrand, with support from Claude (Anthropic) in an iterative coproduction mode: the clinical direction and final wording are mine; Claude wrote the code, proposed structures, and pushed back where useful. Every MI principle, every word choice on the screens, every safety threshold was decided and approved before it was shipped.

Clinical grounding: fifteen years working in UK health and social care.

## A note

This platform exists because the Missing Middle — millions of UK older adults living independently but at risk of sudden decline — deserve to be heard before they are in crisis. Not assessed. Heard.

If this software can reach one person who has given up trying to find help, and let them feel, for twenty minutes, that their own words matter, then it has done its job.
