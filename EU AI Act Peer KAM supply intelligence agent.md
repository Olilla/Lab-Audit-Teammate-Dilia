# EU AI Act Peer Audit — KAM Supply Intelligence Agent
Auditor: Olalla Murciego
System built by: Dilia Navarro

---

## Phase 2 — First-pass Classification

| Question | Answer |
|---|---|
| Does this system fall under any prohibited category (Article 5)? | No. The system does not perform biometric identification, social scoring, subliminal manipulation, or any other prohibited practice. |
| Does this system operate in any of the eight Annex III areas? | No. The system operates in a B2B internal productivity context. It does not touch employment decisions, credit assessment, education, law enforcement, critical infrastructure, migration, administration of justice, or democratic processes. |
| If Annex III: does it significantly influence decisions in that area? | Not applicable. |
| Does this system interact with end users or generate content requiring disclosure (Article 50)? | The system interacts with internal users (KAMs) via Slack and generates structured answers automatically. The users are professionals operating within their own organisation, not members of the public. Article 50 transparency obligations are not triggered in this context. |
| First-pass risk tier | Minimal risk — no specific AI Act obligations apply. |
| One-sentence justification | The system is an internal business intelligence tool that answers operational queries from professional users, produces no decisions with legal or similarly significant effects on individuals, and does not operate in any Annex III area. |

---

## Phase 3 — Clarifying Questions Log

**Question 1: Are KAM decisions that rely on this output communicated externally to clients?**

What I need to know: whether the system's output influences decisions that are then presented to third parties — for example, if a KAM uses the answer to negotiate a contract or communicate account status to a client.

Why it matters: if the output materially shapes decisions that affect external parties, the compliance picture may shift depending on the nature of those decisions and whether they touch any Annex III area.

Provisional assumption: the system is used for internal preparation only. KAMs review the output and exercise independent judgement before any external communication. The system is not the decision-maker.

---

**Question 2: What happens when the SQL query returns incorrect or incomplete data and the KAM acts on it without detecting the error?**

What I need to know: whether there is any error-flagging, confidence indicator, or built-in caveat in the output that alerts the KAM when the answer may be unreliable.

Why it matters: the brief notes the system includes a warning if a client is not recognised or a query fails, but does not describe what happens when a query succeeds but returns inaccurate data. In a commercial context, acting on wrong supplier or rate information could have material business consequences.

Provisional assumption: the system includes the raw SQL query in every response, which allows a technically literate KAM to spot obvious errors. This is a partial safeguard but not a systematic one.

---

**Question 3: Who has access to the Slack channel where outputs are posted, and could outputs containing client or supplier data be seen by unauthorised users?**

What I need to know: whether the Slack thread where answers are posted is restricted to the relevant KAM or visible to a broader internal audience.

Why it matters: the outputs contain client profile data from Salesforce and supplier and product data from Supabase. If the Slack channel is shared beyond the intended users, this creates a data exposure risk under GDPR even if the AI Act classification is unaffected.

Provisional assumption: the system posts to the same Slack thread where the question was asked, which is likely a KAM-specific channel. Access controls are assumed to be in place but are not confirmed by the brief.

---

**Question 4: Is the assigned KAM name pulled from Salesforce ever included in the Slack output?**

What I need to know: whether personal data from the Salesforce record — specifically the KAM name field — appears in the generated answer or in the Excel export.

Why it matters: the brief explicitly identifies the KAM name as personal data present in the input. If that name surfaces in outputs that could be seen by others, it triggers GDPR data minimisation and purpose limitation obligations regardless of the AI Act classification.

Provisional assumption: the KAM name is used as a lookup field to retrieve the correct account but is not included in the structured output. This is unconfirmed.

---

## Phase 4 — Audit Report

### Section 1: System Summary

The KAM Supply Intelligence Agent is an internal question-answering tool built for a car-rental company's Key Account Management team. A KAM types a free-text question in Slack — typically about a specific client's suppliers, products, routes, or contract details — and the system returns a structured answer, the SQL query used to retrieve the data, and optionally an Excel file. The pipeline runs through an n8n webhook, a Flask server, and a LangGraph agent that queries Salesforce for client profile data and Supabase for supplier and product data via dynamically generated SQL. There is no human approval step before the answer is posted to Slack. The system was built by Dilia Navarro and is intended for daily KAM use.

---

### Section 2: Risk Classification

First-pass tier: Minimal risk.

The system is an internal productivity tool that helps professional users retrieve structured business data from two internal sources. It does not make decisions about individuals, does not produce outputs with legal or similarly significant effects on natural persons, and does not operate in any of the eight Annex III areas. The users are employees operating within their own professional context, not members of the public interacting with an AI system without awareness. Article 50 transparency obligations are not triggered because the system is not designed to interact with natural persons in a way that could lead them to believe they are speaking with a human, and the outputs are clearly machine-generated structured data responses rather than human-like conversational content.

The one area of genuine uncertainty is whether KAM decisions informed by this output ever reach external parties in a way that could constitute a significantly influenced decision in a regulated domain. Based on the brief, this appears unlikely, but it is the single question that would change the classification if the answer were yes.

---

### Section 3: Role Map

