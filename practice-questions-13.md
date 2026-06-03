# Practice Questions 13 — RAG, Conversation Design & Token Budgets

> Domain 5 deep-dive: Chunk size tradeoffs, reranking, sliding window vs summarization, persistent memory, needle-in-haystack, token budget allocation.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A RAG system uses 50-token chunks for a legal knowledge base. Retrieved chunks often lack the surrounding context needed to interpret a clause correctly. What is the root cause?

A) Chunks are too large  
B) Chunks are too small — 50 tokens often cuts through the middle of a paragraph or clause; legal text has high contextual dependency within paragraphs; minimum meaningful chunk for legal text is 200-400 tokens per paragraph/clause  
C) Legal text cannot be chunked  
D) Retrieval model is wrong  

**Answer: B**  
**Explanation:** Chunk size selection depends on content type. Legal text: highly contextual, clause-level reasoning requires complete paragraphs. 50-token chunks cut clauses mid-sentence. Guidelines by content type: (1) Legal/regulatory: 200-400 tokens (full paragraphs/clauses). (2) Technical documentation: 150-300 tokens (full procedures). (3) FAQ/conversational: 50-150 tokens (Q&A pairs). Always test chunk boundaries for interpretability.

---

**Q2.** A developer compares two RAG approaches:

Option A: Retrieve 20 chunks with top-20 cosine similarity scores  
Option B: Retrieve 50 chunks, then rerank using a cross-encoder model, keeping top-10

Which produces better answer quality and why?

A) Option A — fewer processing steps  
B) Option B — two-stage retrieval+rerank: initial retrieval maximizes recall (find all potentially relevant chunks); reranking maximizes precision (select the most contextually relevant from candidates); the combination outperforms pure similarity search  
C) They are equivalent  
D) Option A — 20 chunks provides more context  

**Answer: B**  
**Explanation:** Two-stage RAG pipeline: (1) Dense retrieval (cosine similarity) is fast but approximate — maximizes recall. (2) Cross-encoder reranking is slower but more accurate — maximizes precision. Using both: cast a wide net (50 candidates) then precisely select the best 10. This outperforms top-20 direct retrieval because the reranker understands query-document interaction, not just semantic similarity.

---

**Q3.** A conversational AI maintains full conversation history for every user. After 6 months, users with long conversation histories experience degraded response quality. What is happening?

A) The model gets tired  
B) Very long conversation histories contain: contradictory information from earlier conversations, outdated preferences, stale context, and the relevant recent context is "drowned out" by the volume of old context — classic "needle in a haystack" problem  
C) The API rate limit is being hit  
D) History files are corrupted  

**Answer: B**  
**Explanation:** Long-term history degradation: Claude's ability to focus on the most relevant context decreases as history length grows. Old conflicting information can override current behavior. Fix: implement CALM-M lifecycle management — (1) Archive old conversations beyond a certain age. (2) Extract long-term facts into a structured memory object. (3) Only include recent conversation + persistent memory in context.

---

**Q4.** A developer embeds documents using model A but later switches to embedding model B for new documents. Both old and new documents are in the same RAG vector store. What problem occurs?

A) Old documents cannot be retrieved  
B) Embedding space mismatch: vectors from model A and model B are not comparable; similarity search across mixed embeddings returns nonsensical results; all documents must be embedded with the same model  
C) Model B produces better results  
D) This only affects new documents  

**Answer: B**  
**Explanation:** Vector embedding spaces are model-specific. Cosine similarity between embeddings from different models is meaningless. When switching embedding models: (1) Re-embed ALL documents with the new model. (2) Clear the old embeddings. (3) Never mix embeddings from different models in the same vector store. This is a common mistake when upgrading embedding models.

---

**Q5.** A RAG system has 10,000 chunks. Average retrieval returns 20 chunks. Claude uses only 2-3 of the 20 retrieved chunks in its answer. What optimization reduces context waste?

A) Reduce chunk count to 10,000  
B) Reduce k (retrieved chunks) from 20 to 5-7 through better reranking, OR implement a relevance threshold (only include chunks above a minimum similarity score), OR use a contextual compressor that extracts only the relevant sentences from each retrieved chunk  
C) Increase chunk size to reduce total chunks  
D) Remove unused chunks from the knowledge base  

