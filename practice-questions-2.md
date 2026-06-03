# Practice Questions 2 — SPIDER Framework & Human-in-the-Loop

> Domain 1 deep-dive: Every SPIDER letter in isolation, combined scenarios, HITL patterns, escalation design.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** An agent processes a customer order: validate → charge card → update inventory → send confirmation. The card charge succeeds but inventory update crashes. The system retries from the beginning. The customer is charged twice. Which SPIDER letter was violated?

A) S — did not stop on failure  
B) P — did not preserve state (no checkpoint after the charge)  
C) I — side effects were not isolated; charge was not idempotent  
D) Both B and C  

**Answer: D**  
**Explanation:** Two SPIDER violations occurred. SPIDER-P: there was no checkpoint after the successful card charge, so retry restarted from the beginning. SPIDER-I: the charge side effect was not idempotent — it should include an idempotency key so re-running it returns the original result rather than creating a new charge.

---

**Q2.** An agent is analyzing legal contracts and encounters a clause it has never seen before. Its confidence in interpreting the clause is 42% (threshold: 65%). According to SPIDER-E, what must happen?

A) The agent proceeds with its 42% confidence interpretation  
B) The agent skips the clause and continues analyzing the rest  
C) The agent stops, flags the specific clause with its confidence score, and escalates to a human legal reviewer  
D) The agent requests more context from the user  

**Answer: C**  
**Explanation:** SPIDER-E: when confidence falls below the defined threshold, escalate to human. The agent must stop processing the specific ambiguous element, preserve the current state, and hand off to a human with full context (the clause, confidence score, available interpretations). Continuing with low confidence produces unreliable output.

---

**Q3.** An orchestrator's log shows only: "Task completed | Status: SUCCESS | Timestamp: 2024-03-15 14:22:11". An auditor asks: "What data did the agent process? What decision did it make? Why?" The team can't answer. Which SPIDER letter was not implemented?

A) S — Stop on failure  
B) D — Determine retry strategy  
C) R — Report outcomes  
D) I — Isolate side effects  

**Answer: C**  
**Explanation:** SPIDER-R: outcome reporting must include enough information to reconstruct what happened — inputs, outputs, reasoning, decisions. A timestamp and "SUCCESS" fails this standard completely. Audit trails require decision rationale, data processed, actions taken, and all relevant context.

---

**Q4.** An agent's retry logic for a database write failure is: "Retry immediately up to 10 times." The database is under high load. What problem does this create and which SPIDER letter addresses it?

A) The database doesn't see the agent — irrelevant  
B) Immediate retries during high load amplify the load problem (retry storm); SPIDER-D requires exponential backoff for transient load-related failures  
C) 10 retries is too many; SPIDER-S should limit to 3  
D) This is correct behavior  

**Answer: B**  
**Explanation:** SPIDER-D: Determine retry strategy. For load-related failures (database under stress), immediate retries make the situation worse by adding more load. Exponential backoff (wait 1s, 2s, 4s, 8s...) gives the database time to recover between attempts. Retry storms are a known failure mode in distributed systems.

---

**Q5.** An agent sends an email, then updates a database record. The email sends successfully but the database update fails. On retry, the email is sent again (duplicate). What is the SPIDER-I fix?

A) Check if the email was already sent before re-sending; use idempotency key or "sent" flag in state  
B) Send the email and database update in the same transaction  
C) Reverse the order: database first, then email  
D) Do not retry on database failures  

**Answer: A**  
**Explanation:** SPIDER-I: Isolate side effects. The fix is checking state before re-triggering irreversible actions. Track a flag ("email_sent: true") in persistent state after the email sends. On retry, check this flag — if already sent, skip. This is idempotent side-effect management. Database-first ordering (C) is also good practice but doesn't prevent the duplicate on retry.

---

**Q6.** A medical diagnosis assistant has a system prompt that says: "If you are less than 90% confident in a diagnosis, you must present multiple differential diagnoses and escalate to physician review." A developer changes this threshold to 50% to reduce escalations. What risk does this create?

A) None — 50% is still a majority confidence  
B) Patients receive confident-sounding diagnoses from the agent that may be wrong 50% of the time; SPIDER-E thresholds exist for patient safety  
C) More diagnoses will be escalated than before  
D) The agent will respond slower  

**Answer: B**  
**Explanation:** SPIDER-E thresholds are safety parameters. In a medical context, presenting a diagnosis with only 50% confidence without escalation means the agent is wrong half the time while the interface may not convey that uncertainty. The threshold exists to protect patients — reducing it for operational convenience directly increases safety risk.

---

