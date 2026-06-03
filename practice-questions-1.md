# Practice Questions 1 — Agentic Architecture: Orchestration Patterns

> Domain 1 deep-dive: Router, Pipeline, Parallel — when to use each, how to combine them, what breaks them.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A B2B SaaS platform needs an AI assistant that handles four different request types: account billing, technical bugs, feature requests, and onboarding help. Each type requires completely different tool access and context. Which pattern is the primary architecture?

A) Pipeline — process all requests through 4 sequential stages  
B) Router — classify intent, dispatch to the appropriate specialist  
C) Parallel — run all four agents simultaneously, return the most relevant  
D) Single agent with all four tool sets combined  

**Answer: B**  
**Explanation:** When you have multiple distinct intents that map to specialized handlers, Router is the correct pattern. Each specialist is independent and only activated when its domain is relevant. Running all four in parallel wastes resources. A single agent with all tools violates least privilege and reduces accuracy.

---

**Q2.** A content pipeline processes submitted articles: Step 1 extracts key claims, Step 2 fact-checks each claim, Step 3 calculates a credibility score, Step 4 generates an editorial summary. Step 3 cannot run without Step 2's output. What pattern applies?

A) Parallel — run all steps concurrently  
B) Router — route based on article topic  
C) Pipeline — each step feeds the next  
D) Single agent — all in one prompt  

**Answer: C**  
**Explanation:** Sequential dependency (each step needs the previous step's output) is the defining characteristic of Pipeline orchestration. The pipeline is deterministic, ordered, and debuggable. Parallel would fail because Steps 3 and 4 can't run without prior outputs.

---

**Q3.** A due-diligence tool simultaneously requests: company financial data from a financial DB, news sentiment from a news API, social media sentiment from a social API, and competitive analysis from an internal knowledge base. All four are independent. Which pattern?

A) Pipeline — process in a safe, sequential order  
B) Router — determine which source is most relevant  
C) Parallel — all four requests are independent; run concurrently  
D) None — this requires a human analyst  

**Answer: C**  
**Explanation:** Four independent data sources with no dependencies between them = Parallel pattern. This minimizes total latency (total time ≈ slowest individual request, not the sum). An orchestrator dispatches all four, waits, then synthesizes. Pipeline would add unnecessary latency.

---

**Q4.** A router agent classifies a request as billing-related with 52% confidence (below a configured 70% threshold). What should it do?

A) Route to billing anyway since 52% is the highest probability  
B) Ask the user a clarifying question to determine intent before routing  
C) Route to the "general" fallback agent  
D) Return an error to the user  

**Answer: B**  
**Explanation:** When classification confidence is below threshold, the correct behavior is disambiguation — ask the user a targeted clarifying question. Routing on 52% confidence will be wrong ~48% of the time. A general fallback is acceptable for some systems but asking is more accurate. Never surface an internal error to the user for a classification problem.

---

**Q5.** A pipeline has 8 steps. Steps 1-4 run successfully. Step 5 fails. The team considers: "Should we retry from step 1 or step 5?"

What is the correct principle and answer?

A) Always retry from step 1 for consistency  
B) Retry from the last successful checkpoint (step 4 output); only re-run step 5 onward  
C) Skip step 5 and continue from step 6  
D) Abandon the pipeline; manual intervention required  

**Answer: B**  
**Explanation:** SPIDER-P: Preserve state. Checkpoint after each successful step so recovery re-runs only the failed step and beyond. Retrying from step 1 wastes compute and can re-trigger side effects (if steps 1-4 had any). Skipping a failed step would corrupt the pipeline's output integrity.

---

**Q6.** A company uses a router that dispatches to: `SalesAgent`, `SupportAgent`, `TechAgent`. A new type of request arrives that overlaps both Sales and Tech ("What technical integrations does your enterprise plan include?"). What should the router do?

A) Route to SalesAgent only  
B) Route to TechAgent only  
C) Route to both in parallel; synthesize the combined response  
D) Return an "ambiguous query" error  

**Answer: C**  
**Explanation:** Multi-intent requests that span multiple specialist domains should be dispatched to all relevant agents in parallel. The orchestrator synthesizes a unified response. This is a standard router capability — routers can dispatch to multiple agents simultaneously when intent is compound.

---

**Q7.** A pipeline orchestration system processes insurance claims: Ingest → Parse → Validate → Fraud Check → Approve/Deny → Notify. "Approve/Deny" has two branches. How should branching be handled in a pipeline?