**Answer: B**  
**Explanation:** Over-retrieval wastes context. If 2-3 of 20 chunks are used, the 17-18 unused chunks are wasted tokens. Fix: (1) Tighten k (retrieve fewer, better). (2) Relevance threshold (filter out low-similarity chunks). (3) Contextual compression (extract only the relevant sentences from each chunk rather than entire chunks). The goal: high signal density in the retrieved context.

---

**Q6.** A customer service bot uses a sliding window of the last 10 messages. A user asks a question that depends on something said 15 messages ago. The bot doesn't remember. What conversation design pattern fixes this without keeping all 15 messages?

A) Increase the window to 20  
B) Sticky context pattern: explicitly extract and maintain key facts from the conversation in a structured "session memory" block that persists across the sliding window; the user's earlier information is preserved in compressed form even after sliding out of the window  
C) Use summarization  
D) Ask the user to repeat themselves  

**Answer: B**  
**Explanation:** Sticky context complements the sliding window. When a significant fact is established ("my account number is 12345", "I'm having trouble with the pro subscription", "I've already tried restarting"), extract it to a persistent session memory. Even as old messages slide out, the extracted facts remain. The window preserves recency; sticky context preserves importance.

---

**Q7.** A long-form document editor using Claude needs to maintain context of a 30,000-word document being edited while handling user commands. The document doesn't fit in context with the conversation. What architecture works?

A) Truncate the document to fit  
B) Active context strategy: keep only the current section (3,000 words) in context + a document outline (key headings and summaries of other sections ~2,000 tokens); use tools to navigate and retrieve other sections when needed  
C) Process the full document in batches  
D) Use a larger model  

**Answer: B**  
**Explanation:** Active context with structured navigation: (1) Current section — fully in context (the user is editing this). (2) Document outline — provides navigational awareness of the full document. (3) Tool to retrieve other sections on demand. This gives Claude: full fidelity on the current section, structural awareness of the whole document, and on-demand access to any other section. Far more practical than trying to fit 30,000 words in context.

---

**Q8.** A RAG system for a financial document library retrieves paragraphs based on semantic similarity to the user's query. Sometimes the retrieved paragraphs are semantically similar but refer to different fiscal years (e.g., 2022 vs 2024 data). What retrieval improvement addresses this?

A) Use longer chunks  
B) Metadata filtering: embed year/fiscal_period as metadata on each chunk; at retrieval time, filter by the relevant year from the query before semantic search; hybrid retrieval (semantic + metadata filter) ensures temporal accuracy  
C) Better embedding model  
D) Use keyword search instead  

**Answer: B**  
**Explanation:** Metadata filters are the standard RAG improvement for structured attributes. "What was revenue in 2024?" — extract year filter → search only 2024 documents → then semantic similarity within that filter. This prevents temporally-mismatched results. Common metadata filters: date/year, document type, author, category, confidence level. Hybrid retrieval (semantic + metadata) consistently outperforms pure semantic search for structured knowledge bases.

---

**Q9.** A developer is building a persistent memory system for a personal assistant. Users interact over months and want the assistant to remember their preferences, projects, and history. What is the correct architecture for persistent memory?

A) Store entire conversation history and include it in every request  
B) Extract memory at the end of each conversation: identify key facts, preferences, project updates; store as structured facts; prepend a compressed memory object (not raw history) at the start of each new conversation  
C) Use a database and retrieve all memories  
D) No persistent memory — privacy concerns  

**Answer: B**  
**Explanation:** Persistent memory architecture: (1) Post-conversation extraction: "What should be remembered from this conversation?" → structured facts. (2) Structured storage: `{preferences: [...], active_projects: [...], key_facts: [...], last_updated: "..."}`. (3) Pre-conversation injection: prepend compressed memory (not raw history) to new conversations. This preserves essential long-term context without ballooning context size with raw history.

---

**Q10.** A developer allocates max_tokens = 200 for a task that requires a detailed technical explanation. Users report truncated, incomplete answers. What is wrong?

A) The model doesn't support long outputs  
B) max_tokens is a hard ceiling on output length; 200 tokens (~150 words) is insufficient for a detailed technical explanation; the output is cut off when this limit is reached; increase max_tokens to match expected output complexity  
C) Use a different model  
D) The prompt is too long  

**Answer: B**  
**Explanation:** max_tokens controls the output length ceiling. Common token-to-word ratios: ~1 token per 0.75 words (English). 200 tokens ≈ 150 words — barely a paragraph. For technical explanations: 500-2,000 tokens (400-1,500 words). Set max_tokens appropriate to the expected output: simple answers (100-300), paragraphs (300-800), detailed explanations (800-3,000), comprehensive reports (3,000+).

