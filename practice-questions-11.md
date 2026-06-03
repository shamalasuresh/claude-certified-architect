# Practice Questions 11 — Error Handling, Idempotency & MCP Architecture

> Domain 4 deep-dive: Error response design, retry patterns, idempotency, Resources vs Tools, Prompts capability, MCP server lifecycle.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A tool call fails with the error: "Something went wrong." Claude retries 5 times, each time performing the same potentially destructive write operation. What two design problems caused this?

A) Only one problem — Claude shouldn't retry  
B) (1) Non-descriptive error doesn't tell Claude whether the error is retry-able or not. (2) The write operation is not idempotent — retrying creates duplicates. Both must be fixed: return structured errors with `retryable: false/true` and implement idempotency keys for write operations.  
C) Only one problem — the operation should be idempotent  
D) Claude should never retry  

**Answer: B**  
**Explanation:** Two independent problems: error design and idempotency. Even a retry-able error with an idempotent operation is safe (same result). Even a non-idempotent operation with `retryable: false` won't be retried. Both problems together cause multiple destructive writes. Fix both independently: descriptive errors signal retry intent; idempotency keys make write operations safe to retry.

---

**Q2.** An MCP tool implements idempotency using a client-generated `idempotency_key` parameter. The server stores `{key: result}` and returns the cached result on duplicate calls. What additional constraint must the key design enforce?

A) Keys must be integers  
B) Idempotency keys must be unique per operation (UUID or similar), scoped to a time window (e.g., 24 hours), and tied to the specific operation parameters — a key reused with different parameters should be rejected, not re-used with cached results  
C) Keys must match the tool name  
D) Keys are optional  

**Answer: B**  
**Explanation:** Idempotency key design requirements: (1) Uniqueness per operation — prevents different operations from sharing a key. (2) Time scoping — prevents very old keys from causing issues in new contexts. (3) Parameter binding — a key should be valid only for the exact same operation (same tool + same parameters). If Claude calls `create_order` twice with the same key but different amounts, reject the second call.

---

**Q3.** A developer must choose between exposing a product catalog as an MCP Tool or an MCP Resource. The catalog is: read-only, doesn't require parameters, and is large (5,000 products). Which is more appropriate?

A) Tool — more flexible  
B) Resource — Resources are designed for content that is read, not actions that are performed; the catalog is static content identified by URI, not a computation triggered by parameters; large content accessed by URI fits the Resource model  
C) Both are equivalent  
D) Neither — return it in the system prompt  

**Answer: B**  
**Explanation:** MCP Resources vs Tools distinction: Resources = content identified by a URI (data, documents, files); Tools = actions that perform operations. Product catalog is content → Resource. The catalog URI might be `products://catalog/all` or `products://category/electronics`. Resources can be listed, subscribed to, and read. Using a Tool for static content misuses the tool paradigm.

---

**Q4.** An MCP server returns this error for a validation failure: `{"error": "400"}`. A developer argues this is sufficient since HTTP 400 means bad request. What is missing?

A) Nothing — HTTP status codes are self-explanatory  
B) Missing: (1) Human-readable error message explaining WHAT is invalid. (2) Machine-readable error code for programmatic handling. (3) Specific field information if applicable: which parameter is invalid and why. Example: `{"error_code": "INVALID_PARAM", "message": "Parameter 'date' must be in YYYY-MM-DD format. Received: '15/01/2024'", "field": "date"}`  
C) Should use 422 instead  
D) Error should include the full request  

**Answer: B**  
**Explanation:** Error responses should be actionable: tell the caller exactly what to fix. For Claude: a structured error with specific guidance allows it to correct the call and retry successfully. `{"error": "400"}` only tells Claude something is wrong — not what or how to fix it. Structured errors with field-level details are essential for agentic systems where Claude must self-correct.

---

**Q5.** An agent calls `book_meeting(attendees, time, room)`. The operation succeeds and meeting is booked. Later the agent calls it again with the same parameters because it "forgot" it already called it. The result: duplicate meeting booked. What architectural pattern prevents this?

A) Use short context windows so the agent doesn't "forget"  
B) Idempotency keys: the agent generates a unique key before the first call and passes it with every retry attempt; the server recognizes the key and returns the previous result without executing again  
C) Lock the calendar after booking  
D) Use a confirmation step before booking  

