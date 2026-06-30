# Existing Solutions & Tech to Use or Reference

A working catalog of apps, libraries, standards, and platforms relevant to the
Rogue Raise homelessness-data work (de-duplication, PIT count, syndromic
surveillance, self-report). Goal: **don't reinvent what already exists** — know
what to build on, what to interoperate with, and where the real gaps are.

Each entry notes what it is, why it matters to us, and whether it's a
*build-on*, *interoperate-with*, or *reference* item.

---

## 1. PIT count & field survey apps

These largely solve the "collect a survey in the field" problem. The collection
layer is mostly a solved problem — our leverage is elsewhere.

| Tool | What it is | Relevance | Use as |
|---|---|---|---|
| **Counting Us** (Simtech Solutions) | Volunteer-facing PIT survey app; orgs not on a region's HMIS can participate via a setup key | Closest off-the-shelf fit for a volunteer-driven count | Build-on / reference |
| **Survey123** (Esri) | Configurable form-based survey app; **works offline** and syncs when reconnected | Many 2025 PIT counts ran on it; proven offline-first pattern we need | Build-on |
| **Show The Way** | Mobile app for outreach workers/first responders to collect info from unsheltered people | Model for street-outreach capture and incentive flows | Reference |
| **PIT Regional Command Center** (PointinTime.info) | Real-time dashboard of incoming surveys, GPS tallies, summary reports for HUD CoC regions | Model for our live web dashboard + reporting | Reference |

**Offline is non-negotiable** — coverage is unreliable in the field, so any field
app needs a small local data bank that syncs to a web dashboard later.

---

## 2. HMIS platforms (the systems of record)

HMIS = Homeless Management Information System, the HUD-mandated system CoCs use
for reporting. Anything we build has to **interoperate with**, not fight, the
local HMIS. Worth confirming which vendor the Jackson County CoC uses.

| Vendor | Product | Notes |
|---|---|---|
| **Bitfocus** | Clarity Human Services | Largest market share among California CoCs |
| **WellSky** (formerly Mediware) | Community Services / ServicePoint | Long-standing, widely deployed |
| **Eccovia** | ClientTrack | Common mid-market CoC choice |
| **CaseWorthy / Foothold (AWARDS) / others** | various | Round out the approved-vendor lists |

*Action item: identify the local CoC's HMIS vendor before the Raise so integration assumptions are real.*

---

## 3. De-duplication & record linkage (our highest-leverage gap)

This is the unlock: turn messy, repeated records into **one trustworthy story
per person**, ideally without exposing identities. It's a mature science — we
build on existing libraries, we don't invent the math.

### Open-source libraries (build-on)

| Library | Strengths | When to reach for it |
|---|---|---|
| **Splink** (UK MoJ) | Fast, scalable probabilistic linkage; Fellegi-Sunter model; SQL backends (DuckDB/SQLite packaged); interactive diagnostics; links ~1M records on a laptop in ~1 min | Strongest default starting point for transparent probabilistic linkage |
| **dedupe** (Python) | Human-in-the-loop active learning for fuzzy matching | Structured fuzzy matching where a person can train the model |
| **Zingg** | Entity resolution on Spark / modern data stack | Larger-scale, data-engineering workflows |
| **Python Record Linkage Toolkit** | Modular: preprocessing → indexing → comparing → classification → evaluation | Prototyping / assembling a custom pipeline step by step |

### Privacy-preserving record linkage (PPRL)

Lets us match people **across organizations without sharing identities** — key
given HIPAA and the fact that no single org sees everyone.

- **Bloom-filter encodings** + EM-estimated match probabilities for privacy-preserved linkage.
- **De-identified Bayesian / fuzzy matching** that tolerates typos and partial/zero data.
- Demonstrated specifically for **joining health-care and homeless-services data** while preserving confidentiality — directly analogous to linking OHA/county health data with service-provider data.

**What to build:** a de-duper that assigns a persistent **de-identified match
ID**, emits a **confidence score** for fuzzy matches ("possible same person —
0.82"), and gracefully handles orphans and zero-data cases. Pair it with a
CRM-style client record as the human-facing layer.

---

## 4. Syndromic surveillance standards & libraries

The backbone for the continuous baseline/trend track. National, standardized,
and free — build on these rather than rolling our own analytics.

| Resource | What it is | Use as |
|---|---|---|
| **Rnssp** (CDC, R) | Official NSSP analytics toolkit; standardized data pulls, templates, `get_essence_data()` to hit the NSSP-ESSENCE API | Build-on |
| **pynssp** (CDC, Python) | Python equivalent of Rnssp | Build-on |
| **NSSP-ESSENCE** | CDC's secure web tool/API for the surveillance workflow | Interoperate-with |
| **ICD-10 Z-codes** | Housing status inside clinical data: `Z59.0` homelessness (+ `Z59.00/.01 sheltered/.02`), `Z59.811` at-risk | Reference / standard |

> Today's workflow is a manual API → terminal → Excel pull. The obvious upgrade is
> a **live ESSENCE/API integration** using Rnssp/pynssp, with date-range params
> driven by the tool instead of by hand.

---

## 5. Precedents & comparable programs

- **New Mexico augmented count** — surveillance-style approach producing counts far higher than one-time PIT figures. Request their reports and, ideally, their code as a starting point. *(Open question: can we get the data-collection method, not just the outputs?)*
- **Continuum of Care (the local count operator)** — a nonprofit currently runs the 1-2-3 counts. Identify them, map the players, and consider inviting them as a judge so their data can come to the table.

---

## 6. Self-report & anonymous reporting

The genuinely under-built area. Aim: let people "add their existence" on their
own terms, anonymously, without going through an authority figure.

- **Pattern, not product:** a mobile-friendly opt-in web app / shareable link, no name required. Carefully designed against spam and bad-faith reporting.
- **Device reality:** kiosks were explicitly ruled out (vandalism, device management, cost). Lean on people's own phones, donated/old smartphones, and free hotspots/Wi-Fi logins instead.
- **Trauma-informed design** (safety, trust/transparency, choice, voice) and **real incentives** (bus passes, meals, agency) are what drive participation — the tech is the easy part.

---

## 7. Quick reference — links

- Splink — https://moj-analytical-services.github.io/splink/index.html
- dedupe / Zingg / data-matching software list — https://github.com/J535D165/data-matching-software
- CDC Rnssp — https://github.com/CDCgov/Rnssp
- CDC pynssp — https://github.com/CDCgov/pynssp
- CDC NSSP / ESSENCE — https://www.cdc.gov/nssp/index.html
- ICD-10-CM Z59.0 — https://www.icd10data.com/ICD10CM/Codes/Z00-Z99/Z55-Z65/Z59-/Z59.0
- PPRL for health + homeless data — https://pmc.ncbi.nlm.nih.gov/articles/PMC8489603/
- De-identified Bayesian matching — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10163749/
- Counting Us / PIT support (Simtech) — http://www.simtechsolutions.com/homeless-data-services/point-in-time-count-support/
- Survey123 / 2025 PIT resources — https://cohmis.zendesk.com/hc/en-us/articles/32805589816205-Balance-of-State-2025-Point-in-Time-Count-Resources
- PIT Regional Command Center — https://www.pointintime.info/regional-command-center/
- WellSky HMIS — https://wellsky.com/hmis-software/
- California CoC / HMIS vendor landscape — https://homelessstrategy.com/california-continuums-of-care-and-homeless-management-information-system-hmis-vendors-who-they-are-and-next-steps/
