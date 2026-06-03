# Practice Questions 14 — Cross-Domain Mixed Scenarios

> Multi-domain: Questions spanning agentic architecture + prompt engineering + context management + tool design simultaneously.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A customer support pipeline: (1) Router agent classifies ticket, (2) Specialist agent handles, (3) Human reviews critical cases. The specialist agent's context fills up after 15 messages. Routing accuracy is 89%. Human reviewers approve 95% of cases. Which combined improvement addresses the most risk?

A) Improve routing accuracy to 99%  
B) Combine CALM-M (manage specialist context to prevent overflow) + SPIDER-E (escalate to human earlier when context nears limit, not just for critical cases); context overflow in the specialist creates wrong answers that humans then approve 95% of the time — the human review is a false safety gate if it doesn't catch context-degraded responses  
C) Replace human review with automated checks  
D) Increase context window  

**Answer: B**  
**Explanation:** Cross-domain: CALM-M (Domain 5) addresses context overflow. SPIDER-E (Domain 1) addresses escalation before context degrades. The interaction: context overflow degrades answer quality, but the HITL gate (95% approval) doesn't catch quality degradation because humans approve most things without checking depth. Fix both: manage context (so quality stays high) and escalate proactively when approaching context limits.

---

**Q2.** A medical record summarization agent uses: parallel pattern (summarize multiple sections simultaneously), GPT-style few-shot examples in the prompt, and no access control on the MCP server. Which risk is most severe?

A) Parallel pattern inefficiency  
B) No access control on the MCP server is a critical security vulnerability: anyone with network access can retrieve patient medical records; HIPAA violation; the few-shot examples and parallel pattern are operational concerns but the security failure is the existential risk  
C) Few-shot examples are insufficient  
D) Parallel pattern is incorrect for medical data  

**Answer: B**  
**Explanation:** Severity ranking: (1) Security/compliance failure (no access control = HIPAA violation, potential legal liability, regulatory action) — existential threat. (2) Quality issues (few-shot examples) — operational concern. (3) Architecture inefficiency (parallel pattern) — engineering concern. When multiple issues exist, address the most severe first. Operational improvements are worthless if the system is shut down for compliance violations.

---

**Q3.** A developer designs: (1) router agent with PRECISE-compliant prompt, (2) tool schema with excellent descriptions, (3) RAG with 1,000-token chunks for context retrieval. The system fails on ambiguous queries that require cross-domain reasoning. What is the gap?

A) Tool descriptions need improvement  
B) The pipeline pattern (router → specialist) may not be optimal for cross-domain queries that require synthesis across knowledge areas; a parallel pattern (multiple specialists simultaneously) + a synthesis agent may be needed; also CALM-L: large chunks may be including irrelevant cross-domain content  
C) The PRECISE prompt needs more explicit instructions  
D) Increase chunk count  

**Answer: B**  
**Explanation:** Cross-domain: Agentic Architecture (Domain 1) + Context Management (Domain 5). Router/pipeline handles in-domain queries well. Cross-domain queries need parallel processing (Domain 1 parallel pattern: multiple specialists run simultaneously) + synthesis (agent that combines results). Additionally, CALM-L with 1,000-token chunks means irrelevant domain content may be included — reduce chunk size or improve filtering for cross-domain retrieval.

---

**Q4.** A CLAUDE.md file specifies: "Always use TypeScript." An MCP server's tool schema has no TypeScript-specific type examples. A developer asks Claude to generate code. Claude generates JavaScript. Which configuration level is failing?

A) CLAUDE.md is wrong  
B) CLAUDE.md instruction applies to code generation behavior; the MCP tool schema (Domain 4) is the interface for tools, not code generation; if Claude generates JavaScript despite CLAUDE.md, the issue is that CLAUDE.md may not be properly scoped or the tool's output format overrides the CLAUDE.md instruction in the current interaction context  
C) The MCP server should enforce TypeScript  
D) Use the system prompt instead of CLAUDE.md  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code Configuration (Domain 2) + Tool Design (Domain 4). CLAUDE.md's scope is the working session behavior. If a tool call triggers code generation and returns a code template, the tool's output format may override the CLAUDE.md preference. Fix: ensure CLAUDE.md is in project scope (not just user scope), or add TypeScript format specification to the tool's schema description for code-generating tools.

