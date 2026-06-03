# Practice Questions 7 — CoT, Few-Shot, Output Formatting & Hallucination

> Domain 3 deep-dive: When/how to use chain-of-thought, few-shot design, output reliability, reducing hallucination.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A mortgage approval system uses Claude to evaluate applications. Applications involve: income verification, debt-to-income calculation, credit history analysis, and risk scoring. Which prompt technique most improves decision accuracy?

A) High temperature for creativity  
B) Chain-of-thought prompting: "Work through each evaluation factor step by step before giving a final approval recommendation"  
C) Few-shot examples of approved applications  
D) Reduce max_tokens  

**Answer: B**  
**Explanation:** Mortgage approval involves multi-step reasoning with numeric calculations and risk evaluation. CoT prompting forces Claude to reason through each factor before making a final decision, reducing the chance of jumping to conclusions. This improves accuracy on complex, multi-factor decisions compared to direct prompting.

---

**Q2.** A sentiment analysis system must classify product reviews as: `positive`, `negative`, `neutral`, `mixed`. Without few-shot examples, Claude sometimes returns `mostly positive` or `somewhat negative`. What is the root cause and fix?

A) The model isn't capable of exact classification  
B) Without examples showing the exact labels, Claude uses natural language variations; fix: provide 1-2 few-shot examples per label showing the exact string output required  
C) Use a temperature of 0  
D) Use a JSON schema  

**Answer: B**  
**Explanation:** Few-shot examples establish the exact output vocabulary. Without them, Claude uses natural language variations of sentiment labels. With examples showing `positive`, `negative`, `neutral`, `mixed` as the exact outputs, Claude anchors to those exact strings. Combined with a JSON schema (`"enum": ["positive", "negative", "neutral", "mixed"]`), this eliminates non-canonical labels.

---

**Q3.** A developer uses extended thinking (with `thinking_budget: 10000`) for every request to a simple FAQ bot. Users complain about slow response times. What is wrong?

A) Extended thinking should always be used for quality  
B) Extended thinking is for complex reasoning tasks; a FAQ bot answering simple factual questions doesn't benefit from deep reasoning — it adds significant latency for no quality gain  
C) The thinking budget is too small  
D) Extended thinking is required for all production applications  

**Answer: B**  
**Explanation:** Extended thinking adds latency proportional to the thinking budget. For simple FAQ lookups, direct retrieval from context is sufficient — no deep reasoning is needed. Extended thinking is valuable for: complex multi-step problems, architectural decisions, ambiguous situations requiring analysis. Not for: simple lookups, classification, factual retrieval.

---

**Q4.** A developer provides 3 few-shot examples for a task. The examples all show the same output format but different input types. After deployment, performance is poor on inputs that look very different from the examples. What few-shot improvement helps?

A) Add more examples of the same type  
B) Add examples specifically covering the failing input types — examples should represent the actual distribution of inputs the system will encounter  
C) Remove examples and rely on instructions  
D) Use longer examples  

**Answer: B**  
**Explanation:** Few-shot examples must cover the input distribution. If edge cases or uncommon input types aren't represented in examples, performance degrades on them. The key question: "What are my system's real inputs?" Then: "Do my examples cover the tricky cases?" Add examples for inputs where the task behavior is non-obvious.

---

**Q5.** A legal contract analysis tool consistently invents clause numbers and section headers that don't exist in the contracts. This is happening despite the contract text being provided in the prompt. What is the most targeted fix?

A) Use a more powerful model  
B) Grounded generation instruction: "ALL references (clause numbers, section headers, dates) MUST be copied verbatim from the contract text. Never paraphrase or invent references. If you cannot find a specific clause, say 'No such clause found.'"  
C) Lower temperature  
D) Add more context  

