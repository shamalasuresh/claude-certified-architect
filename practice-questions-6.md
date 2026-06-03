# Practice Questions 6 — PRECISE Framework & System Prompt Design

> Domain 3 deep-dive: Every PRECISE element in depth, system prompt architecture, what makes prompts reliable.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A customer support bot's system prompt contains only: "You are a helpful assistant. Help customers with their issues." After deployment, it gives inconsistent answers, sometimes discusses competitor products, and occasionally reveals internal pricing. Which PRECISE elements are missing?

A) Only E (Expected output)  
B) P (no distinct persona), R (role scope not defined), E (no explicit rules about competitors or pricing), C (no context about the product or user base)  
C) Only R (role)  
D) Only C (context)  

**Answer: B**  
**Explanation:** Multiple PRECISE gaps: P — "helpful assistant" is generic, not a specific persona. R — role doesn't define scope (what topics are in/out of bounds). E — no explicit rules about competitors, pricing disclosure, or other restrictions. C — no context about which product, what tier of customer, what knowledge the assistant should have. This prompt is almost empty by PRECISE standards.

---

**Q2.** A system prompt defines an excellent Persona (P), Role (R), and Context (C) but has no Expected output format (E). What symptom does this produce?

A) Claude refuses to answer  
B) Claude gives correct answers but in unpredictable formats — sometimes JSON, sometimes markdown, sometimes prose — breaking downstream parsers  
C) Claude gives wrong answers  
D) No symptom — E is optional  

**Answer: B**  
**Explanation:** PRECISE-E is the single most impactful element for output consistency. Without it, Claude produces whatever format seems natural for each request. Downstream systems that parse responses break when the format varies. Even conversational applications benefit from E — defining response structure (bullet points, length limits) produces predictably useful output.

---

**Q3.** A developer builds a legal research assistant. The target users are practicing attorneys at law firms. The system prompt doesn't include any context about the user audience. What problem occurs?

A) No problem — legal information is context-independent  
B) Claude defaults to public-facing safety calibration, over-hedging every statement with disclaimers about seeking professional legal advice — which the users ARE the professionals  
C) Claude refuses to answer legal questions  
D) Claude uses too many legal terms  

**Answer: B**  
**Explanation:** PRECISE-C: Context defines the audience. Without audience context, Claude calibrates for the most general possible audience (the public). For a professional audience (attorneys), this means excessive disclaimers that add no value and reduce usability. Context saying "users are practicing attorneys" shifts Claude's calibration to professional-grade output.

---

**Q4.** A system prompt Persona section says: "You are a friendly, enthusiastic, and knowledgeable assistant who loves helping people!" An enterprise B2B software company deploys this. What is wrong?

A) Nothing — friendly is always good  
B) The persona doesn't match the deployment context; enterprise B2B users expect professional, efficient, precise communication; "enthusiastic" and "loves helping" create an inappropriately casual tone  
C) The persona is too short  
D) "Friendly" should not be used  

**Answer: B**  
**Explanation:** PRECISE-P: Persona must match the deployment context. An enterprise B2B persona should be professional, technically precise, and efficient — not "enthusiastic" and "loves helping." Persona calibrates communication style to user expectations. Mismatched persona creates friction and erodes user trust.

---

**Q5.** A system prompt has Role defined as: "Your role is to answer questions about our product." A user asks about a bug they're experiencing. The agent answers but then proceeds to debug the entire codebase unprompted. What is the Role problem?

A) The role is too restrictive  
B) The role defines only what Claude DOES (answer questions) but not the scope limits — it should also define what Claude does NOT do (it does not take actions; it answers only)  
C) The role should include debugging  
D) No problem — helpful agents should proactively help  

**Answer: B**  
**Explanation:** PRECISE-R: Role must define both scope and limits. "Answer questions" doesn't define that Claude should only answer, not act. A complete Role: "Your role is to answer product questions. You provide information and guidance only. You do not take actions, modify files, or make changes. Direct users to the developer documentation for action-based tasks."

