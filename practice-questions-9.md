# Practice Questions 9 — Tool Schema Design & Anti-Patterns

> Domain 4 deep-dive: Description quality, parameter design, naming conventions, schema constraints, tool count optimization.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A developer defines a tool with `"description": "Gets data"`. Claude uses it inconsistently — sometimes calling it for the right purpose, sometimes not. What is the root cause?

A) The tool has too many parameters  
B) The description is too vague; Claude relies on the description to decide when and how to use a tool; "Gets data" provides no information about what data, from where, or when to call it  
C) Claude cannot use tools with short descriptions  
D) The tool needs more parameters  

**Answer: B**  
**Explanation:** Tool descriptions are the primary signal Claude uses for tool selection. "Gets data" is unusable — every tool "gets data" in some sense. A good description: "Retrieves a customer's account details (balance, tier, status, last activity) from the CRM by customer_id. Use when the conversation requires customer-specific account information." This tells Claude what data, the return value shape, and the trigger condition.

---

**Q2.** A MCP server exposes 47 tools. Claude's tool selection accuracy is 62%. A developer reduces this to 12 tools. Tool selection accuracy jumps to 91%. What explains this?

A) Fewer tools means more API calls per task  
B) Tool selection degrades with large tool counts; Claude must distinguish between 47 options, increasing the chance of choosing the wrong one; organizing tools into domains and reducing count to ~15 or fewer per context maintains high selection accuracy  
C) The removed tools were broken  
D) API latency improved  

**Answer: B**  
**Explanation:** Tool selection accuracy degrades with count. When too many tools are available: (1) Similar tools create ambiguity. (2) Claude's attention to each tool description is diluted. (3) Naming collisions increase. Best practice: expose only the tools needed for the current task context. If many tools are required, use tool routing (a meta-tool that selects a sub-set of tools) or domain-specific tool sets.

---

**Q3.** A tool parameter is defined as `"type": "string"` for a date parameter. Claude sends dates in formats: `2024-01-15`, `January 15, 2024`, `1/15/24`, `15th January 2024`. Which schema constraint eliminates this variation?

A) Add a description saying "use ISO format"  
B) Use `"format": "date"` or add a `"pattern": "^\\d{4}-\\d{2}-\\d{2}$"` regex constraint in the JSON schema, AND document in the description: "Date in ISO 8601 format YYYY-MM-DD (e.g., 2024-01-15)"  
C) Use integer timestamps instead  
D) Validate on the server side and return an error  

**Answer: B**  
**Explanation:** Both schema constraint + description work together: the schema pattern enforces the format structurally (Claude will produce schema-compliant output), and the description example clarifies intent. Schema constraints are the primary enforcement mechanism; descriptions provide semantic context. Integer timestamps (C) solve the format problem but reduce human readability of logs and Claude's reasoning.

---

**Q4.** A tool is named `doTheThing()`. Why is this a problematic tool name?

A) It's too short  
B) Tool names should be descriptive and action-oriented verbs that indicate what the tool does; `doTheThing` provides zero semantic signal for tool selection; name it based on the action and object: `send_email`, `get_customer_record`, `calculate_tax`  
C) Underscores should not be used  
D) Only camelCase should be used  

**Answer: B**  
**Explanation:** Tool naming is critical for selection accuracy. Convention: `{verb}_{noun}` in snake_case or camelCase. `send_email`, `search_products`, `delete_record`, `validate_address`. The name should unambiguously identify the action (verb) and the target (noun). Claude uses the name + description together for selection — a bad name forces the description to do more work and increases ambiguity.

---

**Q5.** A tool has 15 parameters, most optional. Claude frequently calls it with missing required parameters and wrong optional parameter combinations. What redesign helps most?

A) Add more descriptions to each parameter  
B) Split into multiple focused tools with fewer parameters each; a tool with 15 parameters is doing too many things; split by use case so each tool has 3-5 parameters covering a specific scenario  
C) Make all parameters required  
D) Use a single `options` object parameter  

**Answer: B**  
**Explanation:** Tool design principle: each tool should do one thing well. 15 parameters suggests a multi-purpose tool. Splitting: `create_basic_order(product_id, quantity)` + `create_custom_order(product_id, quantity, customizations, delivery_date)` + `create_bulk_order(product_ids, quantities, discount_code)` — each focused, fewer parameters, clear use case. Claude selects the right tool rather than struggling with a complex parameter set.

