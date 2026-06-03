# Practice Questions 12 — CALM Framework & Caching Strategies

> Domain 5 deep-dive: Every CALM letter in depth, ephemeral vs persistent caching, cache ordering, invalidation, when caching doesn't help.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A developer is processing a 500,000-word corporate document library with Claude. The entire library doesn't fit in one context window. Which CALM letter addresses this directly?

A) A — Aggressively cache  
B) C — Chunk: break the document library into topic-based chunks, process each chunk independently, and synthesize results  
C) L — Limit context  
D) M — Manage conversation history  

**Answer: B**  
**Explanation:** CALM-C: Chunk large content that exceeds context limits. Strategies: (1) Topic-based chunking — group related documents. (2) Task-based chunking — process one section per request. (3) Hierarchical chunking — summarize chunks and work with summaries. The goal: each individual request fits in the context window while preserving the ability to work with the full corpus across multiple requests.

---

**Q2.** A customer service application sends the full 8,000-token company policy document with every user request (10,000 requests/day). API costs are high. Which CALM letter offers the most cost reduction?

A) C — Chunk the policy document  
B) A — Aggressively cache: place the policy document in the system prompt and use Anthropic's prompt caching feature; the document is cached after the first request and subsequent requests reuse the cached version at ~90% discount  
C) L — Limit context to reduce the policy  
D) M — Manage conversation history  

**Answer: B**  
**Explanation:** CALM-A: Prompt caching is the solution for repeated static content. The policy document is sent once; Anthropic caches it at a specific cache checkpoint. Subsequent requests that include the same prefix hit the cache. At scale (10,000 requests/day), ~90% cost reduction on that content. The document stays fresh (update the system prompt when policy changes, clearing the cache).

---

**Q3.** A conversational AI has been running for 200 turns. The earliest turns contain questions about topics no longer relevant to the current task. Including all 200 turns consumes most of the context budget. Which CALM strategy is most appropriate?

A) C — Chunk the conversation  
B) M — Manage conversation history: use a sliding window (keep the last N turns), or summarize early turns into a compressed memory, or extract key facts/decisions into a structured summary to replace verbose history  
C) L — Limit context  
D) A — Cache conversation history  

**Answer: B**  
**Explanation:** CALM-M: conversation history management. Strategies for long conversations: (1) Sliding window — keep only recent N turns. (2) Summarization — replace old turns with a compressed summary. (3) Selective preservation — keep turns with key decisions/context, discard small talk. (4) Structured memory — extract facts from conversation into a structured format. Old irrelevant turns waste context that could be used for the current task.

---

**Q4.** A developer places a 50,000-token document cache block AFTER the user's query in the prompt. The caching is configured correctly but doesn't save tokens. Why?

A) The document is too large to cache  
B) Cache breakpoints must be placed AFTER stable content; for caching to work, the cache checkpoint must appear AFTER all content that will be reused, not after dynamic content; placing the document after the query means each query creates a unique prefix, defeating caching  
C) The token count is too low  
D) System prompt caching requires the full 100k minimum  

**Answer: B**  
**Explanation:** CALM-A cache ordering rule: stable content first, dynamic content last. The cache key is the prompt prefix up to the cache checkpoint. If the query (dynamic, changes every request) comes before the document (stable, same every request), every request has a unique prefix → no cache hits. Correct order: system prompt + stable context (with cache checkpoint) → then the user's query.

---

**Q5.** A RAG system retrieves and adds 20,000 tokens of context to every request, but most retrieved documents are only marginally relevant. What CALM letter addresses this?

A) C — Chunk the documents  
B) L — Limit context: aggressively filter retrieved documents to only those that are directly relevant; use reranking or relevance scoring to keep only the most pertinent content; cutting from 20,000 to 5,000 tokens of highly relevant content often outperforms 20,000 tokens of mixed relevance  
C) A — Cache the documents  
D) M — Manage retrieval history  

**Answer: B**  
**Explanation:** CALM-L: Limit context to the highest-value content. More context ≠ better performance. The "needle in a haystack" problem: highly relevant content buried in 20,000 tokens of mixed relevance is harder for Claude to utilize than 5,000 tokens of directly relevant content. RAG quality metric: relevance density (signal/noise ratio), not just recall (quantity retrieved).