---

**Q5.** An agentic system collects data via 8 parallel tool calls, then synthesizes the results. Each tool result is 3,000 tokens. After synthesis, context is 26,000 tokens of tool results + 2,000 tokens of synthesis instructions. Total: 28,000 tokens before any output. The response max_tokens is 15,000. Context limit: 32,000. What will happen?

A) The system works perfectly  
B) Context overflow risk: 28,000 + 15,000 (output) = 43,000 tokens, which exceeds the 32,000-token context limit; the synthesis cannot produce its full 15,000-token output; fix: CALM-L on tool results — have each tool return 500-1,000 tokens instead of 3,000, keeping total context under 32,000 including output  
C) The synthesis agent can work within the constraints  
D) Increase max_tokens  

**Answer: B**  
**Explanation:** Cross-domain: Agentic Architecture (Domain 1 parallel pattern) + Context Management (Domain 5 CALM-L). Parallel pattern collects 8 × 3,000 = 24,000 tokens of results. Synthesis needs output space. 24,000 + 2,000 + 15,000 = 41,000 > 32,000 context limit. Fix: CALM-L — each tool returns a compressed summary (500 tokens) rather than full results (3,000 tokens). 8 × 500 = 4,000 + 2,000 + 15,000 = 21,000 — within budget.

---

**Q6.** A system prompt for a coding assistant includes the PRECISE framework elements but the "Expected Output" section specifies: "Return a JSON object with keys: code, explanation, tests." A developer also adds few-shot examples but the examples use markdown code blocks instead of JSON. What will Claude do?

A) Always follow the few-shot examples (higher priority)  
B) Behavior will be inconsistent — the PRECISE-E instruction says JSON, the few-shot examples show markdown; Claude may switch between formats; the contradiction between explicit instruction and examples must be resolved by either updating examples to use JSON or changing the instruction  
C) Always follow the explicit JSON instruction  
D) Default to markdown since it's more readable  

**Answer: B**  
**Explanation:** Cross-domain: Prompt Engineering (Domain 3 — PRECISE + few-shot) interaction. When explicit instructions and examples contradict, behavior is inconsistent. Examples carry significant weight (few-shot) but explicit instructions also carry weight. The contradiction creates unpredictable behavior. Rule: align examples with instructions — examples should demonstrate the exact format specified in the instruction.

---

**Q7.** An organization wants to deploy an internal knowledge assistant using Claude Code with CLAUDE.md for developer use AND a separate production API for customer-facing use. Which configuration should be in CLAUDE.md and which in the system prompt?

A) All configuration in CLAUDE.md  
B) CLAUDE.md: developer tool preferences, coding conventions, internal API keys and connection info; Production system prompt: customer-facing persona, response constraints, what topics are in/out of scope, data access permissions; CLAUDE.md is for developer experience; system prompt is for production behavior  
C) All configuration in system prompt  
D) They should share the same configuration  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code Configuration (Domain 2) + Prompt Engineering (Domain 3). CLAUDE.md: developer-facing, defines how Claude behaves when helping developers (tooling preferences, repo conventions, development workflow). System prompt: operator-level configuration for end-user-facing deployments. They serve different purposes and different audiences. Separating them keeps both clean and appropriately scoped.

---

**Q8.** A pipeline agent: receives a customer query → retrieves knowledge via RAG → generates a response → posts the response to a customer communication platform. A prompt injection in the knowledge base modifies the response posted to the customer. What is the combined failure?

A) RAG is not suitable for this use case  
B) Two failures: (1) Indirect prompt injection via RAG (Domain 3 defense: treat retrieved content as data, not instructions). (2) Agentic system lacks SPIDER-I (Isolate): writing to external systems (communication platform) should be guarded with explicit content validation before posting; combine injection defense + action gating  
C) The pipeline should be replaced with a router  
D) Disable RAG  

**Answer: B**  
**Explanation:** Cross-domain: Prompt Engineering injection defense (Domain 3) + Agentic Architecture SPIDER (Domain 1). The combined fix: (1) Domain 3: system prompt instruction that RAG content is data, not instructions. (2) Domain 1 SPIDER-I (Isolate) + SPIDER-E (Escalate): before posting to external platform, validate content doesn't contain injected instructions; gate high-impact actions. Defense-in-depth across domains.