---

**Q6.** A read tool `search_customer_data` sometimes takes 10+ seconds. Claude times out and tries again, causing duplicate searches. What is the correct design?

A) Increase the timeout  
B) Read operations should be idempotent (calling multiple times produces the same result) — duplicate searches are fine; the real issue is UX: return a progress indicator or stream results; for truly long operations, use a job ID pattern: start_search() returns a job_id, check_search_status(job_id) polls for completion  
C) Make the search faster  
D) Disable retries  

**Answer: B**  
**Explanation:** Search tools are idempotent — duplicate calls are harmless. But the UX problem (multiple searches appearing to the user) is addressable with the async job pattern: `start_search()` returns a job_id immediately, `get_search_results(job_id)` polls or waits for completion. This pattern handles long-running reads without timeout issues or user-visible duplicates.

---

**Q7.** A tool schema defines an enum: `"status": {"enum": ["active", "inactive", "pending", "suspended", "archived", "deleted", "banned"]}`. Claude struggles to select the right status value in context. What redesign helps?

A) Remove the enum constraint  
B) If this tool is for different operations, split it; if statuses are used in different contexts, group them: user-facing status changes have different valid values than admin operations; alternatively, reduce the enum to the values valid for the specific operation  
C) Add descriptions to each enum value  
D) Use a string with validation  

**Answer: C**  
**Explanation:** The fix is actually C: enums should have descriptions for each value in the parameter description: `"status: One of: 'active' (account in good standing), 'suspended' (temporarily restricted), 'archived' (read-only, non-deletable). Note: 'banned' and 'deleted' require admin privileges and use separate tools."` Descriptions within the enum parameter description help Claude select the right value. Splitting is also valid for privilege-restricted operations (B).

---

**Q8.** A tool for sending emails has the parameter `"recipients": "string"`. A developer realizes this should accept multiple recipients. Which schema change is correct?

A) Change to `"recipients": {"type": "string", "description": "comma-separated emails"}`  
B) Change to `"recipients": {"type": "array", "items": {"type": "string", "format": "email"}, "minItems": 1, "maxItems": 50}`  
C) Keep as string and handle parsing server-side  
D) Use a separate parameter per recipient  

**Answer: B**  
**Explanation:** Use the correct JSON type for the data structure. Array type with email format validation is semantically correct for multiple recipients. It: (1) signals to Claude that multiple recipients are expected (array). (2) Validates each item is an email. (3) Constrains bounds (1-50 recipients). Option A (comma-separated string) is ambiguous, harder to validate, and forces server-side parsing. Arrays are the correct type for lists.

---

**Q9.** A developer creates a tool `get_user_and_process_request()` that both retrieves user data AND performs business logic. What is wrong with this tool design?

A) The name is too long  
B) The tool violates the single-responsibility principle; tools should be atomic — either side-effect-free (reads) or targeted actions (writes); combining retrieval with processing makes the tool harder to predict, test, and selectively retry  
C) Both operations should be in separate services  
D) Tools can combine operations for efficiency  

**Answer: B**  
**Explanation:** Tool design principle: separate reads from writes. `get_user(user_id)` is a safe, idempotent read. `process_request(user_id, request_data)` is a write with side effects. Combined: if the process fails, was it the read or the write? Can you safely retry? Combining makes error handling ambiguous and the tool hard to reason about. Separate tools, clear contracts.

---

**Q10.** A tool description says: "This powerful tool can handle many different types of requests and will process them accordingly." What is wrong?

A) "Powerful" is marketing language  
B) The description is circular and content-free; it describes every tool in existence; it provides no information about: what inputs trigger this tool, what it does, what it returns, or when NOT to use it  
C) Too short  
D) Missing return value description  

**Answer: B**  
**Explanation:** Tool descriptions must answer: (1) What does this tool do? (2) When should Claude call it? (3) What parameters matter? (4) What does it return? (5) When should Claude NOT call it? A description that answers none of these is useless. Concrete: "Submits a support ticket to Zendesk. Use when a customer issue requires human agent follow-up. Returns a ticket_id and estimated_response_time. Do NOT use for FAQs that can be answered immediately."

