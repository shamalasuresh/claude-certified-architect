# Practice Questions 18 — Scale, Cost & Latency Architecture Decisions

> High-volume systems, cost optimization, latency SLA tradeoffs, caching ROI, architectural patterns for scale.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A system processes 1 million Claude API requests per day. The average system prompt is 3,000 tokens. Without caching, the daily input token cost for system prompts alone is 3 billion tokens. With prompt caching enabled, the cached tokens cost ~10% of standard rate. What is the approximate daily savings?

A) 10% savings  
B) ~2.7 billion tokens saved per day (90% of the 3 billion system prompt tokens are cached rather than re-processed); at scale, prompt caching on a shared system prompt is the single highest ROI optimization available  
C) 50% savings  
D) No savings — each request has a unique prefix  

**Answer: B**  
**Explanation:** At 1M requests/day with a 3,000-token system prompt: without caching = 3B tokens/day for system prompt alone. With caching: ~300M tokens (10% of standard rate for cached reads) + 3M tokens for first-time cache creation. Net: ~2.7B tokens/day saved. At $3/MTok (Sonnet), that's ~$8,100/day saved. At this scale, prompt caching pays for architectural investment within days.

---

**Q2.** A high-volume classification service needs to classify 100,000 customer support tickets per hour. Target latency: < 2 seconds per ticket. Each classification uses Claude Haiku (fastest, cheapest). Estimated single-request latency: 0.5-1.5s. What architecture pattern handles 100k requests/hour?

A) Sequential processing  
B) Parallel async processing with a request pool: fan-out requests asynchronously (100+ concurrent), use connection pooling, implement rate limit handling with exponential backoff; at 100k/hour ≈ 28 requests/second, async parallel processing with ~50-100 concurrent connections handles this comfortably  
C) Use Opus for better accuracy  
D) Queue all requests and process in daily batches  

**Answer: B**  
**Explanation:** Throughput math: 100k requests/hour = 28 req/s. With 1s average latency: need ~28 concurrent connections to sustain throughput. Async parallel processing with a connection pool of 50-100 handles this with headroom. Key patterns: (1) Async request pool. (2) Rate limit awareness + backoff. (3) Circuit breakers for API unavailability. (4) Dead letter queue for failed requests. Haiku is the correct model choice for high-volume, latency-sensitive, classification.

---

**Q3.** A developer observes that Claude API response times vary significantly: P50 = 800ms, P95 = 3.2s, P99 = 8.5s. For a user-facing application with a 2-second UX expectation, what does this distribution mean?

A) P50 is fine — most users will be happy  
B) 5% of requests (P95) exceed 3.2 seconds — significant tail latency that creates poor UX for 1 in 20 users; strategies: streaming responses (user sees output immediately), timeout + fallback for requests exceeding 2.5s, skeleton UI while waiting  
C) The API is unreliable and should not be used  
D) Set a 10-second timeout and accept the variance  

**Answer: B**  
**Explanation:** Tail latency UX impact: P95 and P99 are what determines UX quality for heavy users. A user who makes 20 requests will likely hit P95 once. Strategies for tail latency: (1) Streaming: start showing output immediately, so perceived latency = time-to-first-token (usually 200-400ms), not total response time. (2) Optimistic UI: show skeleton/loading state. (3) Timeout with graceful degradation. P50 optimization is insufficient for production UX.

---

**Q4.** A company is evaluating: Claude Haiku at $0.25/MTok vs. Claude Sonnet at $3/MTok (12x cost difference) for a summarization task. In testing, Haiku achieves 78% acceptable quality. Sonnet achieves 94% acceptable quality. The application processes 500,000 summaries/month with an average of 1,000 tokens each. What is the cost-quality analysis?

A) Always use the cheapest model  
B) Calculate: Haiku cost = 500M tokens × $0.25/MTok = $125/month with 22% failure rate (requiring human review or re-run). Sonnet cost = $1,500/month with 6% failure rate. If each failure costs >$2.27 in human review + business impact, Sonnet pays for itself. Calculate the cost of the 16% quality gap.  
C) Always use the most accurate model  
D) Split 50/50 between models  

**Answer: B**  
**Explanation:** ROI analysis for model selection: don't just compare model costs — compare total system costs including failure handling. Haiku: $125/month + cost of 110,000 failures/month (22% × 500k). Sonnet: $1,500/month + cost of 30,000 failures/month (6% × 500k). If failures require $0.01 in rework: Haiku total ≈ $1,225, Sonnet total ≈ $1,800. If failures require $1 in rework: Haiku total ≈ $110,125, Sonnet total ≈ $31,500 — Sonnet wins by a factor of 3.