---

**Q6.** A system uses ephemeral caching (default Anthropic prompt caching). The system receives 1,000 requests, then experiences 30 minutes of low traffic. After the low-traffic period, the next request is slow again (cache miss). Why?

A) The cache was manually cleared  
B) Ephemeral caches have a TTL (time-to-live); Anthropic's ephemeral cache expires after approximately 5 minutes of non-use; a 30-minute gap guarantees a cache miss; for guaranteed cache hits, use persistent caching (extended cache TTL)  
C) The content changed  
D) Too many tokens were cached  

**Answer: B**  
**Explanation:** Cache TTL is critical to understand. Ephemeral cache: ~5 minute TTL — suitable for sustained high-traffic applications where requests arrive continuously. Persistent cache: longer TTL — suitable for applications with variable traffic or that need guaranteed cache hits regardless of traffic gaps. Match cache type to traffic pattern.

---

**Q7.** A developer is designing context for a complex research task. They have: a 10,000-token research brief (stable), a 5,000-token knowledge base extract (stable for this session), and a 500-token user query (dynamic). What is the optimal CALM-A ordering?

A) User query → Research brief → Knowledge base  
B) Research brief (with cache checkpoint) → Knowledge base extract (with cache checkpoint) → User query  
C) Knowledge base → Research brief → User query  
D) Interleave all three  

**Answer: B**  
**Explanation:** CALM-A ordering: most stable content first, with cache checkpoints after each stable section. The research brief is most stable (same for all queries on this task) → checkpoint. Knowledge base may vary per session → checkpoint. User query is dynamic → goes last, no cache needed. Each cache checkpoint caches everything before it. The user query, being last, only adds its own tokens at inference time.

---

**Q8.** A code review tool has: a 2,000-token code style guide (same for all reviews), a 500-token file being reviewed (changes per request), and a 200-token review checklist (same for all reviews). For optimal caching, where should the cache checkpoint be placed?

A) After the file being reviewed  
B) Code style guide + review checklist (both stable) → cache checkpoint → file being reviewed; group all stable content before the checkpoint; the checkpoint caches everything before it; the file is dynamic so it goes after the checkpoint  
C) Separate cache for each component  
D) Cache only the style guide  

**Answer: B**  
**Explanation:** CALM-A: group all stable content before a single cache checkpoint. Even though style guide and checklist are separate, both are stable → place both before the checkpoint → cache both. The file being reviewed changes per request → place after checkpoint. This caches 2,200 tokens per request instead of only 2,000 (if only the guide was cached). Always group stable content.

---

**Q9.** An application processes customer feedback. Each feedback item is new and unique (no repeated content). The developer enables prompt caching. What happens to costs?

A) Costs decrease significantly  
B) Costs may not decrease at all; caching requires repeated identical content to hit the cache; unique content per request means every request is a cache miss; CALM-A only helps when there is genuinely repeated content  
C) Costs decrease by 50%  
D) Caching causes errors with unique content  

**Answer: B**  
**Explanation:** Caching is not universally beneficial. It reduces costs only when the same content appears across multiple requests. Unique customer feedback = no cache hits = no savings (actually small overhead from cache management). CALM-A is effective for: system prompts, knowledge bases, policy documents — any content that appears in many requests. Don't enable caching for purely unique content expecting savings.

---

**Q10.** A developer has a 200,000-token knowledge base. They want to cache the entire knowledge base. What is the correct chunking and caching strategy?

A) Cache the entire 200,000 tokens as a single block  
B) Chunk the knowledge base into topic domains, cache the most frequently accessed domains in the system prompt; use RAG to retrieve only relevant chunks per request rather than including all 200,000 tokens per request  
C) This is impossible to cache  
D) Increase context window size  

**Answer: B**  
**Explanation:** CALM-C + CALM-A together: 200,000 tokens exceeds context window limits. Chunk by topic domain. Cache the most frequently needed domains in the system prompt (they'll hit cache repeatedly). Use RAG to retrieve relevant chunks per request. The full knowledge base never enters context at once — only the relevant portions do. This is the canonical pattern for large knowledge bases.

---

**Q11.** A multi-user application uses the same system prompt for all users but adds user-specific context (name, preferences, history) before the user query. Where should cache checkpoints be placed?

A) After user-specific context  
B) System prompt (with cache checkpoint) → user-specific context (no checkpoint, unique per user) → user query; the system prompt is shared across all users → cache it; user-specific context is unique per user → don't cache  
C) Nowhere — multi-user apps can't use caching  
D) After the user query  