---

**Q11.** A developer creates a `search` tool and a `find` tool that do almost the same thing (full-text search vs. exact match search). Claude consistently picks the wrong one. What is the fix?

A) Delete one tool  
B) Clearly differentiate in names and descriptions: `search_full_text(query)` ("Use for natural language queries, partial matches, relevance-ranked results") vs. `find_exact(field, value)` ("Use when you know the exact value of a specific field; faster, returns only exact matches"); the distinction must be unambiguous  
C) Add more parameters to each  
D) Merge them into one tool  

**Answer: B**  
**Explanation:** When two similar tools exist, their descriptions must explicitly contrast them. Each description should include: "Use THIS tool when [condition]" and optionally "Use [other tool] instead when [different condition]." Cross-references in descriptions help Claude understand the distinction and choose correctly.

---

**Q12.** A financial tool uses the parameter `"amount": {"type": "number"}`. Transactions are failing because Claude sometimes sends `100` (USD dollars) and sometimes `10000` (USD cents). What schema fix prevents this?

A) Use string type for amounts  
B) Add unit specification in the description: `"amount: Transaction amount in USD cents (integer). Example: $10.50 should be sent as 1050."` AND use `"type": "integer"` to prevent decimal amounts; document units explicitly  
C) Use two separate parameters  
D) Add a `currency` parameter  

**Answer: B**  
**Explanation:** Ambiguous units are a common API bug. Fix: (1) `"type": "integer"` prevents `100.50` (forces cents, not dollars). (2) Description explicitly states the unit: "in USD cents." (3) Example converts between units (Claude can verify its math). This prevents unit confusion. For multi-currency: add a `"currency": {"type": "string", "enum": ["USD", "EUR"]}` parameter.

---

**Q13.** A developer wants to prevent Claude from calling a destructive tool (delete_record) without confirmation. The tool is defined normally. What additional mechanism is needed?

A) Add "require confirmation" to the tool description  
B) Implement a two-step pattern: the tool call itself does a "dry run" and returns a confirmation token; a separate `confirm_delete(token)` tool executes the deletion; Claude must call both in sequence  
C) Set the tool to read-only  
D) Add a `confirmed: boolean` parameter  

**Answer: B**  
**Explanation:** The two-step confirmation pattern architecturally enforces confirmation: (1) `delete_record(id)` → returns `{preview: "This will delete...", token: "abc123", expires_in: 60}`. (2) `confirm_delete(token)` → executes. Without the token from step 1, step 2 cannot execute. This is more robust than a `confirmed: boolean` parameter, which Claude might set to `true` without an actual confirmation step.

---

**Q14.** A tool returns a 2000-token JSON response for every call, but Claude only ever uses 3 fields from it. What problem does this create and what is the fix?

A) No problem — Claude filters relevant fields  
B) Large tool responses consume context window unnecessarily; fix: redesign the tool to return only the requested fields via a `fields` parameter, or create targeted tools: `get_customer_name(id)`, `get_customer_balance(id)` instead of `get_customer_full_record(id)`  
C) Compress the JSON response  
D) Cache the responses  

**Answer: B**  
**Explanation:** Tool response size directly impacts context window consumption (CALM-L: Limit context). If Claude uses 3 of 50 fields, 47 fields are wasted context. Solutions: (1) `fields` parameter: `get_customer(id, fields=["name", "balance", "tier"])`. (2) Targeted tools for specific needs. (3) Response summarization in the tool server before returning to Claude. Design tools to return what Claude needs, not everything available.

---

**Q15.** A developer is designing a tool that queries a database. Should they use `"type": "string"` or `"type": "integer"` for a database record ID?

A) Always string — IDs can be alphanumeric  
B) Match the actual ID type in the system: if IDs are integers (auto-increment), use integer; if IDs are UUIDs or slugs, use string with appropriate format; and always document the format in the description with an example  
C) Always integer — more efficient  
D) Depends on the database  

