# CCA Foundations — Full Mock Exam 1

> Simulate real exam conditions: **120 minutes**, no notes (except after you're done).  
> Score: 45/60 (75%) to pass.

**Start timer before reading Q1.**

---

## Instructions
- 60 questions, 120 minutes
- All questions are scenario-based multiple choice
- One best answer per question
- Scoring is weighted by domain
- Mark questions you're unsure about and return to them

---

## SECTION 1 — Agentic Architecture (Questions 1–16)

---

**1.** A logistics company is building an agent that schedules truck deliveries. The agent needs to: (1) check driver availability, (2) check truck availability, (3) check route feasibility, then (4) create the booking. Which orchestration pattern is optimal?

A) Router — route to the appropriate specialist  
B) Parallel for steps 1-3, then Pipeline for step 4  
C) Pipeline for all 4 steps in sequence  
D) A single agent with all tools accessible simultaneously  

---

**2.** An agent repeatedly calling a payment API encounters a "402 Payment Required — insufficient balance" error. According to SPIDER-D, what should it do?

A) Retry with exponential backoff  
B) Retry immediately once, then escalate  
C) Stop retrying — this is not a transient error; escalate to a human  
D) Retry three times with delays  

---

**3.** A content moderation agent reviews posts and can take three actions: approve, flag for review, or delete. Posts are being deleted incorrectly at a rate of 0.5%. Management wants to reduce incorrect deletions. What is the best architectural change?

A) Increase the agent's confidence threshold before taking any action  
B) Require human confirmation for all delete actions (inline HITL for delete only)  
C) Add a second agent to double-check the first  
D) Switch to a more powerful model  

---

**4.** An orchestrated system consists of: Orchestrator → Agent A → Agent B → Agent C, where each agent passes results to the next. Agent B is connected to an external third-party data source. Agent C takes action based on Agent B's output. What validation should occur before Agent C executes?

A) None — Agent B is a trusted internal agent  
B) Validate Agent B's output schema before passing to Agent C; treat the external data as untrusted  
C) Validate Agent C's inputs only, since it's taking the action  
D) Full human review of all Agent B outputs  

---

**5.** A developer argues: "We should give our agent all available tools so it can handle any situation." An architect disagrees. Which architectural argument is MOST compelling against this approach?

A) More tools increase token usage  
B) Claude may time out processing the tool list  
C) The tool selection accuracy drops as the number of tools grows; unnecessary tools also expand the blast radius of mistakes  
D) The API has a maximum tool count per request  

---

**6.** An HR automation agent processes employee leave requests. It can approve or reject requests up to 5 days. A request comes in for 3 weeks of medical leave. What should happen?

A) Auto-approve since it's medical leave  
B) Auto-reject since it exceeds 5 days  
C) Pause, compile full context, and escalate to an HR manager  
D) Approve the first 5 days and ask the employee to re-submit for the remainder  

---

**7.** A pipeline agent: (1) retrieves data, (2) analyzes it, (3) generates a report, (4) emails the report. Step 4 runs, but the email library crashes. The agent retries from step 1. What SPIDER improvement is needed?

A) Implement SPIDER-S: stop the pipeline entirely on any failure  
B) Implement SPIDER-P: checkpoint after step 3 so retry resumes from step 4 only  
C) Remove the retry mechanism entirely  
D) Implement SPIDER-E: escalate to human after any failure  

---

**8.** Two agents exchange messages about a document editing task. Agent 1 says the document has 10 pages. Agent 2 (which has fresh access to the document) says it has 12 pages. The orchestrator must proceed. What is correct?

A) Trust Agent 1 since it spoke first  
B) Trust Agent 2 since it has direct access  
C) Flag the discrepancy, use Agent 2's data (fresher source), and log the conflict for review  
D) Use the average (11 pages)  

---

**9.** A company wants an agent to manage social media posts — drafting, scheduling, and publishing. The security team is concerned. Which capability should absolutely require human-in-the-loop approval?

