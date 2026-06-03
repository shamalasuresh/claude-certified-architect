# Practice Questions 15 — Tradeoffs & Edge Cases

> Advanced scenarios: Budget vs. latency vs. reliability, novel constraints, edge cases not covered in single-domain files.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A healthcare startup has $500/month API budget, needs 10,000 patient queries/month answered, requires 95%+ accuracy. Which model and architecture optimize this constraint set?

A) Use Opus for maximum accuracy  
B) Tier by complexity: route simple/known queries to Haiku (~80% of queries), complex/ambiguous queries to Sonnet (~20%); use prompt caching on the shared system prompt; this achieves high accuracy on complex queries while keeping cost within budget  
C) Use Haiku for everything to minimize cost  
D) Use Sonnet for everything  

**Answer: B**  
**Explanation:** Cost-accuracy tradeoff with tiered routing: estimate costs at scale. Haiku is ~20x cheaper than Opus. If 80% of queries are routine (clear questions, well-covered by knowledge base), Haiku handles them well at low cost. The 20% complex queries get Sonnet's higher accuracy at higher cost. Combined: higher average accuracy than Haiku-only, lower cost than Sonnet-only. Prompt caching adds additional savings on shared content.

---

**Q2.** A real-time trading system needs AI-generated recommendations. Latency SLA: 200ms. Typical Claude Sonnet response time: 400-800ms. The developer insists on using Claude. What architectural options exist?

A) Accept the SLA violation  
B) Options: (1) Pre-compute common scenarios: generate recommendations for likely market conditions offline, cache for fast lookup. (2) Use Haiku (faster) + accept slightly reduced accuracy. (3) Redesign: Claude provides background analysis asynchronously, recommendations are rule-based with AI-generated risk context  
C) Override the SLA requirement  
D) Claude cannot be used for real-time systems  

**Answer: B**  
**Explanation:** Latency constraints require architectural adaptation. Claude's inference time is fundamentally limited by model size and API infrastructure. Solutions: (1) Pre-computation: move Claude offline, cache results. (2) Model selection: Haiku is significantly faster. (3) Architectural redesign: separate Claude's planning (async, latency-tolerant) from execution (fast, rule-based). Never promise a latency SLA Claude's API cannot consistently meet.

---

**Q3.** A developer must choose between: (A) single large well-tuned prompt, (B) 3-agent pipeline with specialized prompts. Task: complex document analysis and summarization. Single prompt: 82% accuracy, 1 API call. Pipeline: 94% accuracy, 3 API calls, 3x cost. When is the pipeline justified?

A) Never — cost optimization always wins  
B) When the accuracy improvement (82% → 94%) matters for the use case: for legal, medical, or high-stakes analysis where errors are costly, 3x API cost is trivial compared to error costs; for low-stakes internal summaries, the single prompt is preferred  
C) Always — 94% is always better than 82%  
D) Depends on response time requirements only  

**Answer: B**  
**Explanation:** Cost-accuracy tradeoff evaluation: "What is the cost of a 12% error rate?" For a legal contract: a missed clause could cost millions — 3x API cost is immaterial. For an internal meeting summary: a missed detail has low impact — single prompt preferred. The evaluation formula: `cost_of_accuracy_improvement / (3 × single_call_cost)` — if the accuracy gain justifies the cost, use the pipeline.

---

**Q4.** A developer has a working system but discovers that 5% of responses violate a new regulatory requirement added after launch. They need a fix that works within 24 hours without full system retest. What is the safest approach?

A) Retrain the model  
B) Add a targeted Explicit Instruction (PRECISE-E) to the system prompt covering the new regulation: specific, verifiable rule for the new requirement; test the specific failing scenarios before deploying; this is a targeted fix that can be tested quickly without full regression  
C) Deploy a separate compliance filter model  
D) Manually review all responses  

**Answer: B**  
**Explanation:** Targeted system prompt addition is the fastest safe fix. Adding a specific explicit instruction (rather than a system-wide change) localizes the fix and minimizes regression risk. Test procedure: (1) Write the specific instruction. (2) Test against the 5% failing scenarios. (3) Verify passing scenarios still pass. (4) Deploy with monitoring. A full system retest is appropriate for major changes, not for targeted PRECISE-E additions.