---

**Q11.** A RAG system retrieves chunks from a policy document. The policy has been updated, but the RAG index still contains the old version. Users receive outdated policy information. What is the correct index management approach?

A) Periodic full re-index (weekly)  
B) Event-driven re-indexing: when a document is updated in the source system, trigger re-embedding and re-indexing immediately; also implement a "last_updated" metadata field and display it in answers so users know the timeliness of information  
C) Version the index and keep old versions  
D) Add a disclaimer that information may be outdated  

**Answer: B**  
**Explanation:** RAG index freshness is critical for policy/compliance applications. Event-driven re-indexing: connect to document update events (webhook, file system watcher, database trigger) → re-embed changed documents → update the index. Plus transparency: surface the document's last_updated timestamp in answers. For regulated industries, stale policy information is a compliance risk, not just a UX issue.

---

**Q12.** A developer is testing the "needle in a haystack" problem with their RAG system. They insert a specific fact into a large document corpus and then query for that exact fact. The system fails to retrieve it. What is most likely wrong?

A) The fact doesn't exist  
B) The embedding of the query and the embedding of the chunk containing the fact may have low cosine similarity due to vocabulary mismatch (the question uses different words than the answer); try query expansion or a hybrid (semantic + keyword) retrieval  
C) The RAG system is broken  
D) The fact must be duplicated in multiple chunks  

**Answer: B**  
**Explanation:** Vocabulary mismatch is the #1 RAG retrieval failure. Query: "What is the maximum fine?" Fact: "Penalties not exceeding $50,000" — different words, same meaning, potentially low similarity. Fix: (1) Query expansion: generate multiple phrasings of the query and retrieve for all. (2) Hybrid retrieval: BM25 keyword search + semantic search → rerank combined results. Keywords catch exact matches; semantics catch paraphrases.

---

**Q13.** For a complex multi-step task requiring 10 tool calls and significant reasoning, how should the token budget be allocated?

A) Maximize input tokens, minimize output  
B) For complex agentic tasks: input (system prompt + context + tool schemas): 20-40% of context; reserve 30-50% for tool call results (each tool result adds to context); reserve 15-25% for output generation; leave buffer (5-10%) to avoid hitting the ceiling  
C) Equal split between input and output  
D) Output tokens don't need to be budgeted  

**Answer: B**  
**Explanation:** Agentic token budget: input + tools are just the starting point — each tool call adds its result to context. A 10-tool pipeline can add significant tokens through results. Budget breakdown: (1) Initial context: ~25%. (2) Tool results (accumulated): ~40%. (3) Final output: ~25%. (4) Buffer: ~10%. Plan for accumulation — the context grows with each tool call.

---

**Q14.** A developer adds a full product catalog (5,000 products, 300 tokens each = 1.5M tokens) to the context via RAG. Obviously this doesn't fit. They reduce to the top 100 most popular products (30,000 tokens) in the system prompt. Still too large. What is the correct RAG design?

A) Use a smaller catalog  
B) RAG per query: retrieve only the products most relevant to the user's specific query (5-10 products, 1,500-3,000 tokens); the full catalog is in the vector store, not in context; only the query-relevant subset is retrieved each time  
C) Summarize each product to 30 tokens  
D) Use a different model  

**Answer: B**  
**Explanation:** RAG is designed for exactly this: putting a large knowledge base in a vector store and retrieving only the query-relevant subset per request. 5,000 products → vector store. User asks about "wireless headphones" → retrieve top 5-10 headphone products → include those ~3,000 tokens in context. The system scales to any catalog size without increasing per-request context.

---

**Q15.** A developer uses a chunk size of 1,000 tokens for a technical manual. They find that answers sometimes cite incorrect page numbers and section headers. What chunking approach fixes this?

A) Use smaller chunks  
B) Include structural metadata in chunks: each chunk embeds document metadata (document_name, section_title, page_number, subsection) in the chunk text or as metadata; this ensures the retrieved chunk carries attribution information for accurate citation  
C) Use larger chunks to avoid splitting sections  
D) Post-process to add citations  

**Answer: B**  
**Explanation:** Chunk metadata is essential for accurate attribution. When chunking: prepend metadata to each chunk: `"[Section 3.2 - Installation Requirements, Page 47]\n{content}"`. This way: (1) The metadata is embedded with the content (semantically associated). (2) Retrieved chunks carry their own attribution. (3) Claude can accurately cite sections and pages. Without metadata in chunks, Claude guesses citations.

