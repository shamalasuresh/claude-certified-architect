# Claude Certified Architect (CCA) — Foundations

> Complete end-to-end preparation material for the CCA Foundations exam.

---

## Exam at a Glance

| Parameter | Details |
|-----------|---------|
| Questions | 60 scenario-based, multiple choice |
| Duration | 120 minutes |
| Passing Score | 75% (weighted by domain) |
| Delivery | Online proctored |
| Validity | 2 years from passing date |

**Key principle:** Every question is scenario-based. You will never be asked "what does X mean?" — you will always be asked "given this situation, what is the best approach?"

---

## Domain Weighting

```
Domain 1 — Agentic Architecture      ████████████████████████████  27%
Domain 2 — Claude Code Configuration ████████████████████          20%
Domain 3 — Prompt Engineering        ████████████████████          20%
Domain 4 — Tool Design & MCP         ██████████████████            18%
Domain 5 — Context Management        ███████████████               15%
```

| Domain | Weight | Questions (approx.) |
|--------|--------|---------------------|
| 1. Agentic Architecture | 27% | ~16 |
| 2. Claude Code Configuration | 20% | ~12 |
| 3. Prompt Engineering | 20% | ~12 |
| 4. Tool Design & MCP | 18% | ~11 |
| 5. Context Management | 15% | ~9 |

---

## Study Material Index

### Core Domain Notes
| File | Domain | Weight |
|------|--------|--------|
| [domain-1-agentic-architecture.md](domain-1-agentic-architecture.md) | Agentic Architecture | 27% |
| [domain-2-claude-code-configuration.md](domain-2-claude-code-configuration.md) | Claude Code Configuration | 20% |
| [domain-3-prompt-engineering.md](domain-3-prompt-engineering.md) | Prompt Engineering | 20% |
| [domain-4-tool-design-mcp.md](domain-4-tool-design-mcp.md) | Tool Design & MCP | 18% |
| [domain-5-context-management.md](domain-5-context-management.md) | Context Management | 15% |

### Reference & Practice
| File | Purpose |
|------|---------|
| [frameworks-cheatsheet.md](frameworks-cheatsheet.md) | SPIDER, CALM, PRECISE quick reference |
| [practice-questions.md](practice-questions.md) | 60+ scenario questions with explanations |
| [practice-questions-1.md](practice-questions-1.md) | 60+ scenario questions with explanations |
| [practice-questions-2.md](practice-questions-2.md) | 60+ scenario questions with explanations |
| [practice-questions-3.md](practice-questions-3.md) | 60+ scenario questions with explanations |
| [practice-questions-4.md](practice-questions-4.md) | 60+ scenario questions with explanations |
| [practice-questions-5.md](practice-questions-5.md) | 60+ scenario questions with explanations |
| [practice-questions-6.md](practice-questions-6.md) | 60+ scenario questions with explanations |
| [practice-questions-7.md](practice-questions-7.md) | 60+ scenario questions with explanations |
| [practice-questions-8.md](practice-questions-8.md) | 60+ scenario questions with explanations |
| [practice-questions-9.md](practice-questions-9.md) | 60+ scenario questions with explanations |
| [practice-questions-10.md](practice-questions-10.md) | 60+ scenario questions with explanations |
| [practice-questions-11.md](practice-questions-11.md) | 60+ scenario questions with explanations |
| [practice-questions-12.md](practice-questions-12.md) | 60+ scenario questions with explanations |
| [practice-questions-13.md](practice-questions-13.md) | 60+ scenario questions with explanations |
| [practice-questions-14.md](practice-questions-14.md) | 60+ scenario questions with explanations |
| [practice-questions-15.md](practice-questions-15.md) | 60+ scenario questions with explanations |
| [practice-questions-16.md](practice-questions-16.md) | 60+ scenario questions with explanations |
| [practice-questions-17.md](practice-questions-17.md) | 60+ scenario questions with explanations |
| [practice-questions-18.md](practice-questions-18.md) | 60+ scenario questions with explanations |
| [practice-questions-19.md](practice-questions-19.md) | 60+ scenario questions with explanations |
| [practice-questions-20.md](practice-questions-20.md) | 60+ scenario questions with explanations |
| [mock-exam-1.md](mock-exam-1.md) | Full timed mock exam (60 questions) |
| [mock-exam-2.md](mock-exam-2.md) | Full timed mock exam (60 questions) |
| [mock-exam-3.md](mock-exam-3.md) | Full timed mock exam (60 questions) |
| [mock-exam-4.md](mock-exam-4.md) | Full timed mock exam (60 questions) |

---

## Critical Exam Tips

1. **The exam tests tradeoffs** — When two answers seem correct, choose based on the constraint in the question (budget vs latency vs reliability)
2. **Read the scenario carefully** — The size of the system, team, and budget are deliberate clues
3. **Domain 5 is the tie-breaker** — Context Management questions separate passing from failing scores
4. **SPIDER, CALM, PRECISE** — Memorize these frameworks; they appear directly in questions
5. **Don't ignore small domains** — 15% is 9 questions; getting them all wrong can fail you

---

## Key Frameworks Summary (Memorize These)

### SPIDER — Agentic Reliability
- **S**top on failure
- **P**reserve state
- **I**solate side effects
- **D**etermine retry strategy
- **E**scalate to human
- **R**eport outcomes

### CALM — Context Management
- **C**hunk long content
- **A**ggressively cache
- **L**imit conversation length
- **M**anage token budgets explicitly

### PRECISE — Prompt Engineering
- **P**ersona
- **R**ole
- **E**xplicit instructions
- **C**ontext
- **I**nput format
- **S**tyle
- **E**xpected output

---

## Common Mistakes (Don't Do These)

- Over-studying Agentic Architecture and neglecting Context Management
- Memorizing tool names without understanding *when* to use them
- Not practicing under timed conditions
- Treating questions as recall tests instead of decision problems