**Answer: B**  
**Explanation:** Cache what is shared, skip caching what is unique. System prompt: same for all users → cache saves tokens for every request across all users. User-specific context: unique per user → no cache benefit per individual request but the system prompt cache still saves the shared tokens. The cache checkpoint should be placed at the stable/dynamic boundary.

---

**Q12.** A developer implements conversation summarization (CALM-M) for a long-running support conversation. When the conversation exceeds 50 turns, the first 40 turns are summarized into 3 sentences. What risk exists with aggressive summarization?

A) No risk — summaries always work  
B) Critical context loss: early turns may contain key decisions, error descriptions, or constraints that don't survive condensed summarization; mitigation: extract structured facts before summarization, preserve "sticky" items (key decisions, constraints) explicitly  
C) Summaries are too expensive  
D) Only the last 10 turns matter anyway  

**Answer: B**  
**Explanation:** CALM-M summarization risk: information loss. Critical conversation elements that must survive summarization: (1) Key decisions made ("customer agreed to the enterprise plan"). (2) Constraints established ("customer cannot change their username"). (3) Errors encountered. (4) Verified facts. Strategy: before summarizing, extract structured facts into a persistent memory block. The summary covers general context; the structured facts preserve critical specifics.

---

**Q13.** A token budget analysis shows that tool results are consuming 60% of the context window in an agentic pipeline. Which CALM letter addresses this and what is the fix?

A) A — cache tool results  
B) L — Limit context: tool results should return only the data Claude needs, not entire API responses; implement field filtering in tools, paginate large results, and summarize verbose tool outputs before adding to context  
C) C — chunk tool calls  
D) M — manage tool call history  

**Answer: B**  
**Explanation:** CALM-L for tool results: (1) Field filtering: tool returns only the fields Claude will use. (2) Pagination: don't return 1,000 records when 10 are needed. (3) Summarization: for verbose outputs, have the tool or a post-processing step summarize before adding to context. Tool results can be the largest context consumers in agentic pipelines — explicit size management is required.

---

**Q14.** An application caches a medical protocol document (3,000 tokens) in the system prompt. The protocol is updated by the clinical team at irregular intervals (sometimes daily, sometimes monthly). What cache management approach is correct?

A) Set a fixed 24-hour cache expiry  
B) Implement a content hash (MD5/SHA) of the document; when the hash changes, update the system prompt and the cache automatically invalidates; this ensures Claude always uses the current protocol regardless of update frequency  
C) Manually clear the cache after each update  
D) Use a weekly cache refresh schedule  

**Answer: B**  
**Explanation:** Content-hash-based cache invalidation: (1) Hash the document content. (2) Store the hash alongside the cached content. (3) On each system startup or on a check interval, compare current document hash to cached hash. (4) If different, update system prompt → automatic cache invalidation. This handles irregular update frequencies correctly without over-invalidating (schedule-based) or under-invalidating (manual process dependency).

---

**Q15.** A developer adds tool call results to the conversation history verbatim. After 10 tool calls, the conversation history with tool results exceeds the context window. What CALM strategy applies?

A) Increase the context window  
B) CALM-M: extract the key information from tool results and convert to compact conversational context: "The database returned 47 matching records; top 3 by relevance are: [summarized list]" replaces the full JSON tool output  
C) CALM-C: chunk the tool results  
D) CALM-A: cache tool results  

**Answer: B**  
**Explanation:** CALM-M for tool result history: don't store raw tool responses in conversation history. Extract and compress: raw JSON (2,000 tokens) → key facts (100 tokens). The conversation history should be a human-readable narrative of what happened, not an archive of raw API responses. Claude only needs the meaningful outcomes, not the full verbose tool responses, to continue the task.

---

**Q16.** A developer tests whether prompt caching is working. What is the best way to verify a cache hit?