---

**Q5.** A developer evaluates two approaches for a multi-step agent task:

Approach A: One agent with extended thinking (50k thinking tokens)  
Approach B: 3-agent pipeline, each with standard thinking

Approach A is 40% cheaper. Approach B is 25% more accurate. Which factors determine the choice?

A) Always choose the cheaper option  
B) Factors: (1) Task risk level (high-risk → accuracy; low-risk → cost). (2) Latency tolerance (extended thinking adds significant latency; pipeline adds network hops). (3) Interpretability (pipeline provides intermediate results that can be inspected; extended thinking is opaque). (4) Reliability (one agent failure vs. three independent failure points)  
C) Always choose the more accurate option  
D) Always choose the pipeline (more modern)  

**Answer: B**  
**Explanation:** Multi-factor tradeoff. Neither option dominates on all dimensions. Extended thinking: cheaper, single failure mode, opaque. Pipeline: more accurate, interpretable intermediate results, multiple failure modes, more latency. Decision matrix: (1) High-stakes + auditing required → pipeline. (2) High-volume low-stakes → extended thinking (cost). (3) Latency-sensitive → pipeline (no extended thinking latency). No universal answer — evaluate per use case.

---

**Q6.** An organization deploys Claude across 50 internal applications. Maintaining 50 separate system prompts is becoming unmanageable. Prompts have diverged, creating inconsistent behavior. What architecture addresses this?

A) Use one system prompt for all applications  
B) Prompt library with layered composition: a core organizational prompt (brand, ethics, security rules) + domain overlays (legal prompt, marketing prompt, technical prompt) + application-specific instructions; compose at runtime; maintain each layer separately with version control  
C) Hire a team to maintain 50 prompts  
D) Use a single standard model API without customization  

**Answer: B**  
**Explanation:** Prompt library architecture: (1) Core layer: organizational constants (brand voice, security rules, data handling) — maintained by one team. (2) Domain layer: specialized additions per domain. (3) Application layer: minimal application-specific additions. Composition at runtime: `core + domain + app_specific = final_system_prompt`. When the core layer is updated, all 50 applications immediately benefit. This is prompt engineering DevOps.

---

**Q7.** A developer must choose between chunking strategy A (500-token chunks, 92% retrieval accuracy, lower latency) and strategy B (200-token chunks, 95% retrieval accuracy, higher latency + more chunks = more API calls to embed/store). For a high-traffic production system, which factors matter most?

A) Always use the most accurate strategy  
B) For high-traffic production: cost per request (more chunks = more embedding + storage costs), retrieval latency (smaller chunks = more API calls to the vector store), and the 3% accuracy difference impact on the use case; also consider the "lost in the middle" effect — smaller chunks mean more retrieved chunks in context  
C) Always use the lower latency strategy  
D) Chunk size doesn't affect latency  

**Answer: B**  
**Explanation:** High-traffic chunking tradeoff: (1) Storage cost: 200-token chunks mean ~2.5x more chunks to store and index. (2) Retrieval: same k (e.g., 5 chunks) retrieves less content with smaller chunks — may need higher k, increasing latency. (3) Context quality: more smaller chunks retrieved = more context consumed. (4) The 3% accuracy gain may be significant or trivial depending on use case. Benchmark at production scale.

---

**Q8.** A developer is told: "never use extended thinking because it's too slow." The developer's task is: optimizing a complex supply chain with 12 constraints. Extended thinking would improve accuracy from 76% to 91%. What is the correct response?

A) Follow the blanket rule — no extended thinking  
B) Challenge the blanket rule with data: extended thinking's latency cost may be justified for this specific task (supply chain optimization affects millions in logistics costs); propose a tiered approach: extended thinking for optimization runs (daily batch), standard for status queries (real-time); latency rules should be task-specific, not universal  
C) Use a pipeline instead  
D) Accept 76% accuracy  

**Answer: B**  
**Explanation:** Context-dependent tool use: blanket rules often don't hold for all tasks. A rule "never use extended thinking" may be appropriate for real-time chat (200ms SLA) but inappropriate for a daily batch optimization (latency irrelevant). Propose: apply the latency constraint where it matters (real-time queries), use extended thinking where accuracy matters and latency is tolerable (batch jobs). Architect by task requirements.