**Answer: B**  
**Explanation:** Idempotency key pattern: (1) Agent generates UUID before first attempt: `key = uuid()`. (2) First call: `book_meeting(..., idempotency_key=key)` → books and stores result. (3) Accidental retry with same key → server returns cached result, no new booking. The key makes duplicate calls safe by returning the same result without duplicate side effects.

---

**Q6.** A developer defines an MCP Prompt (not Tool) for a common task: "Generate a weekly summary report based on the provided data." When is a Prompt the right choice versus a Tool?

A) Prompts are always better  
B) Prompts are reusable prompt templates that users or clients can retrieve and use; they package common task instructions; use Prompts for: standard task templates, complex multi-step instructions users want to reuse; use Tools for: operations that interact with systems or data  
C) Prompts and Tools are interchangeable  
D) Prompts are deprecated  

**Answer: B**  
**Explanation:** MCP Prompts capability: stores parameterized prompt templates that clients can list and use. "Generate weekly summary report with parameters: [data_format, time_period, recipient_type]" is a good Prompt. Users select it from a menu rather than writing instructions from scratch. Tools perform operations; Prompts provide reusable task frameworks. The distinction mirrors templates vs. actions.

---

**Q7.** An MCP server's startup sequence fails to connect to a required database. The server starts anyway and the tools return cryptic null reference errors when called. What is the correct startup behavior?

A) Start anyway and handle errors per-request  
B) Fail fast at startup: if required dependencies are not available, the server should fail to start with a clear error indicating what's missing; this surfaces problems immediately rather than returning misleading errors during tool calls  
C) Use retry logic during startup  
D) Start in degraded mode  

**Answer: B**  
**Explanation:** Fail-fast principle: if a server cannot function correctly, it should not pretend to be available. Startup health checks: (1) Verify database connectivity. (2) Verify required environment variables. (3) Validate configuration. If any required dependency is unavailable → exit with clear error. Degraded mode (tools that return null errors) is worse than no server — it produces misleading failures.

---

**Q8.** Claude makes a tool call: `delete_file(file_id: "abc123")`. The server successfully deletes the file but the network response is lost before Claude receives it. Claude assumes failure and calls `delete_file(file_id: "abc123")` again. What happens?

A) The file is deleted twice  
B) If `delete_file` is implemented as idempotent: second call recognizes the file is already deleted and returns success (or a clear "already deleted" success response); file is only deleted once  
C) The second call throws an error  
D) This scenario cannot happen  

**Answer: B**  
**Explanation:** Idempotent delete: "delete this file" should produce the same final state whether called once or ten times — the file is deleted. An idempotent implementation returns success even if the file is already gone (since the desired state is achieved). The alternative: return a specific "resource not found" error that Claude interprets as success (desired state achieved) not failure (must retry).

---

**Q9.** An MCP server hosts Resources for a document library. Documents are updated frequently (every few minutes). Claude reads a document at the start of a task, then 20 minutes later still references that document's content. What MCP feature addresses this?

A) Claude should re-read documents more often  
B) MCP Resource subscriptions: clients can subscribe to resource changes and receive notifications when content is updated; Claude (via the MCP host) can be notified when a document changes and refresh its context  
C) Set short cache TTLs  
D) Disable resource caching  

**Answer: B**  
**Explanation:** MCP Resources support change notifications via subscriptions. The workflow: (1) Claude reads document at start. (2) Subscribes to changes. (3) Server sends notification when content updates. (4) Claude re-reads the updated document. Without subscriptions, Claude works with stale data. This is the designed mechanism for keeping resource-based context fresh.

---

**Q10.** An MCP tool `process_payment` sometimes returns `{"status": "pending"}` instead of success/failure. Claude doesn't know what to do and either retries (double-charge risk) or fails. What response design fixes this?

A) Always return success or failure, no pending state  
B) Return a clear status model: `{"status": "pending", "transaction_id": "txn_123", "check_after_ms": 3000}` — Claude knows to wait, then call `get_transaction_status(transaction_id)` to poll for the result; document the async pattern in the tool description  
C) Increase response timeout  
D) Make payments synchronous  