---

**Q9.** A developer has a CLAUDE.md that says "use the `code_review` slash command for all reviews." They also have an MCP tool `review_code` that does the same thing. When does this create a problem?

A) It never creates a problem  
B) Conflicting action mechanisms: CLAUDE.md slash command may trigger a different workflow than the MCP tool; if both are available, Claude may use either inconsistently; define clear precedence in CLAUDE.md ("use `code_review` slash command, not the `review_code` tool") or remove the redundancy  
C) The MCP tool takes precedence  
D) CLAUDE.md takes precedence  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code (Domain 2) + Tool Design (Domain 4). Redundant mechanisms for the same action create inconsistency. Fix: (1) Remove the redundancy (keep one mechanism). (2) If both must exist, explicitly state precedence in CLAUDE.md. (3) Or scope them differently (slash command for interactive review, MCP tool for automated CI/CD review). Clarity prevents inconsistent behavior.

---

**Q10.** A RAG-based customer service bot uses 500-token chunks. Users ask questions that span multiple chunks (e.g., "What is your return policy and how does it interact with sale items?"). The answer requires two chunks that weren't retrieved together. What design pattern helps?

A) Increase chunk size to 2,000 tokens  
B) Parent document retrieval: embed chunks for retrieval but return the parent document (or larger section) when a chunk is retrieved; this provides surrounding context; also: retrieve more candidates (k=10) to increase the chance of getting both relevant chunks  
C) Reduce chunk overlap  
D) Add more policy documents  

**Answer: B**  
**Explanation:** Parent document retrieval: chunks are used for retrieval (small = precise matching) but the full parent section is returned (large = complete context). Retrieved chunk: "Returns allowed within 30 days" → return full parent section including "Sale items: final sale, no returns." This gives Claude the complete policy including the interaction the user asked about. Separates retrieval granularity from context granularity.

---

**Q11.** A multi-agent system: Orchestrator → [Research Agent, Calculator Agent, Writer Agent]. The Writer Agent's system prompt uses excellent PRECISE framework. Research Agent returns 8,000-token results. The Writer Agent produces hallucinated citations. What is the cross-domain diagnosis?

A) PRECISE is insufficient  
B) Research Agent's results overwhelm the Writer Agent's context (Domain 5 CALM-L), causing the "needle in haystack" problem where citation information is lost in the bulk; Writer Agent's PRECISE (Domain 3) should include a grounding instruction: "all citations must be verbatim from the research results"  
C) Use a different writer model  
D) Reduce the research agent's scope  

**Answer: B**  
**Explanation:** Cross-domain: Context Management (Domain 5) + Prompt Engineering (Domain 3). Large research results dilute citation accuracy. Dual fix: (1) CALM-L: Research Agent returns structured summaries with citations pre-extracted rather than 8,000-token full results. (2) PRECISE-E: Writer Agent's explicit instructions include "cite only from [research_results] verbatim." Both domains contribute to the hallucination; both need fixes.

---

**Q12.** A developer deploys an AI tool using stdio transport, a detailed CLAUDE.md, and a RAG knowledge base. A colleague on another machine wants to use the same setup. What changes are required?

A) Share the CLAUDE.md and that's enough  
B) Switch stdio → SSE transport (for multi-user network access), configure authentication for the SSE server, share CLAUDE.md (project scope applies per user), and either share the vector store connection or replicate the knowledge base index  
C) Each developer needs separate knowledge bases  
D) CLAUDE.md cannot be shared  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code Config (Domain 2) + MCP Transport (Domain 4). stdio = single user, local only. Adding a second user requires SSE for network access + authentication. CLAUDE.md at project scope is shared via version control. Knowledge base: centralize the vector store with authentication (so both developers query the same indexed knowledge). Each piece of the stack has specific multi-user considerations.

---

**Q13.** A parallel agent system processes financial data from 10 sources simultaneously. Results are synthesized by a central agent. The synthesizer uses CoT reasoning before producing a final recommendation. Users report the CoT reasoning is visible in the final output. How is this fixed?