---

**Q9.** An agentic coding assistant can: read files, write files, execute code, call external APIs. A user accidentally asks it to "clean up old test files." The agent deletes production configuration files that don't match standard test file naming patterns. What SPIDER element failed?

A) S — Stop  
B) D — Determine: the agent failed to verify whether files were actually test files before deletion; also I — Isolate: deletion is an irreversible action that should require explicit confirmation; the agent over-interpreted "clean up" without validating scope  
C) R — Report  
D) P — Preserve state  

**Answer: B**  
**Explanation:** Multiple SPIDER failures: (1) D (Determine): the agent should have analyzed whether files were genuinely test files before acting on "old test files" — it needed to verify scope. (2) I (Isolate): file deletion is irreversible and high-risk; it should always require explicit user confirmation ("I found 47 files matching 'test*' patterns. Are these safe to delete?"). Neither was implemented.

---

**Q10.** A system achieves 99% accuracy in testing but 89% in production. The test dataset had 1,000 carefully curated examples. Production has 50,000+ diverse inputs. What does this gap indicate?

A) The model needs retraining  
B) Distribution shift: the test dataset doesn't represent production diversity; edge cases, unusual phrasings, rare topics, and real user behavior patterns are underrepresented in the curated test set; production testing with real inputs is essential; consider adversarial testing and synthetic data augmentation for testing  
C) The production system has bugs  
D) 89% is good enough  

**Answer: B**  
**Explanation:** Test-production gap = distribution mismatch. Curated test sets overfit to "expected" inputs. Real users produce: unusual phrasings, topic combinations, edge cases, adversarial inputs. Fixes: (1) Sample real production inputs for the test set (after handling). (2) Adversarial test generation. (3) Coverage analysis: what input types are in production but not in tests? (4) Continuous evaluation: sample production inputs weekly and evaluate.

---

**Q11.** A developer builds a system with comprehensive HITL oversight for all actions. The human reviewer is consistently approving actions within 2 seconds without reading them (rubberstamping). What does this indicate about the HITL design?

A) Human review is working correctly  
B) Alert fatigue / HITL design failure: when humans always approve without reading, the oversight is theater — it adds latency without safety value; redesign: (1) Reduce HITL to genuinely high-risk actions only. (2) Make the review UI highlight the specific risk requiring review. (3) Add meaningful review time requirements for truly critical actions.  
C) Reduce HITL to improve latency  
D) The model is accurate enough to not need review  

**Answer: B**  
**Explanation:** HITL effectiveness requires genuine human engagement. Rubberstamping indicates: (1) Too many items in the queue (reviewer fatigue). (2) Items are too low-risk to warrant review (crying wolf). (3) Review UI doesn't surface relevant risk information. Fix: scope HITL to genuinely high-risk actions, provide reviewers with actionable risk summaries, and measure review quality (not just approval rate). Fake oversight is worse than no oversight.

---

**Q12.** A developer must decide: add 10 new tools to an MCP server (bringing total to 25) vs. build a tool-routing agent that manages a larger tool set. At what point is routing preferable?

A) Always use tool routing for more than 10 tools  
B) Routing becomes valuable when: tool count exceeds 15-20 (selection accuracy degrades), tools cluster into domains (routing by domain is natural), or different tools are needed for different task types; for a general-purpose assistant with <15 tools, direct tool list is simpler and more reliable  
C) Routing is never worth the complexity  
D) Always use routing — more organized  

**Answer: B**  
**Explanation:** Tool routing threshold: direct tool list works well up to ~15 tools. Beyond that, selection accuracy degrades. Routing architecture: a meta-agent (or classifier) determines which domain/subset of tools to use, then presents only that subset to the main agent. Trade-offs of routing: adds a step, adds latency, adds a potential failure point. Worth it when the selection accuracy improvement justifies the complexity.

---

**Q13.** A developer is designing a system prompt that will be reused across 5 similar applications. They want maximum reuse but each application has slightly different Explicit Instructions. What pattern achieves this?