**Answer: B**  
**Explanation:** Async operations require a complete status model: (1) `pending` with a transaction_id provides a handle for future checks. (2) `check_after_ms` hints at polling interval. (3) `get_transaction_status(id)` provides the resolution mechanism. Claude can then implement: wait → poll → resolve, without guessing or retrying the original operation. Document this pattern explicitly in the tool description.

---

**Q11.** A developer is building an MCP server that wraps a legacy API. The legacy API returns HTML error pages instead of JSON. Claude receives these HTML responses as tool results. What is the fix?

A) Ask the legacy API team to fix their responses  
B) The MCP server is the adapter layer — it must translate legacy API responses (HTML errors, XML, plain text) into proper MCP-compliant JSON responses before returning to Claude; the adapter pattern is the MCP server's job  
C) Parse HTML in the Claude prompt  
D) Return the HTML as-is and add instructions for Claude to parse it  

**Answer: B**  
**Explanation:** The MCP server is the integration layer that normalizes external API behavior. It translates: HTML error → `{"error_code": "LEGACY_API_ERROR", "message": "extracted error message"}`. This is the adapter pattern. Claude should never receive raw HTML, XML, or legacy formats — always JSON following MCP conventions. The MCP server absorbs the impedance mismatch.

---

**Q12.** An MCP tool call is taking 45 seconds. The default client timeout is 30 seconds. How should this be architected?

A) Increase the timeout to 60 seconds  
B) For long-running operations: implement the async job pattern — the tool call starts the operation and immediately returns a job_id; a separate tool polls for completion; this decouples execution time from response time  
C) Split the operation into smaller parts  
D) Warn users about expected latency  

**Answer: B**  
**Explanation:** The async job pattern solves long operation timeout problems: (1) `start_report_generation(params)` → returns `{job_id: "job_abc", estimated_seconds: 45}` immediately. (2) `get_job_status(job_id)` → returns `{status: "running", progress: 60}` or `{status: "complete", result_url: "..."}`. Claude can poll, inform the user of progress, and retrieve results when ready. No timeout issues, better user experience.

---

**Q13.** Two MCP tools are defined: `search_by_name` and `search_by_id`. They share identical implementation except for the search field. A developer proposes merging them into `search(field: "name"|"id", value: string)`. Is this a good idea?

A) Always consolidate similar tools  
B) Usually yes — reduces tool count (improving selection accuracy), but only if the descriptions are clear enough for Claude to always choose the right field; test that field selection accuracy doesn't degrade; if field confusion remains, keep them separate  
C) Never consolidate tools  
D) Only consolidate if performance improves  

**Answer: B**  
**Explanation:** Consolidation tradeoff: fewer tools = better overall selection accuracy, but field confusion within a tool = errors. The merged tool is better if the field distinction is unambiguous. The test: does Claude always select the correct `field` value? If yes, consolidate. If there's consistent confusion, keep separate tools. Always validate empirically.

---

**Q14.** An MCP server is being shut down for maintenance. Ongoing tool calls may be interrupted mid-operation. What is the correct graceful shutdown procedure?

A) Kill the process immediately  
B) Graceful shutdown: (1) Stop accepting new connections. (2) Wait for in-flight operations to complete (with a timeout). (3) For operations that cannot complete: roll back if possible, mark as interrupted if not, persist state for recovery. (4) Send appropriate error to connected clients before closing.  
C) Only stop stdio connections  
D) Let timeouts handle it  

**Answer: B**  
**Explanation:** Graceful shutdown is essential for data integrity. For write operations mid-flight: if they can be rolled back, roll them back and notify Claude of the interruption. If they completed partially, mark the state clearly so recovery is possible. Claude receives a clear error (not a timeout) that it can communicate to the user. Abrupt shutdown causes data corruption and ambiguous states.

---

**Q15.** An MCP server returns tool results in XML format. The system works correctly with the current Claude version. The developer plans to upgrade the Claude version. What risk exists?

A) No risk — XML is XML  
B) Different Claude versions may parse or interpret XML differently; the MCP specification uses JSON for tool results; using XML is a non-standard implementation that creates fragility and version-dependency  
C) The XML must be updated to include namespaces  
D) XML is faster than JSON  