A) Drafting posts  
B) Scheduling posts for future dates  
C) Publishing posts (immediate public action)  
D) Viewing analytics  

---

**10.** When designing an agentic system, at what point in the process should you decide whether to use agents vs. simpler API calls?

A) After building the system and measuring performance  
B) During the initial architecture phase — agents add complexity and should only be chosen when the task genuinely requires autonomous decision-making or tool use  
C) Always choose agents — they are more capable  
D) Choose based on the number of API calls needed  

---

**11.** An agent logs the following: "Tool: send_email | To: user@example.com | Subject: Order confirmed | Status: Success". An hour later, the user reports they never received the email. The log is complete. Which SPIDER element could be improved?

A) S — Stop on failure  
B) R — Report outcomes (the log should include message ID and delivery confirmation, not just send status)  
C) P — Preserve state  
D) I — Isolate side effects  

---

**12.** A Router agent receives: "What is the weather today and what should I wear?" Which routing decision is correct?

A) Route to weather agent only  
B) Route to fashion/recommendation agent only  
C) Route to both (weather agent + recommendation agent) in parallel, synthesize  
D) Return an error — multi-intent queries are not supported  

---

**13.** An agentic system uses a tool that queries a CRM. The tool is callable by both the internal support agent AND an external partner agent that integrates via API. What access design is most secure?

A) Same tool access for both — they should have parity  
B) Internal agent: full CRM tool access; external partner: separate tool with limited field access and rate limits  
C) External partners should not have any CRM access  
D) Use the same tool but log external calls more verbosely  

---

**14.** A parallel orchestration system dispatches 5 sub-agents. 4 complete successfully. 1 fails with a timeout. What should the orchestrator do with the partial results?

A) Discard all results and retry from scratch  
B) Wait indefinitely for the 5th agent to finish  
C) Use the 4 available results, note the missing source, and return a partial-but-labeled result to the user  
D) Request human approval before proceeding  

---

**15.** A developer is debugging an agentic system and discovers the agent called a "delete" tool when it should have called an "archive" tool. The tools have similar descriptions. What is the root architectural issue?

A) The model isn't powerful enough to distinguish them  
B) The delete and archive tools have insufficiently differentiated descriptions; the delete tool should also include a warning about irreversibility  
C) The tools should be merged into one  
D) Tool selection should always require human confirmation  

---

**16.** An agent needs to book flights for employees. The workflow: search → select → confirm booking → charge company card. Where MUST human-in-the-loop approval be inserted?

A) Before every step for safety  
B) Only before charging the company card (irreversible financial action)  
C) No HITL needed — agents should be fully autonomous  
D) Before search and before booking confirmation  

---

## SECTION 2 — Claude Code Configuration (Questions 17–28)

---

**17.** A CLAUDE.md file contains: "Execute database migrations when schema changes are detected." A developer accidentally triggers a migration against a production database. Which is the MOST appropriate immediate fix?

A) Add a Forbidden section: "Never run database migrations without explicit user confirmation and a specified target environment"  
B) Remove all database-related instructions from CLAUDE.md  
C) Add the rule to user-level CLAUDE.md instead  
D) Encrypt the migration commands  

---

**18.** A project CLAUDE.md includes: "You may use npm install to add any packages that are useful." A developer discovers Claude Code installed 12 unreviewed packages. What is the architectural problem?

A) npm install should never be in CLAUDE.md  
B) The permission is too broad — it should specify which packages or require confirmation for new package installations  
C) This is expected Claude Code behavior  
D) Package installation should be in user-level config  

---

**19.** A team is creating a CLAUDE.md for a React frontend project. Which content is MOST valuable to include?

A) The name and role of each team member  
B) The project's TypeScript config, component naming conventions, state management pattern, and test runner commands  
C) Current open bugs and their status  
D) Git history of recent changes  

---

**20.** A developer's user-level CLAUDE.md says: "Always use tabs for indentation." The project CLAUDE.md says: "Use 2-space indentation (enforced by Prettier)." What happens when Claude Code writes code in this project?