---

**Q5.** A system uses Claude Sonnet for all requests. A cost analysis reveals: 60% of requests are simple FAQ lookups, 25% are moderate reasoning tasks, 15% are complex analysis. What model routing architecture reduces cost while maintaining quality?

A) Use Haiku for everything  
B) Tiered routing: classify request complexity → route simple (60%) to Haiku, moderate (25%) to Sonnet, complex (15%) to Sonnet or Opus; this maintains quality where needed while cutting cost significantly on the 60% simple requests  
C) Use a single model for consistency  
D) Use extended thinking for all requests  

**Answer: B**  
**Explanation:** Model tiering math: if current cost = 1M requests × Sonnet rate = X. After tiering: (600k × Haiku rate) + (250k × Sonnet rate) + (150k × Sonnet rate) = (600k × 0.083X) + (400k × X) ≈ 450k × X ≈ 45% of original cost. Quality maintained: complex tasks still get Sonnet; FAQ quality on Haiku is typically fine. The 60% of simple requests routed to Haiku provides the largest savings.

---

**Q6.** An API gateway in front of Claude receives 1,000 identical requests per second for the same information (a breaking news event creating a spike in the same query). What architectural pattern prevents this from 1,000 Claude API calls per second?

A) Queue all requests — process 1 at a time  
B) Response caching at the API gateway: identical queries get the cached Claude response until the cache TTL expires (e.g., 60 seconds); 1,000 requests/second → 1 Claude call per 60 seconds; for high-traffic identical queries, response caching is more effective than prompt caching  
C) Increase Claude API rate limits  
D) Reject duplicate requests  

**Answer: B**  
**Explanation:** Response-level caching vs. prompt caching: prompt caching saves token cost per request but still makes one API call per request. Response caching (at the application layer or API gateway) serves many requests from a single Claude call. For a spike of identical queries: cache the response for 60 seconds → 1 Claude call serves 60,000 users (at 1,000/s). Cache key = normalized query hash. TTL = appropriate freshness window for the content type.

---

**Q7.** A streaming response system uses Claude's streaming API. Users report that responses start quickly but occasionally stop mid-sentence and complete after a 3-second pause. What infrastructure issue causes this?

A) The model is slow  
B) Network backpressure or connection interruption in the streaming pipeline: one of the hops between Claude's API and the user is buffering (not forwarding tokens as they arrive); check: proxy/load balancer streaming configuration, TCP buffer settings, application server streaming support; streaming requires every hop to be configured for pass-through, not buffering  
C) Claude's streaming is unreliable  
D) The response is too long  

**Answer: B**  
**Explanation:** Streaming pauses are typically infrastructure issues, not model issues. Common culprits: (1) Nginx/load balancer not configured for `proxy_buffering off`. (2) Application server collecting chunks before forwarding. (3) CDN/WAF buffering full responses. Each network hop must be configured for streaming pass-through. Test by calling Claude's API directly — if streaming is smooth direct but pauses through the stack, the issue is in the infrastructure.

---

**Q8.** A batch processing system uses Claude to analyze 50,000 documents overnight. Documents average 2,000 tokens each. Using standard API rate, estimated completion time is 22 hours (exceeds the overnight window). What options exist?

A) Only option is to wait  
B) Options: (1) Anthropic Batch API (if available): 50% cost reduction, designed for async batch, processes in parallel with higher throughput. (2) Multiple API keys with different rate limit tiers. (3) Reduce per-document context: CALM-L to reduce 2,000-token documents to 1,000-token extracts. (4) Pre-filter: only analyze documents that need AI analysis (maybe 30% of 50k actually need it).  
C) Use a smaller model  
D) Split into 2-night batches  

**Answer: B**  
**Explanation:** Batch processing optimization: (1) Batch API: designed for exactly this — 50% cheaper, higher throughput for non-real-time workloads. (2) Context reduction: smaller inputs = faster processing + lower cost. (3) Pre-filtering: if 70% of documents are routine (can be handled by rules), only 15,000 need AI → 6-hour window. (4) Parallelism: multiple concurrent requests (respecting rate limits) reduces wall clock time.

---

**Q9.** A real-time chat application targets 200ms time-to-first-token. The current TTFT is 350ms. The system prompt is 4,000 tokens. What optimizations reduce TTFT?