**Answer: B**  
**Explanation:** Invented references are a specific hallucination pattern. The fix is a targeted negative instruction + grounding requirement: all references must be verbatim from the source + explicit fallback for missing items ("not found"). Temperature reduction helps somewhat but doesn't prevent the underlying tendency to fill gaps with plausible-sounding invented content.

---

**Q6.** A system processes customer feedback and needs to: (1) classify sentiment, (2) extract key issues, (3) assign priority. A developer considers: one prompt for all three vs. three separate prompts. Which is better and why?

A) One prompt — always more efficient  
B) Three separate prompts — each task is focused and the output of each can be validated before use; errors in step 1 don't contaminate steps 2 and 3  
C) One prompt with chain-of-thought covering all three  
D) Depends on cost requirements  

**Answer: C**  
**Explanation:** Chain-of-thought in a single structured prompt is often the best balance: "First classify sentiment. Then extract key issues from a negative/mixed sentiment review. Then assign priority based on the issues found." This maintains the sequential reasoning (step 2 benefits from step 1's output) without the overhead of three API calls. Separate prompts add latency and cost; a single CoT prompt achieves the same result.

---

**Q7.** A data extraction prompt needs Claude to extract structured data from unstructured text. The instructions describe the output schema in detail. After deployment, ~10% of extractions have missing fields or wrong types. What technique most reduces this?

A) More detailed instructions  
B) Assistant prefill with the opening JSON structure + example-based few-shot showing the exact schema populated with sample data  
C) Higher temperature for more creative extraction  
D) Add output validation  

**Answer: B**  
**Explanation:** For structured extraction, prefill + few-shot is highly effective. Prefill (`{`) forces JSON output. Few-shot examples show the populated schema, teaching Claude exactly what each field should contain. The combination of structural constraint (prefill) and semantic guidance (examples) reduces schema violations and missing fields more effectively than instructions alone.

---

**Q8.** Extended thinking is enabled for a classification task. The thinking block contains correct reasoning, but the final answer is wrong. What does this indicate?

A) Extended thinking doesn't work  
B) The model corrected itself from wrong thinking, producing the right final answer  
C) There is a disconnect between the reasoning and the final answer; this can happen when the output format constraints override the model's reasoning — review how the final answer section is prompted  
D) The thinking budget is too small  

**Answer: C**  
**Explanation:** Correct thinking but wrong final answer indicates a structural issue in how the output is prompted. The model may be forced into a specific format that doesn't naturally accommodate the nuanced reasoning. Review the Expected Output (PRECISE-E) section — it may be overconstrained or ambiguously specified in a way that causes the model to disconnect the answer from the reasoning.

---

**Q9.** A developer wants to use CoT for a code generation task. The concern is that the CoT reasoning in the response will confuse users who just want the code. What technique separates reasoning from user-visible output?

A) Ask Claude to think internally  
B) Use `<thinking>` tags for reasoning and instruct that only content outside these tags is shown to users; or use extended thinking which keeps the thinking block separate from the response  
C) Run reasoning in a separate API call  
D) There is no way to hide CoT reasoning  

**Answer: B**  
**Explanation:** Two approaches: (1) `<thinking>` tags in the prompt with post-processing to strip them from user-visible output. (2) Extended thinking API — the thinking block is a separate response field not shown to users by default. Both achieve separation of reasoning from user-facing output. The extended thinking approach is cleaner architecturally.

---

**Q10.** A few-shot prompt has 8 examples. The developer notices performance is slightly worse than with 4 examples. What explains this?

A) Fewer examples always perform better  
B) Beyond a certain point, more examples can dilute the signal, introduce conflicting patterns, or push important instructions further from the query; 3-5 high-quality examples usually outperform 8+ mediocre ones  
C) The model cannot handle more than 5 examples  
D) The 8 examples have a bug  

**Answer: B**  
**Explanation:** Example quality > example quantity. Too many examples can: dilute the signal with edge cases that add noise, push the actual task instructions further from the query (primacy/recency effect), or introduce subtle patterns that conflict. 3-5 carefully chosen, diverse, high-quality examples usually produce better results than 10+ examples.