**Answer: B**  
**Explanation:** MCP specifies JSON for tool results. Non-standard formats create: (1) Fragility across Claude versions. (2) Incompatibility with MCP-compliant clients. (3) Ambiguity in how Claude interprets the results. Use the specified format (JSON) to ensure compatibility, predictability, and long-term stability. Deviation from spec = technical debt.

---

**Q16.** A tool `get_recommendations` takes no parameters and returns personalized recommendations. How does the server know which user to personalize for?

A) Claude passes the user ID in a hidden parameter  
B) User identity comes from the authenticated session/token associated with the MCP connection — the server extracts the user ID from the authentication context, not from tool parameters; tool parameters should not contain identity information the server should already know from authentication  
C) Add a `user_id` parameter  
D) Use a global variable  

**Answer: B**  
**Explanation:** Authentication context carries identity. When a user authenticates to the MCP server, their identity is associated with the connection/session. Tools should extract user identity from this server-side context, not require it as a parameter. Requiring `user_id` as a parameter: (1) Is redundant (server already knows). (2) Allows clients to request other users' data. (3) Is an IDOR (Insecure Direct Object Reference) risk if not validated.

---

**Q17.** MCP Resources can be listed (`resources/list`), read (`resources/read`), subscribed to (`resources/subscribe`), and can send notifications (`notifications/resources/updated`). A developer builds a resource but doesn't implement `resources/list`. What is the impact?

A) Resources still work — listing is optional  
B) Without `resources/list`, Claude cannot discover available resources; it can only access resources whose URIs it already knows; discoverability is lost; implement `resources/list` for any resources Claude should be able to discover dynamically  
C) The resource server is non-compliant  
D) Both A and C  

**Answer: D**  
**Explanation:** Resources technically work if URI is known (A), but the MCP specification recommends implementing `resources/list` for discoverability. Without it, resources are "dark" — only accessible if the URI is hardcoded in the system prompt or otherwise pre-known. For dynamic resource discovery, `resources/list` is essential. A resource that can't be discovered is of limited value in most deployments.

---

**Q18.** A developer wants to test MCP error handling without deploying a real server. What approach works best?

A) Deploy a real server with simulated errors  
B) Use a mock MCP server that returns predetermined error responses for specific tool calls; this allows testing Claude's error handling logic in a controlled environment without side effects  
C) Test error handling manually  
D) Skip error handling tests  

**Answer: B**  
**Explanation:** Mock servers for error testing: create a test MCP server that: (1) Returns `retryable: true` errors for specific calls to test retry logic. (2) Returns `retryable: false` errors to test graceful failure. (3) Returns partial success to test partial handling. (4) Simulates timeouts. Testing error paths is as important as testing the happy path, and mocks enable deterministic error scenarios.

---

**Q19.** A developer exposes a `execute_sql` tool in an MCP server. The tool is marked `"read_only": true` in the description. What does this NOT prevent?

A) SELECT queries  
B) "read_only: true" in the description is advisory — it's a signal to Claude, not a technical constraint; a determined or injected instruction could still call it with INSERT/UPDATE SQL; technical enforcement (database user with SELECT-only privileges) is required to actually enforce read-only access  
C) Table scans  
D) Stored procedure calls  

**Answer: B**  
**Explanation:** Description metadata is advisory, not enforced. "read_only: true" tells Claude to only use SELECT queries, but: (1) Prompt injection could instruct Claude to execute UPDATE. (2) A misconfigured Claude could ignore the advisory. Technical enforcement: (1) Connect to database with a read-only user account. (2) The database-level permission prevents write operations regardless of what Claude sends. Description = advisory; technical control = enforcement.

---

**Q20.** An MCP server's `create_user` tool successfully creates a user but then fails on a subsequent step (sending a welcome email). The tool returns an error. Claude retries the entire tool call. What problem occurs?

A) The email is not sent  
B) Duplicate user creation: the partial success (user created) is not tracked; the retry creates a second user; the server must implement transactional semantics or track partial completion via idempotency key before retrying  
C) The retry will work correctly  
D) The email failure should be ignored  