A) It uses tabs because user-level config always wins  
B) It uses 2-space indentation because project-level config overrides user-level for project work  
C) It alternates between tabs and spaces unpredictably  
D) It asks the developer to resolve the conflict  

---

**21.** A slash command `/deploy` is defined in the project CLAUDE.md with no output format or safety constraints. A developer types `/deploy` and Claude Code deploys to production without confirmation. What should be added to the command definition?

A) Nothing — if the command is in CLAUDE.md, it should execute fully  
B) An explicit confirmation step in the command definition: "Before executing, display the deployment target and ask for explicit confirmation"  
C) Move the command to user-level CLAUDE.md  
D) Require MCP authentication  

---

**22.** An MCP server is configured in the project's `mcp_config.json`. Which statement is TRUE?

A) MCP server config goes inside CLAUDE.md, not a separate file  
B) MCP server config is in a separate file (`mcp_config.json`); CLAUDE.md references the available tools by name and describes when to use them  
C) MCP servers must be configured at the user level, not project level  
D) All MCP servers must use SSE transport  

---

**23.** A new team member sets up Claude Code on their laptop. The project has a CLAUDE.md with team conventions. However, Claude Code behaves differently for this developer than for others. What is the MOST likely cause?

A) Claude Code has different capabilities on different operating systems  
B) The developer has conflicting user-level CLAUDE.md settings that override some project conventions  
C) The project CLAUDE.md is not committed to version control  
D) MCP servers are not installed on the developer's machine  

---

**24.** Which statement about the scope hierarchy of CLAUDE.md files is CORRECT?

A) User-level always overrides project-level for consistency  
B) Project-level overrides user-level; subdirectory-level overrides project-level  
C) All CLAUDE.md files have equal authority; last-read wins  
D) Subdirectory-level can only restrict permissions, not expand them  

---

**25.** A `CLAUDE.md` file includes the instruction: "Check for test coverage before submitting changes." Claude Code runs tests but doesn't enforce a coverage threshold. What is missing?

A) The instruction is clear enough — Claude will enforce coverage  
B) A specific threshold: "Ensure test coverage is above 80% for modified files before completing any task"  
C) A slash command for running tests  
D) A Forbidden section preventing low-coverage code  

---

**26.** A sensitive microservice has its own directory in a monorepo. The team wants Claude Code to treat files in this directory as read-only and never suggest changes without explicit instruction. Where should this be configured?

A) In the root project CLAUDE.md with a path exception  
B) In the user-level CLAUDE.md  
C) In a CLAUDE.md file inside the sensitive microservice directory itself  
D) As a Git pre-commit hook  

---

**27.** An organization uses a shared MCP server for Jira integration. Different projects need to connect to different Jira boards. How should this be configured?

A) One user-level MCP config for the Jira server; each project specifies board IDs in its CLAUDE.md  
B) A separate MCP server instance for each project  
C) Jira MCP is not suitable for project-scoped configuration  
D) All projects share one hardcoded Jira board  

---

**28.** A developer adds to CLAUDE.md: "The stack is Next.js 14, TypeScript, Tailwind CSS, Prisma ORM with PostgreSQL. Database schema is in prisma/schema.prisma. Run dev server with `npm run dev`. Tests run with `npm test`." What PRECISE element does this most closely correspond to?

A) P — Persona  
B) C — Context  
C) E — Explicit instructions  
D) I — Input format  

---

## SECTION 3 — Prompt Engineering (Questions 29–40)

---

**29.** A developer tests the same prompt twice with temperature 0 and gets slightly different outputs. What is the most likely explanation?

A) Temperature 0 guarantees identical outputs; this is a bug  
B) Very long prompts can cause slight non-determinism even at temperature 0 due to floating-point operations  
C) Temperature 0 is not actually deterministic by design  
D) The context window was exceeded  

---