A) Maintain 5 completely separate system prompts  
B) Template prompt with parameterized sections: a base template with clear placeholder markers for application-specific instructions; at deployment time, fill in the application-specific sections; the base content (Persona, Role, Context, Style) is shared; only the application-specific Explicit Instructions differ  
C) Use the most restrictive prompt for all applications  
D) Use dynamic system prompt generation for each request  

**Answer: B**  
**Explanation:** Parameterized prompt templates: `{SHARED_PERSONA} {SHARED_ROLE} {APP_SPECIFIC_INSTRUCTIONS} {SHARED_STYLE}`. The shared sections maintain consistency; the parameterized section enables customization. This is software engineering applied to prompts: DRY principle (Don't Repeat Yourself), version-controlled base template, managed per-application overrides. Update the base → all applications benefit.

---

**Q14.** A system is designed with multiple Claude models (Haiku for classification, Sonnet for generation). The Haiku classifier incorrectly classifies edge cases 8% of the time. These errors compound: the Sonnet generator receives wrong task context and produces wrong output. End-to-end accuracy: 91%. Should the developer optimize the classifier or the generator?

A) Always optimize the weakest link  
B) Analyze the failure mode: if classifier errors only affect edge cases (rare in production), fixing the generator to be robust to mild misclassification may be more effective; if classifier errors are evenly distributed (8% of all inputs), fixing the classifier has broader impact; measure error distribution before deciding  
C) Always optimize the classifier  
D) Replace the classifier with a manual rule system  

**Answer: B**  
**Explanation:** System optimization requires failure analysis, not just fixing the weakest link. Ask: (1) What is the distribution of the 8% classifier errors? (Rare edge cases vs. common patterns.) (2) Do generator errors align perfectly with classifier errors? (Or does the generator sometimes self-correct minor misclassifications?) (3) Is it easier to fix the classifier or make the generator robust to misclassification? Data-driven optimization over intuition.

---

**Q15.** A developer must handle the case where Claude's context window is nearly full but the task is not complete. The pipeline has no recovery mechanism. What SPIDER element is this and what is the fix?

A) SPIDER-R (Report)  
B) SPIDER-S (Stop) + SPIDER-D (Determine): the system must detect approaching context limits, stop before quality degrades, determine what work has been completed, and SPIDER-P (Preserve): save the current state before stopping so the task can be resumed from where it stopped  
C) SPIDER-E (Escalate)  
D) SPIDER-I (Isolate)  

**Answer: B**  
**Explanation:** Context limit handling uses multiple SPIDER letters: (1) S (Stop): detect 80-90% context utilization → stop adding new work. (2) D (Determine): assess what is complete and what remains. (3) P (Preserve): save state (completed steps, remaining steps, intermediate results). (4) E (Escalate, optional): if the task cannot be segmented, escalate to human. State preservation before stopping is the key to resumable tasks.

---

**Q16.** An agent has been running for 2 hours on a complex task, making 200 tool calls. The context is full. The task is 70% complete. What should happen?

A) Start over  
B) SPIDER-P (Preserve state): serialize the current state (completed steps, extracted data, intermediate results, remaining tasks) to an external store; SPIDER-S (Stop) the current run; start a new conversation with the preserved state as context for the remaining 30%  
C) Request a larger context window  
D) Complete the remaining 30% at reduced quality  

**Answer: B**  
**Explanation:** Long-running task continuation: (1) State serialization: the agent's intermediate results are saved externally (structured JSON of progress). (2) Context pruning: the new conversation starts with a compact "task status report" rather than full conversation history. (3) Task resumption: the new conversation picks up from where the previous stopped. This is the correct pattern for multi-session agentic tasks.

---

**Q17.** A team is building a CLAUDE.md file for a large codebase. A junior developer suggests adding every coding convention, style guide, and best practice document to CLAUDE.md (estimated: 15,000 tokens). A senior developer argues for a concise 2,000-token CLAUDE.md with links to external documents. Who is right?

A) Junior developer — more information is better  
B) Senior developer — CLAUDE.md is loaded with every request; 15,000 token CLAUDE.md = 15,000 tokens consumed every single session; a concise CLAUDE.md with references (and tools to retrieve them when needed) is more efficient; CALM-L applies to CLAUDE.md too  
C) Both approaches are equivalent  
D) Junior developer — full context prevents misunderstandings  

