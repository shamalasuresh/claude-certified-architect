# Practice Questions 16 — Industry Verticals: Healthcare, Finance & Legal

> Domain-specific scenarios: Regulatory constraints, professional requirements, data sensitivity, liability considerations.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A healthcare startup deploys an AI chatbot that provides medication dosage information. The system prompt says: "You are a medical assistant. Help users with their medication questions." A user asks about safe dosage for their child. The chatbot gives a specific dosage. The child is harmed. Who bears primary liability?

A) Anthropic, as the model provider  
B) The operator (startup) who deployed the system without adequate safeguards, disclaimers, or scope restrictions; the system prompt should have included: "Do not provide specific dosage recommendations. Always direct users to their healthcare provider or pharmacist for dosing decisions."  
C) The user who asked  
D) Claude, as the responding agent  

**Answer: B**  
**Explanation:** Operator liability: the startup configured the system, deployed it to users, and defined its behavior through the system prompt. Operator responsibility includes: (1) Appropriate scope restriction ("information only, not medical advice"). (2) Disclaimers. (3) Emergency escalation paths. (4) Directing users to professionals for clinical decisions. The absence of these safeguards in a healthcare deployment is operator negligence.

---

**Q2.** A hospital system wants to use Claude to analyze patient records for treatment pattern research. What data handling requirements must the architecture address before any implementation?

A) Standard data security is sufficient  
B) HIPAA requirements: (1) Business Associate Agreement (BAA) with Anthropic. (2) PHI must not be retained in Claude's training data (use API with zero-data-retention). (3) Minimum necessary information — include only what's needed for the analysis. (4) Audit logging of all PHI access. (5) De-identification if possible (safer than PHI access). (6) IRB review for research use.  
C) Encrypt the data before sending  
D) Only de-identified data can be used  

**Answer: B**  
**Explanation:** Healthcare data architecture requires HIPAA compliance as a prerequisite. Each element is required: (1) BAA is a legal requirement before processing PHI with a business associate. (2) Zero-data-retention ensures PHI doesn't persist. (3) Minimum necessary is a HIPAA principle. (4) Audit logs demonstrate compliance. (5) De-identification removes most restrictions. Skipping any of these creates HIPAA liability.

---

**Q3.** A mental health support chatbot is designed to help users cope with anxiety. A user expresses active suicidal ideation. What must the system do?

A) Continue the conversation and provide support  
B) Immediate protocol: (1) Provide crisis hotline information immediately (988, local emergency services). (2) Escalate to human crisis counselor if available. (3) The PRECISE-E explicit instructions must define this mandatory response pattern. (4) This is a non-negotiable HITL escalation point — SPIDER-E.  
C) Terminate the conversation  
D) Ask clarifying questions  

**Answer: B**  
**Explanation:** Crisis escalation is mandatory for mental health applications. This is not optional — it is a clinical and ethical requirement. The architecture must: (1) Detect crisis signals (trained classifier or keyword detection + Claude's judgment). (2) Mandatory response: crisis resources + human escalation. (3) Never just "continue providing support" during active crisis. (4) Document the incident for clinical review. SPIDER-E is non-negotiable for safety-critical escalations.

---

**Q4.** A financial advisory firm deploys an AI assistant that provides investment advice. The advice tool calls live market data APIs. The assistant says "buy XYZ stock" based on the live data. What regulatory problem exists?

A) Live data creates latency issues  
B) "Buy XYZ stock" is personalized investment advice requiring SEC registration as an Investment Advisor; the system must explicitly scope to "general market information" not personalized advice; system prompt must include: "You provide general market information only. You do not provide personalized investment recommendations. Consult a licensed financial advisor for investment decisions."  
C) Market data APIs require special licensing  
D) Claude should not access live data  

**Answer: B**  
**Explanation:** SEC regulation: personalized investment advice requires registration as an Investment Advisor. An AI providing "buy XYZ" recommendations without this registration creates legal liability. The system prompt scope restriction is the architectural safeguard. "General market information" (educational) vs. "personalized recommendation" (regulated) — this line must be clearly defined and enforced in the system's behavior.

---

**Q5.** A legal research assistant helps attorneys find relevant case law. It returns cases with quotes. The attorney uses a cited case in court but the case doesn't exist — it was hallucinated. What architectural safeguards prevent this?

