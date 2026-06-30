# Counting People, Not Just the Counted
### Better homelessness data for Jackson, Josephine & Klamath counties

**Rogue Raise · July 17–19, 2026**
Presentation content / slide deck draft — audience: builders, designers, and volunteers at the Raise

> Working goal in one line: *Build the trustworthy baseline data that unlocks the funding, the better count, and the honest narrative.*

---

## Slide 1 — Why we're here

Two parallel jobs, one mission: **tell the accurate story of homelessness in our region so resources get allocated honestly.**

- **The PIT count** is event-driven and federally required once a year (always January — cold, people in warming shelters or gone, widely seen as an undercount).
- **Syndromic surveillance** is continuous (day-over-day, week-over-week) and can build a *baseline* that makes the PIT count smarter and lets us run counts more often than the federal minimum.

The insight from the June 22 working session: you use the ongoing surveillance data to establish the baseline, the baseline sets the targets for the PIT, and the trend data over time becomes the thing commissioners and funders can actually act on.

> "The reason we need that first data to work is so we can get the money to do the second data the way we want." — paraphrased from the working session

---

## Slide 2 — The problem, in the field's own words

- Current methodology is **inconsistent and unclear** — the count isn't trusted, by either side of the political debate.
- **Duplication is the single biggest data hurdle.** The same person shows up multiple times across notes and systems with no reliable way to match them.
- Data lives siloed: each jurisdiction protects its own datasets; there's no single org that sees everyone (and for good reason — different needs, HIPAA, trust).
- The people being counted have **little reason to participate** and real reasons to distrust the process.
- Public health data is *easily weaponized* — a loose, assumption-driven narrative hands ammunition to counter-narratives.

**Design implication for the Raise:** the hard part isn't the technology for gathering data — it's manpower, trust, and de-duplication. Build for those.

---

## Slide 3 — Three buckets of work (pick your team)

The work sorts cleanly into three tracks. A blue-sky "PIT Cover Tool" sits on top of all three, but the foundation has to come first.

1. **Clean & de-duplicate the data we already have.**
   Turn messy, repeated records into one trustworthy story per person. (This is the unlock — everything else depends on it.)
2. **Improve the Point-in-Time count itself.**
   Better instrument, better coverage, offline-capable, less burden on volunteers.
3. **Add new data.**
   Capture what's currently invisible: meal services, syringe exchange, Narcan distribution, self-report.

> These are roughly linear, not just tiered: you can't get to a credible PIT (and the budget it unlocks) until the baseline data is clean.

---

## Slide 4 — What already exists vs. what we build

The most useful reframe for a 48-hour Raise: **much of the surveillance scaffolding is already done and free.** Don't burn the weekend rebuilding it.

**Already exists (adopt / configure):**

- Oregon OHA's **ESSENCE** instance and the **ESSENCE API** for automated pulls
- CDC **Rnssp / pynssp** libraries + ready-made **RMD report templates**
- Validated **homelessness syndrome definitions** (WA / Idaho / Maricopa)
- Upstream **visit-level de-duplication** in the BioSense Platform
- Mature **record-linkage** and **privacy-preserving linkage** libraries (Splink, dedupe, PPRL methods)

**Genuinely green-field (where the Raise should spend its energy):**

- **Person-level, cross-source de-duplication** — linking the same human across health, services, and self-report
- **Anonymous self-report** — opt-in, no-name, phone-based
- **The storytelling / counter-narrative layer** — making the data land with both audiences

> If a team is rebuilding something in the left column, redirect them. The win is in the right column.

---

## Slide 5 — Best practice #1: De-duplication is a record-linkage problem

This is a solved-ish science — we don't have to invent it.

- **Probabilistic record linkage** brings together records within one dataset (de-duplication) or across datasets using pairwise comparisons and matching probabilities (commonly via an expectation-maximization approach).
- **Privacy-preserving record linkage (PPRL)** lets you match people *without* exposing identities — using techniques like Bloom-filter encodings and Bayesian fuzzy matching that tolerate typos and partial data.
- This has been done specifically to **join health-care and homeless-services data** while preserving confidentiality — directly relevant to linking OHA/county health data with service-provider data.