---

**Q11.** A code review tool using Claude produces reviews that say "the code looks good" for obvious security bugs. This is false confidence. What prompt technique most directly prevents this?

A) Use a stricter tone  
B) Include few-shot examples showing security vulnerabilities being caught AND a negative instruction: "Never state 'the code looks good' without explicitly verifying: SQL injection, XSS, auth bypasses, insecure deserialization, hardcoded credentials"  
C) Lower the temperature  
D) Add more examples of good code  

**Answer: B**  
**Explanation:** "Looks good" is the hallucination of a secure codebase. The fix: (1) Few-shot examples showing vulnerabilities being identified (teaches Claude what to look for). (2) Explicit instruction preventing premature positive conclusions without security verification. The checklist format ensures Claude actually checks for each category rather than pattern-matching to "no obvious issues."

---

**Q12.** A developer uses zero-shot CoT: "Think step by step and then answer." An expert review finds the steps are correct but verbose and the final answer is buried in text. What single addition improves usability most?

A) Remove the CoT instruction  
B) Add a structural separator: "Think step by step, then provide your final answer after '### Answer:'" — this separates the reasoning from the extractable answer  
C) Use XML tags for the answer  
D) Add max_tokens constraint  

**Answer: C**  
**Explanation:** Using XML tags like `<answer>` is actually the cleaner approach but B also works. The key is structural separation between reasoning and answer. Either `### Answer:` (easy to parse after the fact) or `<answer>your answer here</answer>` (easy to extract with regex/parsing) creates a reliable extraction point. Without it, parsing the final answer from verbose CoT is fragile.

---

**Q13.** A developer asks: "Should I use few-shot or zero-shot CoT for my complex classification task?" The task has 12 possible output categories and tricky edge cases. What is the answer?

A) Zero-shot CoT — it's simpler  
B) Few-shot — provide 2-3 examples per category covering the tricky edge cases; additionally, add CoT instruction for ambiguous inputs  
C) Few-shot examples alone without CoT  
D) Neither — use a fine-tuned model  

**Answer: B**  
**Explanation:** 12 categories with tricky edge cases = combine few-shot (establish the category vocabulary and handle edge case examples) + CoT (for ambiguous inputs where the classification requires reasoning). Few-shot examples cover the expected cases; CoT handles novel ambiguous ones. This combination is more robust than either alone.

---

**Q14.** An AI assistant frequently confuses two similar products in the company catalog (ProductA and ProductB have similar names). What prompt technique most effectively prevents this confusion?

A) A longer product description for each  
B) Few-shot examples specifically contrasting ProductA and ProductB in the same examples: "User asks about [feature that only ProductB has] → answer is ProductB, not ProductA because..."  
C) Mention the products alphabetically  
D) Increase temperature  

**Answer: B**  
**Explanation:** Confusion between similar items is best addressed with contrastive examples — examples that show the difference explicitly. "User asked about X → classified as ProductA (not ProductB) because ProductA has [distinguishing feature]" directly teaches the distinction. This is more effective than separate examples for each product or more descriptive text alone.

---

**Q15.** A medical documentation assistant is asked: "Can beta blockers cause bradycardia?" It confidently states "No" — which is incorrect. This is a factual hallucination. What prompt technique reduces this specific risk?

A) Higher temperature  
B) Confidence calibration instruction: "Only make definitive claims (yes/no, always/never) when you are certain. For medical facts where you have any uncertainty, state 'This is an area where you should verify with current clinical guidelines' rather than giving an incorrect definitive answer."  
C) Grounded generation with source documents  
D) Both B and C  

**Answer: D**  
**Explanation:** Both techniques together are most effective. B (confidence calibration) prevents false confident answers by instructing Claude to acknowledge uncertainty. C (grounded generation with clinical reference documents) gives Claude accurate source material to cite. For medical applications, both are essential — uncertainty acknowledgment prevents confident errors, grounding provides accurate facts.