A) Use a more powerful model  
B) RAG-only citation: the assistant must retrieve citations from a verified legal database (Westlaw, LexisNexis, or similar); no citation should be generated from training knowledge; explicit instruction: "Cite only cases retrieved from the legal database. If you cannot find a case in the database, say 'No matching case found.'"  
C) Add a disclaimer to all responses  
D) Have attorneys verify all citations manually  

**Answer: B**  
**Explanation:** Legal citation hallucination is a documented problem with serious professional consequences (sanctions, bar complaints, client harm). The only reliable fix: ground citations to verified databases. The system must never generate citations from training knowledge. The instruction "cite only from retrieved sources" + RAG-only architecture + "not found" fallback eliminates hallucinated citations.

---

**Q6.** A healthcare AI assistant is deployed in an emergency department triage setting. Speed is critical (< 5 second response). The most accurate model (Sonnet) takes 3-8 seconds. The fastest model (Haiku) takes 0.5-2 seconds with slightly lower accuracy. Which is more appropriate and why?

A) Always use the most accurate model  
B) For ED triage: latency constraint (< 5 seconds) is a clinical requirement — a 10-second triage delay can be dangerous; Sonnet within 5 seconds for most queries + fallback to immediate "assess patient directly" escalation if response exceeds threshold; triage is a time-sensitive support tool, not the decision-maker  
C) Use Haiku — it's within the SLA  
D) The AI should not be used in emergency settings  

**Answer: B**  
**Explanation:** Healthcare AI architecture must balance accuracy and clinical workflow requirements. For ED triage: (1) Time constraints are clinical requirements, not just UX preferences. (2) The AI is a support tool — the physician makes the final decision. (3) If response time exceeds threshold: fallback to "direct clinical assessment required" — the safest default. A fast approximate suggestion is often more clinically useful than a slow perfect analysis in emergency settings.

---

**Q7.** A law firm wants Claude to draft legal documents (contracts, NDAs, etc.) for clients. Which scope restriction must be in the system prompt?

A) No restrictions needed — legal drafting is technical  
B) "These documents are drafted for review and customization by the attorney of record. They do not constitute legal advice and are not finalized legal documents until reviewed and approved by a licensed attorney. Do not present draft documents as ready for signature."  
C) Only a copyright disclaimer is needed  
D) Legal drafting cannot be automated  

**Answer: B**  
**Explanation:** Legal drafting scope: AI-generated documents require attorney review before being presented to clients as final documents. The scope restriction: (1) Identifies documents as drafts. (2) Requires attorney review. (3) Prevents the document from being used as a "final" legal document. (4) Manages liability (documents are not AI-certified legal instruments). This is a professional responsibility (ethics) requirement.

---

**Q8.** A fintech company processes credit card transactions and wants to use Claude to flag suspicious transactions. The flagging decisions affect access to customer funds. What HITL design is required?

A) Fully automated — AI decisions are final  
B) HITL for consequential decisions: AI flags suspicious transactions for HUMAN REVIEW before blocking funds; the AI score is a risk signal, not a final decision; blocking a customer's card based solely on AI judgment without human review is both a regulatory risk and customer service failure  
C) Only flag very high-risk transactions for review  
D) Human review is only for false positive remediation  

**Answer: B**  
**Explanation:** Financial regulation (Regulation E, CFPB guidelines) requires fair and accurate dispute processes for account actions. Blocking customer funds based solely on AI without human review exposes the company to: (1) Regulatory action. (2) Customer harm (inability to access funds). (3) Liability for incorrect blocks. The AI is a risk triage tool; humans make the blocking decision. This is a required HITL design in fintech.

---

**Q9.** A clinical decision support tool helps physicians with drug interaction checking. The physician selects "penicillin" and "methotrexate" — a known serious interaction. The system prompt doesn't include instructions for critical interactions. Claude's response is: "These medications may interact. Consult a pharmacist." Is this response sufficient?

A) Yes — it's accurate and safe  
B) For known critical drug interactions: the response must be proportional to the risk level; a "may interact" response for a potentially fatal interaction (methotrexate + penicillin can cause methotrexate toxicity) understates the severity; the system must have explicit instructions mapping interaction severity to response urgency  
C) The response is too strong  
D) The system should refuse to answer drug questions  

