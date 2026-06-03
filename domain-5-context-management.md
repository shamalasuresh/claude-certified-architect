# Domain 5: Context Management — 15%

> The "tie-breaker" domain. Context management questions separate candidates who pass from those who fail. Do not underestimate this domain.

---

## What This Domain Covers

- Token budget optimization
- Prompt caching (ephemeral and persistent)
- Multi-turn conversation design
- The CALM framework
- Context window strategies
- Managing long conversations

---

## 1. Understanding the Context Window

Claude has a **context window** — the maximum amount of text it can process in a single request. Everything Claude "knows" in a conversation is limited to this window.

### Context Window Components

```
┌─────────────────────────────────────────┐  ← Context window limit
│  System Prompt                          │
│  (instructions, persona, examples)      │
├─────────────────────────────────────────┤
│  Conversation History                   │
│  (previous human/assistant turns)       │
├─────────────────────────────────────────┤
│  Retrieved Context                      │
│  (RAG results, tool outputs)            │
├─────────────────────────────────────────┤
│  Current User Message                   │
│                                         │
├─────────────────────────────────────────┤
│  Response (output tokens)               │
│                                         │
└─────────────────────────────────────────┘
```

### Token Counting Rules of Thumb

| Content Type | Approx. Token Count |
|-------------|---------------------|
| 1 word | ~1.3 tokens |
| 100 words | ~130 tokens |
| 1 page of text | ~500 tokens |
| Full code file (100 lines) | ~500-1000 tokens |
| Detailed system prompt | ~500-2000 tokens |
| Long conversation (50 turns) | ~10,000-50,000 tokens |

**Key insight:** In long applications, conversation history is often the biggest consumer of the context window — bigger than system prompts or current messages.

---

## 2. The CALM Framework

CALM is the primary framework for context management. The exam will reference it directly.

### C — Chunk Long Content

Break large documents into smaller pieces rather than stuffing the entire document into context.

```
✓ Correct: Retrieve and inject only the relevant 3-5 chunks (500-1000 tokens each)
✗ Wrong: Paste the entire 50-page document into context
```

**Chunking strategies:**
- **Fixed-size chunking:** Split at every N tokens (simple but ignores meaning)
- **Semantic chunking:** Split at natural boundaries (paragraphs, sections)
- **Recursive chunking:** Try large chunks, split smaller if needed
- **Sliding window:** Overlapping chunks to preserve cross-boundary context

### A — Aggressively Cache

Use prompt caching for static content that appears in every request.

**What to cache:**
- System prompts (stable instructions)
- Base documentation (rarely changes)
- Few-shot examples
- Tool schemas

**What NOT to cache:**
- Dynamic user data
- Session-specific context
- Anything that changes per-request

### L — Limit Conversation Length

Design conversations to not accumulate indefinitely.

**Strategies:**
- **Sliding window:** Keep only the last N turns
- **Summarization:** Replace old turns with a summary
- **Topic detection:** Start fresh context when topic changes significantly
- **Explicit reset:** Allow users to clear conversation history

### M — Manage Token Budgets Explicitly

Don't guess; measure and allocate.

```python
# Example token budget allocation
CONTEXT_BUDGET = 100_000  # total context window

system_prompt_tokens = 2_000    # measured
conversation_history = 20_000  # last 20 turns
retrieved_context = 10_000     # top-5 RAG chunks
current_message = 500          # current user input
response_reserve = 4_000       # max response length

used = (system_prompt_tokens + conversation_history + 
        retrieved_context + current_message + response_reserve)
# = 36,500 — well within 100k budget
```

---

## 3. Prompt Caching

Prompt caching is one of the most impactful cost and latency optimizations available.

### How Prompt Caching Works

When you mark a prefix of your prompt as cacheable, Anthropic stores the computed KV cache. Subsequent requests that share that exact prefix reuse the cached computation instead of reprocessing it.

```
Request 1:  [SYSTEM PROMPT (cacheable)] + [User message A]
            ↑ Computed and cached

Request 2:  [SYSTEM PROMPT (cacheable)] + [User message B]
            ↑ Cache hit! Not recomputed. 
            Only [User message B] is new computation.
```

### Cache Types

| Type | Duration | Best For |
|------|----------|---------|
| **Ephemeral** | ~5 minutes | Within a conversation session |
| **Persistent** | Up to 1 hour | Cross-session reuse of large prompts |

### When Caching Saves the Most