**30.** A system prompt defines a persona but the persona breaks down when users directly ask "Are you ChatGPT?" or "What AI model are you?" What addition to the prompt fixes this?

A) Explicit instructions covering identity questions: "If asked which AI you are or who made you, say 'I'm [name], your [description]. I can't share details about my underlying technology.'"  
B) Increase context length  
C) Add a stronger persona description  
D) Use few-shot examples of the persona  

---

**31.** A legal chatbot must respond formally in all cases. A user writes in Spanish: "¿Puedes ayudarme con mi contrato?" (Can you help me with my contract?). The system prompt says "respond formally." What is the best behavior?

A) Respond only in English — the system prompt's language  
B) Respond formally in Spanish — matching the user's language while maintaining the formality instruction  
C) Ask the user to write in English  
D) Respond in both English and Spanish  

---

**32.** A developer wants Claude to reason through complex medical diagnoses before giving answers. Which prompt structure is BEST?

A) `"Answer the medical question: {question}"`  
B) `"<thinking>Work through the symptoms, rule out differentials, consider risk factors</thinking> Provide your diagnosis and reasoning: {question}"`  
C) `"Think step by step. Symptoms: {question}. Provide diagnosis."`  
D) Use extended thinking with a high thinking budget for diagnostic accuracy  

---

**33.** A classification prompt needs Claude to output `{"category": "X", "confidence": 0.85}` but Claude sometimes adds commentary before the JSON. What is the MOST reliable fix?

A) Tell Claude "respond with JSON only"  
B) Add `{"category": "` as assistant prefill AND add "Respond with JSON only. No text before or after the JSON."  
C) Post-process Claude's output to extract JSON  
D) Use a JSON mode API parameter  

---

**34.** An application is using 20 few-shot examples to guide Claude's behavior. A developer notices the first few and last few examples are having more influence than the middle examples. This is an example of:

A) A prompt engineering bug  
B) Normal model behavior — primacy and recency effects mean early and late examples carry more weight  
C) Too many examples causing confusion  
D) A context window limitation  

---

**35.** A customer support prompt says "be empathetic" but users rate Claude's responses as cold and impersonal. What is the most effective prompt improvement?

A) Change model to a more empathetic version  
B) Show exactly what empathetic tone looks like with few-shot examples; replace the abstract instruction "be empathetic" with concrete behavioral examples  
C) Add "very" before "empathetic"  
D) Increase temperature for warmer responses  

---

**36.** A developer is testing a prompt and finds it works well for typical cases but fails on rare edge cases. What is the most efficient fix?

A) Redesign the entire prompt from scratch  
B) Add explicit instructions for each failing edge case; add a few-shot example for the trickiest ones  
C) Use a more powerful model  
D) Lower the temperature  

---

**37.** The PRECISE framework recommends defining both Persona (P) and Role (R). Why are two separate elements needed?

A) They serve different functions: Persona defines WHO Claude is (identity, expertise, communication style); Role defines WHAT Claude does (function, scope, limitations)  
B) Redundancy improves reliability  
C) Persona applies to all responses; Role applies only to tool use  
D) Persona is required; Role is optional  

---

**38.** A production system's system prompt contains 15 separate bullet-point rules. Several rules contradict each other. What effect does this have?

A) Claude ignores contradictory rules  
B) Contradictory rules produce unpredictable behavior — Claude may follow different rules on different requests  
C) Claude will ask for clarification before each request  
D) The last rule always takes precedence  

---

**39.** A developer uses chain-of-thought prompting for a simple yes/no classification: "Is this email spam? Think step by step, then answer Yes or No." The results are less accurate than a direct prompt. Why?

A) CoT always improves accuracy  
B) For simple binary classification, CoT adds noise — the model may "think" itself into wrong conclusions that direct pattern matching gets right  
C) The word "step" confuses the model  
D) Yes/No questions cannot use CoT  

---

**40.** A multilingual application serves users in 12 languages. The system prompt is written in English. Users write in their native language. Claude consistently responds in English. What system prompt change fixes this?