**Q7.** An agent successfully completes a complex 12-step task. No errors occurred. Which SPIDER letters still apply when no failures happen?

A) Only S (Stop) — the others are only for failure scenarios  
B) Only P (Preserve state)  
C) All of SPIDER apply regardless — P (preserve state throughout), I (isolate side effects), R (report outcomes) are ongoing requirements, not just failure responses  
D) SPIDER only applies to failure scenarios  

**Answer: C**  
**Explanation:** SPIDER is not just a failure framework — it's an architecture framework. P: preserve state continuously (for potential future failures). I: isolate side effects in every execution. R: report all outcomes, including successful ones. Only S (stop), D (retry), and E (escalate) are primarily failure responses, but P, I, R apply to every run.

---

**Q8.** An HR system has an agent that can approve vacation requests up to 5 days. It approves a 4-day request from an employee who is currently on a performance improvement plan (PIP). The company policy says PIP employees need manager approval for any vacation. What does this reveal about the HITL design?

A) The agent behaved correctly — 4 days is within its authority  
B) The agent's authority boundary was defined only on duration, missing a context-based trigger (PIP status); SPIDER-E escalation triggers must account for relevant context, not just thresholds  
C) PIP status is not an agent concern  
D) The manager should have checked the system  

**Answer: B**  
**Explanation:** SPIDER-E escalation triggers must be comprehensive. Duration is one dimension of authority, but employee status (PIP) is another that overrides duration rules. Authority boundaries should model ALL relevant dimensions, not just the most obvious one. Missing a context-based trigger is a HITL design gap.

---

**Q9.** When designing HITL, a developer says: "We should require human approval for every action the agent takes." An architect says: "We should require human approval only for actions above a risk threshold." Who is right and why?

A) The developer — safety always requires full human oversight  
B) The architect — requiring approval for everything defeats the purpose of automation and creates alert fatigue; risk-based HITL targets safety effort where it matters  
C) Both are wrong — agents shouldn't require any HITL  
D) It depends on the model being used  

**Answer: B**  
**Explanation:** Blanket human approval for every action creates alert fatigue (humans approve everything without reading it) and negates the agent's value. Risk-based HITL applies human judgment precisely where it's needed: high-stakes, irreversible, novel, or high-uncertainty actions. This is the correct architectural principle.

---

**Q10.** An agent makes 1,000 decisions per day. 980 are routine (fully within policy). 15 are edge cases (need judgment). 5 are high-risk (irreversible, above authority). What HITL configuration handles this correctly?

A) HITL for all 1,000 — too risky to automate  
B) Auto-approve 980 routine cases; exception-based escalation for 15 edge cases; inline approval for 5 high-risk cases  
C) Auto-approve all 1,000 with post-hoc audit  
D) Human reviews a random 10% sample  

**Answer: B**  
**Explanation:** This is the classic tiered HITL model: fully automated for well-defined routine cases; exception-based escalation for anomalies; mandatory inline approval for genuinely high-risk irreversible actions. This optimizes human attention while maintaining safety controls where they matter.

---

**Q11.** An agent is about to send a mass email to 50,000 customers with a promotional offer. The email contains a pricing error (30% off instead of 3% off). Which HITL design would have caught this?

A) Post-hoc review after sending  
B) Inline approval for all mass communications above a recipient threshold (e.g., >1,000 recipients)  
C) The agent should self-audit its own emails  
D) Rate limiting email sends  

**Answer: B**  
**Explanation:** Mass communications are inherently high-risk (irreversible at scale, reputation and financial impact). A threshold-based HITL trigger — requiring human approval for emails to more than N recipients — would have surfaced the draft for review. SPIDER-E: irreversible actions above a defined impact level require human approval.

---

**Q12.** A SPIDER-P checkpoint stores the state: `{"step": 3, "partial_results": {"item_a": "processed"}}`. The system crashes and recovers. It resumes from the checkpoint. What must the system verify before resuming from step 4?

A) Nothing — the checkpoint is trusted as authoritative  
B) That the checkpoint data is internally consistent, that the current environment state matches the checkpoint's assumptions, and that step 3's side effects are confirmed  
C) That the user still wants to proceed  
D) That a human has reviewed the checkpoint  

**Answer: B**  
**Explanation:** Checkpoints save state, but on recovery you must validate: (1) the checkpoint data hasn't been corrupted; (2) the external world is consistent with what the checkpoint assumed (e.g., "item_a processed" means it's actually in the target system); (3) side effects completed (not just that the step ran). Resuming on stale or inconsistent checkpoint data can cause worse problems than starting over.

---

**Q13.** What is the difference between SPIDER-S (Stop) and SPIDER-E (Escalate)?