**Answer: B**  
**Explanation:** Clinical tool calibration: responses must be calibrated to clinical risk level. A severity mapping in PRECISE-E: (1) Contraindicated interactions → "CRITICAL INTERACTION: [specific risk]. Do not co-administer without specialist consultation." (2) Significant interactions → "Significant interaction: monitor carefully." (3) Minor interactions → "Minor interaction: awareness advised." "May interact" for contraindicated combinations is dangerously understated.

---

**Q10.** A legal AI system processes attorney-client privileged communications to provide case strategy suggestions. What data retention policy is required?

A) Standard 90-day retention  
B) Attorney-client privilege requires: (1) Zero-data-retention with the AI provider (API-level ZDR commitment). (2) No storage of privileged content in logs, training data, or caches. (3) The BAA or contract with the AI provider must explicitly address privilege. (4) Access limited to the specific attorneys of record for each matter.  
C) Encrypt and retain for 7 years  
D) Only anonymized summaries can be retained  

**Answer: B**  
**Explanation:** Attorney-client privilege is absolute in most jurisdictions — unauthorized disclosure waives privilege. AI systems processing privileged content must: (1) Zero-data-retention (content not retained by the AI provider). (2) Explicit contractual protection against training data use. (3) Access controls limiting to authorized attorneys. (4) No logging of content. This is the required architecture for any AI system handling privileged legal communications.

---

**Q11.** A health insurance company wants to use AI to process claims and automatically deny claims based on the AI's decision. What is the minimum required human oversight?

A) None — AI can make final coverage decisions  
B) Insurance regulation requires: (1) Human review of all denials before finalization. (2) Appeals process with human review. (3) Explanation of reasons for denial. (4) Compliance with state insurance regulations on coverage decisions. AI can score and draft denial explanations, but a human claims professional must make and sign off on final denial decisions.  
C) Human review only for claims over $10,000  
D) AI decisions are sufficient if accuracy is >90%  

**Answer: B**  
**Explanation:** Insurance coverage decisions are regulated. Fully automated denials without human review violate: (1) State insurance regulations (most require human decision-maker). (2) ACA/Affordable Care Act requirements. (3) Patient rights to explanation and appeal. AI role: assist in assessment, flag likely denials for human review, draft explanation letters. Human role: final coverage decision, accountability for denials. This is regulatory, not optional architecture.

---

**Q12.** A financial AI system produces quarterly earnings forecasts for investors. The system sometimes expresses high confidence in forecasts that later prove incorrect. What disclosure architecture is required?

A) No disclosure needed — forecasts are clearly speculative  
B) SEC disclosure requirements: (1) Clear labeling as AI-generated forward-looking statements. (2) Standard risk disclaimers (forward-looking statements involve risks and uncertainties). (3) Confidence scores should not be presented as investment advice. (4) Material risk factors must accompany any projections. This is securities law, not just good practice.  
C) Just add a standard disclaimer footer  
D) AI confidence scores are reliable and don't need special disclosure  

**Answer: B**  
**Explanation:** Securities law: forward-looking statements (earnings forecasts) require specific safe harbor disclosures. AI-generated forecasts are additionally scrutinized. The architecture must: (1) Label outputs as AI-generated with limitations. (2) Include required forward-looking statement disclaimers. (3) Ensure the system doesn't create the impression of certainty. An AI that presents forecasts as high-confidence recommendations without proper disclosure creates securities liability.

---

**Q13.** A pharmacist's tool uses Claude to verify drug dosage calculations before dispensing. The tool should never reduce the pharmacist's vigilance or create automation bias. What design principle addresses this?

A) Make the AI decision final to be consistent  
B) Automation bias prevention: (1) Present the AI check as one input, not the final answer. (2) Require the pharmacist to enter their OWN calculation before seeing the AI result. (3) Show confidence levels. (4) Never display "CORRECT" — only highlight discrepancies for review. (5) Regular calibration exercises where the AI is deliberately wrong to keep humans engaged.  
C) Remove the AI if it creates complacency  
D) Use a disclaimer that the AI is supplementary  

**Answer: B**  
**Explanation:** Automation bias is a real clinical risk: humans over-rely on AI outputs and stop exercising independent judgment. Prevention design: (1) Human-first input (prevents anchoring). (2) Discrepancy highlighting (not "correct/wrong" — just differences). (3) No final validation — always a second set of eyes. (4) Active engagement measures. This is human factors design for safety-critical AI systems.

---

**Q14.** A lawyer uses an AI assistant to research potential conflicts of interest before taking a new client. The AI searches the firm's client database. What data access control is critical?