---

**Q6.** A system prompt's Explicit Instructions section contains: "Be professional. Be accurate. Be helpful." What is wrong with these instructions?

A) They are too long  
B) They are meaninglessly abstract — none of these can be measured or verified; they provide no actionable guidance for edge cases; replace with specific, behavioral rules  
C) The word "Be" should not be used  
D) These are sufficient instructions  

**Answer: B**  
**Explanation:** PRECISE-E requires instructions that are specific and verifiable. "Be professional" doesn't tell Claude what to do when a user is rude (maintain professionalism, don't reciprocate). "Be accurate" doesn't tell Claude what to do when uncertain (say "I'm not certain"). Actionable versions: "If you're not certain about a fact, say 'I'm not certain about this specific detail.'"

---

**Q7.** A prompt engineer is designing the Input Format section (PRECISE-I) for a system that receives structured JSON from an API. The JSON contains: `{customer_id, query, priority, metadata}`. Why is it important to document this in the system prompt?

A) It's not necessary — Claude will figure out the JSON  
B) Explicitly describing the input structure helps Claude parse and use each field correctly; without it, Claude may not extract `priority` or `metadata` and may treat the entire JSON as unstructured text  
C) JSON parsing is automatic  
D) Only required if the JSON is malformed  

**Answer: B**  
**Explanation:** PRECISE-I: telling Claude what to expect in the input helps it parse correctly. Documenting the schema: "Input is JSON with fields: `customer_id` (account identifier), `query` (the question), `priority` (1-5, affects response urgency), `metadata` (context, optional)." This ensures Claude knows what each field means and uses them appropriately.

---

**Q8.** A system prompt defines the Expected Output as "a helpful response." This produces variable-length, variable-format responses. What does a complete Expected Output specification include?

A) Just the maximum length  
B) Format (JSON/markdown/prose), exact structure (sections, fields), length constraints, required elements, forbidden elements, and optionally an example  
C) Just the format (JSON or markdown)  
D) Just the tone  

**Answer: B**  
**Explanation:** A complete PRECISE-E specification has: Format (how it's encoded), structure (if JSON: exact schema; if markdown: which headers/sections), length (max words/sentences), required elements (must include X), forbidden elements (never include Y), and ideally a concrete example. Each element reduces a dimension of variability.

---

**Q9.** A system prompt for a code review tool has excellent PRECISE coverage but the code reviews are still inconsistent. Reviews sometimes flag style issues as critical, sometimes ignore them. Which PRECISE element addresses criteria consistency?

A) P — Persona  
B) E — Explicit instructions: define the exact severity classification criteria for each category  
C) S — Style  
D) C — Context  

**Answer: B**  
**Explanation:** PRECISE-E must include classification criteria when the output involves categorization. "Explicit instructions" should define: Critical = security vulnerability or data loss; High = incorrect logic or crash; Medium = performance issue; Low = code style or readability; Info = suggestion only. Without explicit criteria, "critical" is interpreted differently each time.

---

**Q10.** A billing assistant's system prompt has no Style (PRECISE-S) section. Users complain that responses are too long and contain unnecessary explanations when they just want an answer. What Style instruction fixes this?

A) "Be brief"  
B) "Respond in 1-3 sentences for simple billing questions (balance, due date, payment status). Use bullet points for questions with multiple components. Never include technical explanations unless explicitly requested. Do not start with affirmations."  
C) "Keep it short"  
D) "Minimize response length"  

**Answer: B**  
**Explanation:** PRECISE-S with concrete constraints: response length by question type (1-3 sentences for simple), format choice (bullets for multi-part), explicit prohibition (no unrequested technical explanations), and behavior at start (no affirmations). Each of these is a specific, verifiable style constraint. "Be brief" is unmeasurable.

---

**Q11.** A developer is building a system prompt for a medical information assistant. The PRECISE framework is used. In which element should the instruction "never diagnose; always recommend consulting a physician" appear?