A) Pipelines cannot have branches — redesign as a router  
B) Add conditional logic at the Approve/Deny step; it either sends to Approve pipeline or Deny pipeline based on the fraud check result  
C) Run both Approve and Deny in parallel, discard the inapplicable one  
D) Branch by creating two separate pipelines  

**Answer: B**  
**Explanation:** Pipelines can have conditional branches — a step's output determines which subsequent step runs. This is a conditional pipeline. Running both branches in parallel and discarding one wastes resources and may trigger irreversible side effects (e.g., both an approval notice and denial notice).

---

**Q8.** An agentic system is described as: "The orchestrator takes the user's request, breaks it into 5 parallel sub-tasks, waits for all results, then synthesizes." A latency SLA requires the entire flow in under 3 seconds. Sub-task 3 averages 4 seconds. What is the correct architectural fix?

A) Remove sub-task 3  
B) Switch from parallel to sequential to better control timing  
C) Set a timeout for sub-task 3 and proceed with partial results if it misses the deadline  
D) Cache sub-task 3's output permanently  

**Answer: C**  
**Explanation:** In parallel orchestration with latency SLAs, each sub-agent must have a timeout. If sub-task 3 misses the deadline, the orchestrator proceeds with the 4 available results, labels the response as partial, and optionally triggers sub-task 3 asynchronously. Removing sub-task 3 loses functionality. Sequential would make ALL tasks take 4+ seconds.

---

**Q9.** A nested orchestration pattern has: `TopOrchestrator → Router → [SalesAgent, TechAgent]`. Within TechAgent, there are sub-steps that run sequentially. What pattern does this describe?

A) Invalid — patterns cannot be nested  
B) Router at the top level; Pipeline within TechAgent  
C) All Pipeline  
D) All Parallel  

**Answer: B**  
**Explanation:** Orchestration patterns can and should be nested. The top level uses Router (classifies intent, dispatches). Inside TechAgent, sequential steps use Pipeline. This is common in real systems — a router dispatches to specialized agents that internally run their own orchestration logic.

---

**Q10.** After a parallel orchestration run, the orchestrator needs to synthesize results from 5 sub-agents. Sub-agents 1, 3, and 5 returned valid data. Sub-agents 2 and 4 returned errors. What is correct synthesis behavior?

A) Discard all results since the set is incomplete  
B) Use results 1, 3, 5; clearly mark in the output which sources were unavailable  
C) Re-run sub-agents 2 and 4 before synthesizing  
D) Ask the user whether to proceed with partial data  

**Answer: B**  
**Explanation:** Graceful degradation — partial results with transparency are better than failure. The response should use available data, clearly indicate which sources were unavailable, and potentially flag the response as partial. Re-running in-line adds latency and may not succeed. Asking the user introduces unnecessary friction for a routine partial-data scenario.

---

**Q11.** A pipeline for generating marketing copy: Extract product info → Research competitors → Generate copy → Review for brand compliance → Finalize. Research (step 2) is slow (avg 8 seconds). Can this pipeline be optimized with parallelism?

A) No — pipelines are always sequential  
B) Yes — if step 1's output can feed both step 2 (competitor research) and an early draft of step 3 simultaneously, run them in parallel then merge  
C) Skip step 2 to reduce latency  
D) Move step 2 to after step 3  

**Answer: B**  
**Explanation:** Hybrid patterns are valid. If step 3 can begin drafting from step 1's output while step 2 researches concurrently, this reduces latency. When step 2 completes, its results refine the draft. This "parallel branches that merge" is a valid pipeline optimization for independent sub-flows.

---

**Q12.** What is the key design question that determines whether a task should use an agent vs. a simple chain of API calls?

A) Does the task require more than 3 API calls?  
B) Is the task complex?  
C) Does the task require autonomous decision-making, dynamic tool selection, or handling of unpredictable branching logic based on intermediate results?  
D) Does the task involve external data sources?  

**Answer: C**  
**Explanation:** The agent vs. pipeline decision hinges on autonomy and dynamic decision-making. If you can define the exact sequence of steps in advance (even with branching), a pipeline suffices. Agents are for tasks where the next action depends on what was just discovered — the flow cannot be fully pre-specified.

---

**Q13.** A Router agent is given a message: "Cancel my subscription and also explain why my last invoice was $50 higher than expected." The billing specialist handles cancellations. The support specialist handles billing questions. They use completely separate data sources. What is the correct routing architecture?