A) Disable CoT  
B) Two approaches: (1) Use extended thinking API (thinking block is separate from response). (2) Use XML tags: `<thinking>` for internal reasoning, and post-process to strip thinking tags from the user-visible response. This is a Prompt Engineering (Domain 3) fix for a multi-agent (Domain 1) system.  
C) Move reasoning to the parallel agents  
D) Use a different output format  

**Answer: B**  
**Explanation:** Cross-domain: Agentic Architecture (Domain 1 parallel) + Prompt Engineering (Domain 3 CoT). The CoT reasoning is valuable for accuracy but not user-visible. Fix: extended thinking (thinking block in API response, not in user-facing content) OR XML tag separation with post-processing. The parallel architecture doesn't change — only the output handling of the synthesis agent changes.

---

**Q14.** A company evaluates: deploy Claude Code with CLAUDE.md for developer-facing use OR build a custom API integration with system prompts for internal tooling. When is the CLAUDE.md approach clearly correct?

A) Always — CLAUDE.md is simpler  
B) CLAUDE.md/Claude Code is correct when: the users are developers working in code editors, the primary task is software development, and the team wants quick iteration on coding assistance without API infrastructure; API + system prompt is correct when: building end-user-facing tools, requiring programmatic control, needing custom UIs  
C) API integration is always better  
D) Depends only on cost  

**Answer: B**  
**Explanation:** Cross-domain: Domain 2 (Claude Code) vs. Domain 3 (API system prompts). Decision criteria: (1) User type: developers in editors → CLAUDE.md. End users in custom apps → API + system prompt. (2) Task type: coding assistance → Claude Code. Custom application logic → API. (3) Control requirements: full programmatic control → API. Convention-based configuration → CLAUDE.md. Know when each approach is designed to be used.

---

**Q15.** A developer creates a slash command `/security-review` in CLAUDE.md. The command runs a security checklist. They also want to add the same checklist to an MCP tool for CI/CD automation. What is the correct architecture?

A) Duplicate the checklist in both places  
B) Define the security checklist in one MCP Prompt (reusable prompt template); the CLAUDE.md slash command calls this prompt; the CI/CD automation also calls this prompt; single source of truth for the checklist, multiple invocation methods  
C) Use CLAUDE.md only  
D) Use MCP tools only  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code Config (Domain 2) + MCP Architecture (Domain 4 Prompts capability). MCP Prompts provide reusable task templates. Define the security checklist once as an MCP Prompt. The CLAUDE.md `/security-review` command invokes this MCP Prompt. CI/CD also invokes the same MCP Prompt. Single definition, multiple callers. Changes to the checklist update everywhere simultaneously.

---

**Q16.** A chatbot's RAG retrieves 5 chunks. The system prompt is 2,000 tokens. RAG adds 5,000 tokens. Conversation history is 8,000 tokens. The user query is 200 tokens. Total: 15,200 input tokens. The model is Haiku with 200k context. Why might performance still degrade?

A) 15,200 tokens is near the limit  
B) Performance does not simply degrade at context limits; more likely culprits: the conversation history (8,000 tokens) may contain irrelevant early turns diluting the recent context; the 5 RAG chunks may not all be relevant (low signal density); both are CALM issues within a comfortable context window  
C) Haiku cannot handle this context size  
D) The system prompt is too long  

**Answer: B**  
**Explanation:** Context size ≠ the only performance factor. 15,200 tokens in a 200k context is very comfortable. But: (1) Irrelevant history (CALM-M: manage history quality, not just quantity). (2) Low-relevance RAG chunks (CALM-L: limit to high-relevance content). Performance issues within comfortable context windows are usually signal-to-noise problems, not size problems.

---

**Q17.** A multi-agent content moderation system: (1) Image classifier, (2) Text analyzer, (3) Context agent that considers previous violations by the same user. The context agent reads the user's violation history (potentially large). What context management strategy is correct for the context agent?

A) Include full violation history  
B) CALM-C: chunk the violation history by time period (last 7 days full, last 30 days summarized, older summarized at higher compression); CALM-M: extract persistent facts (total violations, categories, severity trend); this provides the context agent with relevant history without excessive context consumption  
C) Only use the most recent violation  
D) Exclude historical context  