---

**Q16.** A developer wants Claude to extract 15 specific fields from unstructured invoices. Some fields are often absent from invoices. The extraction consistently hallucinates values for missing fields. What instruction prevents this?

A) Increase example count  
B) Explicit instruction: "For fields not present in the invoice, output `null`. Never infer, estimate, or approximate missing values. If you are uncertain whether a field is present, output `null` rather than guessing."  
C) Make all fields optional in the schema  
D) Post-process results to remove low-confidence extractions  

**Answer: B**  
**Explanation:** The null-for-missing instruction is the targeted fix for extraction hallucination. Without it, Claude fills empty slots with plausible-looking values. With it, missing fields are explicitly represented as null — which downstream systems can handle correctly. This is a classic negative instruction that defines the correct behavior for the failing case.

---

**Q17.** A developer compares two approaches for a complex reasoning task:

Option A: Single prompt with CoT  
Option B: Pipeline of 3 prompts (classify → analyze → recommend)  

Option B is more accurate but 3x more expensive. When is Option B justified?

A) Always — more prompts = better accuracy  
B) When the accuracy improvement has meaningful business value (e.g., medical, legal, financial decisions) AND the cost fits the use case budget; for casual or high-volume low-stakes tasks, single-prompt CoT is preferred  
C) Never — optimize for cost  
D) Only for latency-insensitive applications  

**Answer: B**  
**Explanation:** The accuracy/cost tradeoff depends on the stakes and volume. A multi-prompt pipeline for medical decision support is justified even at 3x cost — wrong answers have severe consequences. A pipeline for ad headline generation is not worth 3x cost. Always evaluate: "What is the cost of a wrong answer?" vs. "What is the cost of 3 API calls?"

---

**Q18.** When using structured few-shot examples, XML tags `<example>` around each example help because:

A) XML is faster to process  
B) XML tags create clear, unambiguous boundaries between examples and between examples and the actual task; Claude can clearly distinguish example input/output from the real input it must process  
C) XML requires less tokens  
D) XML is required for few-shot  

**Answer: B**  
**Explanation:** XML tags are a structural delimiter that prevents example content from "bleeding" into the actual task or being interpreted as instructions. Without delimiters, Claude may confuse example outputs with the format of the actual response, or miss where examples end and the real task begins. Clear boundaries = reliable parsing.

---

**Q19.** A developer instructs Claude: "If you don't know something, make your best guess." This causes hallucination. What is the correct instruction?

A) "Never guess; always refuse unknown questions"  
B) "If you're uncertain, say 'I'm not certain about this specific detail' and provide what you do know with appropriate confidence markers. Clearly distinguish confident facts from uncertain ones."  
C) "Guess with low confidence"  
D) "Only answer questions you are 100% certain about"  

**Answer: B**  
**Explanation:** "Best guess" causes Claude to produce plausible-sounding false information. The better instruction acknowledges uncertainty explicitly, provides what is known, and uses confidence markers. This gives users accurate meta-information about reliability. "Never guess / always refuse" is too restrictive — Claude has genuine uncertainty gradients that should be communicated, not suppressed.

---

**Q20.** A prompt uses the sandwich pattern (instructions → long content → repeat instructions). Which instructions should be repeated at the end?

A) All instructions, exactly as written at the beginning  
B) The most critical behavioral rules and the exact output format specification — the elements most likely to be "forgotten" in long context  
C) The persona and role only  
D) Nothing should be repeated  

**Answer: B**  
**Explanation:** The sandwich pattern is targeted — not every instruction needs repeating. Repeat: (1) Output format specification (easy to drift in long contexts). (2) Critical behavioral rules (things that must always be followed). Don't repeat: persona/role (established once, stable), context (already provided), style (general guidance). Targeted repetition keeps the prompt concise while addressing the "lost in the middle" problem.