```
Savings = (cached_tokens / total_tokens) × request_volume

High impact: Large system prompt + high request volume
Low impact:  Small system prompt or low request volume
```

**Exam insight:** Caching is most valuable when:
1. The cached prefix is large (thousands of tokens)
2. The same prefix is used many times
3. The uncached portion (user message) is relatively small

### Cache Invalidation

The cache is invalidated when:
- The cached prefix changes (even by one character)
- The cache TTL expires
- You explicitly remove cache markers

**Practical implication:** Put stable content first, dynamic content last. Changing early content invalidates the entire cache for that prefix.

```
✓ Correct ordering:
[System prompt — stable] → [Few-shot examples — stable] → [User message — dynamic]

✗ Wrong ordering:  
[Current date — changes daily] → [System prompt — stable] → [User message]
(Cache invalidated every day for everything after the date)
```

---

## 4. Multi-Turn Conversation Design

### Conversation State Models

**Stateless (each request is independent):**
```
Request: [System prompt] + [Full history] + [New message]
Use when: History fits in context, simplest implementation
Problem: Context grows unbounded over long conversations
```

**Stateful with sliding window:**
```
Request: [System prompt] + [Last N turns] + [New message]
Use when: Conversation can be understood from recent turns
Problem: Loses early context (user's initial preferences)
```

**Stateful with summarization:**
```
Request: [System prompt] + [Summary of early turns] + [Last N turns] + [New message]
Use when: Early context matters but full history is too long
Problem: Summarization adds latency and cost
```

**Stateful with persistent memory:**
```
Request: [System prompt] + [Extracted key facts] + [Last N turns] + [New message]
Use when: Specific facts from early turns are important long-term
Problem: Requires extracting and storing facts explicitly
```

### When to Summarize vs. Truncate

| Approach | When | Cost |
|----------|------|------|
| Keep full history | Short conversations, all turns equally relevant | Low |
| Sliding window | Long conversations, recent context most relevant | Low |
| Summarize then window | Long conversations, early context partially relevant | Medium |
| Extract key facts | Long conversations, specific facts matter | Medium |
| Full summarization | Very long conversations, need complete overview | High |

### Designing for Conversation Continuity

**Carry forward user preferences:**
```python
# At start of new session, inject remembered preferences
system_prompt = base_prompt + f"""
User preferences from previous sessions:
- Prefers concise responses
- Works in Python 3.11
- Uses VS Code
"""
```

**Handle context loss gracefully:**
```
If you need information about a previous step that isn't in 
the current context, say "I don't have that information in 
the current context — could you remind me of X?" rather than 
making assumptions.
```

---

## 5. Context Window Strategies

### RAG (Retrieval Augmented Generation)

RAG is the primary strategy for giving Claude access to large knowledge bases without hitting context limits.

```
User Question
     │
     ▼
Embedding Model → Query Vector
     │
     ▼
Vector Database → Top-K Relevant Chunks
     │
     ▼
Context Assembly: [System] + [Chunks] + [Question]
     │
     ▼
Claude → Answer grounded in retrieved context
```

**RAG chunk sizing:**
- Too small: Missing context, incoherent chunks
- Too large: Wastes context on irrelevant content
- Sweet spot: 200-500 tokens with semantic boundaries

**Reranking:** After retrieving top-K chunks, rerank by relevance before injecting. This improves quality when initial retrieval is noisy.

### Long Document Strategies

**Problem:** Document is larger than context window.

| Strategy | How | When |
|----------|-----|------|
| Hierarchical summarization | Summarize sections, then summarize summaries | Need full-document understanding |
| Map-reduce | Process each chunk independently, combine results | Tasks parallelizable by chunk |
| Sliding window | Process overlapping windows sequentially | Sequential tasks (translation, review) |
| Targeted extraction | Extract only relevant sections | Have a specific query |

### Structured Context Injection

When injecting multiple pieces of context, use clear separators:

```xml
<context type="user_profile">
Name: Alex Johnson
Account tier: Enterprise
Previous issues: None
</context>

<context type="current_ticket">
Issue: Login fails after password reset
Priority: High
Created: 2024-01-15
</context>

<context type="relevant_docs">
[Password Reset Documentation excerpt]
...
</context>
```

**Why:** XML tags help Claude distinguish between different types of context and reference the right information.

---

## 6. Token Budget Optimization

### Techniques to Reduce Token Usage