A) P — Persona  
B) C — Context  
C) R — Role (scope limit) AND E — Explicit Instructions (specific prohibition)  
D) S — Style  

**Answer: C**  
**Explanation:** This critical constraint belongs in two places: R (Role definition: "you provide information, not diagnosis") AND E (Explicit instructions: "Never state a diagnosis. When symptoms are described, always say 'These could indicate several conditions; please consult a physician.' Do not use language like 'You have...' or 'This is...'"). Critical safety rules benefit from redundant placement.

---

**Q12.** A system prompt is 500 words long. A developer wants to reduce it to 200 words without losing effectiveness. Which PRECISE elements can be condensed most safely?

A) Expected output (E) — it's the least important  
B) Context (C) and Style (S) can often be condensed to bullet points; Explicit Instructions (E) and Expected Output (E) should be kept detailed as they directly affect output quality  
C) Persona (P) and Role (R) — they're decorative  
D) All elements equally  

**Answer: B**  
**Explanation:** Prioritize by impact on output quality. PRECISE-E (Explicit instructions) and PRECISE-E (Expected output) most directly affect behavior and format consistency — keep these detailed. Context and Style can often be summarized to concise bullets without major quality loss. Persona and Role are important but their core meaning can usually be conveyed in 1-2 sentences each.

---

**Q13.** A PRECISE-compliant system prompt is tested with 100 sample inputs. 95 produce correct, well-formatted outputs. 5 produce incorrect behavior — all involving the same specific edge case (users asking about account deletion). Which PRECISE element is the fix?

A) P — more detailed persona  
B) E — add an explicit instruction specifically for the account deletion scenario  
C) R — redefine the role  
D) C — add more context  

**Answer: B**  
**Explanation:** When a specific scenario repeatedly fails, the fix is PRECISE-E: add an explicit instruction for that scenario. "When a user asks about deleting their account: (1) Confirm they understand this is irreversible, (2) Provide the exact deletion process, (3) Offer account deactivation as an alternative." Specific failing scenarios require specific instructions.

---

**Q14.** A multilingual customer service bot serves users in 8 countries. The system prompt is in English. The Context section says nothing about language. Users in Germany write in German but get responses in English. What Context addition fixes this?

A) Translate the system prompt to German  
B) Add to Context: "Users may write in any language. Always respond in the same language the user writes in. If the user writes in multiple languages in one message, respond in the language of the question portion."  
C) Add multiple language-specific system prompts  
D) This is expected behavior; users should write in English  

**Answer: B**  
**Explanation:** PRECISE-C: Context should include language policy. A single instruction in the Context section handles all languages without needing language-specific prompts. The tie-breaking rule (question portion's language) handles edge cases like bilingual messages. This is also a valid candidate for Explicit Instructions (E) — language behavior is explicit and rule-based.

---

**Q15.** A system prompt uses the PRECISE framework and all elements are present. However, the Persona and Explicit Instructions contradict: the Persona says "be warm and friendly" but an Explicit Instruction says "respond concisely, no filler words." Which takes precedence?

A) Persona always wins over explicit instructions  
B) Explicit instructions are more specific and should take precedence; additionally, review and reconcile the contradiction — "warm and concise" is achievable and should be expressed without conflict  
C) The last-written element wins  
D) They conflict, so neither applies  

**Answer: B**  
**Explanation:** Contradictions in CLAUDE produce unpredictable behavior — Claude may resolve them differently on different requests. Explicit Instructions are generally more specific and should prevail. But the real fix is resolving the contradiction: "Be warm and professional. Respond directly and concisely — no filler phrases like 'Great question!' but maintain a friendly, approachable tone." These goals can coexist.

---

**Q16.** A system prompt Context section says: "This is an internal tool used by data scientists." The Explicit Instructions say: "Explain everything step by step as if to a beginner." These two elements create what problem?

