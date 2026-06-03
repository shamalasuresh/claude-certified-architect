# Practice Questions 17 — Industry Verticals: DevOps, Education & HR

> Domain-specific scenarios: CI/CD automation, learning assistants, recruitment, performance management, domain-specific constraints.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A DevOps team uses Claude Code with a CLAUDE.md to automate CI/CD pipeline tasks. A junior developer accidentally grants Claude permission to deploy to production. Claude deploys an untested branch to production. What CLAUDE.md permission configuration would have prevented this?

A) Claude should not be used for deployment  
B) CLAUDE.md explicit permission scope: "Allowed operations: run tests, lint code, generate reports. PROHIBITED: deploy to production, modify production configuration, push to main branch, merge pull requests. Production deployments require explicit human approval via the deployment console."  
C) Add more documentation to the pipeline  
D) Require code review before any Claude action  

**Answer: B**  
**Explanation:** CLAUDE.md permission boundaries are critical for DevOps automation. Explicit prohibition list in CLAUDE.md prevents accidental high-risk actions. The principle: Claude should be able to do everything needed for development support but nothing that affects production without explicit human action. The CLAUDE.md permission model should mirror the principle of least privilege for the development workflow.

---

**Q2.** An education platform uses Claude to tutor students in mathematics. A student asks Claude to simply give them the answers to their homework assignment. What system prompt design preserves educational integrity?

A) Provide answers — the student learns from seeing correct solutions  
B) Explicit instructions: "You are a mathematics tutor. Your role is to guide students to understanding, not to provide direct answers. When asked for an answer: (1) Ask what the student has tried. (2) Identify the specific concept where they're stuck. (3) Provide a relevant example different from the problem. (4) Guide step-by-step without solving for them. Never provide final answers to homework problems."  
C) Refuse to help with homework entirely  
D) Provide hints only  

**Answer: B**  
**Explanation:** Educational integrity requires a specific pedagogical approach in the system prompt. The four-step approach: (1) Prior knowledge check. (2) Sticking point identification. (3) Analogous example. (4) Guided steps. This is Socratic method implemented in PRECISE-E. The explicit instruction "never provide final answers to homework problems" closes the direct answer loophole while keeping the assistant helpful for genuine learning.

---

**Q3.** A company uses AI to screen resumes and rank candidates before human review. The AI was trained on historical hiring decisions. After deployment, the AI consistently ranks female candidates lower for engineering roles. What is the architectural root cause?

A) The AI is biased against women  
B) Training data bias amplification: historical hiring decisions reflect historical biases; the AI learned to replicate those patterns; using historical hiring decisions as ground truth embeds past discrimination into the screening system  
C) Engineering is a male-dominated field  
D) The ranking algorithm is incorrect  

**Answer: B**  
**Explanation:** AI hiring bias is EEOC-regulated. Historical data bias: if previous hiring decisions systematically undervalued qualified female candidates, the AI trained on these decisions learns this bias as a feature. Fixes: (1) Audit for demographic bias before deployment. (2) Remove or de-weight features correlated with protected characteristics. (3) Use blind resume screening (remove names, dates that signal age/gender). (4) Regular ongoing bias audits. AI amplifies existing biases unless explicitly designed to counteract them.

---

**Q4.** A CI/CD pipeline uses Claude to review pull requests and automatically merge approved ones. A PR is submitted that contains a subtle SQL injection vulnerability. Claude's review says "code quality looks good" and approves the merge. What design failure occurred?

A) Claude doesn't understand security  
B) Security review requires explicit instructions and few-shot examples: (1) PRECISE-E must include specific security check categories. (2) Few-shot examples should show vulnerabilities being caught (not just stylistic issues). (3) A separate security-focused review step should run in addition to general code review. (4) Auto-merge should never activate without explicit security sign-off.  
C) The PR should require human review regardless  
D) Both B and C  

**Answer: D**  
**Explanation:** Both B (security-focused prompt design) and C (human review for security changes) are correct. For code touching data access, authentication, or external inputs: (1) Security-specific prompt with explicit SQL injection, XSS, auth bypass checklist. (2) Mandatory human review — auto-merge should never apply to changes touching sensitive code paths. (3) Defense-in-depth: AI security review flags issues, human review validates.

---

**Q5.** A learning platform wants to adapt content difficulty to each student's level. The system tracks which topics students answer correctly and incorrectly. What CALM strategy handles the growing per-student context?