A) The lawyer should have access to all client data  
B) Conflict checking requires: read access to client names and matter types only — NOT access to privileged matter content; the conflict check tool should return "potential conflict found/not found" without exposing the other client's privileged information to the attorney requesting the check  
C) Full database access is needed for comprehensive checking  
D) Conflict checks don't need AI  

**Answer: B**  
**Explanation:** Conflict checking architecture: the AI needs to know that Client A and Client B have adverse interests — it does NOT need to know the details of Client A's matter. Tool design: `check_conflicts(new_client_name, new_matter_type)` → returns `{conflict: true, reason: "Existing client on opposite side of similar matter type"}` without revealing Client A's privileged information. Minimum necessary access principle applied to professional ethics requirements.

---

**Q15.** A medical device company wants to use AI in a diagnostic software product. The FDA regulates this as Software as a Medical Device (SaMD). What regulatory requirement affects the AI architecture?

A) Standard software development practices apply  
B) FDA SaMD requirements: (1) 510(k) clearance or PMA approval for AI-assisted diagnostic outputs that influence clinical decisions. (2) Algorithmic change protocol (ACP) defining which model updates require re-clearance. (3) Real-world performance monitoring. (4) Transparency about training data and model limitations. This determines whether model updates can be deployed freely or require FDA review.  
C) HIPAA is the only regulatory requirement  
D) AI in medical devices is currently unregulated  

**Answer: B**  
**Explanation:** FDA SaMD regulation is critical for medical AI architecture decisions: (1) ACP determines deployment agility — some AI updates require re-clearance (significant changes) vs. pre-approved change control (minor updates). (2) This means: you cannot simply upgrade the Claude model version without assessing if it constitutes a "significant change" under the FDA framework. (3) System testing and validation requirements exceed standard software. Medical AI architects must understand regulatory pathways.

---

**Q16.** A robo-advisor platform uses Claude to generate personalized investment portfolios. The system considers: age, risk tolerance, investment goals. It provides specific asset allocation percentages (e.g., 60% equities, 30% bonds, 10% alternatives). What must the architecture include beyond standard AI safety?

A) Only standard disclaimers  
B) Investment Advisor registration: (1) If providing personalized investment advice for compensation, SEC/FINRA registration as Registered Investment Advisor (RIA) is required. (2) Fiduciary duty documentation. (3) KYC (Know Your Customer) verification. (4) Suitability analysis documented and auditable. (5) Client agreements acknowledging AI-based advice. This is securities regulation, not just AI ethics.  
C) The AI exempts the platform from RIA registration  
D) Only disclosure is needed  

**Answer: B**  
**Explanation:** SEC investment advisor regulation: providing personalized investment advice for compensation requires RIA registration. No AI exception exists. The robo-advisor must: (1) Register as RIA or use an RIA-registered entity. (2) Document fiduciary analysis. (3) Verify client identity and suitability. (4) Maintain audit records of advice given. The AI is the delivery mechanism; the regulatory requirements are unchanged by automation.

---

**Q17.** A healthcare application uses Claude to summarize patient notes for care transitions (patient moving from hospital to home care). What information must be preserved regardless of summarization?

A) Only the primary diagnosis  
B) Critical clinical information that must survive any summarization: active medications (with dosages and frequencies), allergies, active diagnoses requiring ongoing treatment, pending lab/test results, follow-up appointments, discharge instructions, and red flags/warning signs requiring emergency contact  
C) Summarize everything equally  
D) Summarization is not appropriate for clinical use  

**Answer: B**  
**Explanation:** Clinical care transition summarization requires a required-fields approach (never summarize away): (1) Medications — wrong dosage = medication error. (2) Allergies — omission = allergic reaction. (3) Active diagnoses — continuity of care. (4) Pending results — follow-up required. (5) Warning signs — patient safety. The CALM-M approach for healthcare: extract structured critical facts FIRST, then summarize context.

---

**Q18.** A legal AI is used to predict outcomes of litigation (will we win/lose?). It claims 85% accuracy based on training data. A law firm uses this to decide whether to take cases. Two years later, it's discovered the 85% was overfitted to historical cases and doesn't generalize. Clients paid fees for cases the AI should have flagged as weak. What architectural failure occurred?