A) Write the system prompt in all 12 languages  
B) Add: "Always respond in the same language the user writes in. Never switch to English unless the user writes in English."  
C) Use separate Claude deployments per language  
D) Post-process responses to translate them  

---

## SECTION 4 — Tool Design & MCP (Questions 41–51)

---

**41.** An MCP server has a tool `get_report` that retrieves a financial report. Internally, it also caches the user's query pattern for analytics. A security audit flags this. Why?

A) Caching is not allowed in MCP tools  
B) A "get" tool with hidden side effects (analytics tracking) violates the principle that read-like tools should be free of side effects; the tracking should be a separate, explicit action  
C) Analytics data is sensitive and should not be stored  
D) Performance impact of caching makes the tool slower  

---

**42.** An MCP tool returns numerical data, but Claude sometimes misinterprets the units. For example, a `get_temperature` tool returns 22 but Claude sometimes treats this as Fahrenheit, sometimes Celsius. What is the fix?

A) Add unit information to the return type description: "Returns temperature in Celsius as a float"  
B) Return a string instead: "22°C"  
C) Add conversion logic to always return Fahrenheit  
D) Both A and B are valid; A is the more architectural solution  

---

**43.** An MCP server provides a `delete_file` tool. The tool's description says "deletes a file." Claude uses this tool for both permanent deletion and archiving, causing data loss. What two changes address this?

A) Remove the tool from the server  
B) Rename to `permanently_delete_file` AND add to the description: "Permanently removes the file from disk. This action CANNOT be undone. For reversible removal, use `archive_file` instead."  
C) Add a confirmation parameter  
D) Move the tool to a separate MCP server  

---

**44.** A company has built an internal MCP server for project management tools. They want to expand its use from 3 developers to 150 developers. Currently using stdio transport. What change is required?

A) Increase the number of stdio processes  
B) Migrate from stdio to SSE transport with authentication  
C) Distribute the stdio binary to all 150 developers  
D) Use HTTP polling instead of SSE  

---

**45.** An MCP tool that sends SMS messages has an idempotency key parameter. Claude generates idempotency keys using: `message_content + recipient + timestamp_seconds`. After a network failure, Claude retries with the same content and recipient but the timestamp has changed by 1 second. What problem does this cause?

A) The idempotency key is too long  
B) The timestamp makes the key change on retry, defeating the idempotency purpose — duplicates can be sent  
C) The recipient should not be part of the key  
D) Idempotency keys shouldn't include message content  

---

**46.** A developer is designing a tool that executes SQL queries against a company database. Which of the following is the MOST important security control?

A) Rate limit the number of queries per minute  
B) Only allow pre-defined parameterized query templates; reject freeform SQL input  
C) Log all queries for auditing  
D) Require a minimum query length  

---

**47.** An MCP server returns an error with status code 500 but no error body. Claude retries 10 times before giving up. What two improvements would reduce unnecessary retries?

A) Return a machine-readable error body with `retriable: true/false` and `retry_after_seconds` when relevant  
B) Disable retries entirely  
C) Use HTTP 200 with an error flag in the body  
D) Reduce the server timeout  

---

**48.** An MCP tool needs to access user-specific data. Multiple users call the same MCP server. What pattern ensures users only see their own data?

A) Return all data and let Claude filter  
B) Include a `user_id` parameter that Claude must provide  
C) Authenticate via OAuth and enforce row-level security server-side based on the authenticated identity  
D) Cache data per user ID in the tool's description  

---

**49.** A tool named `analyze_and_store_data` receives data, analyzes it, and writes results to a database. A developer complains that retrying after failures causes duplicate database entries. What design change prevents this?

A) Make the tool read-only  
B) Split into two tools: `analyze_data` (read, returns analysis) and `store_analysis` (write, takes idempotency key)  
C) Add error handling in the client  
D) Use transactions  

---