---

**Q21.** A developer needs Claude to produce numbered lists consistently. Sometimes Claude produces: `1. 2. 3.` and sometimes `1) 2) 3)` and sometimes `- - -`. What prompt engineering fix is most reliable?

A) Specify "numbered list" in the instructions  
B) Provide an example in the Expected Output section: "Format your list exactly as: `1. First item\n2. Second item\n3. Third item`" OR use assistant prefill with "1. "  
C) Use a style instruction  
D) Ask Claude to be consistent  

**Answer: B**  
**Explanation:** Format consistency requires showing the exact format, not describing it. "Numbered list" is ambiguous (periods vs. parentheses vs. other). Show the exact format OR use assistant prefill to start the list for Claude. Describing format in words ("use 1. format") is less reliable than showing it or starting it.

---

**Q22.** A developer is building a prompt for a task that requires comparing two options. Which CoT structure produces the most balanced comparison?

A) "What is better, A or B?"  
B) "Evaluate A: list its strengths and weaknesses. Evaluate B: list its strengths and weaknesses. Compare the two across: [specific dimensions]. Then make a final recommendation given [specific constraints]."  
C) "Think step by step and compare A to B"  
D) "First think about A, then about B"  

**Answer: B**  
**Explanation:** Structured CoT for comparisons: separate evaluation of each option (prevents anchoring bias from early comparison), explicit comparison dimensions (prevents arbitrary criteria), and constrained recommendation (based on given constraints, not general preference). This structure produces balanced, reproducible comparisons.

---

**Q23.** A developer tests a prompt and finds Claude's reasoning (in CoT) is sound but it reaches the wrong final answer 20% of the time. What is the most likely cause?

A) The model has a bug  
B) The final answer prompt structure may cause Claude to "second-guess" its reasoning at the answer stage; try: "Based on the reasoning above, your final answer is:" to anchor the answer to the completed reasoning  
C) The thinking budget is too small  
D) The examples are wrong  

**Answer: B**  
**Explanation:** "Reasoning correct, answer wrong" is a real phenomenon. The final answer prompt may inadvertently invite reconsideration. "Based on the reasoning above" anchors the answer to the completed reasoning rather than re-opening the question. This often reduces the disconnect between correct reasoning and incorrect final answers.

---

**Q24.** A retrieval-augmented generation (RAG) system provides Claude with retrieved documents. Claude still produces hallucinated facts not in the documents. What additional prompt technique is needed?

A) Retrieve more documents  
B) Strict grounding instruction + citation requirement: "Answer ONLY using information in the provided documents. If the answer is not in the documents, say exactly: 'The provided documents do not contain information about this.' Cite the specific document/section for every factual claim."  
C) Use higher quality documents  
D) Reduce retrieved document count  

**Answer: B**  
**Explanation:** RAG alone does not prevent hallucination — it just provides better source material. Claude may still supplement retrieved facts with training knowledge. The strict grounding instruction + citation requirement creates a verifiable chain between answer and source. Citation requirements are particularly effective because Claude must find the specific text supporting each claim.

---

**Q25.** A developer wants to test whether their few-shot examples are actually helping performance. What is the correct methodology?

A) Add more examples if performance seems low  
B) A/B test: run the same evaluation set with (a) zero-shot, (b) current few-shot, (c) variations of few-shot — measure accuracy on each; compare to establish whether examples help, hurt, or are neutral for each input type  
C) Use user feedback as the metric  
D) Trust that more examples always help  

**Answer: B**  
**Explanation:** Empirical testing is the only reliable way to know if few-shot examples help. Sometimes zero-shot outperforms few-shot (examples introduce wrong anchors). Sometimes examples help enormously. The A/B methodology with a consistent evaluation set gives objective data. This is the correct engineering approach to prompt optimization — measure, don't assume.

---

## Score: /25 | Pass: 19/25 (75%)