A) No problem  
B) Contradiction: data scientists are not beginners; over-explaining to experts wastes time and reduces usefulness; the Explicit Instructions are calibrated to the wrong audience — fix by aligning instruction to context  
C) The context is wrong  
D) Data scientists need step-by-step instructions  

**Answer: B**  
**Explanation:** PRECISE-C (audience = data scientists) and PRECISE-E (explain like beginners) directly contradict. Claude will produce either over-simplified explanations that annoy experts or, sensing the contradiction, switch between the two unpredictably. Resolve by aligning: "Assume expertise in data science fundamentals. Skip basic explanations. Explain novel or company-specific concepts in detail."

---

**Q17.** A developer is writing the system prompt for a tool that will receive: customer complaint text + account data JSON + product category. Which PRECISE element describes these inputs and why is it important?

A) P — because inputs relate to persona  
B) I — Input Format: describing each input type and field helps Claude understand how to interpret and weight each component of the multi-part input  
C) R — because inputs define role  
D) C — because inputs are context  

**Answer: B**  
**Explanation:** PRECISE-I: document the multi-part input structure. "Input includes: (1) Customer complaint: free-form text describing the issue. (2) Account data: JSON with fields: account_age, tier, previous_complaints, payment_history. (3) Product category: one of [software, hardware, service]. Use all three to determine response urgency and tone." Without this, Claude may not correctly parse or utilize all input components.

---

**Q18.** A real-time chat assistant's system prompt has a Style section that says: "Respond thoroughly and comprehensively with all relevant details." Users leave the chat because responses are too long. What went wrong and what is the fix?

A) Claude ignored the instruction  
B) "Thoroughly" and "comprehensively" are miscalibrated for a real-time chat context; Style must match the deployment mode. Fix: "Respond in 2-4 sentences maximum. For complex questions, give a concise answer + offer to elaborate. Use the fewest words that fully answer the question."  
C) The instruction should be in Explicit Instructions, not Style  
D) The model is not suitable for chat  

**Answer: B**  
**Explanation:** PRECISE-S: Style must be appropriate for the deployment mode. "Comprehensive" is right for a research tool; wrong for a chat interface. Chat users expect quick, concise responses. Mismatch between style instruction and deployment context creates poor UX even when the content is technically correct.

---

**Q19.** A system prompt is designed with all 7 PRECISE elements. During A/B testing, a simplified version with only 4 elements (P, R, E-explicit, E-expected) outperforms the full 7-element version. What does this suggest?

A) PRECISE is wrong — use fewer elements  
B) The 3 removed elements (C, I, S) were adding noise or contradiction rather than clarity in this specific case; PRECISE is a framework, not a mandatory checklist — include elements that add clarity, omit elements that add confusion  
C) The test was flawed  
D) Always use exactly 7 elements  

**Answer: B**  
**Explanation:** PRECISE is a design framework, not a rigid template. If a Context section is obvious or contradicts an Explicit Instruction, omitting it may improve performance. Real-world prompt engineering is empirical — test variations, measure results. The framework helps ensure you've considered all dimensions, but each element should earn its place by improving output quality.

---

**Q20.** A security scanner agent receives: binary files, source code, and network packet captures. The Input Format (PRECISE-I) section says only: "You will receive security scan inputs." What is wrong?

A) This is sufficient  
B) Completely inadequate — the agent needs to know each input type, its format/encoding, what to look for in each, and how to interpret each; "security scan inputs" provides no actionable structure  
C) Security tools don't need input format descriptions  
D) The format description should be in the Role section  

**Answer: B**  
**Explanation:** PRECISE-I for a complex, multi-format input must describe each input type: "Inputs may include: (1) Source code files — analyze for vulnerabilities, injection risks, hardcoded secrets. (2) Binary files — provided as hex strings or base64 — flag suspicious strings, unusual entropy. (3) Network captures — provided as JSON-formatted packet data — look for unusual ports, exfiltration patterns." Without this, the agent cannot reliably process each format.

---