**Answer: B**  
**Explanation:** Partial failure with retry = duplicate creation. Solutions: (1) Transactional: roll back user creation if email fails — either both succeed or neither does. (2) Idempotency key: on retry with same key, recognize user was already created and only retry the email step. (3) Separate tools: `create_user` and `send_welcome_email` — Claude handles the failure of each independently. Multi-step operations need explicit failure handling.

---

**Q21.** A developer builds an MCP server that returns rich formatted markdown in tool results (bold text, tables, headers). Why might this cause problems?

A) Markdown is not valid JSON  
B) Claude may interpret markdown formatting in tool results as display formatting and pass it through literally to users; or different Claude versions may render markdown differently; tool results should return structured data (JSON fields) not formatted display content  
C) Markdown slows processing  
D) Only HTML is supported in tool results  

**Answer: B**  
**Explanation:** Tool results should be structured data, not pre-formatted display content. Return: `{"items": [{"name": "Product A", "price": 29.99}]}` not `"**Product A**: $29.99\n---"`. Claude decides how to present data to users; the tool provides data. Markdown in tool results creates display formatting leakage and inconsistent presentation.

---

**Q22.** An MCP tool call returns a very large result (100,000 tokens). Claude's context is nearly full before the tool call. What should the server do?

A) Return the full result regardless  
B) Tools should support pagination or size limits: `max_results`, `page`, `page_size` parameters; additionally, the MCP server should be aware of practical result size limits; returning 100k tokens when context is nearly full causes context overflow  
C) Compress the result  
D) Claude handles context overflow automatically  

**Answer: B**  
**Explanation:** Tool results consume context window (CALM-L: Limit context). A tool that can return 100k tokens must support size controls: (1) `max_results` limits result count. (2) `fields` limits returned fields per result. (3) Pagination allows fetching in chunks. Good tool design respects context constraints. A tool that can exhaust the context window on a single call is an operational hazard.

---

**Q23.** An MCP server must serve both synchronous (immediate response) and asynchronous (long-running) tool calls. What pattern separates these cleanly?

A) Use two separate servers  
B) Use naming convention and response contract: synchronous tools return results directly; asynchronous tools return a `{job_id, status: "started", poll_tool: "get_job_status"}` response; document which tools are async in their descriptions; async tools always return a job handle, never block  
C) Asynchronous tools should be removed  
D) Use different HTTP endpoints  

**Answer: B**  
**Explanation:** The naming + response contract pattern cleanly separates sync and async: (1) Description identifies async behavior: "Async tool. Returns job_id for status polling." (2) Response structure is consistent: every async tool returns `{job_id: string, poll_tool: string, check_after_ms: number}`. (3) Claude knows the pattern and applies it universally. Consistency is key — all async tools follow the same pattern.

---

**Q24.** When an MCP tool returns an error, what should Claude do by default if no error handling is specified?

A) Retry indefinitely  
B) Communicate the error to the user/orchestrator clearly and stop the current action; do not assume errors are transient; do not retry destructive operations without explicit instructions; present the error information so a human or orchestrator can decide next steps  
C) Ignore the error and continue  
D) Retry once automatically  

**Answer: B**  
**Explanation:** Default error behavior: stop, don't assume, communicate. Retrying destructive operations without explicit guidance risks duplicate side effects. Assuming errors are transient and retrying indefinitely can amplify failures. The safest default: communicate the error clearly, preserve state, allow human or orchestrator to decide on retry strategy. This is the SPIDER-E (Escalate) behavior for tool errors.

---

**Q25.** A developer is designing a new MCP server from scratch. What is the recommended implementation order for features?

A) Build all features simultaneously  
B) Implement in order: (1) Core tool definitions and happy-path responses. (2) Input validation and error responses. (3) Authentication and authorization. (4) Idempotency for write operations. (5) Rate limiting. (6) Monitoring and logging. (7) Resource subscriptions and async patterns. Core functionality first, security and reliability second, advanced features last.  
C) Start with authentication  
D) Start with monitoring  

**Answer: B**  
**Explanation:** Implementation order: core → safety → reliability → advanced. Core tools first (business value). Input validation + error responses second (correct behavior for invalid inputs). Auth third (required before any real deployment). Idempotency fourth (required for production write operations). Rate limiting + monitoring add operational maturity. Subscriptions/async last (enhancement, not required for basic operation).

---

## Score: /25 | Pass: 19/25 (75%)