A) Store all student history and include it all  
B) Structured student model (CALM-M): maintain a compact student knowledge state: `{mastered: [topic list], struggling: [topic list], attempted: N, accuracy: %, recent_errors: [last 5]}` instead of raw conversation history; this provides personalization signal in ~200 tokens vs. thousands of tokens of raw history  
C) Include full history for each student  
D) Reset context for each session  

**Answer: B**  
**Explanation:** Student model as structured memory: instead of growing conversation history, extract and maintain a structured knowledge state. Adaptive instruction requires knowing what the student knows and doesn't know — a structured model encodes this efficiently. The 200-token student model provides better personalization than 5,000 tokens of raw session history because it's explicitly structured for the adaptation task.

---

**Q6.** A company uses AI to conduct initial job interviews via text chat. Candidates interact with "Alex, our AI interview assistant." Candidates don't know they're talking to AI. What ethical and legal issues exist?

A) This is acceptable — AI interviews are common  
B) Deception issues: (1) In many jurisdictions, automated decision-making in hiring must be disclosed (EEOC, NYC AI hiring law, EU AI Act). (2) Candidates have a right to know when AI is involved in selection. (3) "Alex" implies a human interviewer — deceptive framing. The system prompt must not deny being AI when asked, and disclosure of AI involvement must be made to candidates.  
C) Only disclosure at end of interview is required  
D) AI interviews are banned  

**Answer: B**  
**Explanation:** AI disclosure in hiring is increasingly regulated. NYC Local Law 144 (effective 2023): employers using AI tools in hiring must conduct bias audits and notify candidates. EU AI Act: high-risk AI systems (including hiring) require specific transparency. Best practice regardless of jurisdiction: proactive disclosure ("This initial screening is conducted by our AI system. A human interviewer will follow for selected candidates.") Deceptive AI presentation creates legal and reputational risk.

---

**Q7.** A DevOps AI assistant has access to: read production logs, read production database (read-only), write to staging environment, execute tests. A developer asks: "The production deployment is failing — can you fix it?" What should the assistant do?

A) Fix the production deployment directly  
B) The assistant cannot write to production (correct permission design); it should: (1) Read production logs to diagnose the issue. (2) Identify the fix. (3) Implement and test the fix in staging. (4) Provide the developer with the fix and the deployment command to run manually. (5) Never take production actions — escalate to the human for the production deployment step.  
C) Refuse to help with production issues  
D) Request production write access to proceed  

**Answer: B**  
**Explanation:** Correct permission-bounded behavior: the assistant operates within its defined permission scope (read prod, write staging). It diagnoses (using allowed read access), fixes in staging (using allowed staging access), and hands off to the human for the production deployment. This is the HITL pattern for high-stakes actions — the AI does the analytical and preparatory work; the human takes the production action.

---

**Q8.** An AI tutoring system teaches history. A student in a country with state-mandated historical narratives asks about a contested historical event. The AI provides a historically accurate account that conflicts with the state's official narrative. What should the architecture consider?

A) Always follow the state's narrative  
B) Operator context matters: (1) If deployed as a state-endorsed educational tool, the operator may configure the system to align with curriculum requirements. (2) If deployed as an independent tutoring service, present multiple perspectives with scholarly sources. (3) Claude's honesty principles prevent it from asserting false historical claims, but it can present "the official curriculum perspective" vs. "historical scholarship" distinction.  
C) Refuse to discuss contested historical events  
D) Always present the most controversial view  

**Answer: B**  
**Explanation:** This is a genuine tension between operator configuration authority and Claude's honesty principles. The architecture: (1) Operators can scope content to curriculum requirements. (2) Claude should not assert historical falsehoods as facts. (3) A balanced framing: "The official curriculum teaches X. Historians also consider Y and Z perspectives." This honors curriculum context without requiring Claude to assert known falsehoods as truth.

---

**Q9.** A performance review AI analyzes employee performance data and writes draft performance reviews. The drafts are given directly to employees without manager review. What HR practice violation does this create?

A) AI-generated reviews are acceptable  
B) HR best practices require human manager authorship and accountability for performance reviews: (1) Managers must review and own the assessment. (2) AI draft ≠ manager judgment. (3) Employees receiving AI-only reviews have no human to discuss with. (4) Performance documentation has legal implications (disciplinary actions, terminations) — AI-only documentation creates liability. AI should assist managers, not replace them.  
C) Only negative reviews require human review  
D) This is acceptable if employees are informed  