| Role | Entity | Key AI Act obligations |
|---|---|---|
| Provider | Dilia Navarro, as the developer who built and would deploy the system | Documentation of capabilities and limitations; ensuring the system does not fall into prohibited or high-risk categories; no further obligations given minimal risk classification |
| Deployer | The car-rental company whose KAM team uses the system | Responsible for ensuring the system is used within its intended scope; responsible for any data protection obligations toward employees and clients whose data flows through the system |
| Vendor — OpenAI | Provider of the LLM used in Node 6 for final answer generation | Carries GPAI provider obligations under the AI Act: transparency documentation, copyright policy, technical documentation |
| Vendor — n8n | Workflow orchestration and Slack integration | Infrastructure provider; not an AI system provider under the Act |
| Vendor — LangGraph / LangChain | Agent orchestration framework | Framework provider; not an AI system provider under the Act |

---

### Section 4: Compliance Findings

**Finding 1 — GDPR: personal data transmitted to a third-party LLM**

Severity: Significant

The system passes data retrieved from Salesforce — including at minimum the assigned KAM name, which the brief identifies as personal data — to OpenAI's API as part of the answer generation step in Node 6. This constitutes a transfer of personal data to a third-party processor. GDPR requires a lawful basis for this processing and a Data Processing Agreement with OpenAI covering the specific data categories involved. The brief does not indicate whether either is in place.

Recommended action: Conduct a data mapping exercise to confirm exactly which fields from Salesforce and Supabase are passed to the OpenAI API. Verify that OpenAI's DPA covers this processing. Consider whether the KAM name field can be excluded from the LLM prompt without affecting output quality, applying the GDPR data minimisation principle.

Escalation needed: Yes — to a Data Protection Officer or privacy lawyer before production deployment.

---

**Finding 2 — No systematic output quality safeguard**

Severity: Minor

The system includes error handling for failed queries and unrecognised clients, and it exposes the raw SQL query in every response. However, the brief does not describe any mechanism for flagging when a successful query may have returned incomplete or misleading data — for example, when a client has partial records in Supabase or when the schema RAG layer retrieves an outdated schema. In a commercial context, a KAM acting on incorrect supplier or rate information could make materially wrong decisions.

Recommended action: Add a standard caveat to every Slack response noting that the output is generated from available data and should be verified against source systems for high-stakes decisions. This is a one-line addition to the output formatting prompt and does not require architectural changes.

Escalation needed: No.

---

**Finding 3 — Slack output access controls unconfirmed**

Severity: Minor

The brief does not describe the access controls on the Slack channels where outputs are posted. If client profile data and supplier details are posted to a shared channel accessible beyond the intended KAM audience, this creates a data exposure risk under GDPR.

Recommended action: Confirm that the Slack channels used by the system are restricted to authorised KAM users. If the system could post to public or broadly shared channels, implement channel restrictions at the n8n configuration level.

Escalation needed: No, unless the channel access review reveals a broader exposure.

---

### Section 5: Overall Recommendation

**Clear to proceed — with two conditions.**

The system is minimal risk under the EU AI Act and does not present any blocking compliance findings under the regulation itself. The two conditions relate to GDPR rather than the AI Act, and both are addressable without structural changes to the system. The deployer should confirm that a Data Processing Agreement with OpenAI is in place covering the personal data fields transmitted to the API, and should verify that Slack channel access controls limit output visibility to authorised users. Neither condition requires significant technical work, but both should be resolved before the system handles live production data.

| Step | Auditor notes | Builder notes | Agreed / disagreed? | Why it matters |
|---|---|---|---|---|
| Auditor presents | Classified the system as minimal risk. No Article 5 prohibited practices, no Annex III category triggered. Main findings: GDPR exposure from Salesforce data passed to OpenAI API, no systematic output quality safeguard, and Slack access controls unconfirmed. | | | An external auditor without builder context may classify more conservatively or miss operational details that change the risk picture. Presenting the audit before the builder responds tests whether the classification holds up independently. |
| Builder responds | The brief does not confirm whether a DPA with OpenAI is in place, whether the KAM name field appears in outputs, or whether Slack channels are restricted to authorised users. These were flagged as clarifying questions requiring a response before finalising the audit. | | | Builder context often resolves ambiguities that an external auditor has to assume. If the builder's response changes any finding, that gap in the brief is itself a compliance risk — undocumented systems are harder to audit and harder to defend. |
| Compare classifications | Auditor: minimal risk. | | | If the classifications differ, the reason is usually either a difference in regulatory interpretation or information that did not make it into the brief. Both are useful findings: the first reveals genuine ambiguity in the regulation, the second reveals a documentation gap. |
| Compare gap lists | Auditor identified: (1) GDPR — personal data to OpenAI API, severity significant; (2) no output quality safeguard, severity minor; (3) Slack access controls unconfirmed, severity minor. | | | Gaps that appear in the external audit but not the self-audit reveal builder blind spots. Gaps that appear in the self-audit but not the external audit reveal brief gaps — things the builder knew but did not document. Both directions matter. |
| Joint closing note | | | | The closing note captures what the debrief itself revealed about the difference between auditing your own work and auditing someone else's. It is the meta-finding of the exercise: not what the system does wrong, but what the audit process surfaces that neither party could have reached alone. |

---

### Section 6: What This Report Is Not

This report is a first-pass compliance assessment prepared for educational and peer review purposes. It is not a legal opinion, a conformity assessment, or a certification of any kind. It does not constitute advice from a qualified lawyer or data protection specialist. The conclusions in this report should be verified with legal counsel before any production deployment or EU market placement.

---

## Phase 5 — Debrief Closing Note

*To be completed jointly with Dilia after the debrief conversation.*