A) The AI model was poorly trained  
B) Multiple failures: (1) No uncertainty quantification — the 85% was presented without confidence intervals or out-of-sample validation. (2) No ongoing performance monitoring comparing predictions to outcomes. (3) Human attorneys were replaced rather than augmented — the AI's prediction became the decision. (4) No disclosure to clients that AI was used for case assessment.  
C) Insufficient training data  
D) The AI should not have been used for this task  

**Answer: B**  
**Explanation:** This case study covers multiple architectural failures: (1) Overfitting presenting as accuracy (requires out-of-sample validation, not training accuracy). (2) Overconfidence without uncertainty bounds. (3) No performance monitoring (would have caught degradation early). (4) Human decision-making replaced (attorneys should augment with AI, not defer to it). (5) Client disclosure ethics (clients deserve to know AI influenced case selection). All are architectural decisions, not just model quality.

---

**Q19.** A financial trading firm wants to use AI to automatically execute trades based on market analysis. The trades are large (>$1M per transaction). What mandatory control architecture applies?

A) Standard logging is sufficient  
B) Large-value automated trading requires: (1) Hard dollar limits per AI-initiated trade. (2) Risk circuit breakers (automatic pause if portfolio impact exceeds threshold). (3) Human authorization required above specific dollar thresholds. (4) Real-time monitoring with human review capability. (5) Pre-trade and post-trade compliance checks. (6) Regulatory reporting (MiFID II, Reg NMS depending on jurisdiction).  
C) Only approval from the CRO is needed before deployment  
D) AI trading above $1M is prohibited  

**Answer: B**  
**Explanation:** Algorithmic trading regulation: large-value automated trading requires multiple mandatory controls beyond standard software deployment. These aren't optional — they're regulatory requirements (SEC Rule 15c3-5 market access rule, FINRA requirements). No AI financial trading system should go live without: position limits, circuit breakers, human oversight mechanisms, compliance checks, and regulatory review. The dollar amounts at stake and systemic market risk make these mandatory.

---

**Q20.** A telemedicine platform uses AI for preliminary symptom assessment before connecting patients to physicians. A patient with a heart attack presents with atypical symptoms (fatigue, jaw pain) instead of classic chest pain. The AI misclassifies as "low urgency." What design failure caused this?

A) The AI model needs more training  
B) Multiple design failures: (1) Atypical presentations not adequately represented in training/few-shot examples. (2) The triage system should default to escalation rather than downgrade for any cardiac symptom combination. (3) No SPIDER-P (safety trigger) for high-risk symptom combinations regardless of confidence score. (4) No "when in doubt, escalate" instruction in the system prompt for cardiovascular presentations.  
C) The patient described symptoms incorrectly  
D) Telemedicine AI cannot handle emergencies  

**Answer: B**  
**Explanation:** Atypical presentations are a known clinical challenge. The architecture must address this explicitly: (1) Few-shot examples covering atypical presentations of high-acuity conditions. (2) Safety trigger: any symptom combination including {jaw pain, shoulder pain, fatigue, shortness of breath, nausea} → automatic cardiac escalation regardless of AI confidence. (3) "Err on the side of urgent escalation for cardiovascular symptoms." (4) Default safe behavior: uncertainty → escalate, not "low urgency."

---

**Q21.** A law firm's AI contract review tool flags "risk clauses" with red, yellow, green ratings. Junior associates trust the ratings and don't read flagged green clauses. Three months in, a clause that creates unlimited liability is missed — it was rated green. What went wrong architecturally?

A) The AI was inaccurate  
B) Automation bias reinforced by UI design: the green rating created excessive confidence; design should prevent over-reliance: (1) No "safe" classification — only "requires urgent review," "requires standard review," "flagged for attorney attention." (2) Random sampling where attorneys are asked to review "green" clauses to maintain vigilance. (3) Explicit instructions that attorneys must read, not just check ratings.  
C) Junior associates should not use AI tools  
D) The rating system needs more categories  

**Answer: B**  
**Explanation:** UI design creates automation bias. A green "safe" rating is the problem — it creates the "this is fine" mental shortcut. Redesign: (1) Never categorize clauses as "safe" — all require attorney review. (2) The AI's role: prioritize what needs attention first, not certify what can be skipped. (3) Build in verification mechanisms. Legal AI should increase attorney efficiency, not replace attorney judgment. This is human-AI teaming design.

---

**Q22.** A healthcare AI startup wants to claim their AI is "HIPAA compliant." What is technically required for this to be accurate?