A) Route to billing specialist only — cancellation is higher priority  
B) Route to support specialist only — billing inquiry is more complex  
C) Route to both; billing specialist handles cancellation, support specialist explains the invoice discrepancy; responses are combined  
D) Ask the user to send two separate messages  

**Answer: C**  
**Explanation:** This is a compound intent — two clearly distinct tasks for two different specialists. Route both in parallel. The orchestrator synthesizes a unified response covering both. Forcing the user to send two messages is poor UX when the system can handle it automatically.

---

**Q14.** In a pipeline with steps A → B → C → D, step C has an unpredictable failure rate of 15% due to an unstable external API. What is the BEST resilience design?

A) Remove step C from the pipeline  
B) Implement retry (with backoff) for step C up to 3 times; if all fail, use a fallback implementation of C (or flag result as incomplete)  
C) Wrap the entire pipeline in a try-catch and restart from A on any failure  
D) Route around step C when it fails by connecting B directly to D  

**Answer: B**  
**Explanation:** SPIDER-D: transient external API failures should be retried with backoff. A fallback handles sustained outages gracefully. Restarting from A throws away B's work and may re-trigger side effects. Skipping C entirely or connecting B→D may corrupt the pipeline's semantics.

---

**Q15.** An orchestrator dispatches to three parallel agents. Agent 1 takes 2 seconds. Agent 2 takes 4 seconds. Agent 3 takes 1 second. The orchestrator times out after 3 seconds. Which agents' results are available?

A) Only Agent 3 (1 second)  
B) Agents 1 and 3 (1 and 2 seconds)  
C) All three if the timeout is measured from last dispatch  
D) None — a timeout fails the entire request  

**Answer: B**  
**Explanation:** At 3-second timeout, Agents 1 (2s) and 3 (1s) have completed; Agent 2 (4s) has not. The orchestrator has partial results from Agents 1 and 3. A well-designed system proceeds with these two results, notes Agent 2 was unavailable, and returns a partial response — not a total failure.

---

**Q16.** A team has a pipeline with 10 steps. Steps 1-5 are fast (milliseconds). Steps 6-10 each call external services (seconds). How should checkpointing be designed?

A) Checkpoint after every step for maximum safety  
B) Checkpoint after the last fast step (step 5) and after each external service call (steps 6-10)  
C) Checkpoint only at the very end  
D) No checkpointing needed for pipelines under 20 steps  

**Answer: B**  
**Explanation:** Checkpoint cost-benefit: checkpointing fast in-memory steps adds overhead with little benefit (they're cheap to re-run). Checkpointing before and after expensive/slow external calls is high-value (avoids re-running slow steps on failure). Checkpoint strategy should align with the cost of the work being preserved.

---

**Q17.** A Pipeline and a Router are both given the same task: "A user submits a support ticket; classify it and then route it to the right team." Which is correct?

A) This is a Pipeline task  
B) This is a Router task  
C) This is both: a Pipeline of [classify → route]; classification uses the Pipeline pattern while routing uses the Router pattern  
D) Neither — this is a single-step task  

**Answer: C**  
**Explanation:** Real systems combine patterns. "Classify then route" is a two-step Pipeline where the first step IS the router's classifier. This is correct — a Router IS a specific pipeline pattern where one step is classification and the next is dispatch. Understanding that patterns compose and nest is essential for the exam.

---

**Q18.** An orchestrator runs 5 sub-agents in parallel. If one sub-agent goes rogue (loops indefinitely), what safeguard prevents the orchestrator from hanging forever?

A) Claude automatically kills infinite loops  
B) Per-agent execution timeouts set by the orchestrator; if an agent exceeds its time budget, the orchestrator kills it and proceeds  
C) The orchestrator will naturally detect the loop and terminate the agent  
D) Infinite loops cannot occur in agentic systems  

**Answer: B**  
**Explanation:** Infrastructure-level timeouts per sub-agent are the required safeguard. Each parallel agent should have a maximum wall-clock time after which the orchestrator abandons it. Claude itself won't necessarily detect an infinite tool-call loop at the platform level — that's the system architect's responsibility.

---

**Q19.** An e-commerce agent receives: "Find me a laptop under $800, add the best one to my cart, and schedule a delivery for next Monday." Which orchestration pattern best handles this?

A) Router — three different intents  
B) Pipeline — sequential: search → select → add to cart → schedule delivery  
C) Parallel — run all three actions simultaneously  
D) Single LLM call with all tools available  

