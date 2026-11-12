---
name: local-service-ad-creative
description: >-
  Turn an agency's local-service ad plan (roofers, plumbers, home services) into Arcads creative that passes
  advertising and consumer-protection rules before a single credit is spent. Maps each ad's "Creative:" line to
  the right Arcads route (b-roll, Nano Banana still, scene), forbids synthetic customer testimonials, runs the
  agency's language rules on every script, and presents a credit estimate for approval. Use when the user asks
  for ad creative, Meta or Instagram ad videos, b-roll, or statics for a front-desk, receptionist or
  local-service offer, or mentions a client-facing ad plan file.
---

# Local-service ad creative (with guardrails)

The agency writes the ads; this skill makes the pictures and clips for them. It sits on top of
`arcads-external-api` and adds the rules that keep the creative honest. Read that skill for endpoints,
auth, polling and file layout.

## Hard rules (stop and say why; never work around)

1. **Generated people are spokespeople only.** A generated person never presents as a customer, a
   business owner, an employee of a real company, or anyone with a personal result ("this got me 12 jobs").
   No testimonials, reviews or before-and-after stories from generated people. The US FTC rule on fake
   reviews and testimonials (16 CFR Part 465) covers these, and so do the FTC Endorsement Guides.
2. **No invented proof.** No stats, counts, star ratings or rankings on screen unless the user gives the source.
   Never promise rankings, revenue or a number of jobs.
3. **Real businesses need written OK.** Creative that shows a real company's name, logo, truck or crew needs
   that company's written approval. Otherwise use a fictional demo company.
4. **Follow the ad platform's rules for realistic generated media.** Check the platform's current policy on
   labeling generated or altered content before launch; don't assume.
5. **The agency's language rules apply to every word a prospect sees** (script, captions, on-screen text).
   If the user's agency has a lint tool, run it. Otherwise apply the default list below.
6. **Every generation costs credits.** Estimate first, present the total as an estimate (Arcads has no billing
   endpoint), and wait for a yes. Log every call to `logs/arcads-api.jsonl`.

### Default language list (when the agency has no lint tool)

| Never | Say instead |
|---|---|
| "AI", "artificial intelligence" | front desk, assistant, agent, employee |
| the software vendor's name | the agency's own brand |
| "widget" | the benefit ("talk to us right here") |
| "answers every call" | "picks up when you can't" |
| "real person answers" | (the assistant never claims to be human) |
| "guaranteed #1 / first page / more leads" | (no guaranteed results) |

If the agency's repo is checked out next to this one, lint each script with
`python3 ../<agency-repo>/tools/language_lint.py --stdin` and fix every error before generating.

## Workflow

1. **Read the ad plan.** Ask for the ad file (for example an `ads/meta-ads-v1.md`). For each ad, copy the
   primary text, headline and the `Creative:` line.
2. **Pick a format per ad.** Prefer, in order:
   - **Demo audio:** a real recording of the demo line answering, with waveform and captions. Not generated;
     the user supplies the audio. Strongest proof because it's real.
   - **Phone-screen still or motion** (Nano Banana image, then optional b-roll): "Missed call 7:42 PM" →
     "Inspection booked". No people needed.
   - **Situation b-roll** (Kling via `POST /v1/b-roll`, 5 or 10 s, silent): storm sky, a phone buzzing on a
     ladder, a truck in a driveway. People appear only as anonymous workers, faces not featured.
   - **Spokesperson** (Veo 3.1 / Sora 2 / scene): only when the user asks, and only with a script that passes
     rule 1. The spokesperson explains the offer in the second person ("your phone", "your crew"); never "I" or
     "my company".
3. **Write the prompts** with the per-model guides in `arcads-external-api/prompting/`. Keep on-screen text
   short and lint it.
4. **Estimate and ask.** Table: ad, format, model, count, credits each (from `MASTER_CONTEXT.md`), total.
   Generate nothing until the user approves the total.
5. **Generate, poll, save** per `arcads-external-api`. One variant per ad first; more only on request.
6. **Hand back** a short list: ad name → file path → format → which rules were checked. Flag anything a person
   must still confirm (platform labeling, real-company approval, the source for any number on screen).

## Refuse, with one line of why

- "Make a happy customer saying we booked 40 jobs" → generated testimonial, rule 1.
- "Put 4.9 stars on it" without a source → invented proof, rule 2.
- "Use ABC Roofing's logo" without their written OK → rule 3.