A) The response is faster  
B) Check the API response for `cache_read_input_tokens` in the usage object; a cache hit populates this field with the number of tokens read from cache; also check that `input_tokens` is lower than expected (reflecting that cached tokens don't count as full input)  
C) Use a rate limit error as a proxy  
D) Monitor response latency  

**Answer: B**  
**Explanation:** Anthropic's API returns usage stats including `cache_read_input_tokens` on cache hits and `cache_creation_input_tokens` on cache misses (first creation). These are the definitive indicators of cache behavior. Response speed (A) is affected by many factors. The usage stats are the reliable, precise measure of cache performance.

---

**Q17.** A CALM-L analysis shows that a system prompt is 8,000 tokens but only 1,200 tokens are actually used to influence Claude's behavior. The rest is background context that Claude rarely references. What should the developer do?

A) Keep it — more context is always better  
B) Restructure: move rarely-referenced background to retrieval (RAG) — include it only when the query is relevant to that content; keep the system prompt to the content Claude actually needs for every request  
C) Split into two system prompts  
D) Cache the unused portion  

**Answer: B**  
**Explanation:** CALM-L: the context window should contain high-signal content, not information that rarely influences responses. Unused context is wasted tokens that: (1) Consume cost. (2) Dilute the signal of relevant content. (3) Reduce performance on the relevant portions. Move rarely-used content to RAG where it's retrieved on-demand. The system prompt should be the consistently essential content.

---

**Q18.** An organization's Claude deployment handles: (1) 90% routine queries against a fixed knowledge base, (2) 10% complex queries requiring different context. How should caching be structured?

A) Single system prompt for everything  
B) Two cache tiers: (1) Common baseline (shared knowledge base) cached for all requests. (2) Task-specific context added dynamically for the 10% complex cases; the 90% routine queries hit the full cache; the 10% complex queries hit the baseline cache + add dynamic context  
C) No caching — variety prevents cache hits  
D) Cache only for the 90% queries  

**Answer: B**  
**Explanation:** Tiered caching: layer 1 (always cached, 100% of requests) + layer 2 (dynamic, added as needed). This maximizes cache ROI: every request benefits from the baseline cache hit. Complex requests pay for their dynamic additions but still save on the shared baseline. This is the optimal CALM-A strategy for mixed workload applications.

---

**Q19.** A developer uses CALM-C to process a 200-page legal document by splitting it into 20-page chunks. After processing each chunk independently, the summarizer needs information from chunk 1 to properly contextualize information in chunk 15. What problem is this and what is the fix?

A) Process the document without chunking  
B) Cross-chunk context dependency: fix with a two-pass approach — (1) First pass: process each chunk and extract key entities, definitions, and facts into a running context object. (2) Second pass: process each chunk with the accumulated context object prepended.  
C) Use a larger context window  
D) Process chunks sequentially, feeding each output as input to the next  

**Answer: B**  
**Explanation:** Cross-chunk dependencies are a chunking design problem. The two-pass approach: pass 1 builds a "knowledge accumulator" (key terms defined in the document, parties, defined terms, section cross-references). Pass 2 processes each chunk with this accumulated context. For legal documents: defined terms in §1 affect interpretation of §20 — the accumulated context carries these definitions across chunks.

---

**Q20.** A developer is allocating the token budget for a complex analysis task. The breakdown is: system prompt (3,000 tokens), knowledge base (7,000 tokens), conversation history (10,000 tokens), user query (1,000 tokens). The model has 100,000-token context. The analysis requires significant output. How should the budget be thought about?

A) Only input matters — output doesn't have a budget  
B) Total context = input + output; with 21,000 input tokens used, 79,000 tokens remain; allocate part of this for output (max_tokens parameter); if the analysis needs 15,000-token output, Claude has 64,000 tokens available — the budget is healthy; if output requirements are 60,000 tokens, the input must be reduced  
C) Output tokens don't consume context  
D) Maximum input tokens = maximum context window  

**Answer: B**  
**Explanation:** Context window = input tokens + output tokens. With 100k context: if input is 21k, the maximum output is 79k (before hitting the window limit). Set max_tokens in the API call to reserve the desired output space. If task requires large output, reduce input accordingly. Understanding the full input+output budget is essential for complex generation tasks.