**Answer: B**  
**Explanation:** The schema type should match the system's actual data type. Integer IDs: `"type": "integer", "minimum": 1`. UUID IDs: `"type": "string", "format": "uuid"`. Slug IDs: `"type": "string", "pattern": "^[a-z0-9-]+$"`. Always include an example: "Example: 12345" or "Example: 550e8400-e29b-41d4-a716-446655440000". Type mismatch causes silent failures or type coercion bugs.

---

**Q16.** A tool is described as: "Gets the weather. Takes a location and returns weather data." After deployment, Claude calls it with: city names, country names, zip codes, coordinates, and airport codes. The tool only accepts city names. What is missing?

A) The tool needs to accept all formats  
B) The description must specify the exact format of accepted input: "Gets current weather for a city. `location` must be a city name in English (e.g., 'London', 'New York', 'Tokyo'). Do not use country names, zip codes, or coordinates."  
C) Validate input server-side  
D) Add a location_type parameter  

**Answer: B**  
**Explanation:** Input format must be explicit in the description. "Takes a location" is ambiguous — location can mean anything. Adding: exact format required + examples + explicit exclusions ("do not use X, Y, Z") prevents format confusion. The description is Claude's only guide to parameter values; ambiguity in descriptions produces inconsistent inputs.

---

**Q17.** A developer has two similar tools: `create_draft_email` and `send_email`. They want to prevent Claude from directly calling `send_email` without first calling `create_draft_email`. What is the best architectural enforcement?

A) Document the order in both tool descriptions  
B) `send_email` requires a `draft_id` parameter obtained from `create_draft_email`'s response; the dependency is encoded in the API contract — you cannot call `send_email` without a valid draft_id, so the workflow order is architecturally enforced  
C) Remove `send_email` from the available tools until a draft exists  
D) Both B and C  

**Answer: D**  
**Explanation:** Both B and C are valid and complementary. B (parameter dependency) enforces the workflow via API contract — `send_email(draft_id)` requires the output of `create_draft_email()`. C (dynamic tool availability) is even stronger — if `send_email` isn't available until a draft exists, it literally cannot be called out of order. Combining both eliminates workflow order violations.

---

**Q18.** A tool has the parameter `"filter": {"type": "string", "description": "filter results"}`. How should this be improved?

A) Change to required  
B) Specify the filter format exactly: `"filter": {"type": "object", "properties": {"field": {"type": "string", "enum": ["status", "date", "type"]}, "operator": {"type": "string", "enum": ["equals", "contains", "before", "after"]}, "value": {"type": "string"}}, "description": "Filter results. Example: {'field': 'status', 'operator': 'equals', 'value': 'active'}"}`  
C) Remove the filter parameter  
D) Use a string with a documented format  

**Answer: B**  
**Explanation:** Structured parameters are more reliable than free-form strings. A structured filter object with typed fields and enums: (1) Limits valid field names. (2) Constrains operators to valid options. (3) Makes the filter syntax unambiguous. A free-form filter string forces Claude to guess the syntax and the server to parse potentially malformed inputs.

---

**Q19.** A developer needs to add a `dry_run` parameter to all tools to enable testing. What is the correct schema pattern?

A) Add `"dry_run": {"type": "boolean"}` to every tool  
B) If dry_run is a cross-cutting concern, document it once in the system prompt as a universal parameter; adding it to every tool schema adds noise, and each tool description should clarify whether dry_run affects it: "If dry_run is true, returns preview without executing"  
C) Create a separate tool set for dry runs  
D) Handle dry_run at the MCP server level, not the tool schema level  

**Answer: D**  
**Explanation:** Cross-cutting concerns like dry_run, correlation_id, trace_id are better handled at the MCP server level (via headers/context/configuration) rather than polluting every tool schema. The tool should execute or not execute based on server-level configuration. If it must be in the schema, document it once in the system prompt rather than repeating it in every tool description.

---

**Q20.** A tool description ends with: "Note: This tool is experimental and may not work correctly." What problem does this cause?

A) No problem — transparency is good  
B) Claude will avoid using the tool or add excessive caveats; tool descriptions create expectations; only deploy tools Claude should use confidently; if a tool is not production-ready, don't expose it  
C) "Experimental" should be replaced with "beta"  
D) This helps Claude understand limitations  

