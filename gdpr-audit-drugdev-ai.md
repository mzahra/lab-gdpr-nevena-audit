# GDPR audit: DrugDev-AI

**Auditor:** Zahra
**Teammate / builder:** Nevena
**Project audited:** [DrugDev-AI](https://github.com/nevena-mi/DrugDev-AI), an AI assistant for pharmaceutical drug development and regulatory science
**Date:** 2026-08-19
**Related work:** I also audited this project for the EU AI Act, see [the PR](https://github.com/ai-consulting-bootcamp/lab-eu-ai-act-peer-audit/pull/5/changes). This is the separate GDPR audit, done independently.

---

## Part 0: Data processing brief

- DrugDev-AI has three modes: Ask (RAG question answering), Learn (curriculum with auto-generated lessons and quizzes), Monitor (aggregates live regulatory updates).
- Personal data collected, all user-provided: questions, learning profiles, conversation history, quiz responses, search keywords. Retained "only for the active session."
- Questions "may contain information that directly or indirectly identifies the user or describes their professional or personal circumstances," and a user "could voluntarily enter" health, ethnicity, political, or religious information in free text.
- No automated decisions with legal effect. Users keep full responsibility for professional decisions.
- Architecture (from README.md, AGENT.md, stack_decision.md, allowed as architecture/data flow material): OpenAI's Responses API generates answers, Pinecone stores vector embeddings, Cohere reranks results. No mention anywhere of user accounts, login, hosting region, or persistent storage, which is itself a gap.

---

## Part 1: Read and annotate

Personal data categories and special-category risk are tabulated in Part 2, so not repeated here.

**Unclear:** vendor-side retention once data leaves the session; whether Learn mode's "progression" tracking needs persistent storage that isn't described anywhere; hosting region.

**Crosses an EU border:** OpenAI, Pinecone, and Cohere are all US companies, so any personal data reaching them (questions, conversation history, likely quiz answers) is an international transfer unless a specific EU region is configured. Monitor mode's calls to ClinicalTrials.gov/openFDA/EMA return public data, but the search keywords a user types still leave the system.

**Possibly incompatible purpose:** whether any user data is reused to train or fine-tune a model, own or vendor's. Not stated either way, see clarifying question 2.

---

## Part 2: Data and role map

**Personal data summary:**

| Data category | Source | Purpose | Crosses EU border? | Special category? |
|---|---|---|---|---|
| Questions (free text) | Ask mode | Generate grounded answers | Yes, via OpenAI/Pinecone/Cohere (all US) | Possible if volunteered |
| Learning profile | Learn mode | Personalize curriculum | Likely, if OpenAI personalizes | Unlikely |
| Conversation history | Ask/Learn | Session context for follow-ups | Yes, same vendors | Same as questions |
| Quiz responses | Learn mode | Assess progress, feedback | Possibly, if LLM-graded | Unlikely |
| Search keywords | Monitor mode | Query public regulatory sources | Unclear, gap in brief | Unlikely |

Retention stated as session-only; vendor-side retention (OpenAI/Pinecone/Cohere) is not confirmed.

**Role map:**

| Entity | Role | Processing activity | DPA needed? |
|---|---|---|---|
| Client | Not defined in the brief; assumed no separate external client yet | Data subject: submits questions, quiz answers, keywords | N/A, revisit if commercialized |
| Builder: Nevena | Controller | Decides what data is collected and how it's used | No, controller not processor |
| OpenAI | Processor | Generates answers from questions and history | Yes, Article 28 |
| Pinecone | Processor | Stores vector embeddings | Yes, Article 28 |
| Cohere | Processor | Reranks retrieved passages | Yes, Article 28 |

International transfer: all three vendors are US-based. Mechanism needed: SCCs per vendor, or an adequacy decision if applicable. No evidence in the brief that either is in place.

---

## Part 3: Clarifying questions log

1. **Accounts or session-only in production?** Affects retention rules and DSR handling. Assumption: session-only, and "progression" is a planned feature not yet built.
2. **DPAs signed with OpenAI, Pinecone, Cohere?** Without one, using a processor for personal data is a gap on its own, and vendor terms may allow training on submitted data. Assumption: no DPA in place yet.
3. **What processing region and transfer mechanism per vendor?** Core Chapter V requirement. Assumption: default US region, SCCs needed and currently undocumented.
4. **Is there a published privacy notice?** Article 13/14 requirement, plus AI-specific transparency. Assumption: none exists yet.

---

## Part 4: Audit report

### Section 1: System summary

DrugDev-AI is a capstone AI assistant for pharma regulatory science, with three modes: Ask (RAG Q&A over FDA/EMA/ICH/WHO documents), Learn (curriculum with auto-generated lessons and quizzes), and Monitor (live regulatory updates). It processes user-provided data (questions, learning profile, conversation history, quiz responses, search keywords), stated to be session-only. Generation, retrieval, and reranking run through three US vendors: OpenAI, Pinecone, and Cohere. No automated decisions with legal effect are made.

### Section 2: Data and role map summary

Personal data is mostly free text plus derived data, all stated session-only (Part 2). Nevena's team is the controller; OpenAI, Pinecone, and Cohere are processors, all US-based, so every one is an international transfer. No external client is defined at this stage.

### Section 3: Compliance findings

> **Finding 1: International transfer mechanism**
> **Severity:** Significant
> **Description:** User data reaches OpenAI, Pinecone, and Cohere (all US), with no evidence of SCCs or an adequacy decision.
> **Recommended action:** Confirm each vendor's processing region and put SCCs in place wherever data leaves the EU.
> **Escalation needed?** Yes, to whoever owns vendor contracts, before production.

> **Finding 2: Data Processing Agreements missing**
> **Severity:** Blocking
> **Description:** OpenAI, Pinecone, and Cohere are processors under Article 28, but no DPA is mentioned anywhere in the reviewed materials.
> **Recommended action:** Sign an Article 28 DPA with each vendor before processing real personal data, and confirm terms exclude use of submitted data for vendor model training.
> **Escalation needed?** Yes, to legal. This blocks production use with real personal data.

> **Finding 3: No privacy notice for end users**
> **Severity:** Significant
> **Description:** No privacy notice is referenced anywhere, telling users their input goes to third-party AI providers.
> **Recommended action:** Publish a notice covering what's collected, why, third parties, retention, and rights, including the AI-specific point that users are talking to an AI.
> **Escalation needed?** Yes, to whoever owns the product, before go-live.

> **Finding 4: Special category data risk in free text**
> **Severity:** Significant
> **Description:** Free text fields can capture health, ethnicity, political, or religious data if a user volunteers it, with no stated safeguard.
> **Recommended action:** Add a visible warning near free text inputs; consider input-side filtering longer term.
> **Escalation needed?** No, a design fix, but should be tracked.

> **Finding 5: Retention beyond the session is undocumented**
> **Severity:** Significant
> **Description:** "Session only" is stated for the app, but vendor-side retention (OpenAI/Pinecone/Cohere) is not confirmed.
> **Recommended action:** Check and document each vendor's retention policy; disclose in the privacy notice if any retain data longer.
> **Escalation needed?** No, but resolve before finalizing Finding 3.

> **Finding 6: Lawful basis not documented in a reviewable form**
> **Severity:** Minor
> **Description:** No stand-alone lawful basis assessment exists. Contractual necessity (Article 6(1)(b)) looks reasonable for the core service, but it's not written down formally.
> **Recommended action:** Write a short, documented lawful basis assessment per processing purpose.
> **Escalation needed?** No, documentation task.

### Section 4: Specific GDPR obligations checklist

| Obligation | Assessment | Note |
|---|---|---|
| Lawful basis identified | Cannot determine from brief | Reasonable basis exists, not formally documented (Finding 6) |
| Purpose limitation respected | Cannot determine from brief | No confirmation on model-training reuse (question 2) |
| Data minimisation | Appears met | Data collected matches what each mode needs |
| Roles mapped, DPAs in place | Gap identified | Roles clear, DPAs missing (Finding 2) |
| International transfer mechanism documented | Gap identified | All vendors US-based, no safeguard documented (Finding 1) |
| DPIA conducted if required | Appears met | No large-scale profiling, vulnerable subjects, or systematic monitoring found; re-check if accounts/analytics are added |
| Article 22 safeguard if automated decisions affect people | Appears met | Educational content only, decision left to the user |
| Privacy notice covers AI processing | Gap identified | None found (Finding 3) |
| DSR can be operationalised within deadlines | Cannot determine from brief | Depends on confirming no persistent or vendor-side storage (question 1, Finding 5) |

### Section 5: Overall recommendation

**Proceed with conditions.** The design (session-only storage, no accounts, no automated decisions with legal effect) keeps baseline risk fairly low for an MVP. But one blocking finding (missing DPAs) and several significant gaps (transfer mechanism, privacy notice, vendor retention, special category risk) must close before real personal data goes through the system in production. Finding 2 first, the rest shortly after or before go-live.

### Section 6: What this report is not

Not a legal opinion, not a DPIA, not a certification of compliance. An independent, structured review based on the data processing brief and public project documentation, done as a training exercise. Get a proper legal review before relying on this for a real launch.

---

## Part 5: Debrief notes

Status: done, held with Nevena on 2026-08-19.

1. **Auditor presents:** walked through the report above with Nevena.
2. **Builder responds:** Nevena reviewed the findings and agreed with the audit overall, no pushback or additional context flagged.
3. **Lawful basis compare:** matches for the core service. Mine: Article 6(1)(b), contractual necessity (Finding 6). Hers: also Article 6(1)(b) for Ask/Learn/Monitor, plus Article 6(1)(f), legitimate interests, for a separate purpose I had no visibility into: runtime token and cost monitoring. Her basis is stronger overall, since it covers a real second purpose (operational analytics) that isn't in the public architecture docs I had access to.
4. **DPIA compare:** same conclusion. Both audits used the EDPB's nine criteria and both found only one present (innovative use of technology), so a full DPIA is not required at this MVP stage.
5. **Gap list compare:**
   - Matched on both sides: missing Article 28 DPAs with OpenAI/Pinecone/Cohere, missing privacy notice, undocumented international transfer mechanism and hosting region, and incidental special-category data risk in free text fields.
   - She caught that I missed: missing Records of Processing Activities (RoPA), a missing Legitimate Interests Assessment for the cost-monitoring feature, missing incident response and breach notification procedure, and partial/unknown security and access-control documentation. I couldn't catch these from the outside, since they depend on knowing the system has an operational analytics feature at all, which isn't described in the public repo docs.
   - I added that she didn't: a severity rating (Blocking/Significant/Minor) and a named escalation path per finding, following the lab's required structure. Her memo has a "top three actions" list but doesn't grade each gap that way.
   - One useful mismatch: I flagged "lawful basis not documented in a reviewable form" as a Minor finding, since nothing about it was visible to me. She had, in fact, already written a full lawful basis table, just not shared per the lab's ground rules. This shows a real limit of external review: an auditor can flag something as "missing" when it actually exists but simply wasn't in scope to share.
6. **Joint closing note (required):** Self-assessment and independent review caught different things for structural reasons, not effort. Nevena's self-audit surfaced more of the internal accountability paperwork (RoPA, LIA, incident response) because she has visibility into features and internal state an outside auditor cannot see.