---

**Q21.** When should a developer use a sliding window (keep last N turns) vs. summarization (condense old turns) for CALM-M conversation management?

A) Sliding window always — simpler  
B) Sliding window: when recent turns are all that matter and old context is genuinely obsolete (task-focused short conversations). Summarization: when early turns contain important context that must be preserved in compressed form (relationship history, key decisions, established facts)  
C) Summarization always — preserves more information  
D) Depends on API cost  

**Answer: B**  
**Explanation:** Choose by conversation structure: (1) Sliding window is simple and correct when the task doesn't depend on early conversation (coding sessions, short Q&A, task-focused workflows). (2) Summarization is correct when context accumulates over time and early information remains relevant (customer relationship management, long research projects, ongoing negotiations). Match the strategy to the temporal dependency structure.

---

**Q22.** A CALM-A cache has been working for 3 months. The system prompt is updated to add a new product category. What happens to cached content?

A) The old cache continues to be used  
B) Updating the system prompt changes the content that is cached; the previous cache is invalidated; the first request after the update experiences a cache miss (paying the full input token cost); subsequent requests build a new cache from the updated content  
C) The cache is never invalidated  
D) The developer must manually clear the cache  

**Answer: B**  
**Explanation:** Cache invalidation is automatic when content changes. The cache key is the content itself (or a prefix hash). Changed content → new cache key → cache miss → new cache created. This is the correct behavior — you always get fresh content after an update. The first post-update request pays the full price; subsequent requests benefit from the new cache. No manual clearing needed.

---

**Q23.** A developer measures context window usage: 50% system prompt, 30% retrieved documents, 15% conversation history, 5% user query. The system is running at 95% context utilization. What CALM strategy addresses the largest consumer first?

A) Reduce the user query  
B) CALM-L targeting the system prompt (50% of context): audit the system prompt for unused instructions, redundant examples, and verbose phrasing; also apply CALM-A caching to make the system prompt cost-efficient; reducing the largest consumer has the most impact  
C) Stop caching retrieved documents  
D) Summarize the user query  

**Answer: B**  
**Explanation:** Optimization principle: tackle the largest consumer first. 50% system prompt is the target. Actions: (1) Audit for redundancy — remove instructions that duplicate other instructions. (2) Remove unused sections (per CALM-L analysis). (3) Tighten phrasing (same meaning, fewer tokens). (4) Apply CALM-A to cache the remaining optimized system prompt. This has the most impact before touching the 30% retrieved documents.

---

**Q24.** An agentic pipeline has 10 sequential tool calls. After each call, the full tool result is added to context. By tool call 8, context overflow occurs. What is the CALM architecture fix?

A) Increase the context window  
B) CALM-M for agentic pipelines: after each tool call, extract only the decision-relevant information into a structured state object; discard the full tool response from context; replace with the extracted state; this keeps context growth linear and bounded  
C) Reduce tool calls to 5  
D) CALM-C: chunk the tool calls into batches  

**Answer: B**  
**Explanation:** Agentic context management: instead of accumulating raw tool results, maintain a structured state: `{step: 3, items_found: 47, selected_item: "abc123", next_action: "verify_inventory"}`. This replaces 2,000-token raw results with 50-token structured state. The pipeline can run indefinitely with bounded context consumption. This is the canonical pattern for long agentic pipelines.

---

**Q25.** A developer applies all 4 CALM strategies to an existing application. Which order of implementation delivers the most value fastest?

A) C → A → L → M  
B) A (Aggressively cache) → L (Limit context) → M (Manage history) → C (Chunk) — A delivers immediate cost reduction on repeated content; L identifies and removes waste; M addresses growing conversation overhead; C addresses fundamentally size-limited content  
C) L → A → M → C  
D) M → C → L → A  

**Answer: B**  
**Explanation:** Implementation priority by impact: (1) A (Caching) — immediate ROI on API costs, zero functional change needed. (2) L (Limit) — remove waste, improve signal density, cost reduction. (3) M (Manage history) — addresses growing conversation contexts in production. (4) C (Chunking) — required for fundamentally large content but most architecturally complex. Start with quick wins (A, L) before architectural changes (C).

---

## Score: /25 | Pass: 19/25 (75%)