A) Use a smaller context window  
B) Multiple optimizations: (1) Reduce system prompt size (shorter prompt = faster processing start). (2) Use a geographically closer API endpoint. (3) Switch to Haiku (faster TTFT than Sonnet). (4) Enable streaming (user perceives faster response). (5) Warm connections (keep connections alive rather than re-establishing per request).  
C) Increase max_tokens  
D) Use batch processing  

**Answer: B**  
**Explanation:** TTFT optimization levers: (1) Model selection: Haiku has lower TTFT than Sonnet. (2) Prompt size: shorter prompts start generating faster. (3) Network: closer API endpoint reduces round-trip latency. (4) Streaming: user sees tokens as they're generated — perceived TTFT = time-to-first-visible-character. (5) Connection management: HTTP/2 multiplexing, keep-alive connections. Streaming is often the highest-impact UX improvement even without changing TTFT.

---

**Q10.** A company's Claude costs grew 400% month-over-month as they scaled. Cost analysis reveals 80% of cost is input tokens, mostly from long conversation histories. Which CALM strategy has the highest cost ROI?

A) Switch to a cheaper model  
B) CALM-M (Manage conversation history): the 80% input token cost is dominated by growing conversation history; implementing sliding window + summarization can reduce average conversation history from 15,000 tokens to 2,000 tokens — a 7x reduction in the largest cost driver  
C) CALM-A (Cache) — most cost-effective  
D) CALM-C (Chunk) — reduce processing size  

**Answer: B**  
**Explanation:** Cost ROI analysis: the 80% cost driver is conversation history. Solving the biggest cost driver first: if average history is 15,000 tokens and can be reduced to 2,000 tokens via CALM-M, input cost drops ~73% (from 80% to ~22% of total input tokens). This single optimization outperforms caching (CALM-A would help if content was repeated, but each conversation is unique). Target the biggest cost driver first.

---

**Q11.** A developer is evaluating: deploy Claude API integration on regional cloud instances (closer to users) vs. centralized deployment (single region). Latency difference: 80ms vs. 200ms to Claude's API endpoint. Application SLA: 500ms total. What factors determine the architecture?

A) Always deploy closer for latency  
B) Multi-factor evaluation: (1) With 500ms SLA and 200ms API latency, application processing has 300ms remaining — sufficient for most use cases. (2) Multi-region adds: complexity, cost, data residency compliance requirements, configuration drift risk. (3) Only deploy multi-region if: latency SLA is tight (< 150ms) OR data residency requires geographic processing.  
C) Always centralize for simplicity  
D) Add 80ms of artificial delay to the central deployment  

**Answer: B**  
**Explanation:** Multi-region deployment tradeoffs: 80ms saved vs. significant operational complexity. With 500ms SLA and 200ms Claude latency, there's 300ms for application logic — ample headroom. Multi-region is worth the cost when: (1) Latency SLA cannot be met centrally. (2) Data residency regulations require in-region processing. (3) Availability requirements justify redundancy. Don't add operational complexity for 80ms savings when the SLA has 300ms headroom.

---

**Q12.** A developer notices Claude API costs are 40% higher than expected. The cost breakdown shows: input tokens 60%, output tokens 40%. Output tokens are priced at 5x input tokens. What optimization has the most impact?

A) Reduce input tokens  
B) Reduce output tokens: despite being 40% of token count, output tokens cost 5x more per token — if the 40% output share is generating 67% of cost, reducing output (max_tokens, concise output instructions) has 67% of the cost reduction opportunity vs. 33% from reducing input  
C) Both are equally important  
D) Switch to a cheaper model  

**Answer: B**  
**Explanation:** Output token cost dominates at 5x rate. Math: if 60% are input at rate X, and 40% are output at rate 5X: cost = (0.6 × X) + (0.4 × 5X) = 0.6X + 2X = 2.6X. Output tokens (40% of count) generate 2X/2.6X = 77% of cost. Reducing output by 30% saves 23% of total cost. Reducing input by 30% saves only 6.9% of total cost. Target output token reduction for highest ROI.

---

**Q13.** A system needs to process user-uploaded PDFs of varying sizes (1 page to 500 pages). Users submit 1,000 PDFs per day. What context management architecture handles this variability?