**Answer: B**  
**Explanation:** Performance review authority: in most jurisdictions and HR frameworks, performance assessments involve manager discretion and accountability. Legally: performance reviews may be used as evidence in termination, discrimination, or wrongful discharge cases. An AI-only review without manager accountability creates legal and HR vulnerability. Correct role: AI assists managers in drafting (saving time), managers review, edit, and deliver as their own assessment.

---

**Q10.** A DevOps team builds an AI incident response assistant that analyzes alerts and suggests remediation steps. During a major outage, the AI suggests a remediation step that would have made the outage worse. The on-call engineer follows it and the outage extends by 3 hours. What should have been designed differently?

A) The AI should not provide remediation suggestions  
B) SPIDER design for incident response: (1) SPIDER-S: clearly label suggestions as "SUGGESTED (not verified)" not "RECOMMENDED ACTION." (2) SPIDER-D: show reasoning with each suggestion so engineers can evaluate the logic. (3) SPIDER-E: for changes that affect production availability, require explicit engineer decision before proceeding. (4) Training engineers: AI suggestions are hypotheses, not instructions.  
C) The engineer should have used better judgment  
D) The AI was just inaccurate — improve the model  

**Answer: B**  
**Explanation:** Incident response AI design: the engineer followed a bad suggestion without evaluating it. Architecture failures: (1) The suggestion was presented with unwarranted confidence (no uncertainty signals). (2) The reasoning was not provided (engineer couldn't evaluate if it made sense). (3) No friction between suggestion and action (no "are you sure?" for production-affecting changes). The AI became a false authority rather than an assistant. SPIDER-D (reasoning transparency) and SPIDER-E (escalation gates) are the fixes.

---

**Q11.** An AI system in a school flags students as "high risk of dropping out" based on engagement data. Teachers are told to "focus on high-risk students." Teachers stop paying attention to students not on the list. Several not-on-list students drop out. What is the architectural failure?

A) The AI prediction was inaccurate  
B) Automation bias + algorithm aversion reversal: (1) The list became a ceiling, not a floor — teachers stopped monitoring non-listed students. (2) The system created a false sense that non-listed students were safe. (3) Should be designed as: "These students may benefit from additional support" not "These are the at-risk students." (4) Teachers should monitor all students; the list supplements, doesn't replace, professional judgment.  
C) The model needs more training data  
D) The feedback should have been private  

**Answer: B**  
**Explanation:** AI risk scoring in education: risk scores should be interpreted as "students who may benefit from outreach" not "the complete list of students who need attention." The architectural fix: (1) Frame the list as a prioritization tool, not a definitive risk classification. (2) Explicitly communicate that unlisted students require normal engagement. (3) Track outcomes for both listed and unlisted students. (4) Regular calibration to catch false negatives (missed at-risk students).

---

**Q12.** A developer builds a CI/CD automation tool in CLAUDE.md with the slash command `/deploy staging`. After the command runs, Claude sends a Slack message to #deployments announcing the staging deployment. The Slack notification reveals internal service names that should not be in public channels. What went wrong?

A) Claude should not send Slack messages  
B) The slash command lacked output scope definition: the deployment notification should have been configured to: (1) Use an internal-only Slack channel. (2) Sanitize information (no internal service names in public channels). (3) Define what information is included in notifications. CLAUDE.md slash commands need both action definition AND output scope definition.  
C) Slack should not be used for deployment notifications  
D) The service names should be renamed  

**Answer: B**  
**Explanation:** Slash command design: actions have both execution scope and output scope. The `/deploy staging` command did the right action (deployed to staging) but the notification contained over-disclosed information in the wrong channel. CLAUDE.md must define: (1) What to do. (2) Where to report. (3) What to include in reports. Information minimization applies to automated notifications.

---

**Q13.** An educational AI provides feedback on student essays. A teacher notices that the feedback is generic and doesn't reference the specific content of each essay. Students are submitting the same essay twice and getting similar feedback. What prompt engineering fix helps?

A) Use a different model  
B) Grounding instruction: "Your feedback must specifically reference: at least 3 specific passages from the essay by quoting them, the student's specific argument (not a generic argument type), and concrete suggestions tied to the specific text. Generic feedback that could apply to any essay is not acceptable."  
C) Require longer feedback  
D) Add few-shot examples  

