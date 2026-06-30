# Rogue Raise: The Unhoused — Top 3 Build Ideas

**White Rabbit · 5 North Main Street, Ashland · July 2026 (~48-hour Raise)**

A focused shortlist for the opening ceremony. These deliberately steer *away*
from rebuilding field-survey collection (Bucket 2) — Survey123 / Counting Us
already solve that — and aim at the genuinely green-field, highest-leverage work.
Each pick is finishable in a weekend, has a clear owner, and is judged on whether
it's genuinely usable, not whether it's clever.

> How they fit together: **#3** produces the baseline, **#1** makes that baseline
> trustworthy by killing duplicates, and **#2** adds the human signal and
> lived-experience voice the other two can't capture.

---

## 1. The person-matching de-duper — "one human, one record"

*Bucket 1 — Clean the data we have. The foundation everything else stands on.*

Build a lightweight, CRM-style core that matches the same person across health
encounters, syringe exchange, meals, and self-report; assigns a **persistent
de-identified ID**; and surfaces a **confidence score** with a human-review
screen for the fuzzy cases ("possible same person — 0.82").

- **Don't write the matching math** — stand it up on **Splink** (probabilistic
  record linkage, runs on a laptop, links ~1M records in ~1 min).
- Make it robust to the **seven-plus ways homelessness gets recorded** in real
  charts (`HOMELESS` in the address field, "General Delivery," `NHA`, junk ZIP
  `ZZZZZ`, shelter address, a checkbox, hospital city/ZIP).
- Remember visit-level de-dup already happens upstream in the BioSense Platform;
  our job is the harder **person-level, cross-source** layer.

**Owner:** Steven Bagley (epidemiologist) — duplication and hand-matching notes
is his single biggest stated pain.

**Why it wins:** the most defensible deliverable. It turns "1,400 vs 800"
arguments into a number that holds up to scrutiny from every side — the page's
*data truth*.

---

## 2. Anonymous self-report — designed *with* the people in the room

*Bucket 3 — Add new data. The most human-centered, least-built piece.*

A phone-based, **opt-in, no-name** way for someone to "add their existence" — a
QR code / link handed out at meals and syringe exchange, with no authority figure
in the loop.

- The build is easy; the value is that it gets **co-designed during the opening
  ceremony with unhoused community members**, so it asks what they actually want
  to say rather than what we imagine.
- Scope in **spam / bad-faith guard rails** from the start.
- Lean on people's own phones, donated/old smartphones, and free hotspots — **no
  fixed kiosks** (vandalism, device management, cost).

**Owner:** Gwynne Head (harm reduction) — she already runs the trusted
touchpoints (Saturday meal, syringe exchange).

**Why it wins:** it's what makes the Raise legitimate rather than another room of
well-meaning people planning peanuts. Directly serves the north star — *creating
avenues for people to advocate for themselves is the point, not an afterthought.*

---

## 3. Local baseline — fork the syndrome definition + live ESSENCE dashboard

*The "foundation underneath it all," made real. Very finishable because most of
it already exists.*

- **Fork** the validated Washington / Idaho / Maricopa **ESSENCE homelessness
  syndrome definition**; tune it for Jackson / Josephine / Klamath; validate with
  the **CSTE Syndrome Definition Development toolkit**.
- **Replace** Steven's manual API → terminal → Excel pull with a live
  **NSSP-ESSENCE API** feed (Oregon OHA's instance).
- **Render** it through a prebuilt **Rnssp R Markdown** dashboard template rather
  than building reporting from scratch.

**Output:** a week-over-week baseline *trend* instead of a single January
snapshot — exactly the capability the event says unlocks a better, more frequent
count.

**Owner:** Steven Bagley (or split with whoever owns reporting).

**Why it wins:** produces the credible, ongoing baseline that the whole funding
argument depends on.

---

## What we're intentionally NOT building from scratch

- **Field survey collection** — use Survey123 (offline-capable) or Counting Us.
- **Syndromic-surveillance analytics** — use CDC `Rnssp` / `pynssp`.
- **The homelessness definition** — fork an existing validated one.
- **HMIS / system of record** — interoperate with the local CoC's HMIS, don't replace it.

## The two truths every project serves

- **Humanity** — start from the people we're trying to help, not from what's impressive to build.
- **Truth** — a credible, defensible count *and* a first-person story. One without the other persuades no one new.

---

### Sources

- Rogue Raise — White Rabbit — https://whiterabbitashland.com/rogue-raise
- See also: `docs/presentation.md`, `docs/existing-solutions.md`, `docs/contacts.md`
