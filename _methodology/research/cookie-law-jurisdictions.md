# Cookie legislation across our jurisdictions — EU / Romania / Moldova / Israel

> **Status:** practical-compliance research compiled from regulator publications, EDPB guidance, and 2024–2026 enforcement cases.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-15.
> **Scope:** `rapoport.studio` (marketing) and `app.rapoport.studio` (B2B SaaS workspace), operated by Rapoport Studio SRL (Moldovan SRL), serving founder-track customers across EU, Romania, Israel, and Moldova.
> **Bottom line:** one well-built EU-grade banner with a true "Reject all" on the first layer covers all four jurisdictions. Moldova flips from soft regime to GDPR-grade on 23 August 2026; Israel already turned GDPR-grade on 14 August 2025. There is no longer a "weak jurisdiction" to fall back on. Google Analytics is technically usable but operationally radioactive — default to Plausible (no banner) and gate everything else behind opt-in.

---

## 1. EU — GDPR + ePrivacy Directive (2002/58/EC, amended by 2009/136/EC)

### 1.1 The two-layer framework

Cookies in the EU are governed by **two** instruments operating in parallel, and you need to satisfy both:

- **ePrivacy Directive Art. 5(3)** governs *the act of storing or accessing information on a user's device* (cookies, localStorage, SDK identifiers, pixels, fingerprinting). It requires **prior consent** unless the access is strictly necessary.
- **GDPR** governs *the subsequent processing of personal data* once read. Lawful basis must exist (Art. 6) and consent, where used, must meet Art. 4(11)/Art. 7 quality (freely given, specific, informed, unambiguous, granular, revocable).

The ePrivacy Directive is **lex specialis** — if it requires consent, GDPR "legitimate interest" cannot override it. This is the single most common compliance mistake: claiming "we use LI for analytics." That argument was killed in CJEU *Planet49* (2019) and reaffirmed repeatedly since.

### 1.2 Status of the ePrivacy Regulation — withdrawn (Feb 2025)

After eight years of stalled negotiations the European Commission **formally withdrew the proposed ePrivacy Regulation on 11 February 2025**. There is no harmonized successor on the horizon. National divergence (each Member State has its own transposition of 2002/58/EC) is the new permanent state. Build for the strictest common denominator — which today is France (CNIL).

### 1.3 "Strictly necessary" — Art. 5(3) exemption

A cookie or storage operation is **exempt from consent** only if one of two cumulative tests is met:

1. It is **solely** for "carrying out the transmission of a communication over an electronic communications network", OR
2. It is **strictly necessary** to provide a service **explicitly requested by the user**.

In practice the EDPB's **Guidelines 2/2023 on the technical scope of Art. 5(3)** (finalized October 2024) confirmed an expansive reading: the rule covers cookies, localStorage, IndexedDB, cache APIs, IP-based identifiers, pixel tracking, URL/click-ID tracking, and device fingerprinting — anything that reads or writes to the terminal.

**Clearly exempt** (no consent needed):
- Authentication/session cookies (`__Host-...`, supabase `sb-auth-token` etc.)
- CSRF tokens
- Load-balancer / routing cookies
- Shopping-cart state for the duration of the session
- UI preference cookies *if the user actively set the preference* (e.g. language switcher)
- Security cookies (rate-limit fingerprints, fraud-detection identifiers in narrow scope)

**Not exempt** (consent required):
- Any analytics cookie not configured to the CNIL audience-measurement exemption (see §2.1)
- Advertising / retargeting / conversion pixels (Meta, LinkedIn, TikTok, Google Ads)
- Cross-site tracking, social plug-ins (X embed, YT embed if not in privacy mode)
- A/B-testing tools that persist beyond the session
- Anything that joins data with a third party

### 1.4 EDPB consent-banner rules ("Reject all" parity)

The **EDPB Guidelines 5/2020 on consent** plus the **Cookie Banner Taskforce Report (Jan 2023)** establish the de-facto rules every Member State now enforces:

- **First-layer parity:** "Reject all" must be reachable in the **same number of clicks** and with **equal visual prominence** as "Accept all". Hiding reject behind "More options" / "Manage" is a dark pattern. CNIL fined Google €150M (2021), Facebook €60M (2021), Microsoft €60M (2022), Yahoo €10M (2023), and **Google €325M + Shein €150M on the same day in September 2025** on this exact ground.
- **No pre-ticked boxes.** Implied consent (scrolling, continuing to browse) is invalid (CJEU *Planet49*).
- **Granular categories.** Must be able to accept analytics and reject marketing separately.
- **Withdrawal as easy as giving.** A persistent "Cookies" footer link or floating icon is the minimum.
- **No cookie walls** (unless an equivalent free path is offered).
- **Technical enforcement.** The November 2025 CNIL American Express fine (€1.5M) made it explicit: cookies must actually *stop firing* on withdrawal — not just flip a flag in the CMP. This is a recurring vendor-integration failure.

### 1.5 Consent-or-pay — EDPB Opinion 8/2024

The EDPB's **Opinion 8/2024** (April 2024) ruled that "consent or pay" models on large online platforms generally **do not yield freely-given consent**. A third, free, non-tracking option should be offered. Meta's legal challenge against the Opinion was dismissed by the EU General Court. This is relevant if you ever consider a "pay €X/year or accept tracking" wall — don't.

### 1.6 Headline penalties

GDPR caps fines at **€20M or 4% of global annual turnover**, whichever is higher (Art. 83(5)). National ePrivacy laws stack on top with their own caps — see §2.

---

## 2. National regimes — Germany, France, Italy, Spain

The four big regulators ranked by 2025 enforcement intensity: **CNIL (FR) >> Garante (IT) > AEPD (ES) > BfDI/LfDs (DE)**.

### 2.1 France — CNIL

- **Transposition:** Loi Informatique et Libertés Art. 82.
- **Banner rule:** strictly enforces first-layer reject parity. As of 2025, this is non-negotiable — CNIL's 2025 sanctions totalled **€486.8M across 83 decisions**, dominated by cookie violations.
- **Analytics carve-out (real, narrow):** CNIL's "Sheet n°16" allows *audience-measurement* cookies **without consent** if they are:
  - used only for site-internal statistics,
  - not shared with third parties,
  - not used to track individuals across sites,
  - IPs truncated and identifiers stored ≤ 13 months,
  - aggregated data retained ≤ 25 months.
  This is why **Matomo** (self-hosted, anonymized) and **Plausible** qualify; **Google Analytics 4** does not (data goes to Google = third-party share).
- **Recent fines 2026:** Free Mobile €27M and Free €15M in January 2026 alone — pace has not slowed.
- **2026 work programme:** CNIL announced in Dec 2025 it will work on "cross-domain consent" frameworks (single consent across same-group properties).

### 2.2 Germany — TDDDG (formerly TTDSG)