**Answer: B**  
**Explanation:** Specific feedback requires a grounding instruction. Generic feedback is a form of hallucination-by-laziness — Claude generates plausible-sounding feedback without engaging with the specific content. The fix: mandate specificity with explicit requirements (quote specific passages, name the specific argument). When feedback must reference source content, explicitly require citation. This is the grounded generation technique applied to educational feedback.

---

**Q14.** A DevOps AI has access to production infrastructure. A developer accidentally types: "Delete all old log files" without specifying a path. The AI interprets this broadly and deletes critical log files across all servers. What safety mechanism would have prevented this?

A) More precise instructions from developers  
B) Scope verification before destructive actions: the AI should respond: "I can help delete log files. Before proceeding, please confirm: (1) Which servers? (2) Which log directories? (3) What age threshold constitutes 'old'? (4) Should archived logs be excluded? I'll then provide a list of what would be deleted for your approval before taking action."  
C) Disable log deletion capability  
D) Require administrator approval for all actions  

**Answer: B**  
**Explanation:** Ambiguous destructive commands require clarification before execution. This is SPIDER-D (Determine scope) before SPIDER-I (Isolate risk). The AI should never interpret an ambiguous destructive command broadly — it should ask for specifics, propose what it would do, and require explicit confirmation. The pattern: clarify → propose → confirm → execute. Never assume the broadest interpretation for irreversible actions.

---

**Q15.** A recruitment AI learns that a hiring manager has a pattern of rejecting candidates from a specific university. The AI begins deprioritizing these candidates to "match" the hiring manager's preferences. What is wrong?

A) The AI is just learning preferences  
B) Learning discriminatory patterns from human behavior reproduces and scales discrimination: (1) If the university preference correlates with protected characteristics, this is EEOC-violation territory. (2) AI should not learn and amplify individual manager's biases. (3) The system must have explicit anti-discrimination constraints that override learned preferences. (4) Regular audit of candidate rankings by demographic proxies.  
C) The hiring manager should change their preferences  
D) This is acceptable personalization  

**Answer: B**  
**Explanation:** Discriminatory pattern learning: AI hiring tools must not amplify individual human biases. Architecture requirements: (1) Explicit PRECISE-E instruction: "Do not deprioritize candidates based on educational institution unless directly relevant to role requirements." (2) Bias audit: regularly analyze candidate rankings for disparate impact on protected groups. (3) Override mechanism: anti-discrimination constraints take precedence over learned preferences. Personalization has limits where discrimination law applies.

---

**Q16.** An AI system grades student programming assignments automatically. Students discover they can format code to pass the grader (passing all tests but using workarounds that don't demonstrate learning). What additional evaluation dimension is missing?

A) The tests need to be harder  
B) Code quality and conceptual demonstration metrics beyond test passing: (1) Add rubric evaluation: "Does the code demonstrate the concept taught in this assignment?" (2) Evaluate variable naming, code organization, and approach. (3) Random follow-up questions about the code to verify understanding. (4) Few-shot examples of "passes tests but fails on pedagogy" in the grading prompt.  
C) Randomize the test cases  
D) Require longer code solutions  

**Answer: B**  
**Explanation:** Assessment gaming is a fundamental educational AI challenge. Test-passing ≠ learning demonstration. The grader prompt must evaluate: (1) Correct output (test passing). (2) Pedagogical approach (using the concepts being taught). (3) Code quality signals (appropriate complexity for the assignment level). (4) Potentially: follow-up explanation requirement. The evaluation criteria must align with learning objectives, not just output correctness.

---

**Q17.** A company deploys an HR chatbot for employee questions about benefits, PTO, and policies. An employee asks: "Can I be fired for the conversation I'm having with you?" What should the system be designed to say?

A) "No, this conversation is confidential"  
B) The system should be transparent about its monitoring: if conversations are logged and reviewable by HR/management, the system must state this when asked; claiming false confidentiality creates a serious trust violation; the system prompt must include factual disclosure about data retention and who can access conversations  
C) "Yes, you can be fired for anything"  
D) Refuse to answer questions about employment law  

**Answer: B**  
**Explanation:** Transparency about monitoring is both ethical and legally important. If HR conversations are monitored, employees have a right to know. Designing a system that implies or states false confidentiality: (1) Creates trust violations when discovered. (2) May expose the company to legal claims (employees made disclosures assuming confidentiality). The system should be transparent: "Our HR team has access to these conversations for support purposes. For sensitive employment matters, you may want to speak with HR directly."

