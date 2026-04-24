# Design Notes

**Design Your Life** is the onboarding conversation for the Assistiv platform. It is a single-file HTML application deployed at [silegrand.github.io/design-your-life](https://silegrand.github.io/design-your-life). These notes explain the clinical reasoning behind the design.

The full working specification lives in `Claude_MD_V11.docx` for those who want the complete account. This is the shorter version.

## The problem

Three and a half to four million UK older adults live independently but at risk of sudden decline. They are not ill enough for the health system, not poor enough for social care, and not old enough in the system's terms to be flagged by anyone. We call them **the Missing Middle**.

Within that population sits a sharper category: **the Lone 9 percent**. These are older adults who live alone, have no close advocate, no family nearby, often no smartphone, and no easy entry point to any service. They are structurally invisible to most digital interventions.

Existing tools fail this population in two ways. First, they arrive too late — typically after a fall, a hospital admission, a bereavement. Second, they feel like a threat to identity for people who value their independence. The problem is not that assessments take weeks. It is that weeks are spent trying to find an assessment, before anyone has heard them at all.

Design Your Life bypasses that navigation burden entirely. It is a single front door that takes twenty minutes, requires no registration, retains no data, and hears the person first.

## The clinical foundation

The conversation is built on **Motivational Interviewing**. Open questions, affirmations, reflective listening, summary. The software never prescribes, advises, or warns. It asks, it hears, and it reflects. The reflection is the work.

The platform is **hardwired for unconditional positive regard**. This is not a tone choice — it is enforced by every system prompt. The person cannot be wrong in this conversation. They can say anything, decline anything, push back on anything, and the software will continue to receive them without correction.

This is why Design Your Life can reach people who resist staffed services. A human professional, however kind, carries an implicit agenda (get the person into services, get them to accept help, complete a risk assessment). The software carries no agenda beyond receiving the person well. That is legible to an older adult, and it is why they do not need to defend themselves to it.

## The Sovereignty Pivot

The central design decision: **a person who says no is not a failed user. They are a successful one.**

This is implemented, not just stated. A hostility classifier detects when a person is withdrawing consent ("I don't want to do this", "none of your business", "leave me alone") and routes them to the Sovereignty Pause — two equal-weight buttons, one to carry on, one to end. Choosing to end produces a warm exit and no plan. That is the successful outcome for that person. They were heard, they exercised agency, and the software got out of the way.

Every decision point in the software preserves the person's ability to say no, and treats saying no as a valid destination rather than a failure state.

## The six life domains

Eighteen questions map invisibly to six life domains, each anchored by a validated instrument. The person never sees the instrument name. The domains face the person as:

- **Staying Sharp** — memory and concentration (SCD-Initiative)
- **Staying Steady** — confidence at home (ASCOT)
- **Staying Connected** — relationships and belonging (UCLA)
- **Staying Well** — mood, wellbeing, meaning (WHO-5)
- **Staying Safe** — home environment and safeguarding
- **Staying Busy** — purpose and meaningful activity (ICECAP-O)

Words like *decline*, *deterioration*, *frailty*, *assessment*, *monitoring*, and *patient* are banned in anything the person reads. They are reserved for clinical contexts where the reader is a GP.

## Safety architecture

Every answered question triggers **four concurrent Claude Opus 4.7 calls**: warm reflection, hostility check, crisis detection, and adult safeguarding detection. The results are evaluated in strict precedence:

1. **Crisis** (self-directed risk) wins first
2. **Safeguarding** (harm from another person) wins next
3. **Hostility** (withdrawal of consent) wins next
4. **Reflection** runs only if none of the above fire

Each classifier is narrow by design and fails closed: on API error, the safest classification is assumed. A false positive shows a caring screen with resources; a false negative misses a real signal.

### Crisis detection

Fires on explicit, present-tense, self-directed expressions of suicidal ideation, self-harm, or acute danger to self. Does not fire on general sadness, grief, past-tense difficulty, metaphorical language, or worry about the future.

When it fires, the person sees the Crisis Screen with UK resources: **Samaritans** (116 123, samaritans.org), **Shout** (text SHOUT to 85258), **999** for emergency, **111** for urgent non-emergency help.

### Adult safeguarding detection

Fires on explicit disclosures matching the UK Care Act 2014 categories: physical, psychological, financial, or sexual abuse; coercive control; enforced isolation; neglect by a carer; or fear of a specific person in the context of harm. Does not fire on ordinary relationship difficulty or historical abuse.

When it fires, the person sees the Safeguarding Screen with **Hourglass** (0808 808 8141 or text 07860 052906, wearehourglass.org), guidance to the local council's adult safeguarding team, and 999 for emergency.

### The signpost-not-treat boundary

Both safety screens hold a firm boundary: **the software signposts and stops.** It does not attempt to help further. Even human professionals know that in acute distress or safeguarding situations, the correct response is to hand off to someone trained and present, not to attempt treatment alone. Doing so is not a failure of care. It is care.

The continue button on both screens reads *"I have reached out, and I would like to continue."* This is an agentic statement by the person, not a safety certification. The software has no standing to certify safety. It registers the person's choice to return.

## What the person gets

If the person completes the conversation:

- A **Staying Well Plan** grounded in their own words — quoted verbatim, with opportunities surfaced only in the domains they actually raised
- An optional **Summary for your GP** in a formal but non-clinical register — no instrument names, no diagnostic categories, just observational language organised thematically for a GP to triage before a consultation

Both are printable. Both can be saved. Neither is uploaded anywhere.

If the person exits at any point, for any reason, they get a clean exit and the knowledge that nothing was stored.

## Technical notes

Single-file HTML, approximately 3,350 lines. Self-contained. Deployed as GitHub Pages. No backend, no database, no build step.

All Claude calls use Opus 4.7 (`claude-opus-4-7`). Five to seven parallel API calls per answered question depending on the path (reflection, hostility, crisis, safeguarding, and voice summariser if the person chose tidied transcripts). Two further calls per session for the plan and GP summary.

API key entry is on a dedicated screen at the start. Held in browser memory only. Never saved, never logged, discarded when the tab closes.

No data is retained beyond the session. The person leaves with only what they chose to print themselves.

## Why Opus 4.7

The four parallel classifiers that make this work would not be tractable on a weaker model. In particular:

- The **reflection** requires sustained understanding of MI register across eighteen sequential answers, without drifting into advice or explanation
- The **hostility classifier** must distinguish genuine withdrawal of consent from sharp-but-engaged register ("None of your business" is hostility; "Bloody hell, that's hard" is not)
- The **crisis classifier** must distinguish explicit present-tense self-directed risk from grief, metaphorical language, or past-tense difficulty
- The **safeguarding classifier** must distinguish a UK Care Act 2014 disclosure from ordinary relationship difficulty or historical abuse

Each of these is a narrow classification task by design, but narrow does not mean easy. Getting the thresholds right on clinically sensitive content is where model capability matters most.

The **GP Summary** is a distinct, single Opus 4.7 call that must sustain a specific register (formal but observational, never diagnostic, never naming instruments) across seven thematic sections while grounding every sentence in the person's own answers.

## What is out of scope

This submission is the onboarding only. The broader Assistiv platform (passive mmWave sensing for fall and decline detection in partnership with Kings Secure Technologies, a FHIR-aligned care wallet, Triple Tap Logic for consented escalation, and a modelled 14:1 return on investment across Kent ICB) sits outside this hackathon build. It exists and is documented elsewhere. It is not being demonstrated here.

Deferred to pilot phase:

- QR codes for multi-recipient professional handoff
- Locally-tailored Your Connections (currently Canterbury CT1)
- Real advisor phone number in Gentle Handover
- Proxy backend so users do not need their own API key
- Multilingual voice support

## Closing

The measure of success for this platform is not uptake. It is that every person who starts the conversation is treated as whole, regardless of whether they finish. A person who exits at the second question because they decided this was not for them is not a failed interaction. They were offered an open door and chose not to walk through it. That is their right, and the door remains open.

Design Your Life is the front door. The rest of Assistiv is what lies beyond it, for those who choose to walk through.