A) Encrypt data in transit and at rest  
B) HIPAA compliance is an organizational/process standard, not just a technical one: (1) BAA signed with all business associates (including AI provider). (2) Security Risk Analysis documented. (3) Technical safeguards (encryption, access controls, audit logs). (4) Administrative safeguards (policies, training, breach procedures). (5) Physical safeguards (hardware security). "HIPAA compliant" requires all three safeguard categories.  
C) Obtain HIPAA certification (it doesn't exist)  
D) Comply with the encryption standard only  

**Answer: B**  
**Explanation:** HIPAA is not a certification — it's an ongoing compliance standard. There is no official "HIPAA certification." Claims of HIPAA compliance require: (1) All three safeguard pillars (technical, administrative, physical). (2) Documentation and policies. (3) BAAs with every business associate. (4) Breach notification procedures. (5) Regular risk assessments. Encryption alone ≠ HIPAA compliance. This is important for architects advising on healthcare deployments.

---

**Q23.** A legal AI tool operates in multiple countries (US, UK, EU). Contract law differs significantly across jurisdictions. The tool uses the same knowledge base for all. What design is required?

A) One knowledge base works globally  
B) Jurisdiction detection + jurisdiction-specific knowledge: (1) Detect applicable jurisdiction from context (client location, governing law clause). (2) Retrieve jurisdiction-specific legal requirements. (3) Flag jurisdictions where the tool is not validated. (4) System prompt: "Always identify the applicable jurisdiction before providing guidance. If the jurisdiction is unclear, ask before proceeding."  
C) Disclaimer that users should consult local counsel  
D) Operate only in one jurisdiction  

**Answer: B**  
**Explanation:** Multi-jurisdictional legal AI requires explicit jurisdiction handling. A contract clause legal in the US may be unenforceable in the EU (GDPR, consumer protection). The architecture: (1) Jurisdiction identification step (before any substantive analysis). (2) Jurisdiction-specific knowledge retrieval. (3) Uncertainty escalation ("this jurisdiction has not been validated — consult local counsel"). A single knowledge base without jurisdiction-routing will give wrong answers in some jurisdictions.

---

**Q24.** A pharmaceutical company uses Claude to assist in clinical trial protocol design. The protocol will be submitted to the FDA. What is the AI's appropriate role?

A) AI can finalize protocols for submission  
B) AI as drafting assistant only: (1) AI drafts protocol sections based on regulatory templates and prior approved protocols. (2) Qualified clinical researchers review and modify every section. (3) Regulatory affairs professionals validate FDA compliance. (4) No AI-only submission — human expert review is mandatory. (5) Document that AI was used in drafting (FDA expects transparency about tools used).  
C) AI should not be involved in regulated submissions  
D) AI is sufficient for standard protocol sections  

**Answer: B**  
**Explanation:** FDA regulatory submissions require human expert accountability. Clinical trial protocols affect patient safety and require GCP (Good Clinical Practice) compliance. AI role: assist qualified humans (accelerate drafting, ensure completeness against regulatory templates). Human role: clinical judgment, regulatory expertise, accountability for the submission. The FDA expects human expert review and sign-off; AI assistance is acceptable, AI autonomy is not.

---

**Q25.** A debt collection agency wants to use AI for automated debt collection calls. What regulatory constraints fundamentally limit this use case?

A) AI is unrestricted in debt collection  
B) FDCPA (Fair Debt Collection Practices Act) constraints: (1) Requires human identification when asked. (2) Prohibits calling at certain times. (3) Requires ability to honor cease communication requests immediately. (4) Prohibits deceptive practices — an AI not disclosing it's AI may violate this. (5) State laws may prohibit fully automated collection calls. (6) CFPB regulations on AI in financial services add additional requirements.  
C) Only privacy disclosures are needed  
D) AI calls are equivalent to human calls under FDCPA  

**Answer: B**  
**Explanation:** FDCPA was written before AI but applies to automated collection. Key constraints: (1) "Are you a human?" — Claude must not deceive. (2) Time restrictions (cannot call before 8am or after 9pm — requires accurate timezone handling). (3) Immediate cease-and-desist compliance — the AI must honor stop-contact requests in real-time. (4) Disclosure that calls are from a debt collector. (5) Many states additionally require disclosure of AI. This is a heavily regulated domain with fundamental constraints.

---

## Score: /25 | Pass: 19/25 (75%)