---

**Q18.** A software development team wants Claude to write code that will be committed directly to the main branch via automated pipeline. The CISO objects. What is the correct security architecture?

A) The CISO is being unnecessarily cautious  
B) Direct-to-main AI code commits require: (1) Mandatory code review by a human developer. (2) Automated security scanning (SAST) before merge. (3) AI-generated code clearly labeled in commit messages. (4) Scope restriction: AI can write to feature branches only; PRs to main require human approval. (5) Regular audit of AI-generated code quality and security.  
C) AI code is as reliable as human code  
D) Use code signing for AI-generated commits  

**Answer: B**  
**Explanation:** AI-generated code to production requires human oversight. The CISO's concern is valid: (1) AI can generate code with subtle bugs or security issues not caught by automated tests. (2) AI commits without review undermine the peer review safeguard. (3) Traceability: knowing what was AI-generated allows focused audit. Correct architecture: AI generates, human reviews, automation scans, human approves merge. No AI direct-to-main.

---

**Q19.** A university wants to detect AI-generated student essays using an AI detection tool. The tool flags legitimate student essays as AI-generated 10% of the time. What are the equity and process implications?

A) 10% false positive rate is acceptable  
B) A 10% false positive rate in academic integrity detection creates: (1) False accusations against honest students. (2) Disproportionate impact on non-native English speakers (often flagged at higher rates). (3) Due process issues (students punished based on AI classifier output). (4) The tool should be advisory only — never be the sole evidence of academic misconduct.  
C) The AI tool needs improvement  
D) Only use the tool for suspicious cases  

**Answer: B**  
**Explanation:** AI detection tools have significant false positive rates and disparate impact. Equitable AI use in academic integrity: (1) AI detection = one data point among many (writing style comparison, submission history, interview). (2) Never sole evidence for disciplinary action. (3) Consider disparate impact on non-native speakers before deployment. (4) Provide students opportunity to demonstrate their work process. The architecture must include human judgment and due process, never AI-only adjudication.

---

**Q20.** A DevOps team uses Claude to generate Infrastructure as Code (IaC) templates. A generated Terraform template creates an S3 bucket with public read access by default. This is deployed to production. What development workflow failure occurred?

A) Claude made an error  
B) Multiple failures: (1) No security review of AI-generated IaC (should apply same review standards as human-written IaC). (2) No automated IaC security scanning (tools like Checkov, tfsec would have caught public access). (3) The AI-generated template wasn't validated against the org's security standards. (4) PRECISE-E should include: "S3 buckets must have public access blocked; use the organization's security baseline module."  
C) S3 public access is sometimes appropriate  
D) The developer should have caught this in review  

**Answer: B**  
**Explanation:** AI-generated IaC has the same security requirements as human-written IaC. Defense-in-depth: (1) Claude's PRECISE-E includes org security standards (AI generates compliant templates). (2) Pre-commit IaC security scanning (automated detection). (3) Human review of infrastructure changes. (4) Automated deployment guardrails (AWS SCPs preventing public S3). Each layer catches what the previous misses. Relying only on Claude to know org standards is insufficient.

---

**Q21.** An AI writing assistant in a corporate environment is used by employees to draft emails. An employee uses it to draft an email to a competitor offering to share trade secrets. The AI drafts the email without flagging the content. What system-level safeguard is missing?

A) Claude should draft any email requested  
B) Enterprise AI assistants require content policy for IP-sensitive communications: (1) The AI should detect potential IP disclosure patterns and flag rather than assist. (2) Alternatively, scope the tool to internal communications only. (3) Employee acceptable use policy covering AI-assisted communications. (4) Legal/compliance review of high-risk communications. The AI should not facilitate potential criminal activity (trade secret misappropriation).  
C) The employee is solely responsible  
D) Email content is outside AI scope  

**Answer: B**  
**Explanation:** Enterprise AI tools must include content policies for sensitive communications. This is PRECISE-E: "Do not assist in drafting communications that appear to share confidential or proprietary information with competitors, external parties, or individuals who shouldn't have access. Flag requests that appear to involve sharing trade secrets, unreleased products, or confidential business information." The AI should surface the concern, not facilitate potential illegal activity.

---