A) Set a fixed 50-page limit  
B) Dynamic context allocation by document size: (1) Short docs (< 20 pages): process in full. (2) Medium docs (20-100 pages): extract key sections + table of contents + process relevant sections per query. (3) Long docs (100+ pages): hierarchical summarization (summarize each section, then query against summaries + retrieve full sections on demand). (4) Always estimate token count before sending.  
C) Reject documents over 50 pages  
D) Convert all documents to fixed 1,000-token summaries  

**Answer: B**  
**Explanation:** Tiered document handling: one approach cannot handle 1-500 page variability efficiently. Tier by size: small = full context; medium = smart extraction; large = hierarchical indexing with on-demand retrieval. Token estimation before sending prevents context overflow surprises. The architecture scales from single-page forms to 500-page contracts without manual intervention.

---

**Q14.** A cost monitoring alert fires: cost per request increased 3x over 2 weeks with no code changes. What is the most likely cause?

A) The API pricing changed  
B) Context growth: most likely average conversation history or retrieved context has grown significantly over the 2 weeks as users engage in longer sessions; check: average input token count per request over time — if it grew 3x, that's the cause; implement CALM-L and CALM-M to bound context growth  
C) The model was upgraded  
D) Rate limiting increased costs  

**Answer: B**  
**Explanation:** Context creep is the #1 cause of unexplained cost increases in deployed systems. As users have longer conversations, context grows. After 2 weeks of usage, average session length may have grown from 3 turns to 15 turns. Monitoring requirement: track average input tokens per request over time (not just total cost). When input tokens grow, context management policies need tightening. Cost monitoring without token-count monitoring is insufficient.

---

**Q15.** A developer wants to implement a cost cap per user per day to prevent bill shock from abuse or runaway agentic tasks. What architecture implements this?

A) Trust users to be responsible  
B) Token bucket or counter pattern: (1) Track cumulative token usage per user per day in a cache/database. (2) Before each request, check current usage against the daily cap. (3) If cap reached: return a clear error ("Daily limit reached") rather than processing. (4) Implement alerts at 80% cap utilization. (5) Admin override for legitimate heavy users.  
C) Set a global rate limit, not per-user  
D) Only implement monthly caps  

**Answer: B**  
**Explanation:** Per-user cost caps are an operational safety requirement for any production Claude deployment. Without them: a single user in an agentic loop, a compromised account, or an abusive user can generate thousands of dollars in API costs. Implementation: Redis counter with daily TTL, pre-request check, clear error messaging, monitoring. The 80% alert allows investigation before the cap is hit.

---

**Q16.** A developer compares: (A) make 3 sequential Claude calls to refine an answer, or (B) make 1 call with a more complex prompt. Option A: 3 × 1s latency = 3s, higher quality. Option B: 1 × 1.5s latency = 1.5s, moderate quality. When is Option A preferred despite higher latency?

A) Never — minimize API calls  
B) Option A when: task quality has high business value (accuracy matters more than 1.5s difference), users can see progress (streaming partial results from each step), the task is async (human not waiting in real-time), or the quality difference is significant enough to affect key business metrics  
C) Always — more calls = better  
D) Option A only when cost doesn't matter  

**Answer: B**  
**Explanation:** Sequential vs. single call tradeoffs: (1) Synchronous user-facing: Option B preferred (1.5s is much better UX than 3s for real-time chat). (2) Background/async processing: Option A preferred (quality over speed, user isn't waiting). (3) High-stakes decisions: Option A preferred (3s is acceptable for medical/legal/financial accuracy). (4) Can show progress: Option A with streaming each step creates good UX even at 3s total. Context determines the correct choice.

---

**Q17.** A multi-tenant SaaS product uses a shared Claude integration. Tenant A is a large enterprise that generates 80% of all API calls. During peak hours, all other tenants experience high latency because Tenant A consumes most of the API rate limit. What architecture fixes this?

A) Increase rate limits  
B) Tenant-level rate limiting and quota management: (1) Per-tenant rate limits (not just global limits). (2) Priority queuing: distribute rate limit across tenants equitably or by tier. (3) Tenant A gets a dedicated rate limit bucket separate from shared pool. (4) SLA-based queuing: premium tenants get priority processing.  
C) Tell Tenant A to reduce usage  
D) Deploy a separate Claude integration for Tenant A  

**Answer: B**  
**Explanation:** Multi-tenant fairness requires tenant-level resource management. Global rate limits with no per-tenant controls create the "noisy neighbor" problem. Solutions: (1) Per-tenant rate buckets with independent limits. (2) Priority queuing by tenant tier. (3) Enterprise tenants may warrant dedicated API keys/endpoints. (4) Rate limit dashboards so tenants can monitor their usage. This is standard multi-tenant infrastructure design applied to AI API consumption.