**Answer: B**  
**Explanation:** These steps have strict sequential dependencies: you must search before you can select; you must select before you can add to cart; cart must be updated before scheduling delivery. Pipeline is correct. Router would be wrong (there aren't separate specialists). Parallel would fail because each step needs the prior step's result.

---

**Q20.** A router is trained to route to one of three agents. Over time, a new type of request appears that doesn't match any agent. What is the correct design behavior?

A) Route to the closest-matching agent  
B) Return an error  
C) Route to a "catch-all" or "general" agent; log unmatched intents for future agent development  
D) Ask the user to rephrase their request  

**Answer: C**  
**Explanation:** Routers must have a fallback path. A catch-all agent handles unmatched intents gracefully. Crucially, unmatched intents should be logged and analyzed — they are signals that new specialist agents may be needed. This is how agentic systems evolve: unmatched patterns indicate capability gaps.

---

**Q21.** What distinguishes a "fan-out/fan-in" pattern from basic parallel orchestration?

A) They are identical  
B) Fan-out dispatches identical work to multiple agents; fan-in/fan-out with different tasks is parallel orchestration. Fan-out specifically means splitting one task into N identical parallel executions then recombining  
C) Fan-out is sequential; fan-in is parallel  
D) Fan-out is a hardware concept, not applicable to agents  

**Answer: B**  
**Explanation:** Fan-out: one task → N copies running in parallel (e.g., "analyze this document for tone, accuracy, and legal risk" = fan-out to 3 identical analysis processes). Fan-in: collecting results back. Basic parallel orchestration dispatches *different* tasks. Understanding this distinction helps design map-reduce style patterns for large-scale analysis.

---

**Q22.** A Pipeline is described: Extract → Transform → Load. The Load step writes to a database. The Transform step fails intermittently. Which design prevents partial writes on retry?

A) Wrap everything in a transaction  
B) Only retry Transform; Load is not re-run if Transform re-succeeds with the same data  
C) Checkpoint after Transform; on retry, verify the data hasn't changed before re-running Load; use idempotent Load (upsert with unique key)  
D) Add a human approval step before Load  

**Answer: C**  
**Explanation:** Idempotent Load operations (upsert with unique key) combined with SPIDER-P checkpointing means retrying the full ETL is safe — Load can be re-run and won't create duplicates. This is the standard pattern for reliable ETL pipelines with agents.

---

**Q23.** An orchestrator has sub-agents A, B, and C. A and B can run in parallel. C depends on BOTH A and B. What execution graph describes this?

A) A → B → C (fully sequential)  
B) [A || B] → C (A and B in parallel, then C when both complete)  
C) A → [B || C] (A sequential, then B and C in parallel)  
D) [A || B || C] (all parallel)  

**Answer: B**  
**Explanation:** [A || B] → C is the correct dependency graph. A and B run concurrently (saving latency), then C waits for BOTH to complete before executing. This is a fork-join pattern — a fundamental building block for complex multi-agent workflows.

---

**Q24.** A developer argues that their Router should "pick the single best agent and route only to it" for every request. A solution architect disagrees and says some requests need multi-agent routing. Which types of requests specifically justify multi-agent routing?

A) No request justifies routing to multiple agents  
B) Requests where multiple intents are present simultaneously, each requiring different specialist knowledge or tool access  
C) All requests over 100 words should go to multiple agents  
D) Requests that use the word "and"  

**Answer: B**  
**Explanation:** Multi-agent routing is justified when a single request contains genuinely distinct intents that map to different specialists. "Cancel my subscription AND explain my invoice" requires billing (cancellation) + support (invoice explanation). These are distinct concerns, different tools, different context — single-agent routing would leave one concern poorly served.

---

**Q25.** A development team says: "We designed our orchestrator but we're not sure if we need Router, Pipeline, or Parallel. How do we decide?" Which decision framework is BEST?

A) Choose based on team experience with each pattern  
B) Always start with Pipeline, then add Router and Parallel as needed  
C) Ask: Do tasks have different intents requiring specialists? → Router. Do tasks have sequential dependencies? → Pipeline. Are tasks independent with latency pressure? → Parallel. Combine as needed.  
D) Use Router for user-facing systems and Pipeline for internal systems  

**Answer: C**  
**Explanation:** The decision framework maps task characteristics to patterns: intent diversity → Router; sequential dependency → Pipeline; independent + latency-sensitive → Parallel. Real systems combine all three. This is the exam's core reasoning model for agentic architecture questions.

---

## Score: /25 | Pass: 19/25 (75%)