**Answer: B**  
**Explanation:** Cross-domain: Agentic Architecture (Domain 1) + Context Management (Domain 5). The context agent needs history but not all of it equally. Tiered history: (1) Recent violations (full detail — most relevant). (2) Medium-term summary (30-day patterns). (3) Long-term facts (total count, repeat offender status). This provides decision-relevant context without including years of verbose violation records.

---

**Q18.** A developer builds a PRECISE-compliant system prompt for a data analysis tool. The Expected Output specifies JSON with specific fields. An MCP tool in the pipeline returns data in a slightly different JSON schema. Claude sometimes follows the system prompt schema, sometimes follows the tool's schema. What is the fix?

A) Remove the system prompt schema  
B) Normalize in the MCP server: the MCP server (adapter pattern) should transform the tool's output JSON to match the system prompt's expected schema before returning to Claude; Claude should receive only the schema it expects  
C) Modify the system prompt to match the tool  
D) Use XML instead of JSON  

**Answer: B**  
**Explanation:** Cross-domain: Prompt Engineering (Domain 3 PRECISE-E) + Tool Design (Domain 4 adapter pattern). Schema conflicts cause inconsistent behavior. Fix at the MCP layer: transform tool output to match the system prompt's expected schema. The MCP server is the normalization layer. Don't fight between system prompt and tool output schema — normalize at the boundary so Claude always receives consistent structure.

---

**Q19.** A developer's Claude Code setup has CLAUDE.md with project-level instructions. A new developer joins and has conflicting instructions in their user-level `~/.claude/CLAUDE.md`. When they work on the project, project instructions are sometimes overridden. What is the correct understanding?

A) User-level always wins  
B) Claude Code scope hierarchy: user-level CLAUDE.md (applies globally to all projects for this user) vs. project-level CLAUDE.md (applies to specific project); user-level is loaded first but project-level can add/override for the specific project; conflicts should be resolved by making project-level instructions explicit and complete  
C) Project-level always wins  
D) The most recently loaded file wins  

**Answer: B**  
**Explanation:** Domain 2: scope hierarchy. User-level CLAUDE.md is the user's personal defaults. Project-level CLAUDE.md is project-specific rules. Both are loaded; project-level additions/overrides apply for the project. If a user's global settings conflict with project requirements, the project team should document the expected project behavior explicitly in project CLAUDE.md and onboard new developers to align their user settings.

---

**Q20.** An API has 3 Claude deployments: customer chat (haiku), technical support (sonnet), architectural review (opus). Each has different max_tokens, context sizes, and tool sets. A developer wants to unify the PRECISE system prompts. What should be shared and what should differ?

A) All elements should be identical  
B) Shared across all: PRECISE-P core persona (consistent brand voice), PRECISE-R base role. Differentiated: PRECISE-E explicit instructions (different rules per use case), PRECISE-E expected output (different formats/depths), PRECISE-I input format (different input types per use case), and max_tokens/tool sets per deployment  
C) Nothing should be shared  
D) Only the persona should differ  

**Answer: B**  
**Explanation:** Cross-domain: Prompt Engineering (Domain 3 PRECISE) + multiple deployment configurations. Brand consistency: shared Persona and base Role. Task specificity: different Explicit Instructions, Expected Output format, and Input formats per deployment. A shared base template with deployment-specific overrides is the practical architecture. This ensures brand consistency while allowing task-appropriate customization.

---

**Q21.** A developer measures: agentic pipeline produces correct results 90% of the time. The 10% failure cases all occur when the context window is above 80% utilization. What is this telling the developer?

A) Replace the pipeline with a simpler architecture  
B) Context saturation is causing failures: at high utilization, Claude's attention to all context portions degrades; CALM-L and CALM-M must be applied to reduce peak context utilization; SPIDER-S (Stop) should trigger when context reaches 80% with an escalation before quality degrades  
C) Increase the context window size  
D) Reduce the number of pipeline steps  

**Answer: B**  
**Explanation:** Cross-domain: Context Management (Domain 5) + Agentic Architecture SPIDER (Domain 1). Data says failures correlate with context saturation. Dual fix: (1) Domain 5: apply CALM to reduce peak context utilization below the failure threshold. (2) Domain 1 SPIDER-S (Stop): add a circuit-breaker that detects high context utilization and escalates to human or falls back to a simpler response before quality degrades.

---