---

**Q16.** A RAG system for a support knowledge base retrieves based on semantic similarity to questions, but a user asks a very specific question containing a product model number (e.g., "XR-2000 firmware update"). Semantic search returns vague articles about firmware updates in general. What retrieval improvement helps?

A) Bigger embeddings model  
B) Hybrid search: combine semantic search (captures meaning) with BM25/TF-IDF keyword search (captures exact terms like model numbers); model number "XR-2000" is an exact match problem that semantic search handles poorly but keyword search handles perfectly  
C) Add more firmware articles to the knowledge base  
D) Extract the model number and search programmatically  

**Answer: B**  
**Explanation:** Hybrid retrieval solves the exact-match problem. Semantic search: good for conceptual queries ("how do I update firmware"). Keyword search: good for exact terms (product numbers, error codes, specific API names, legal citations). Combining both: the XR-2000 keyword search finds exact matches; semantic search finds related concepts. The reranker combines scores for the final ranking.

---

**Q17.** A developer is designing a RAG system where the same question might be asked in many different ways ("How do I cancel?", "What is the cancellation process?", "I want to cancel my subscription"). What technique improves retrieval consistency?

A) Add all variations to the knowledge base  
B) Query normalization or hypothetical document embedding (HyDE): generate a hypothetical "ideal answer" for the query and use that as the retrieval query; this normalizes query vocabulary to match the answer space rather than the question space  
C) Use keyword matching for common questions  
D) Store all question variations as metadata  

**Answer: B**  
**Explanation:** HyDE (Hypothetical Document Embedding): instead of embedding the question, embed a hypothetical answer to the question. "How do I cancel?" → generate: "To cancel your subscription, go to Settings > Subscription > Cancel. You will receive..." → embed this hypothetical answer → search. This matches the embedding space of actual answers in the knowledge base, dramatically improving recall for paraphrase variants.

---

**Q18.** A conversation history management system uses a summarization approach. After summarizing 20 turns into 2 sentences, the customer asks: "Earlier you mentioned a 20% discount — can I still get that?" The bot has no memory of offering a discount. What failure occurred?

A) The bot didn't offer a discount  
B) Critical context loss in summarization: the specific discount offer didn't survive the aggressive summarization; "20% discount" is a concrete commitment that must be extracted to persistent memory before summarization  
C) Summarization should never be used  
D) The context window is too small  

**Answer: B**  
**Explanation:** Commitments and offers must be extracted to sticky memory before summarization. Summarization should trigger a "commitment extraction" step: "Does this conversation contain: pricing commitments, agreed actions, promises made?" → extract to a `commitments: [{type: "discount", value: "20%", conditions: "..."}]` field in session memory. The summary covers general context; extracted commitments capture binding information.

---

**Q19.** A developer is choosing between chunk sizes of 200, 500, and 1,000 tokens for their RAG knowledge base. Content is a mix of short FAQ answers and long technical procedures. What is the best approach?

A) Use one chunk size for everything  
B) Use content-adaptive chunking: FAQ answers → 100-200 token chunks (one Q&A pair per chunk); technical procedures → 500-800 token chunks (complete procedure steps); don't apply a uniform chunk size to content that varies significantly in natural unit size  
C) Use 500 tokens for all content  
D) Use the largest chunk size  

**Answer: B**  
**Explanation:** Content-adaptive chunking matches chunk size to natural content units. FAQs: one Q&A pair = a complete, self-contained unit (100-200 tokens). Technical procedures: complete procedure steps with context = a complete unit (500-800 tokens). Uniform chunk sizes either split FAQs unnecessarily (too large) or cut procedures mid-step (too small). Chunk at natural semantic boundaries.

---

**Q20.** For a multi-turn conversation with a sales assistant, what is the optimal token budget split between system instructions, product knowledge, and conversation history?

A) Equal thirds  
B) Depends on conversation turn: early turns (few history tokens) → larger product knowledge allocation; later turns (more history) → dynamically adjust by loading only the most relevant product knowledge for the current query; keep system instructions constant; history grows and other content shrinks  
C) Maximize system instructions  
D) Maximize product knowledge  

**Answer: B**  
**Explanation:** Dynamic context allocation: system instructions are fixed (constant tokens). Product knowledge and history compete for the remaining budget. As history grows: (1) Apply CALM-M to compress history. (2) Use RAG to retrieve only product knowledge relevant to the current query (not all products). The total input must stay within budget — dynamic allocation between knowledge and history keeps it manageable.