**Answer: B**  
**Explanation:** Tool descriptions shape Claude's confidence in using them. "May not work correctly" teaches Claude to be uncertain about the tool — it may avoid it or hedge its results. Only expose tools that Claude should use confidently. If a tool is experimental, handle errors robustly and test it before exposing it, rather than warning Claude about its unreliability.

---

**Q21.** A tool has an optional parameter `"reason": {"type": "string"}` that is logged for audit purposes but not functionally required. Claude rarely includes it. How do you increase Claude's use of this parameter?

A) Make it required  
B) In the description, explain why it's important: "reason: (Recommended) Brief reason for this action, logged for audit trail. Providing a reason helps troubleshoot issues and is required for compliance reviews. Example: 'Customer requested account closure per ticket #12345'"  
C) Remove it if rarely used  
D) Add a default value  

**Answer: B**  
**Explanation:** Claude uses parameters when they clearly serve a purpose. Explaining the audit/compliance value + giving a concrete example increases Claude's inclination to populate it. "Recommended" (not "optional") also shifts Claude's default. Making it required (A) is the nuclear option — if it genuinely can't always be provided, required breaks the tool.

---

**Q22.** A tool for searching products takes `min_price` and `max_price` as separate parameters. What schema constraint ensures `min_price <= max_price`?

A) Add a description saying "min must be less than max"  
B) JSON Schema doesn't directly support cross-field validation; use a description constraint + server-side validation that returns a clear error: "min_price must be <= max_price" — Claude will learn from the error response  
C) Use a range object instead  
D) Both A and B  

**Answer: D**  
**Explanation:** Both A and B are correct and necessary. A (description) sets Claude's expectation. B (server-side validation) catches the error if Claude violates it anyway. C (range object `{min: x, max: y}`) is also a clean alternative that makes the relationship explicit in the schema structure. Description + validation is the minimum; structured range object is the cleanest design.

---

**Q23.** A tool returns all errors as HTTP 500 with the message "An error occurred." Claude cannot distinguish between retry-able errors (network timeout) and non-retry-able errors (invalid input). What tool design change is needed?

A) Log errors more verbosely  
B) Return structured error responses that distinguish error types: `{"error_type": "validation_error", "message": "...", "retryable": false}` vs `{"error_type": "service_unavailable", "message": "...", "retryable": true}`  
C) Always return success and include errors in the data  
D) Increase timeout  

**Answer: B**  
**Explanation:** Structured error responses enable intelligent retry logic. Claude's behavior on errors should differ: (1) `retryable: true` + `validation_error` → don't retry, fix the input. (2) `retryable: true` + `service_unavailable` → retry with backoff. Without structured errors, Claude must guess whether to retry, often causing duplicate operations or giving up too early.

---

**Q24.** A developer defines a tool with 300 words in its description. What is the problem?

A) Long descriptions are always better  
B) Tool descriptions should be concise (3-5 sentences); very long descriptions consume tokens from the context budget for every call and may reduce selection accuracy as key information is buried  
C) 300 words is the recommended maximum  
D) Long descriptions prevent errors  

**Answer: B**  
**Explanation:** Description length affects: (1) Context budget — description tokens are consumed at every inference. (2) Selection accuracy — key trigger conditions buried in 300 words may not be weighted correctly. (3) Maintainability — long descriptions are harder to update and may become inconsistent. Target: concise but complete. "What does it do? When to use it? Key parameters? What does it return?" — answer these in 3-5 sentences.

---

**Q25.** A developer is adding a new tool to a production MCP server. The existing 8 tools are working well. The new tool overlaps in function with tool #3. What should the developer do before adding the new tool?

A) Always add new tools — more options are better  
B) Evaluate whether tool #3 can be extended to handle the new use case; if not, clearly differentiate the new tool's description from tool #3 so Claude knows which to use when; and test that existing tool selection accuracy doesn't degrade after adding the new tool  
C) Remove tool #3 first  
D) Merge all overlapping tools immediately  

**Answer: B**  
**Explanation:** Adding overlapping tools risks degrading the accuracy of existing tool selection. Before adding: (1) Can the existing tool handle this? (2) If new tool is necessary, are descriptions clearly differentiated with explicit "use this when X; use tool #3 when Y" guidance? (3) Test existing tool selection after adding — regression testing is critical for tool libraries.

---

## Score: /25 | Pass: 19/25 (75%)