**Q22.** An MCP tool `analyze_contract` returns a 15,000-token full contract analysis. The tool is used in an agentic pipeline where subsequent agents also need context. What Domain 4 + Domain 5 combined fix is most effective?

A) Increase context window  
B) Domain 4: redesign the tool to return a structured summary (key_clauses, risks, recommendations) at 500-1,000 tokens with an option to retrieve full details for specific clauses on-demand; Domain 5 CALM-L: the pipeline uses the summary in context and fetches full text only when a specific clause requires detailed analysis  
C) Remove the tool from the pipeline  
D) Use a separate pipeline for contract analysis  

**Answer: B**  
**Explanation:** Cross-domain fix: Tool Design (Domain 4) + Context Management (Domain 5). Tool redesign: return a compact structured summary by default; support a `clause_id` parameter to retrieve full text for specific clauses when needed. This combines with CALM-L: default context consumption is 500 tokens (summary); deep dives add targeted clause text. The pipeline gets high-level context with on-demand detail.

---

**Q23.** An agent uses a pipeline pattern: classify → retrieve → generate → validate → send. The validate step uses a PRECISE-compliant prompt with explicit quality criteria. The validator consistently marks 40% of outputs as failing. Investigation finds the generator prompt is poorly calibrated. Which fix is correct?

A) Fix the validator's criteria  
B) Fix the generator prompt (Domain 3) to produce outputs that meet the validator's criteria; the validator is the quality gate — 40% failures indicate the generator is producing outputs that don't meet spec; don't lower the bar, improve the generation; analyze what the 40% have in common and add explicit instructions to the generator  
C) Remove the validation step  
D) Use parallel generation and pick the best  

**Answer: B**  
**Explanation:** Cross-domain: Agentic Pipeline (Domain 1) + Prompt Engineering (Domain 3). 40% validation failures = generator calibration problem. The validator is working correctly (that's its job). Fix the generator: analyze failed cases (what do they have in common?), add explicit instructions to the generator's PRECISE-E to prevent the failing patterns, verify fix by measuring validation pass rate. Don't fix the validator to pass bad outputs.

---

**Q24.** A developer is setting up CLAUDE.md for a team's shared repository. The team uses an MCP server with 15 tools. Not all tools are relevant for all tasks. What CLAUDE.md configuration optimizes tool usage?

A) List all 15 tools in CLAUDE.md  
B) In CLAUDE.md, define task-specific tool groups: "For database tasks, use: [db_query, db_update, db_schema]. For API integration, use: [api_get, api_post, api_auth]. For file operations, use: [read_file, write_file, create_dir]." This helps Claude select the right tools for each context without overwhelming it with all 15.  
C) Alphabetically sort all tools  
D) Remove unused tools from the MCP server  

**Answer: B**  
**Explanation:** Cross-domain: Claude Code Config (Domain 2) + Tool Design (Domain 4). CLAUDE.md can include tool guidance that reduces selection overhead. Task-to-tool mappings in CLAUDE.md help Claude pick the right tools without needing to evaluate all 15 for every task. This is effective even though tool selection is ultimately based on tool descriptions — CLAUDE.md context reinforces the appropriate tool set for each workflow.

---

**Q25.** A production system that was designed correctly across all 5 domains (agentic, Claude Code, prompt engineering, tool design, context management) experiences unexpected failures after a model update. What is the correct response?

A) Roll back the model update  
B) Systematic regression testing by domain: (1) Test PRECISE prompt behavior (Domain 3) — did response format/behavior change? (2) Test tool selection (Domain 4) — same inputs still select same tools? (3) Test context utilization (Domain 5) — same context usage patterns? (4) Test SPIDER triggers (Domain 1) — same escalation behavior? Identify which domain regressed and fix specifically.  
C) Apply all configurations as-is  
D) Assume the failure is in the most recently changed domain  

**Answer: B**  
**Explanation:** Cross-domain: Model updates can affect any domain independently. Systematic regression testing isolates which domain(s) regressed. Common model update effects: (1) Tool selection changes (different ranking of similar tools). (2) Output format drift (new model may not follow PRECISE-E exactly the same way). (3) Context utilization patterns shift. Each domain requires separate regression validation. Fix the regressed domain specifically rather than wholesale reconfiguration.

---

## Score: /25 | Pass: 19/25 (75%)