**50.** Which MCP capability type should be used to expose a company's API documentation to Claude so it can reference it when answering questions?

A) Tool — Claude can "fetch" the documentation  
B) Resource — documentation is read-only reference data  
C) Prompt — documentation is a reusable template  
D) Either Tool or Resource — they are interchangeable  

---

**51.** An MCP server uses SSE transport. A developer hardcodes the authentication token in the SSE URL: `https://api.example.com/sse?auth=secret123`. This token appears in server logs. What is the CORRECT fix?

A) Use a shorter token  
B) Base64-encode the token in the URL  
C) Pass the token in the Authorization header, not the URL  
D) Use a different transport (switch to stdio)  

---

## SECTION 5 — Context Management (Questions 52–60)

---

**52.** An application sends a 20,000-token context (system prompt + history + retrieved docs) to Claude. The user asks a simple question answerable with just the first 500 tokens of context. Response quality is lower than expected. What is the most likely cause?

A) Claude cannot handle 20,000 tokens  
B) Excessive irrelevant context dilutes Claude's focus on the relevant information (low signal-to-noise ratio)  
C) The context window is too small  
D) The simple question needs a more powerful model  

---

**53.** A developer measures their application's token budget: System prompt = 3,000, Conversation history = 45,000, User message = 200, Response = 2,000. Total = 50,200. The context window is 100,000 tokens. However, costs are very high. What is the primary optimization target?

A) Compress the system prompt (3,000 tokens)  
B) Reduce conversation history (45,000 tokens) through sliding window  
C) Limit response length (2,000 tokens)  
D) Split into two requests  

---

**54.** Prompt caching is enabled. The system prompt has two parts: Part A (static, 5,000 tokens) and Part B (contains today's date, 100 tokens). Part B comes before Part A. What is the effect on caching?

A) Part A and Part B are both cached efficiently  
B) The date in Part B invalidates the cache daily, making Part A also uncacheable since it comes after Part B  
C) Only Part B is affected by daily invalidation  
D) Prompt caching is unaffected by content ordering  

---

**55.** A user has a 2-hour conversation. Early turns established important constraints ("this is for a children's audience, avoid violence and adult content"). These turns have been dropped by the sliding window. Claude now produces inappropriate content. What should have been implemented?

A) A larger sliding window  
B) Memory extraction: when turns are about to be dropped, extract key constraints into a persistent memory structure that is always included in context  
C) A longer context window model  
D) User-level CALM configuration  

---

**56.** A RAG system retrieves 8 chunks (1,000 tokens each) for every query. Most queries only need 1-2 chunks. What is the correct optimization?

A) Reduce chunk size to 100 tokens  
B) Implement reranking after retrieval to select only the top 2-3 most relevant chunks  
C) Retrieve fewer chunks (1-2 maximum)  
D) Increase retrieval to 20 chunks for better coverage  

---

**57.** A developer wants to use prompt caching for a knowledge base document. The document is updated in real-time as new information is added. Is caching appropriate?

A) Yes — cache everything to reduce costs  
B) No — for real-time updating content, the cache would be invalidated constantly, providing little benefit while adding complexity  
C) Yes — use persistent caching with a 1-hour TTL  
D) Cache only the first half of the document  

---

**58.** A context window breakdown: [Instructions: 2k] [Knowledge base: 40k] [Conversation history: 50k] [User message: 1k]. Total = 93k (93% of a 100k window). Which CALM element directly addresses the knowledge base component?

A) CALM-L (Limit conversation length)  
B) CALM-C (Chunk) — retrieve only relevant chunks instead of the full 40k knowledge base  
C) CALM-M (Manage budgets)  
D) CALM-A (Aggressively cache)  

---

**59.** A production system using Claude has response latencies of 8-10 seconds. The system prompt is 30,000 tokens. Token caching is not enabled. Which change has the highest impact on latency?

A) Reduce max_tokens  
B) Enable prompt caching on the 30,000-token system prompt — subsequent requests process only the delta  
C) Use a faster model  
D) Compress the system prompt to 15,000 tokens  