---

**Q21.** A RAG system has 95% retrieval accuracy (the correct document is in the top-5 retrieved) but only 70% answer accuracy. What does this gap indicate?

A) The retrieval model is wrong  
B) Retrieval is working but answer generation is failing: the correct information is present in context but Claude is not extracting/using it correctly; investigate: prompt instructions for citation, chunk size (may be splitting the answer), or answer generation prompt  
C) Need more training data  
D) Use a different embedding model  

**Answer: B**  
**Explanation:** Retrieval accuracy ≠ answer accuracy. The gap indicates an answer generation problem, not retrieval. Root causes: (1) Chunk splits the relevant information across boundaries (answer straddles two chunks). (2) Prompt doesn't instruct Claude to use only the retrieved context. (3) The relevant passage is in context but diluted by other retrieved content. Debug the generation step, not the retrieval step.

---

**Q22.** A developer wants to implement conversation history for a medical consultation system. Privacy requirements state: no personal health information (PHI) should be retained after the session ends. What conversation memory architecture satisfies both utility and compliance?

A) Don't use conversation history  
B) In-session conversation history (ephemeral, in-memory only): full conversation context within a session; on session end: all data is discarded; no persistent memory; the compliance requirement (no PHI retention) is satisfied; utility (full session context) is preserved  
C) Store anonymized summaries  
D) Use hashed identifiers  

**Answer: B**  
**Explanation:** Ephemeral-only conversation history: full utility within a session + compliance (no persistence). Implementation: conversation history in memory (not database); session ends → memory cleared; no logging of conversation content. This is the correct architecture when regulations prohibit PHI retention: excellent per-session performance, zero cross-session retention. Trade-off: users must re-establish context in each new session.

---

**Q23.** A developer has a 150,000-token context window available. They fill 140,000 tokens of input and set max_tokens=20,000. What happens?

A) The system generates a 20,000-token output  
B) Context overflow: 140,000 (input) + 20,000 (max output) = 160,000 total, which exceeds the 150,000-token context; the API call will fail or truncate; input must be reduced to leave room for the requested output  
C) The system auto-adjusts the output length  
D) Only input tokens count against the context  

**Answer: B**  
**Explanation:** Context window = input + output. This is a common mistake: developers fill the context window with input and forget to reserve space for output. Rule: max usable input = context_window - desired_max_output. With 150k context and 20k output requirement: max input = 130k tokens. Always plan input size with the output budget in mind.

---

**Q24.** A developer builds a research assistant that processes 50 academic papers per session. After reading all 50 papers (far too large for context), it must answer comparative questions across papers. What architecture handles this?

A) Use a model with a larger context window  
B) Build a paper-by-paper extraction pipeline: for each paper, extract key findings, methodology, and claims into a structured summary; store summaries; at query time, retrieve relevant summaries (RAG over summaries) + provide full text of the 2-3 most relevant papers; answer from this combined context  
C) Summarize all 50 papers into one document  
D) Answer each paper separately and aggregate  

**Answer: B**  
**Explanation:** Hierarchical RAG: (1) Per-paper structured extraction. (2) Summaries as the primary retrieval corpus. (3) Full papers available for deep-dive retrieval. Comparative questions ("which paper found the strongest correlation?") work on summaries. Questions requiring exact quotes ("what methodology was used in Paper X?") retrieve the full paper. This scales to any number of papers.

---

**Q25.** A developer notices that adding more retrieved chunks (from 5 to 15) consistently improves recall but degrades answer quality. What explains this paradox and what is the optimal solution?

A) More chunks always improves quality  
B) "Lost in the middle" effect: Claude's attention is less reliable for content in the middle of a long context; relevant information in chunk positions 6-12 (middle of 15 chunks) is less likely to be used than information in positions 1-5 or 13-15; reorder: most relevant chunks first and last  
C) Use smaller chunks  
D) The retrieval model is wrong  

**Answer: B**  
**Explanation:** Lost in the middle is an empirically observed phenomenon: Claude (and most LLMs) use information at the beginning and end of context more reliably than information in the middle. With 15 chunks: put the highest-relevance chunks at positions 1-3 and 13-15; put lower-relevance chunks in the middle. This maximizes the probability that the most relevant information influences the answer. Position matters in RAG.

---

## Score: /25 | Pass: 19/25 (75%)