**1. Compress system prompts**
```
✗ Verbose: "You should always make sure to respond in a professional 
and courteous manner that reflects well on our company and treats 
customers with the respect they deserve."

✓ Compressed: "Always respond professionally and respectfully."
```

**2. Use structured formats over prose**
```
✗ Prose: "The user's name is John Smith. Their account number is 12345. 
They have been a customer since January 2022 and their plan is Pro."

✓ Structured: 
Customer: John Smith | Account: 12345 | Since: Jan 2022 | Plan: Pro
```

**3. Avoid conversation history padding**
```
✗ Keep every "thank you", "sure!", "got it" turn
✓ Filter out low-information turns before including in history
```

**4. Use cache markers on static content**
```python
# Mark stable prefix for caching
messages = [
    {
        "role": "user",
        "content": [
            {"type": "text", "text": LARGE_STABLE_DOCS, 
             "cache_control": {"type": "ephemeral"}},
            {"type": "text", "text": current_question}
        ]
    }
]
```

### Token Budget for Output

Don't forget output tokens have a cost and count toward per-request limits:

```python
response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=1024,  # Set an appropriate limit
    messages=[...]
)
```

**Set max_tokens** based on expected response length. A customer service response doesn't need 4096 tokens. Oversized max_tokens wastes money if the API charges by tokens generated.

---

## 7. Context Quality Over Quantity

**The cardinal rule:** More context is not always better. Irrelevant context increases noise and can degrade performance.

### The Needle-in-a-Haystack Problem

When the relevant information is buried in a large context, Claude may miss it. Studies show performance degrades when relevant information is in the middle of very long contexts.

**Mitigation:**
- Put the most important context near the beginning or end
- Use explicit markers: `<critical_context>` around key information
- Keep context focused and relevant

### Signal-to-Noise Ratio

```
High quality context:
- Directly relevant to the query
- Recent and accurate
- Well-structured and clear

Low quality context:
- Tangentially related
- Outdated information
- Poorly formatted, noisy
```

**Exam insight:** Injecting more retrieved chunks is not always better. 3 highly relevant chunks often outperform 20 loosely relevant chunks.

---

## 8. Exam Scenarios & Right Answers

### Scenario: "Application costs are very high due to large system prompts repeated every request"
**Right answer:** Enable prompt caching on the stable system prompt prefix. Mark it with `cache_control: ephemeral`. Cost reduction is significant for high-volume, large-prompt applications.

### Scenario: "Long conversations lose early context that was important"
**Right answer:** Summarization strategy or persistent memory extraction. Sliding window alone loses early context.

### Scenario: "Context window fills up after 20 turns"
**Right answer:** Sliding window with last N turns + a system-generated summary of earlier important context.

### Scenario: "RAG retrieval returns 20 chunks but answers are poor quality"
**Right answer:** More chunks is not better. Add reranking to select top 3-5 most relevant, and tune chunk size. Focus on signal-to-noise ratio.

### Scenario: "Static documentation (10,000 tokens) included in every request"
**Right answer:** Use prompt caching (persistent) for the documentation. Put it first (stable prefix), dynamic content after.

### Scenario: "User's preferences from turn 1 are needed at turn 50"
**Right answer:** Extract key facts from early turns into a persistent memory structure. Inject as compact structured context in subsequent sessions.

### Scenario: "Need to process a 200-page document with Claude"
**Right answer:** Choose strategy based on task — hierarchical summarization for full understanding, map-reduce for parallel tasks, targeted extraction for specific queries.

---

## 9. Quick Reference Card

### CALM Framework
```
C - Chunk long content (semantic boundaries)
A - Aggressively cache (stable prefixes)
L - Limit conversation length (sliding window/summarize)
M - Manage token budgets explicitly (measure, allocate)
```

### Caching Decision
```
Large prefix + high volume + stable content → Cache it
Small prefix OR low volume OR frequent changes → Skip caching
```

### Conversation History Strategies
```
Short conv (<20 turns)     → Keep full history
Long conv, recent matters  → Sliding window (last N turns)
Long conv, all matters     → Sliding window + summary
Long conv, facts matter    → Persistent memory extraction
```

### RAG Tuning
```
Poor recall    → Larger chunks, more retrieval
Poor precision → Smaller chunks, reranking, fewer chunks
Slow retrieval → Reduce top-K, optimize vector index
High cost      → Cache stable documents, reduce chunk count
```