---

**Q18.** A system has monthly API costs of $50,000. A consultant proposes switching from Sonnet to Haiku everywhere, saving 90% (to $5,000/month). The developer is concerned about quality. What evaluation process determines if this is viable?

A) Accept the proposal — cost savings are obvious  
B) Systematic quality evaluation: (1) Run both models on a representative sample of production requests (500-1,000). (2) Define quality metrics (accuracy, usefulness, formatting compliance). (3) Measure performance gap. (4) Calculate: cost of quality gap (customer churn, increased support tickets, human review costs). (5) Only switch if total system cost with Haiku is lower than total system cost with Sonnet.  
C) Reject — quality must be maintained  
D) Use Haiku for half of all requests  

**Answer: B**  
**Explanation:** Model downgrade evaluation requires total cost analysis, not just API cost comparison. The $45,000/month savings must be weighed against: (1) Quality degradation cost (customer satisfaction, conversion, churn). (2) Increased human review needs for lower-quality outputs. (3) Re-engineering costs if quality is unacceptable. A model that saves $45k/month but causes $100k/month in customer churn is not a good trade.

---

**Q19.** A developer finds that 30% of API calls fail due to rate limiting. They implement simple exponential backoff. Failures drop to 5% but latency increases significantly during peak hours. What additional optimization helps?

A) Increase retry attempts  
B) Request smoothing + queue management: (1) Instead of retry-on-failure, implement token bucket or leaky bucket to smooth requests over time (proactive rate management vs. reactive backoff). (2) Priority queue: delay low-priority requests during peak hours, maintain SLA for high-priority requests. (3) Circuit breaker: detect rate limit exhaustion early and queue rather than fail-and-retry.  
C) Increase API tier  
D) Reduce request concurrency  

**Answer: B**  
**Explanation:** Reactive backoff (retry on failure) is less efficient than proactive rate management. Proactive smoothing: instead of sending 100 requests/second and hitting rate limits, smooth to 80 requests/second using a token bucket — no rate limit failures, lower latency. Priority queueing ensures important requests aren't delayed by low-priority ones during peak. The shift from reactive to proactive rate management is the key architectural improvement.

---

**Q20.** A developer is building a cost dashboard for a Claude deployment. What metrics provide the most operational insight?

A) Just total monthly cost  
B) Essential metrics: (1) Cost per request (by route/endpoint). (2) Input token distribution (histogram). (3) Output token distribution. (4) Cache hit rate and tokens saved. (5) Cost by user/tenant. (6) Cost over time (detect anomalies). (7) P50/P95/P99 latency. (8) Error rate by error type. (9) Model distribution (which models used how much). (10) Cost per outcome unit (cost per resolved ticket, cost per summary generated).  
C) Total tokens per day  
D) Model version distribution  

**Answer: B**  
**Explanation:** Cost observability requires multiple dimensions: (1) Cost/request reveals expensive routes for optimization. (2) Token histograms reveal context growth trends. (3) Cache hit rate reveals caching effectiveness. (4) Per-tenant cost enables chargeback and fairness. (5) Time-series detects runaway costs early. (6) Error rates reveal reliability issues. (7) Cost per business outcome links AI cost to business value. A single aggregate metric is insufficient for operational management.

---

**Q21.** A developer wants to understand the ROI of upgrading from HTTP/1.1 to HTTP/2 for their Claude API integration. What specific benefit does HTTP/2 provide for high-volume Claude API usage?

A) HTTP/2 is faster for all requests  
B) HTTP/2 multiplexing: multiple concurrent Claude API requests over a single connection, eliminating the per-connection overhead (TCP handshake, TLS negotiation) of HTTP/1.1; at high concurrency (50+ simultaneous requests), HTTP/2 reduces connection overhead and eliminates head-of-line blocking  
C) HTTP/2 reduces token costs  
D) HTTP/2 is required for streaming  

**Answer: B**  
**Explanation:** HTTP/2 for high-concurrency Claude API: (1) Multiplexing: N requests per connection (vs. 1 per connection in HTTP/1.1). (2) Header compression: reduces per-request overhead. (3) No head-of-line blocking at the HTTP layer. (4) Fewer TCP connections = fewer TLS handshakes. At high concurrency (50-200 concurrent requests), HTTP/2 reduces connection overhead significantly. Most modern Claude SDK clients use HTTP/2 by default.