---

**60.** Which of the following correctly describes the relationship between CALM-A (Aggressively Cache) and content ordering?

A) Content ordering does not affect caching  
B) Stable content must appear before dynamic content in the prompt; changing content early in the prompt invalidates the cache for all subsequent stable content  
C) Dynamic content should always come first for better performance  
D) Caching works on individual tokens, so ordering is irrelevant  

---

## ANSWER KEY

| Q | A | Q | A | Q | A | Q | A |
|---|---|---|---|---|---|---|---|
| 1 | B | 16 | B | 31 | B | 46 | B |
| 2 | C | 17 | A | 32 | D | 47 | A |
| 3 | B | 18 | B | 33 | B | 48 | C |
| 4 | B | 19 | B | 34 | B | 49 | B |
| 5 | C | 20 | B | 35 | B | 50 | B |
| 6 | C | 21 | B | 36 | B | 51 | C |
| 7 | B | 22 | B | 37 | A | 52 | B |
| 8 | C | 23 | B | 38 | B | 53 | B |
| 9 | C | 24 | B | 39 | B | 54 | B |
| 10 | B | 25 | B | 40 | B | 55 | B |
| 11 | B | 26 | C | 41 | B | 56 | B |
| 12 | C | 27 | A | 42 | D | 57 | B |
| 13 | B | 28 | B | 43 | B | 58 | B |
| 14 | C | 29 | B | 44 | B | 59 | B |
| 15 | B | 30 | A | 45 | B | 60 | B |

---

## Score Sheet

| Domain | Questions | Correct | Score | Pass? |
|--------|-----------|---------|-------|-------|
| Agentic Architecture (27%) | Q1-16 (16 Qs) | /16 | % | ≥75% |
| Claude Code Config (20%) | Q17-28 (12 Qs) | /12 | % | ≥75% |
| Prompt Engineering (20%) | Q29-40 (12 Qs) | /12 | % | ≥75% |
| Tool Design & MCP (18%) | Q41-51 (11 Qs) | /11 | % | ≥75% |
| Context Management (15%) | Q52-60 (9 Qs) | /9 | % | ≥75% |
| **Total** | **60** | **/60** | **%** | **≥75%** |

---

## Explanations for Selected Questions

**Q1 — B:** Steps 1-3 (driver, truck, route checks) are independent — parallel is optimal for speed. Step 4 (booking) depends on all three results — pipeline sequential is correct after synthesis. This is a hybrid pattern.

**Q2 — C:** "402 Insufficient Balance" is not a transient error. The user doesn't have funds — retrying won't change this. SPIDER-D: this type of error requires escalation (notify user, do not loop).

**Q3 — B:** Incorrect deletions (even at 0.5%) need HITL specifically for the delete action — it's irreversible. Approving/flagging decisions can remain automatic. This is targeted HITL for high-risk actions.

**Q7 — B:** Steps 1-3 completed successfully. The retry shouldn't re-do them. SPIDER-P: checkpoint after step 3 so resume from step 4 only. This is the classic use case for checkpointing.

**Q32 — D:** For complex, high-stakes reasoning like medical diagnosis, extended thinking with sufficient budget is the strongest approach. It gives Claude genuine reasoning capacity beyond surface-level CoT prompting.

**Q42 — D:** A is architecturally correct (describe return type with units). B also solves the problem. Choosing D recognizes both are valid but A is the architectural solution — it fixes the root cause (ambiguous type description) rather than just changing the format.

**Q47 — A:** The fix is machine-readable error responses with explicit retriability flags. This gives Claude the information it needs to decide whether to retry or escalate, preventing retry loops.

**Q54 — B:** Cache invalidation is key. Part B contains a date — it changes daily. Because Part B is BEFORE Part A, the cache prefix changes every day at the point where Part B appears. Everything after Part B (including the large Part A) is also invalidated. Fix: put Part A first, Part B last.