**Where the real work is — be precise.** Visit-level de-duplication already happens *upstream*: the CDC BioSense Platform (behind ESSENCE) assigns a unique identifier per visit and collapses a patient's visit data into a single record. Our green-field job is the harder **person-level, cross-source** layer — matching the same human across health data, syringe exchange, meals, and self-report when there's no shared key.

**The messiness it must handle.** In real records, homelessness alone is documented at least seven different ways: `HOMELESS` typed in the address field, "General Delivery" + a post office, `NHA` + hospital ZIP, a pre-designated junk ZIP (e.g. `ZZZZZ`), a shelter address, a homeless checkbox, or just the hospital's own city/ZIP. The de-duper has to be robust to exactly this kind of inconsistency.

**What to build:** a de-duper that assigns a persistent (de-identified) match ID, surfaces a **confidence score** for fuzzy matches ("possible same person — 0.82"), and gracefully handles orphans and zero-data cases. Think CRM-style client record + a normalization/matching layer on top.

*Sources: PPRL for health + homeless data (PMC8489603); de-identified Bayesian matching (PMC10163749); probabilistic linkage overview (PMC4460842).*

---

## Slide 6 — Best practice #2: Use the national standards & libraries

Don't start from a blank file. There are standardized, national methods we can build on.

- **CDC NSSP libraries** — `Rnssp` (R) and `pynssp` (Python) are the CDC's official syndromic-surveillance toolkits. They standardize data pulls and analytics and include `get_essence_data()` to pull from the **NSSP-ESSENCE API**. (Note: today's pull is API → terminal → Excel by hand; a live ESSENCE/API integration is the obvious upgrade.)
- **Ready-made report templates** — the `Rnssp` R Markdown templates are pre-built reports (State Emergency Department, Text Analysis Dashboard, ESSENCE CCDD Categories, Syndrome Definition Evaluation, Data Quality Filter Matrix). A lot of "the dashboard" is template configuration, not from-scratch code.
- **ICD-10 Z-codes for housing status** — `Z59.0` (homelessness) and its more specific children `Z59.00`, `Z59.01` (sheltered), `Z59.02`, plus `Z59.811` (housing instability, at risk). These are how housing status rides along inside clinical data from clinics, hospitals, and urgent cares.
- **Local reality:** Oregon OHA runs its own **ESSENCE** instance (with user guides plus weather/environmental and mass-gathering modules). That's the system Steven already pulls from — the weather data supports heat-illness surveillance, and the mass-gathering module fits event-driven counts at meals/exchanges.

### The big one: we don't have to invent a homelessness definition

The NSSP Knowledge Repository already hosts validated ESSENCE **homelessness syndrome definitions** from Washington State, Idaho, and Maricopa County. WA's is copy-pasteable today:

```
(Z590 OR Z59.0 OR HOMELESS OR NO HOUSING OR LACK OF HOUSING OR WITHOUT HOUSING OR SHELTER)
ANDNOT (ANIMAL SHELTER OR DOMESTIC VIOLENCE SHELTER OR DV SHELTER OR DOG OR CAT)
```

It queries Chief Complaint, Discharge Diagnosis, and Triage Note. **Plan: fork an existing definition and tune it for Jackson / Josephine / Klamath**, validating with the CSTE Syndrome Definition Development toolkit — rather than building from zero.

### Precedent: this method already works on homeless populations

Beyond New Mexico's augmented count (numbers far higher than one-time PIT), there's documented use of ESSENCE *on homeless populations*:

- **Miami-Dade** — tracked a GI outbreak in a homeless shelter
- **San Francisco** — early detection of tuberculosis among the homeless
- **Maricopa County & LA County** — detected Hepatitis A outbreaks among people experiencing homelessness
- **Atlanta** — identified a *Streptococcus pneumoniae* cluster

These map directly to Steven's stated interest in outbreak/batch detection (fentanyl, heat) and to the Hep C / HIV rates raised in the June 22 session.

*Sources: CDCgov/Rnssp & pynssp + Rnssp RMD templates (GitHub/cdcgov.github.io); CDC NSSP/ESSENCE; Oregon OHA ESSENCE guides; NSSP Knowledge Repository "homeless" tag + WA DoH syndrome definition; ICD-10-CM Z59.0.*

---