---

**Q22.** A production system processes sensitive user data through Claude. Compliance requires that no user data leaves the company's cloud region. How does this constraint affect the architecture?

A) This constraint cannot be met with Claude API  
B) Data residency compliance: (1) Check Anthropic's available API regions (e.g., US, EU). (2) Configure the API client to use the compliant region endpoint. (3) Ensure network routing doesn't traverse non-compliant regions. (4) Data residency requires both API endpoint selection AND network path validation. (5) For strict compliance: use zero-data-retention API (no input data retained for training).  
C) Encrypt all data before sending  
D) Data residency only applies to storage, not API calls  

**Answer: B**  
**Explanation:** Data residency for AI APIs: (1) API endpoint determines where data is processed. (2) Zero-data-retention prevents persistence. (3) Network path validation (VPC endpoints, private links) ensures data doesn't traverse non-compliant regions. (4) Contractual commitments (DPA) from Anthropic for GDPR compliance. Data residency applies to processing, not just storage — every API call transmits user data and must comply.

---

**Q23.** An agentic system makes 20 tool calls per complex task. Tool calls take 200-500ms each. Total task time is dominated by tool call latency (4-10 seconds of the 6-12 second total). How can tool call latency be reduced architecturally?

A) Make tools faster  
B) Parallel tool calls where dependencies allow: analyze which tool calls are independent (don't depend on each other's outputs) → execute them simultaneously; 5 independent tool calls in parallel (500ms each) takes 500ms total vs. 2,500ms sequential; identify the critical path and parallelize non-dependent calls  
C) Reduce number of tool calls  
D) Cache all tool results  

**Answer: B**  
**Explanation:** Parallel tool execution for latency: the sequential anti-pattern: call tool 1, wait, call tool 2, wait... For tools without data dependencies: fan-out all independent calls simultaneously → fan-in results → continue. Example: if steps 1-5 are all independent reads (customer data, inventory, pricing, policy, history), call all 5 in parallel → reduces 2,500ms to 500ms. This is the Domain 1 parallel pattern applied to individual tool call optimization.

---

**Q24.** At extreme scale (10M+ Claude API calls/month), a company considers building a prompt optimization layer that automatically shortens prompts before sending. What tradeoffs exist?

A) Always shorten prompts — cost savings  
B) Prompt compression tradeoffs: (1) Removing words reduces cost but may remove semantically important content. (2) Compression can change meaning subtly. (3) Compressed prompts are harder to debug. (4) Quality degradation may outweigh cost savings. (5) Evaluate with rigorous A/B testing at the application level. Recommended: manual optimization over automated compression; test any compression on sample requests before deploying broadly.  
C) Never modify prompts — quality first  
D) Only compress system prompts, never user queries  

**Answer: B**  
**Explanation:** Prompt compression at scale: the ROI can be significant (even 10% prompt reduction at 10M calls = substantial savings) but risks are real: compressed prompts may lose important context, change instruction meaning, or reduce reliability. The correct approach: (1) Profile which prompts are largest. (2) Manually review for removable content. (3) Test compressed versions against original on quality metrics. (4) Deploy only where quality impact is acceptable. Automated compression without quality validation is high-risk.

---

**Q25.** A developer has optimized everything else and wants to squeeze the last 15% of cost out of the system. They've already implemented: prompt caching, model tiering, context management. What advanced optimization is appropriate?

A) No further optimization possible  
B) Advanced optimizations: (1) Fine-tuning (if the use case is narrow and high-volume — can reduce prompt length while maintaining quality). (2) Distillation: use Claude to generate training data for a smaller specialized model. (3) Prompt sharing: multiple similar requests use the same expanded prompt cache. (4) Output token reduction: tighter output format specifications to reduce verbosity. (5) Request deduplication: cache responses for identical requests within a short TTL.  
C) Switch to a competitor  
D) Request a custom pricing plan  

**Answer: B**  
**Explanation:** Advanced optimization tier: after caching, tiering, and context management are optimized, the remaining levers are: (1) Output reduction (tightest ROI per word: output costs 5x input). (2) Response caching for identical queries. (3) Fine-tuning for high-volume narrow tasks (smaller prompt = lower cost per call). (4) Request deduplication (short-TTL cache for repeated identical requests). Each additional optimization requires more engineering investment with diminishing returns — prioritize by cost impact.

---

## Score: /25 | Pass: 19/25 (75%)