**Answer: B**  
**Explanation:** CALM-L applies to all context, including CLAUDE.md. A 15,000-token CLAUDE.md: (1) Consumes 15,000 tokens per session. (2) Most of it is irrelevant to any given task. (3) Dilutes the signal of the most important instructions. A 2,000-token CLAUDE.md with tools or references: Claude reads additional documentation on-demand when needed. High-signal CLAUDE.md + retrieval is superior to bloated CLAUDE.md.

---

**Q18.** A customer asks for their personal data to be permanently deleted (GDPR right to erasure). The AI system has: conversation history in the context, RAG indexed data, fine-tuned model data, and system prompt examples. Which requires special handling beyond standard deletion?

A) Conversation history only  
B) All four require consideration: (1) Conversation history — delete from storage. (2) RAG index — delete chunks and re-index. (3) Fine-tuned model — this is the hardest: data used for fine-tuning cannot be "deleted" from model weights without retraining; must evaluate if fine-tuning used this user's data. (4) System prompt examples — check if any contain this user's data.  
C) Only RAG indexed data  
D) Standard data deletion handles all  

**Answer: B**  
**Explanation:** GDPR right to erasure with AI systems requires addressing all data stores. The most challenging: fine-tuned models that incorporated user data into weights — these cannot have individual data points removed without complete retraining. Architecture implication: design systems to avoid using user data for fine-tuning if right-to-erasure compliance is required, or use federated learning approaches. This is a complex compliance consideration for AI architects.

---

**Q19.** A developer wants to test the robustness of their multi-agent system. What types of edge case testing are most important?

A) Test only the happy path thoroughly  
B) Critical edge case tests: (1) Context window boundary conditions (at 80%, 95%, 100%). (2) Tool failure scenarios (timeout, malformed response, auth error). (3) Prompt injection attempts via tool outputs. (4) Partial task completion with state preservation. (5) Agent conflict resolution (two agents give contradictory results). (6) Cascading failures (agent A failure causes agent B to fail).  
C) Use random input testing only  
D) Test only the most common scenarios  

**Answer: B**  
**Explanation:** Robustness testing for agentic systems must stress the unique failure modes: (1) Context pressure. (2) Tool failures. (3) Security scenarios. (4) State management edge cases. (5) Inter-agent coordination failures. (6) Cascading failures. These are the failure modes that only emerge in agentic systems. Standard happy-path testing misses them entirely. The system's reliability is only as good as its tested edge case handling.

---

**Q20.** An application uses a 10,000-token system prompt with prompt caching. The team proposes adding 500 tokens of logging instructions that may change frequently (weekly updates). What impact does this have on caching?

A) The 500-token addition is negligible  
B) Placing frequently-changing content inside the cached section invalidates the cache on every update; the 10,000-token cache loses its savings on the weekly update weeks; fix: place stable content early (cached), changing content after the cache checkpoint  
C) Caching adjusts automatically  
D) Weekly cache invalidation has no cost impact  

**Answer: B**  
**Explanation:** Cache architecture: the 10,000-token system prompt should be placed before the cache checkpoint; the 500 tokens of frequently-changing logging instructions should be placed AFTER the checkpoint (or in a separate uncached section). This way: the 10,000 stable tokens benefit from caching consistently; the 500 changing tokens don't invalidate the 10,000-token cache. Stable content always before cache checkpoints.

---

**Q21.** A developer has a choice: implement retry logic with exponential backoff OR implement HITL escalation for tool failures. When is each appropriate?

A) Always retry — escalation is unnecessary  
B) Retry with backoff: transient errors (network timeout, rate limit, temporary service unavailability) — same input will likely succeed on retry. HITL escalation: non-transient errors (invalid credentials, business rule violation, data not found, logic error) — retrying won't help; a human or business decision is needed  
C) Always escalate — retries can cause side effects  
D) Never retry  

**Answer: B**  
**Explanation:** Error classification drives retry vs. escalation: (1) Retry-able: service unavailable (503), gateway timeout (504), rate limit (429) — temporary conditions that resolve. (2) Escalate: invalid auth (401) — needs credential refresh. Bad request (400) — logic error needs diagnosis. Not found (404) — data issue needs human verification. The `retryable` field in error responses should encode this distinction directly.