- **Transposition:** **TDDDG** (Telekommunikation-Digitale-Dienste-Datenschutz-Gesetz) — renamed from TTDSG on 13 May 2024. **§ 25 TDDDG** is the operative provision: prior informed consent required before any non-essential storage/access. Practically identical to ePrivacy Art. 5(3).
- **Special fine cap for § 25 violations:** **€300,000 per violation** (separate from GDPR's €20M / 4%).
- **DSK** (Datenschutzkonferenz — joint conference of German DPAs) demands **demonstrable, logged consent** (Nachweisbarkeit). Consent logs are not optional.
- **EinwV (Consent Management Ordinance), April 2025:** voluntary scheme for "recognized consent management services" (RCMS) — designed to address consent fatigue by letting users set preferences once via a recognized service. Approval by **BfDI**. Optional, not required.

### 2.3 Italy — Garante

- **Transposition:** Codice della Privacy + Garante's **Guidelines on cookies and other tracking tools** (10 June 2021, in force since 10 Jan 2022).
- **Strict positions:** scrolling = NOT consent. Banner must appear immediately on first visit. Granular categories required. Cookie walls broadly forbidden unless equivalent content is available without consent.
- **Legitimate interest for analytics = no.** Garante's position aligns with CNIL — only the audience-measurement exemption survives.
- **Enforcement:** Italy issued its first major GDPR fine over employee email metadata in June 2025. Cookie enforcement is steady but smaller-ticket than France.

### 2.4 Spain — AEPD

- **Transposition:** LSSI-CE Art. 22.2 + LOPDGDD.
- **AEPD's "Guía sobre el uso de las cookies"** updated July 2023 (effective 11 Jan 2024) and again May 2024 to align with EDPB Opinion 8/2024 (consent-or-pay).
- **Cookie walls partially allowed:** unique in EU — AEPD permits cookie walls **if** an equivalent alternative (paid or otherwise) is offered. This is a narrower carve-out than it sounds; the alternative must be genuinely equivalent.
- **Penalties:** up to **€30,000** under LOPD-GDD for cookie-specific infringements (this is the LSSI-CE cap, distinct from GDPR's €20M).

### 2.5 Summary — national differences that matter for `rapoport.studio`

| | France | Germany | Italy | Spain |
|---|---|---|---|---|
| Banner reject parity | mandatory (heavy enforcement) | mandatory | mandatory | mandatory |
| Cookie wall | forbidden | forbidden | forbidden (narrow exception) | allowed if equivalent alternative |
| Analytics without consent | only via CNIL exemption | only via similar narrow reading | only "technical" analytics | only if anonymized + 1st-party |
| ePrivacy-specific cap | (GDPR caps apply) | €300k per violation | (GDPR caps) | €30k LSSI-CE cap |
| Enforcement temperature 2025 | very hot | warm | warm | warm |

> **Practical take:** if you build for CNIL you are compliant in the other three. Build for CNIL.

---

## 3. Romania — Law 506/2004 + ANSPDCP

- **Transposition:** **Law 506/2004** on the processing of personal data and protection of privacy in the electronic communications sector implements ePrivacy Directive 2002/58/EC. **Art. 4(5)** governs cookies.
- **Regulator:** **ANSPDCP** (Autoritatea Națională de Supraveghere a Prelucrării Datelor cu Caracter Personal).
- **Substantive rule:** identical to EU — prior, informed, explicit consent before non-essential cookies. "Express consent" is the statutory phrase.
- **Pending amendment (2025–2026):** A draft amendment to Law 506/2004 is in Romanian Parliament. The key change: codifying first-layer reject-parity into national law — making it a statutory rather than just regulatory standard.
- **Penalty regime:** ANSPDCP can impose **administrative fines up to 4% of annual turnover** for cookie violations (mirroring GDPR ceilings), plus GDPR fines stack.
- **Enforcement reality:** ANSPDCP is **less aggressive** than CNIL/Garante. Fines in Dec 2024–Apr 2025 ran 15,000–30,000 RON (~€3k–€6k) for sites placing non-essential cookies without consent. But the precedent is set, and reject-parity will become statute.

> **For `rapoport.studio`:** Romania = no special handling beyond a competent EU banner.

---

## 4. Moldova — Law 133/2011 (until Aug 2026), Law 195/2024 (from 23 Aug 2026), Law 241/2007 (ePrivacy-equivalent)

This is the jurisdiction that **changes most** during the relevant horizon.

### 4.1 Current state (until 22 August 2026)

- **Law 133/2011** on personal data protection is the operative regime. Comparable to a pre-GDPR Data Protection Directive law — consent-based, with NCPDP (Centrul Național pentru Protecția Datelor cu Caracter Personal) as regulator. Substantially weaker than GDPR on online identifiers.
- **Law 241/2007** on electronic communications + amendments via **Law 135/2017** (in force 18 Aug 2017) added an "electronic communications confidentiality" chapter — Moldova's *de facto* ePrivacy-equivalent. It covers interception, traffic/location data, and unsolicited communications, but does **not** spell out a cookie-consent rule the way ePrivacy 2002/58/EC Art. 5(3) does.
- **Law on Information Society Services (2004)** obliges service providers to disclose the "mechanisms used to monitor online activity (including cookies) for marketing purposes and the procedure for obtaining consent" — i.e. a transparency duty rather than a hard opt-in.
- **NCPDP** has not published a dedicated cookies guidance. The NCPDP's own site uses a generic cookie notice (Romanian-language "We use cookies" banner) — the regulator itself is not modelling EU-grade practice yet.

### 4.2 New state from 23 August 2026 — Law 195/2024

- **Adopted:** 25 July 2024, **published 23 August 2024**, **enters into force 23 August 2026** (24-month vacatio legis).
- **Effect:** substantially transposes GDPR into Moldovan law. Once active, Moldovan online-identifier rules will be **GDPR-equivalent**: cookies that read/write to a device combined with online identifiers will count as personal data; consent required on Art. 4(11)-style "freely given, specific, informed, unambiguous" terms.
- **Enforcement:** NCPDP gets GDPR-style powers and fining authority (specific cap structure not yet announced via implementing acts at time of writing).
- **Territorial scope:** processing of personal data of Moldova residents — including foreign sites marketing to Moldovan customers. Same extraterritorial logic as GDPR Art. 3(2).

> **Practical:** as a Moldovan SRL serving an EU customer base, **you are already obliged to apply GDPR** for that audience. Law 195/2024 makes the Moldova-domestic regime catch up, not impose anything new on top of what you must already do for EU users. Build for EU and Moldova is automatically covered.

---

## 5. Israel — Protection of Privacy Law 5741-1981 + Amendment 13 (effective 14 August 2025)

Israel is treated by the EU as having "adequate" status (Commission adequacy decision 2011, re-confirmed in 2024) — but the *domestic* regime moved sharply toward GDPR equivalence with Amendment 13.

### 5.1 Amendment 13 — what changed (effective 14 Aug 2025)

- **Adopted:** August 2024.
- **In force:** **14 August 2025** (one-year vacatio legis).
- **"Personal information" redefined** to expressly include **online identifiers, IP addresses, cookies, and behavioural data** — closing the gap with GDPR Art. 4(1).
- **Consent requirements upgraded:** explicit, informed, freely given, granular, documented. Bundled/blanket consent is invalid. Pre-ticked boxes invalid.
- **Privacy Protection Authority (PPA)** gains: administrative fines (millions of shekels with multipliers for large/sensitive databases), cease-and-desist powers, suspension of processing, criminal investigation.
- **Exemplary damages** of up to NIS 10,000 (~$3,000) per claim available in private actions.

### 5.2 Israeli cookie banner specifics

- **No standalone ePrivacy-equivalent law.** Cookies are regulated through the PPL itself + PPA "Cookies & Similar Technologies Guidance Note" (issued by the PPA's predecessor and updated under Amendment 13).
- **Consent required** for cookies that process personal data — which post-Amendment 13 includes most analytics/marketing cookies.
- **"Strictly necessary" exemption exists in PPA guidance** — cookies "strictly necessary, from the users' perspective, for the provision of the business's services" do not require explicit consent. The carve-out is narrower than EU's Art. 5(3) in form but functionally similar. Operational/security/auth cookies are out of scope of consent.
- **Banner expectations:** disclose categories, purposes, controllers, and obtain affirmative action. PPA expects clear opt-in for analytics/marketing. **A privacy-policy link in the footer is not enough.**
- **No formal "reject-as-easy-as-accept" doctrine** like EDPB's, but PPA explicitly disapproves of pre-ticked boxes, dark patterns, and bundled consent — same outcome with different language.

### 5.3 Territorial scope

The PPL applies based on data-subject location, not server location. A Moldovan SRL serving Israeli founders falls within scope. Practically: **the same consent banner that satisfies EU users will satisfy the PPA**.

---

## 6. Analytics-tool decisions for rapoport.studio in 2026

This is where the practical money is.

### 6.1 Google Analytics 4

**Status:** Conditionally legal, but you don't want it.

- Requires opt-in consent before script loads.
- Requires Google Consent Mode v2 properly wired.
- Requires DPA with Google, IP anonymization, disabled data sharing, data retention configured.
- EU-US Data Privacy Framework (July 2023) made transfer legal in principle, but CJEU challenges are pending — the framework could be invalidated like Privacy Shield was. Carrying GA4 means carrying that risk.
- Falls **outside** the CNIL audience-measurement exemption (data goes to Google = third-party share). So even with consent it will not give you reject-rate-immune analytics.
- Implementation cost in a B2B SaaS context is rarely worth what you get back vs. cookieless alternatives.

**Recommendation: do not use GA4.**

### 6.2 Plausible

**Status:** No consent banner needed. Cookieless by design, no personal data stored, no fingerprinting.

- Built specifically to qualify for the CNIL audience-measurement exemption.
- EU-hosted (Germany), no US transfer.
- Counts every visitor (no reject-rate hit).
- ICO, CNIL, AEPD, Garante all treat it as compliant without consent.
- Data Policy last refreshed March 2026 — no material changes.

**Recommendation: use Plausible as the primary analytics on the marketing site.**

### 6.3 PostHog

**Status:** Consent-required *by default* (writes cookies for session tracking, feature flags, autocapture). Has a documented cookieless mode.

- In `app.rapoport.studio` (authenticated workspace), you can rely on **contract performance / legitimate-interest** for product-analytics on logged-in users — but it must be in your DPA/Terms with the customer, and you must still respect the ePrivacy Art. 5(3) consent rule for any tracking on the **logged-out marketing surface**.
- On `rapoport.studio` (marketing, anonymous), PostHog needs **consent before load** unless you switch to cookieless mode.

**Recommendation:**
- Use PostHog inside `app.rapoport.studio` (authenticated context), surface the data-processing terms in the customer DPA, and document the lawful basis. No banner needed for in-app behaviour because the user has signed a contract.
- On the marketing site, either keep PostHog out entirely (use Plausible only) or run PostHog in cookieless mode behind opt-in consent.

---

## 7. Practical recommendation for Rapoport Studio

### 7.1 What the banner needs to do

Build a single CMP that satisfies CNIL — it will pass everywhere else. Hard requirements:

1. **First layer on first visit**, blocking nothing but clearly visible.
2. **Three buttons, equal prominence:** "Accept all" / "Reject all" / "Customize".
3. **Granular categories** in customize: *Strictly necessary* (always on, no toggle), *Functional*, *Analytics*, *Marketing*.
4. **No pre-ticked toggles** for non-essential.
5. **Withdraw is as easy as give:** persistent footer link "Cookie preferences" or floating icon.
6. **Scripts physically blocked until consent** — not just a flag. Verify Network tab shows no analytics/marketing requests before "Accept".
7. **Consent log** with timestamp, banner version, choice — for ≥ 5 years.
8. **Per-category opt-in/opt-out actually fires the corresponding tags.** Test withdrawal: cookies must stop, not just be marked.

### 7.2 Cookie categorization for `rapoport.studio` + `app.rapoport.studio`

| Category | Examples in our stack | Consent? | Notes |
|---|---|---|---|
| **Strictly necessary** | Supabase Auth session (`sb-...`), CSRF tokens, CF Worker session, language preference if explicitly set | No | Art. 5(3) exemption; equivalent under PPA guidance. |
| **Functional** | UI state (sidebar collapsed), feature-flag overrides | Opt-in (if persistent across sessions) | Borderline — if truly session-only and tied to a user-requested feature, can argue strictly-necessary. Default: ask. |
| **Analytics** | Plausible (cookieless) — *no banner needed* | Not required | Qualifies for CNIL exemption; same posture across all four jurisdictions. |
| **Analytics (logged-in product)** | PostHog inside `app.rapoport.studio` | Covered by contract/DPA | Document in customer Terms + DPA. Not a cookie-banner concern. |
| **Marketing / advertising** | LinkedIn Insight Tag, Meta Pixel, X conversion, Google Ads | Opt-in, blocked-by-default | Required across all four jurisdictions. |
| **Embeds** | YouTube (use `youtube-nocookie.com`), X embeds, Calendly | Opt-in unless privacy-mode variant | Use privacy-enhanced versions where they exist to keep them in "functional". |

### 7.3 Concrete decisions

1. **Marketing site (`rapoport.studio`):** Plausible only, no marketing pixels initially → **no banner required** as long as you also keep YouTube/Calendly embeds in their privacy-mode variants and don't add LinkedIn Insight or Meta Pixel. If you later add any marketing tag, add the full banner.
2. **App (`app.rapoport.studio`):** PostHog on the authenticated side, justified by performance-of-contract + DPA terms with the customer. Banner not needed for in-app behaviour, but **a clear "what we collect" section in the Privacy Policy and DPA is mandatory** — Israel/Moldova/EU all require this.
3. **DPA:** Make sure every B2B customer signs the DPA, since Israel post-Amendment 13 and EU GDPR both demand controller/processor terms in writing. You already have this surface (`/legal/dpa` per recent commits).
4. **Privacy policy:** must enumerate exact cookies/identifiers, purposes, retention, third parties, and link to sub-processor list. Update on every CMP change.
5. **Consent log retention:** ≥ 5 years, queryable. The German BfDI and CNIL will ask to see it.
6. **Geo-detection? No.** Resist the temptation to show the banner only to EU users. Israel and Moldova-from-Aug-2026 both apply on data-subject basis. One global banner is cheaper than three.

### 7.4 What can/cannot live without consent — one-page table

| Item | EU (CNIL/EDPB) | Romania | Moldova (now → 23 Aug 2026) | Israel (post-14 Aug 2025) |
|---|---|---|---|---|
| Auth/session cookies | No consent | No consent | No consent → No consent | No consent |
| CSRF/security | No consent | No consent | No consent → No consent | No consent |
| Load balancer | No consent | No consent | No consent → No consent | No consent |
| Language-pref (user-set) | No consent | No consent | No consent → No consent | No consent |
| Plausible (cookieless) | No consent | No consent | No consent → No consent | No consent |
| Matomo self-hosted (anonymized, CNIL-config) | No consent | No consent | No consent → No consent | No consent |
| Google Analytics 4 | **Opt-in** | **Opt-in** | Transparency now → **Opt-in** | **Opt-in** |
| Meta/LinkedIn/X/Google Ads pixels | **Opt-in** | **Opt-in** | Transparency now → **Opt-in** | **Opt-in** |
| YouTube standard embed | **Opt-in** | **Opt-in** | Transparency now → **Opt-in** | **Opt-in** |
| `youtube-nocookie.com` | No consent | No consent | No consent → No consent | No consent |
| A/B testing (PostHog feature flags on marketing) | **Opt-in** | **Opt-in** | Transparency now → **Opt-in** | **Opt-in** |
| PostHog inside authenticated app | Contract basis (no banner) | Contract basis | Contract basis | Contract basis |

### 7.5 What's changed in 2024–2026 to flag

- **EU:** ePrivacy Regulation **withdrawn 11 Feb 2025**. No harmonization coming. EDPB 2/2023 finalized Oct 2024 — pixels/fingerprinting/URL tracking all in scope of Art. 5(3). CNIL fines crossing **€475M in single days** (Sept 2025).
- **Germany:** TTDSG → **TDDDG renaming May 2024**. EinwV (Consent Management Ordinance) **active April 2025**. § 25 cap stays at €300k.
- **Romania:** **Draft amendment to Law 506/2004** in Parliament 2025 codifying reject-parity into statute.
- **Moldova:** **Law 195/2024 adopted 25 July 2024**, enters force **23 Aug 2026** — Moldova becomes GDPR-equivalent.
- **Israel:** **Amendment 13 enacted Aug 2024, effective 14 Aug 2025** — online identifiers (cookies, IPs) are now expressly personal data; PPA gets administrative-fine power.

### 7.6 What to ship next (suggested follow-up work)

1. Pick a CMP approach — Cookiebot / Iubenda / Didomi, or hand-rolled `OverlayHost` dialog tied to a `?dialog=cookie-preferences` slot (given the linkable-UI contract in `openspec/current/linkable-ui.md`, hand-rolled is on-brand).
2. Audit the marketing site for any non-strictly-necessary cookie/script — if Plausible-only, ship a **transparency notice** (not a consent banner) for now and defer the full CMP until you add marketing tags.
3. Author `/legal/cookies` page with the categorization above, listing every cookie/identifier by name, purpose, controller, retention.
4. In customer DPA (`/legal/dpa`), explicitly cover product analytics (PostHog) so the in-app posture is contract-based.
5. Plan a follow-up OpenSpec change to add a consent-log table (`consent_events` in Supabase) tied to user_id / anonymous_id, populated from the CMP, retained 5 years.
6. **Re-audit in July 2026** before Moldova Law 195/2024 turns on, and again whenever the next CNIL/PPA/Garante guidance drops.

---

## Sources

**EU / EDPB / ePrivacy:**
- [Consenteo — GDPR Cookie Consent in 2026](https://www.consenteo.com/knowledge-hub/GDPR/gdpr_cookie_consent_2026)
- [EDPB Guidelines 2/2023 on technical scope of Art. 5(3)](https://www.edpb.europa.eu/system/files/2024-10/edpb_guidelines_202302_technical_scope_art_53_eprivacydirective_v2_en_0.pdf)
- [EDPB Opinion 8/2024 on Consent or Pay](https://www.edpb.europa.eu/system/files/2024-04/edpb_opinion_202408_consentorpay_en.pdf)
- [EDPB Cookie Banner Taskforce report (Jan 2023)](https://www.edpb.europa.eu/news/news/2022/edpb-adopts-guidelines-right-access-and-letter-cookie-consent_en)
- [Fieldfisher — EDPB broad reading of ePrivacy](https://www.fieldfisher.com/en/insights/edpb-reiterates-its-broad-reading-of-the-cookie-rule-in-the-e-privacy-directive)
- [Simmons & Simmons — Opinion 8/2024 key points](https://www.simmons-simmons.com/en/publications/clvpeiyzw00s0ua8c0jyqml42/key-points-from-edpb-opinion-08-2024-on-consent-or-pay-models)
- [Kukie — 2025-2026 cookie fines tracker](https://kukie.io/blog/cookie-consent-fines-2025-2026)

**France / CNIL:**
- [CNIL — 2025 sanctions summary](https://www.cnil.fr/en/sanctions-and-corrective-measures-cnils-actions-2025)
- [CNIL Sheet n°16 — analytics consent exemption](https://www.cnil.fr/en/sheet-ndeg16-use-analytics-your-websites-and-applications)
- [Lewis Silkin — "When the Cookie Crumbles"](https://www.lewissilkin.com/insights/2025/09/19/when-the-cookie-crumbles-lessons-from-the-cnil-102l64c)
- [BABL AI — €486.8M in CNIL fines 2025](https://babl.ai/frances-data-protection-authority-issues-e486-8-million-in-fines-as-cookie-violations-and-workplace-surveillance-lead-2025-enforcement/)
- [Matomo — CNIL exemption FAQ](https://matomo.org/faq/how-to/how-do-i-configure-matomo-without-tracking-consent-for-french-visitors-cnil-exemption/)
- [Matomo — €475M CNIL enforcement post](https://matomo.org/blog/2025/09/cookie-regulation-cnil/)

**Germany:**
- [Securiti — German TTDSG/TDDDG guide](https://securiti.ai/blog/german-ttdsg-guide/)
- [Usercentrics — German Consent Management Ordinance](https://usercentrics.com/knowledge-hub/cookie-flood-control-consent-management-ordinance-tdddg/)
- [Kukie — Cookie Consent Germany 2026](https://kukie.io/blog/cookie-consent-germany-ttdsg-dsgvo)

**Italy:**
- [Didomi — Italian Garante guidelines](https://www.didomi.io/blog/italian-garante-new-guidelines)
- [Hunton — Italian Garante updated guidelines](https://www.hunton.com/privacy-and-information-security-law/italian-garante-publishes-updated-guidelines-on-cookies-and-other-tracking-technologies)
- [DLA Piper Privacy Matters — Italy 2025 enforcement](https://privacymatters.dlapiper.com/2025/06/italy-the-garante-issues-first-gdpr-fine-over-employees-email-metadata-privacy-breach/)

**Spain:**
- [IAPP — AEPD Cookie Guide](https://iapp.org/resources/article/aepd-guide-on-use-of-cookies/)
- [Secure Privacy — AEPD guidelines](https://secureprivacy.ai/blog/spanish-aepd-cookie-guidelines)

**Romania:**
- [Lawyers Week — Romania cookie consent alignment](https://lawyersweek.net/26447/new-rules-on-cookie-consent-aligning-romanian-legislation-with-european-standards.html)
- [ANSPDCP (official site)](https://www.dataprotection.ro/index.jsp?page=home&lang=en)
- [GDPRhub — ANSPDCP Online Store cookie case](https://gdprhub.eu/index.php?title=ANSPDCP_(Romania)_-_Online_Store_case)

**Moldova:**
- [Law 195/2024 official PDF on NCPDP site](https://datepersonale.md/wp-content/uploads/2024/09/Law-no.-195-2024-on-personal-data-protection-1.pdf)
- [aci.md — Legal expert review of Law 195/2024](https://aci.md/legal-expert-review-moldovas-new-personal-data-protection-law/)
- [CSOMETER — Moldova new data protection law](https://csometer.info/updates/moldova-new-data-protection-law-adopted)
- [Consentmo — Moldova privacy act overview](https://www.consentmo.com/blog-posts/new-data-protection-law-in-moldova)
- [DLA Piper — Moldova online privacy](https://www.dlapiperdataprotection.com/index.html?t=online-privacy&c=MD)
- [Law 241/2007 on electronic communications](https://cis-legislation.com/document.fwx?rgn=22893)
- [NCPDP official site](https://datepersonale.md/en/)

**Israel:**
- [IAPP — Israel Amendment 13 new era](https://iapp.org/news/a/israel-marks-a-new-era-in-privacy-law-amendment-13-ushers-in-sweeping-reform)
- [Library of Congress — Amendment 13 in effect](https://www.loc.gov/item/global-legal-monitor/2025-11-17/israel-amendment-to-privacy-protection-law-goes-into-effect/)
- [Pearl Cohen — Amendment 13 takes effect](https://www.pearlcohen.com/israel-significant-amendment-to-the-privacy-law-takes-effect/)
- [Safetica — Amendment 13 explained](https://www.safetica.com/resources/guides/israel-s-amendment-13-what-the-new-data-protection-law-means-for-your-business)
- [PPA Cookies & Similar Technologies Guidance Note](https://www.ayr.co.il/wp-content/uploads/2024/04/Israel-Cookies-Similar-Technologies-Guidance-Note-.pdf)
- [Lexology — Practical Guidelines to Lawful Use of Cookies (Israel)](https://www.lexology.com/library/detail.aspx?g=841a3fd1-3a6a-4bdd-875e-4aa5179ecf49)
- [Cookie-Script — Israel PPL guide](https://cookie-script.com/privacy-laws/israel-privacy-protection-law-ppl)

**Analytics tools:**
- [Plausible Data Policy](https://plausible.io/data-policy)
- [Plausible — privacy-focused web analytics](https://plausible.io/privacy-focused-web-analytics)
- [PostHog GDPR compliance docs](https://posthog.com/docs/privacy/gdpr-compliance)
- [PostHog cookieless tracking tutorial](https://posthog.com/tutorials/cookieless-tracking)
- [Usercentrics — GA4 GDPR compliance rulings](https://usercentrics.com/knowledge-hub/google-analytics-and-gdpr-compliance-rulings/)
- [Piwik PRO — Is GA GDPR-compliant?](https://piwik.pro/blog/is-google-analytics-gdpr-compliant/)