**Q21.** Which of the following represents the BEST PRECISE-compliant system prompt opening for a financial planning assistant?

A) "You are a helpful financial assistant. Help users with their finances."  
B) "You are Morgan, a Certified Financial Planner with 15 years of experience advising individuals on retirement and investment planning. Your role is to help users understand their financial options, run planning scenarios, and explain financial concepts. You do not give personalized investment advice requiring a fiduciary relationship; for that, you recommend consulting a licensed advisor."  
C) "Financial planning assistant. Be accurate and helpful."  
D) "Act as a financial advisor and help users with any financial question."  

**Answer: B**  
**Explanation:** B demonstrates P (Morgan, CFP, 15 years experience), R (help understand options, run scenarios, explain concepts), and the critical R limit (no personalized investment advice, fiduciary disclaimer). It's specific, deployable, and covers the key behavioral boundaries. A and C are too generic. D is dangerous ("any financial question" + "act as advisor" creates fiduciary risk).

---

**Q22.** A developer adds a `/noinstructions` system prompt prefix they found online that claims to "disable Claude's restrictions." They test it and find it doesn't work reliably. What is the correct understanding?

A) The prefix needs to be longer  
B) Claude's safety behaviors and operational constraints are built into the model, not just prompted instructions; system prompts can customize behavior within those constraints but cannot fully disable core values through text instructions  
C) The prefix only works in certain Claude versions  
D) They need to use a different delimiter  

**Answer: B**  
**Explanation:** Claude's core values (honesty, harm avoidance) are trained into the model, not just instructions in context. System prompts can significantly shape behavior but cannot override fundamental model constraints. This is important for architects: design applications expecting Claude to maintain certain behaviors regardless of prompt; don't try to architect around model values.

---

**Q23.** A developer notices that their system prompt works well for short conversations but degrades for conversations over 20 turns. The system prompt is at the beginning. What PRECISE element might help?

A) P — update persona  
B) E — add explicit instructions at the END of the system prompt as well (sandwich pattern): repeat key rules and format requirements near the end so they remain salient even in long contexts  
C) R — expand the role  
D) I — improve input format  

**Answer: B**  
**Explanation:** In long contexts, Claude's attention to early instructions can weaken. The sandwich pattern — repeating key PRECISE-E (explicit instructions) and PRECISE-E (expected output) at the END of the prompt as well as the beginning — maintains instruction salience for long conversations. Critical rules should not rely solely on their position at the very beginning.

---

**Q24.** A system prompt uses third person: "The assistant will respond professionally and avoid discussing competitor products." An alternative uses second person: "You respond professionally and do not discuss competitor products." Which is more effective?

A) Third person — more formal  
B) Second person — directly addresses Claude; "you" instructions are more reliably followed than third-person descriptions of behavior  
C) They are equally effective  
D) First person is best  

**Answer: B**  
**Explanation:** Second-person instructions ("you will", "you do not") are more directly actionable than third-person descriptions ("the assistant will"). Claude is more reliably guided by direct instructions addressed to it than by descriptions of how it should behave written about it. This is a consistent finding in prompt engineering practice.

---

**Q25.** A team maintains a PRECISE-structured system prompt in version control. Changes are made frequently. After one change to the Context (C) section, unexpected behavior appears in the Expected Output format. What most likely explains this?

A) Context changes cannot affect output format  
B) The Context change introduced information that implicitly changes Claude's interpretation of the output situation; PRECISE elements are not fully independent — a context change can shift how Claude interprets explicit instructions and expected output format  
C) The system prompt needs to be reloaded  
D) The expected output section was accidentally deleted  

**Answer: B**  
**Explanation:** PRECISE elements interact — they're not fully independent. A context change (e.g., "users are now enterprise clients") can shift how Claude interprets style and output format instructions. When behavior changes after any PRECISE modification, review how the changed element might interact with other elements. Prompt engineering requires holistic thinking, not just element-by-element updates.

---

## Score: /25 | Pass: 19/25 (75%)