A) Stop and Escalate are the same thing  
B) Stop means cease execution (the agent halts); Escalate means stop AND transfer to a human with context for resolution. Stop without escalation is appropriate for recoverable transient errors; Escalate is for situations requiring human judgment  
C) Stop applies only to pipeline steps; Escalate applies only to agents  
D) Stop is automatic; Escalate is manual  

**Answer: B**  
**Explanation:** Critical distinction. SPIDER-S: stop processing to prevent further damage or proceeding with invalid state. SPIDER-E: stop AND actively hand off to a human with all necessary context. Some situations need only S (retry later); others need S+E (human must decide). Understanding when to stop vs. when to escalate is a key exam topic.

---

**Q14.** An agent's log reads: "Failed to send email | Error: SMTP timeout | Action taken: Retried 3 times with 2s delays | All attempts failed | Task abandoned." This is an example of which SPIDER elements working correctly?

A) S and R only  
B) D (correct retry strategy for transient error), S (stopped after 3 failed retries), R (full logging of attempts and outcome)  
C) P and I  
D) E (escalated to human)  

**Answer: B**  
**Explanation:** This log shows: SPIDER-D: SMTP timeout is a transient error; retried 3 times with delays (correct). SPIDER-S: stopped after maximum retries (didn't loop indefinitely). SPIDER-R: complete audit trail of attempts, delays, and final outcome. Missing: SPIDER-E — was this escalated to a human? Task abandonment without escalation may not be the right final action.

---

**Q15.** A customer service agent autonomously issues refunds up to $100. A request comes in for a $75 refund. The account shows 12 refunds in the past 6 months (policy allows max 3). What should the agent do?

A) Issue the refund — $75 is within the dollar limit  
B) Reject the refund — policy is exceeded  
C) Pause and escalate to a human — the refund amount is within authority but the pattern (12 refunds) is anomalous and requires judgment  
D) Issue the refund and flag the account for later review  

**Answer: C**  
**Explanation:** SPIDER-E: escalation triggers include anomalous patterns, not just simple threshold violations. A customer with 12 refunds in 6 months when the policy allows 3 is a fraud/abuse signal. Even though the dollar amount is within authority, the context creates a situation requiring human judgment. The agent must not silently proceed.

---

**Q16.** An "async review" HITL pattern means:

A) Humans review actions before they are executed  
B) Humans review a random sample of actions after they are executed, using logs  
C) The agent executes and logs actions; humans review completed actions on a schedule; reversible actions can be undone if errors are found  
D) The agent waits asynchronously for human approval  

**Answer: C**  
**Explanation:** Async review HITL: agent executes, logs comprehensively, humans review the log. Errors discovered in review can be corrected IF the actions are reversible. This is appropriate for high-volume, reversible, well-understood tasks (e.g., internal document classifications) where real-time approval is impractical.

---

**Q17.** A HITL approval interface shows the human only: "ACTION: Transfer funds | Amount: $500 | Approve/Deny?" What critical element is missing from this interface?

A) The interface is sufficient  
B) The reasoning/context: why is this transfer being requested? What triggered it? What happens if denied? Humans cannot make good approval decisions without context  
C) The user's account balance  
D) A deadline for the approval  

**Answer: B**  
**Explanation:** Good HITL interfaces present the decision context, not just the action. A human seeing "$500 transfer" with no explanation cannot make an informed decision — they'll either approve everything blindly (defeating the purpose) or reject everything cautiously. The interface must show: what triggered the action, what data supports it, what the consequences of each choice are.

---

**Q18.** An agent encounters an error it has never seen before: `ERR_UNKNOWN_STATE_0x7F`. SPIDER-D says to determine retry strategy. What is the correct handling?

A) Retry with exponential backoff — all errors are transient  
B) Retry once, then if it fails again, treat it as non-retriable and escalate  
C) Immediately escalate — unknown errors should never be retried  
D) Log and ignore  

**Answer: B**  
**Explanation:** For unknown error types, a single retry is reasonable (it might be transient) followed by escalation if it persists. This balances between over-retrying (harmful for non-transient errors) and under-retrying (missing recoverable situations). The log must include the full error code so humans can diagnose and add the error to the known-error handling matrix.

---

**Q19.** A pipeline writes to 3 different external systems: database, email service, and analytics platform. The database write is reversible (delete operation exists). The email send is irreversible. Analytics logging is reversible. Which actions need SPIDER-P checkpointing MOST critically?

A) All equally  
B) Database and analytics only (reversible, so checkpointing helps with rollback)  
C) Email send most critically — it is irreversible; checkpoint BEFORE email so recovery doesn't re-trigger it; also add idempotency  
D) None — external system writes don't need checkpointing  