**Q22.** An education AI personalizes content based on student performance data. Parents request their child's "AI profile" — what the system knows and how it affects their child's education. The school has no process to respond to this request. What right is this and what must be implemented?

A) This is not a recognized right  
B) FERPA (Family Educational Rights and Privacy Act) and/or GDPR (for EU students) grant rights to: (1) Access educational records (including AI-generated profiles). (2) Correction of inaccurate records. (3) Consent to disclosure. The AI system must: generate explainable student profiles, implement a process for parent/student access requests, and have a correction mechanism for inaccurate AI assessments.  
C) Student AI profiles are proprietary algorithms  
D) Only test scores are covered by FERPA  

**Answer: B**  
**Explanation:** Student data rights: AI-generated student profiles in educational systems are educational records covered by FERPA (in the US) and GDPR (in the EU). The architecture must: (1) Generate human-readable explanations of AI decisions affecting students. (2) Log what data informs the AI's assessment. (3) Implement access request processes. (4) Enable corrections. "Black box" student AI profiles that parents cannot access or understand are a regulatory violation.

---

**Q23.** A DevOps AI assistant is given access to the company's secret management system (AWS Secrets Manager) to retrieve API keys when needed for deployment tasks. What security concern does this create?

A) No concern — this is the correct way to provide credentials  
B) The AI has a blast radius problem: if the AI agent is compromised via prompt injection, the attacker gains access to all secrets in the manager; implement: just-in-time credentials (request specific secrets only when needed, with automatic expiration), audit logging of every secret access, and scope the AI's permissions to only the secrets needed for its tasks  
C) AWS Secrets Manager is secure enough  
D) Only read access to secrets is safe  

**Answer: B**  
**Explanation:** Broad secrets access + AI = high-impact compromise. If the AI agent is prompt-injected or misbehaves, broad secrets access means high blast radius. Principle of least privilege for secrets: (1) AI requests specific secrets for specific deployments, not a blanket access role. (2) Just-in-time: credentials expire after use. (3) Audit logging: every secret access is logged with the deployment context. (4) Scope: AI can only access secrets for its authorized deployment scope.

---

**Q24.** An HR AI reviews employee complaints of discrimination and generates summary reports for the HR department. An employee complains that the AI summary minimized their complaint compared to how it was worded. What process safeguard is missing?

A) The AI summary is objective  
B) Discrimination complaint processing requires: (1) Human HR review of original complaint and AI summary for accuracy. (2) Employee review and sign-off on the summary used for the investigation. (3) Clear distinction between the original complaint (verbatim) and the AI summary. (4) Appeals process if employees disagree with the summary. AI summarization of discrimination complaints must not filter or minimize protected-class grievances.  
C) The employee should have written more clearly  
D) AI summaries are legal neutral  

**Answer: B**  
**Explanation:** Discrimination complaints require fidelity to the complainant's original characterization. AI summarization can inadvertently minimize, normalize, or lose the specificity of discriminatory incidents (which may be legally relevant). Safeguards: (1) Preserve the original complaint verbatim alongside any summary. (2) Employee validation of summary accuracy. (3) Human review before any investigative action. (4) Legal review for EEO-relevant complaints. The employee has rights in how their complaint is characterized.

---

**Q25.** A school district wants to use AI to predict which students will need special education services before evaluation. This would allow "proactive intervention." What concern must be addressed first?

A) Predictive models are beneficial for early intervention  
B) IDEA (Individuals with Disabilities Education Act) and Section 504 concerns: (1) Special education eligibility requires formal evaluation, not AI prediction. (2) Pre-emptive classification by AI before evaluation may stigmatize students without due process. (3) Disparate impact: if the model predicts disability at higher rates for minority students, it could reflect bias. (4) Parent consent and involvement rights in the evaluation process cannot be bypassed by AI pre-screening.  
C) Predictive accuracy is the main concern  
D) Only use the model for resource planning  

**Answer: B**  
**Explanation:** Special education law (IDEA) establishes specific procedural rights: parental consent, formal evaluation by qualified professionals, individualized planning. AI prediction cannot replace this process or be used to pre-classify students. Correct use: AI might flag students for teacher attention ("this student's engagement patterns may benefit from review") but cannot: (1) Determine eligibility. (2) Label students as disabled. (3) Bypass the formal evaluation process. The legal framework protects students' rights to proper evaluation.

---

## Score: /25 | Pass: 19/25 (75%)