## Slide 7 — Best practice #3: Meet people where they are

The technology is easy; trust and participation are the real constraints. Lessons from the field and from trauma-informed research converge.

- **Trauma-informed survey design** rests on six principles: safety; trustworthiness/transparency; peer support; collaboration; empowerment, voice & choice; and cultural/historical/gender sensitivity. Data collection is relationship-building, not just extraction.
- **Leverage existing, trusted touchpoints** — the Saturday library meal, syringe exchange, Narcan distribution, community events. People show up where they're respected and where there's something real for them.
- **Give people something back.** Incentives that came up: bus passes, rewards, and crucially *agency* — let people self-advocate and tell their own story rather than be spoken for.
- **Self-report, anonymously.** A way to "add their existence" without giving a name or going through an authority figure — opt-in, on their own terms.
- **Represent the population in the room.** Have unhoused community members speak and help judge what gets built, so we don't build something beautiful that nobody uses. (The "we don't have teeth — who's going to eat the carrots?" lesson.)

*Sources: Switchboard, "Why Trauma-Informed Surveys Matter"; AIR Trauma-Informed Organizational Toolkit; USICH data guidance.*

---

## Slide 8 — Existing apps & tools (don't reinvent these)

| Tool | What it does | Relevance |
|---|---|---|
| **Counting Us** (Simtech Solutions) | Volunteer PIT survey app; works for orgs not on a region's HMIS via a setup key | Closest off-the-shelf fit for a volunteer-driven count |
| **Survey123** (Esri) | Form-based survey app; **works offline**, syncs when connected | Many 2025 PIT counts ran on this; proven offline pattern |
| **Show The Way** | Outreach app for collecting info from unsheltered people | Model for street-outreach data capture |
| **PIT Regional Command Center** | Real-time dashboard of incoming surveys, GPS tallies, summary reports | Model for the live web dashboard |
| **HMIS platforms** (e.g. WellSky) | The system of record CoCs use for HUD reporting | What any new tool must interoperate with, not fight |
| **Rnssp / pynssp** | CDC syndromic-surveillance analytics | Backbone for the baseline/trend track |
| **Rnssp RMD templates** | Pre-built ESSENCE report templates (ED report, text-mining dashboard, syndrome eval) | Shortcut for the reporting/dashboard layer |
| **Oregon OHA ESSENCE** | Oregon's own ESSENCE instance + Z-code/health data | The actual local data source Steven pulls from |
| **KR homelessness syndrome defs** | Validated WA / Idaho / Maricopa definitions | Fork-and-tune instead of inventing a definition |

**Takeaway:** the gaps worth building into are (a) **de-duplication / confidence-scored matching**, (b) **anonymous self-report**, and (c) **a unified view** that ties surveillance baseline → PIT → service data together. The survey-collection layer is largely solved.

*Sources: Simtech Solutions; Survey123 / Balance of State 2025 PIT resources; PointinTime.info; WellSky.*

---

## Slide 9 — Possible builds (the menu for teams)

What a team could realistically own across 48 hours, each with a named owner afterward:

- **The De-Duper / CRM core** — client records + probabilistic/privacy-preserving matching layer, persistent de-identified IDs, confidence reports for fuzzy matches. *(Bucket 1 — highest leverage.)*
- **Anonymous self-report channel** — a mobile-friendly web app/link people opt into (no name required); design carefully against spam and bad-faith reporting.
- **Volunteer recruitment + field app** — sign-up + light training/checklist + a mobile survey tool that **works offline** with a small local data bank, syncing to a web dashboard.
- **Live dashboard** — web-based view of trends and counts for the "bridge" role to take to commissioners. *Start from an `Rnssp` RMD template rather than a blank canvas.*
- **ESSENCE/API live integration** — replace the manual API→Excel pull with a real pipeline using `Rnssp`/`pynssp`. The ESSENCE API is built for exactly this: it returns aggregated tables (CSV/JSON) or time-series images for recurring situational reports.
- **Triage-note text mining** — extract housing status and de-dup signals from free-text Chief Complaint / Triage Notes (where the real context lives). CDC's free-text coding series + the `Rnssp` Text Analysis Dashboard template give a head start.
- **Local syndrome definition** — fork the WA / Idaho / Maricopa homelessness definition, tune for our three counties, and validate with the CSTE Syndrome Definition Development toolkit. Small, high-value, finishable in 48 hours.
- **Storytelling / counter-narrative layer** — pairing the sanitized quantitative narrative with first-person stories, so we reach both the empathetic *and* the fact-oriented audience.
- **(Backlog) NSSP knowledge base** — scrape the CDC NSSP Vimeo library + Whisper-transcribe into a searchable reference. A nice agentic side-quest, not core.