---

**Q22.** A team debates: use Claude for all steps in a pipeline vs. use deterministic code for simple steps (sorting, filtering, validation) and Claude for complex reasoning. What is the correct architecture principle?

A) Use Claude for everything  
B) Use the right tool for each step: deterministic code for: sorting, filtering, counting, validation against rules, data transformation — these are reliably handled by code and don't benefit from AI. Claude for: classification with ambiguity, summarization, generation, complex reasoning — tasks where AI adds genuine value that code cannot replicate  
C) Use Claude for everything but call it deterministic  
D) Avoid deterministic code in AI pipelines  

**Answer: B**  
**Explanation:** Principle: AI for uncertain reasoning, code for deterministic operations. A pipeline that uses Claude to sort a list by date is wasteful and unreliable. Use code: `sort(items, key=lambda x: x.date)`. Use Claude: "Classify this support ticket by urgency (1-5) considering these factors..." Hybrid pipelines are more reliable, cheaper, and faster than pure AI pipelines for tasks with code-suitable steps.

---

**Q23.** An MCP server is down, causing an agentic pipeline to fail. The orchestrator agent keeps retrying and accumulating error messages in context. After 15 retries, the context is 60% full with error messages. What should have happened?

A) More retries with longer backoff  
B) Circuit breaker pattern: after N consecutive failures (e.g., 3), the circuit breaker should trip and prevent further retries; escalate the failure to the orchestrator/human immediately; don't accumulate failure context indefinitely; SPIDER-S (Stop) + SPIDER-E (Escalate) + SPIDER-P (Preserve clean state)  
C) The agent should work around the failure  
D) Error messages should not be stored in context  

**Answer: B**  
**Explanation:** Circuit breaker + SPIDER combination: (1) After 3 failures: circuit breaker trips. (2) SPIDER-S: stop retrying immediately. (3) SPIDER-E: escalate with a clear failure report. (4) SPIDER-P: preserve the clean task state (not the accumulated error context). Storing 15 error messages in context is wasteful and confusing. The circuit breaker prevents context pollution and unnecessary API cost.

---

**Q24.** A developer tests a multi-agent system by running 100 test scenarios. 98% pass. The developer is confident the system is production-ready. What is missing from this evaluation?

A) Nothing — 98% is excellent  
B) Adversarial testing (prompt injection attempts), security boundary testing, edge case testing at context limits, tool failure simulation, and long-running session testing (not just single-turn); 98% on happy-path scenarios doesn't validate adversarial robustness or reliability under stress  
C) Run 1,000 test scenarios instead  
D) Test with different Claude models  

**Answer: B**  
**Explanation:** 98% on happy-path test cases misses: (1) Injection attacks. (2) Context overflow behavior. (3) Tool failure handling. (4) Long-running session degradation. (5) Concurrent user behavior. (6) Edge cases in authorization. Production readiness requires testing failure modes, not just success modes. The most critical production failures usually come from scenarios not in the test set.

---

**Q25.** An organization must choose between: (A) building custom Claude integration for each of 5 different departments, or (B) building a shared platform with department-specific configurations. What factors favor approach B?

A) Approach A is always better — more customization  
B) Approach B (shared platform) is favored when: (1) Security and compliance controls need to be consistent across departments. (2) IT wants centralized management, monitoring, and cost tracking. (3) Common capabilities (RAG infrastructure, tool library, audit logging) can be shared. (4) Prompt library versioning benefits all departments simultaneously. Trade-off: less customization flexibility per department.  
C) Approach B is always better — less duplication  
D) Depends only on budget  

**Answer: B**  
**Explanation:** Platform vs. custom evaluation: centralized platform advantages: consistent security, shared infrastructure cost, central monitoring, shared prompt library. Custom per-department: maximum flexibility, no shared dependencies. The key: if 70%+ of the capability is shared across departments, platform saves cost and maintenance. If departments have fundamentally different requirements (different security tiers, different data access, incompatible workflows), custom may be necessary.

---

## Score: /25 | Pass: 19/25 (75%)