**Answer: C**  
**Explanation:** Checkpointing is most critical before irreversible actions. The email send cannot be undone — if the system crashes after sending but before saving that fact, a retry will send duplicate emails. Checkpoint state to record "email sent" before executing other steps. Reversible actions (DB, analytics) have recovery paths even without checkpointing.

---

**Q20.** A developer is implementing SPIDER-R logging. They log: action, inputs, outputs, timestamp. An auditor later asks: "Why did the agent choose this action over other available options?" The logs can't answer this. What is missing?

A) More detailed input logging  
B) Decision rationale — the reasoning that led the agent to choose this action; which alternatives were considered and why they were rejected  
C) Output schema validation  
D) User context  

**Answer: B**  
**Explanation:** SPIDER-R logging must capture decision rationale, not just action + IO. Auditors, compliance teams, and debugging engineers need to understand WHY a decision was made — what alternatives existed, what factors led to this choice, what confidence level was involved. Without rationale, logs show WHAT happened but not WHY.

---

**Q21.** A company's HITL escalation path sends escalated requests to a human operator. The operator queue processes requests in 4-6 hours. The original user is waiting. What HITL design improvement addresses this?

A) Automate more decisions to reduce escalation volume  
B) Provide an intermediate response to the user: "This requires additional review; you'll receive a response within 4-6 hours"; escalate async with user notification  
C) Increase the escalation threshold so fewer things escalate  
D) Hire more human operators  

**Answer: B**  
**Explanation:** Good HITL UX communicates the escalation to the user immediately. The user should know their request is being reviewed and when to expect a response. Silent escalation leaves users uncertain. The agent should send an acknowledgment, set expectations, and proceed with the human review in the background.

---

**Q22.** A SPIDER-I principle says "isolate side effects." Which code design pattern most directly implements this?

A) Use try-catch blocks around all actions  
B) The Command pattern: encapsulate each side effect as a distinct, trackable, reversible command object that is queued, validated, then executed — not scattered through business logic  
C) Microservices architecture  
D) Event sourcing  

**Answer: B**  
**Explanation:** The Command pattern directly implements SPIDER-I isolation: each side effect (email, payment, DB write) is an explicit, bounded command. Commands can be queued, validated, and executed at a controlled point — not as inline side effects during logic execution. This makes side effects visible, auditable, and controllable.

---

**Q23.** What is "alert fatigue" in the context of HITL design and why is it a failure mode?

A) When alerts take too long to trigger  
B) When humans receive so many approval requests that they approve them without reading them, rendering HITL meaningless  
C) When the system generates too many error logs  
D) When escalation paths are too slow  

**Answer: B**  
**Explanation:** Alert fatigue is a real and dangerous HITL failure mode. If every action requires approval, humans click "Approve" reflexively to clear their queue, never actually reviewing. The safety value of HITL is completely negated. This is why risk-based, targeted HITL is architecturally superior to blanket approval requirements.

---

**Q24.** An agent's state machine has three states: RUNNING, PAUSED (waiting for HITL), FAILED. A developer proposes adding a fourth state: ABANDONED. When should ABANDONED differ from FAILED?

A) ABANDONED and FAILED are identical; only one is needed  
B) FAILED: the agent stopped due to a system error; ABANDONED: the agent was stopped because a human decided to cancel the task during HITL review — an explicit human decision, not a system error  
C) ABANDONED is used when the agent times out  
D) ABANDONED means the task will be retried; FAILED means it will not  

**Answer: B**  
**Explanation:** Distinguishing ABANDONED from FAILED matters for audit trails and downstream handling. FAILED triggers automated recovery or alerts. ABANDONED records a deliberate human decision — no retry, no recovery, full human intent logged. This distinction is important for compliance and for understanding system behavior patterns.

---

**Q25.** When implementing SPIDER comprehensively, which order of implementation provides the highest safety improvement per unit of effort?

A) R → P → I → D → E → S (reporting first)  
B) S → E → P → I → D → R  
C) E → S → I → P → D → R (escalation first to get human oversight early)  
D) The order doesn't matter — implement all simultaneously  

**Answer: C**  
**Explanation:** Practical implementation priority: E (escalation) first — establishes human oversight as a safety net for everything else. S (stop) second — prevents cascade failures. I (isolate side effects) third — prevents data corruption. P (preserve state) fourth — enables recovery. D (retry strategy) fifth — optimizes resilience. R (reporting) throughout — build from the start but continue improving. HITL as a safety net makes all other elements safer to iterate on.

---

## Score: /25 | Pass: 19/25 (75%)