**Constraints to honor:** must work offline; built for the actual end-user's existing tools; resource-constrained county agency (no budget for kiosks/devices — kiosks were explicitly ruled out for vandalism/management cost); and every build needs an owner after the weekend.

---

## Slide 10 — The north star

Two virtues to build toward:

1. **Serve the humanity** of people struggling in our communities — give them voice and agency, not just a row in a dataset.
2. **Serve the truth** — get as close as we can to what is real, because every lever we pull downstream depends on trustworthy data.

If we nail the foundation, the payoff is concrete: a credible count → the budget for bus passes, phones, and volunteers → a better count next year → a longitudinal trend an epidemiologist can act on. *It's an empathy-generating moment backed by data nobody can easily weaponize.*

---

## Appendix — Open questions to resolve before/at the Raise

- Can we get the **New Mexico** reports and starter code?
- Who is the **nonprofit currently running the 1-2-3 counts** (the Continuum of Care entity)? Get a map of the players. Could we invite them as a third judge and bring their data in?
- What's the **persistent identifier** situation in the source data — how reliable, how many orphans?
- Which **data sources** do we actually have access to by July 17 (OHA server, demographic/health/Z-code data, arrest records?, Narcan distribution)?
- Who from the **community / unhoused population** will speak and help evaluate?
- Wish-list **volunteer pool size**, drawing on existing community networks rather than ground-building.

---

### Sources
- CDC NSSP R package — https://github.com/CDCgov/Rnssp
- CDC NSSP Python package — https://github.com/CDCgov/pynssp
- CDC NSSP / ESSENCE overview — https://www.cdc.gov/nssp/index.html
- CDC ESSENCE onboarding toolkit — https://www.cdc.gov/nssp/php/onboarding-toolkits/essence.html
- Rnssp R Markdown report templates — https://cdcgov.github.io/Rnssp-rmd-templates/usage/
- NSSP Knowledge Repository — "homeless" tag — https://knowledgerepository.syndromicsurveillance.org/tags/homeless
- WA DoH ESSENCE homelessness syndrome definition — https://knowledgerepository.syndromicsurveillance.org/homelessness-washington-state-department-health
- Oregon OHA ESSENCE user guide — https://www.oregon.gov/oha/PH/DISEASESCONDITIONS/COMMUNICABLEDISEASE/PREPAREDNESSSURVEILLANCEEPIDEMIOLOGY/ESSENCE/Documents/userguide.pdf
- ICD-10-CM Z59.0 Homelessness — https://www.icd10data.com/ICD10CM/Codes/Z00-Z99/Z55-Z65/Z59-/Z59.0
- Privacy-preserving record linkage, health + homeless data — https://pmc.ncbi.nlm.nih.gov/articles/PMC8489603/
- De-identified Bayesian matching — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10163749/
- Probabilistic record linkage (P3RL) — https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4460842/
- Counting Us / PIT support (Simtech) — http://www.simtechsolutions.com/homeless-data-services/point-in-time-count-support/
- Survey123 / Balance of State 2025 PIT resources — https://cohmis.zendesk.com/hc/en-us/articles/32805589816205-Balance-of-State-2025-Point-in-Time-Count-Resources
- PIT Regional Command Center — https://www.pointintime.info/regional-command-center/
- WellSky HMIS — https://wellsky.com/hmis-software/
- Trauma-informed surveys (Switchboard) — https://www.switchboardta.org/why-trauma-informed-surveys-matter-and-how-to-create-them/
- Trauma-Informed Organizational Toolkit (AIR) — https://www.air.org/sites/default/files/downloads/report/Trauma-Informed_Organizational_Toolkit_0.pdf
- USICH — Use Data and Evidence — https://usich.gov/fsp/use-data-and-evidence-to-make-decisions
