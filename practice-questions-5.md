# Practice Questions 5 — MCP Integration, Team Setups & Claude Code Workflows

> Domain 2 deep-dive: MCP server config, team/org deployment, monorepos, permission models in real scenarios.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A startup has 4 developers. They want a shared MCP server that connects Claude Code to their PostgreSQL database for all developers. The database has dev, staging, and production schemas. What is the correct MCP configuration strategy?

A) One MCP config pointing to production — it has the most complete data  
B) One MCP config per developer pointing to their local DB  
C) A shared SSE-transport MCP server pointing to the dev database; each developer's environment variable sets the connection string for their dev instance  
D) MCP cannot connect to databases  

**Answer: C**  
**Explanation:** Shared teams should use a remote SSE-transport MCP server. The connection string should reference environment variables (`${DATABASE_URL}`) so each developer's local environment points to their own dev instance. Never point shared development tooling at production. The MCP server itself is shared; the target database varies per environment variable.

---

**Q2.** A developer configures an MCP server in their project's `.claude/mcp_config.json` with `"api_key": "sk-live-abc123"` hardcoded. The file is committed to Git. What are the two immediate consequences?

A) The API key is now in git history (permanent unless history rewrite) and is visible to all repo access  
B) The MCP server stops working  
C) The API key is encrypted automatically by Git  
D) Only the developer can see the key  

**Answer: A**  
**Explanation:** Two consequences: (1) The key is now visible to anyone with repo access. (2) Even if deleted, it's in git history — the key must be rotated immediately. The fix: use environment variable reference `"${API_KEY}"` in mcp_config.json. Add `mcp_config.json` to `.gitignore` if it contains any environment-specific values (or use a template file with only `${VAR}` references that is committed).

---

**Q3.** A company runs a single MCP server with 40 different tools accessible to all 200 employees using Claude Code. Tool selection accuracy is poor. What architectural change addresses this?

A) Better tool descriptions  
B) Separate MCP servers by function/team (e.g., HR MCP, Engineering MCP, Finance MCP) so each team's Claude Code instance only loads the relevant 4-8 tools  
C) Use a more powerful model  
D) Add a tool router in front of the MCP server  

**Answer: B**  
**Explanation:** This is least privilege + tool focus applied at the MCP server level. 40 tools create selection noise. Team-scoped MCP servers expose only relevant tools. Engineering sees code tools; HR sees people tools; Finance sees financial tools. This dramatically improves selection accuracy and reduces the blast radius of a compromised or misconfigured tool.

---

**Q4.** An organization wants to onboard Claude Code for all engineers but requires: (1) no access to production systems, (2) all Claude Code sessions logged to a central audit system, (3) engineers can add personal slash commands. Which configuration structure achieves all three?

A) User-level CLAUDE.md only  
B) Org-level baseline CLAUDE.md + project-level CLAUDE.md for repo specifics + user-level CLAUDE.md for personal commands; org-level sets production access restrictions and audit logging requirements  
C) One global CLAUDE.md for all users and projects  
D) Environment variables only  

**Answer: B**  
**Explanation:** Layered configuration: org/project baseline (forbidden: production, required: audit logging) committed to all repos; project-level for repo specifics; user-level for personal preferences. This satisfies all three requirements without imposing personal preferences on others or preventing legitimate personal customization.

---

**Q5.** A developer has an MCP server for GitHub configured in their user-level `~/.claude/mcp_config.json`. A project they work on also has a GitHub MCP server in its project-level config pointing to a different GitHub organization. Which MCP server is used when working in that project?

A) User-level always takes precedence  
B) Both servers are available; Claude Code can use tools from both  
C) Project-level overrides user-level entirely  
D) Neither — conflict causes MCP to fail  

**Answer: B**  
**Explanation:** MCP servers from different scopes are additive, not conflicting. Both the user-level and project-level GitHub MCP servers are available. Tools from both are accessible. This allows users to have personal tools (their own repos) AND project tools (project's GitHub organization) available simultaneously.

---

**Q6.** A CLAUDE.md references an MCP tool: "Use the `jira-mcp` tool to create tickets for any bugs found." The MCP server is properly configured. During a code review session, Claude Code doesn't create any Jira tickets despite finding 3 bugs. What is the likely issue?

A) The MCP server is offline  
B) The instruction in CLAUDE.md is conditional ("for any bugs found") but Claude Code didn't classify the findings as "bugs" — it may have called them "issues" or "improvements"; make the trigger criteria more explicit  
C) Claude Code cannot use MCP tools during code review  
D) Jira MCP requires manual activation  

**Answer: B**  
**Explanation:** Trigger criteria must match Claude Code's actual classifications. "Bugs found" may not match what Claude Code calls issues in a code review context. Better instruction: "After completing any code review, for each identified defect, security issue, or behavioral error (not style issues), create a Jira ticket using `jira-mcp` tool with severity and description."

---

**Q7.** An organization deploys an SSE-based MCP server for internal documentation search. 6 months later, the server is regularly unavailable, breaking developer workflows. What CLAUDE.md change creates better resilience?

A) Add a fallback instruction: "If the documentation MCP tool is unavailable, use your built-in knowledge about the tech stack to answer, and note that you couldn't verify against current docs"  
B) Remove MCP tool references from CLAUDE.md  
C) Switch to stdio transport  
D) Increase the server's timeout  

**Answer: A**  
**Explanation:** CLAUDE.md should define fallback behavior for when tools are unavailable. "Use your built-in knowledge + note the limitation" is graceful degradation — Claude Code remains useful even when the MCP server is down. Without fallback instructions, unavailable tools cause Claude Code to halt or produce unhelpful responses.

---

**Q8.** A CLAUDE.md says: "Available MCP Tools: `github` for PR management, `postgres` for DB queries, `slack` for notifications." A developer asks Claude Code: "What tables does our users table join to?" Which tool should Claude Code use?

A) `github` — it might be in the codebase  
B) `postgres` — query the database schema  
C) `slack` — ask the team  
D) None — use built-in knowledge  

**Answer: B**  
**Explanation:** This is a database schema question — `postgres` (database) is the correct tool. The schema is ground truth; built-in knowledge would be a guess. This illustrates why CLAUDE.md tool descriptions should specify when to use each tool. The implicit answer is obvious here, but good CLAUDE.md explicitly maps question types to tools.

---

**Q9.** A team runs Claude Code in a CI/CD pipeline to automatically review PRs. The Claude Code instance runs in a containerized environment. Which CLAUDE.md considerations are specific to this automated (non-interactive) use case?

A) None — CLAUDE.md is the same for interactive and automated use  
B) In automated mode: no interactive prompts are possible; CLAUDE.md must specify default decisions for situations where Claude Code would normally ask; output must be machine-parseable (e.g., JSON) for downstream pipeline steps  
C) Automated use doesn't support CLAUDE.md  
D) Use user-level CLAUDE.md only in automated environments  

**Answer: B**  
**Explanation:** Automated Claude Code cannot pause for human input. CLAUDE.md for automated contexts must: (1) define all decisions explicitly (no "ask if unsure"), (2) specify machine-readable output formats that downstream pipeline steps can parse, (3) define clear pass/fail criteria. Interactive CLAUDE.md often relies on developer judgment for edge cases — automated doesn't have that.

---

**Q10.** An MCP server provides a `run_query` tool that executes SQL against the production database. Claude Code is configured for developer use. What CLAUDE.md instruction should accompany this tool?

A) No special instruction needed  
B) "The `run_query` tool connects to production database. Only use it for read operations (SELECT). Never use it for INSERT, UPDATE, DELETE, or DDL statements. Always EXPLAIN ANALYZE the query before running it if it touches more than one table."  
C) "Developers can use run_query freely"  
D) "Only senior developers can use run_query"  

**Answer: B**  
**Explanation:** Production database access requires explicit safety constraints in CLAUDE.md. These constraints should: (1) clarify the environment (production), (2) restrict to safe operations (SELECT only), (3) require query analysis before execution. CLAUDE.md cannot enforce these programmatically but provides clear behavioral guidance.

---

**Q11.** A company wants all Claude Code sessions to start with the same context: the company's engineering principles, tech stack, and code standards. These are 3,000 tokens of stable content. What is the most efficient way to achieve this?

A) Include all 3,000 tokens in every project's CLAUDE.md  
B) A company-wide "base" CLAUDE.md template that all project-level CLAUDE.md files reference and extend, with prompt caching on the base content  
C) Add the content to every developer's user-level CLAUDE.md  
D) Send it as the first message in every session  

**Answer: A**  
**Explanation:** While B is architecturally elegant, in practice Claude Code uses CLAUDE.md files directly. Including the standard content in project-level CLAUDE.md (perhaps via a tool that generates CLAUDE.md from a template) is the most practical approach. Adding it to every developer's user-level file would be hard to maintain. A is correct for the context of how CLAUDE.md actually works.

---

**Q12.** A developer uses Claude Code in a Python project and a Go project simultaneously. The Python project's CLAUDE.md says "use single quotes for strings." The Go project's CLAUDE.md says "use double quotes for strings." When switching between projects, Claude Code should:

A) Apply the most recently read CLAUDE.md to both projects  
B) Apply each project's CLAUDE.md to its own context; Claude Code uses the CLAUDE.md of the project it's currently working in  
C) Use user-level settings for all projects  
D) Ask the developer which applies  

**Answer: B**  
**Explanation:** CLAUDE.md scope is project-specific. When working in the Python project, Python's CLAUDE.md applies. When in the Go project, Go's CLAUDE.md applies. Claude Code respects the project context it's operating in. This is the fundamental value of project-level configuration — each project has its own standards.

---

**Q13.** An organization has a "golden path" toolchain: GitLab for code, Jira for issues, Confluence for docs, Datadog for monitoring. They want Claude Code to use all four. How should this be documented in CLAUDE.md?

A) List all four tools in a "Tools" section with: what each tool is used for, when to use it (specific triggers), and how to reference them by MCP tool name  
B) Just list the tool names  
C) Link to the internal wiki  
D) This requires custom Claude Code training  

**Answer: A**  
**Explanation:** Each tool needs: (1) what it is, (2) when to use it (trigger conditions), (3) MCP tool name. Example: "Use `jira-mcp` to create tickets when you identify bugs or tasks during code review. Use `confluence-mcp` to check existing documentation before adding new code patterns. Use `datadog-mcp` to check current error rates when debugging performance issues."

---

**Q14.** A team discovers that two different projects' CLAUDE.md files give contradictory instructions about error handling patterns (one uses Result types, one throws exceptions). Both are correct for their respective languages. What does this reveal and what should be done?

A) One team is wrong; standardize all error handling  
B) This is expected — different projects have different conventions; CLAUDE.md is project-specific precisely to accommodate this; no change needed  
C) Merge the two into a global standard  
D) Escalate to the tech lead  

**Answer: B**  
**Explanation:** Project-level CLAUDE.md files are intentionally independent. A Go project using error return values and a Java project using exceptions are both correct for their language. CLAUDE.md's project scope means each project's conventions apply within that project. Inter-project contradictions are not a problem — they're a feature of project-scoped configuration.

---

**Q15.** An MCP server tool called `search_codebase` is slow (avg 3 seconds per call). Claude Code calls it 15 times in a single session. What CLAUDE.md instruction can reduce unnecessary calls?

A) "Avoid searching the codebase"  
B) "Before calling `search_codebase`, check if you already have the information in context from a previous search in this session. Cache search results mentally and reuse them. Only call `search_codebase` when you need new information not already in the current context."  
C) "Call search_codebase only once per session"  
D) "Use grep instead of search_codebase"  

**Answer: B**  
**Explanation:** CLAUDE.md can guide Claude Code's tool usage efficiency. Instructing it to reuse in-context information before calling a slow tool reduces redundant calls. This is CALM-A applied to tool calls: don't re-fetch what's already available. Good CLAUDE.md includes not just WHEN to use tools but how to use them efficiently.

---

**Q16.** A developer writes a CLAUDE.md slash command: `/git-summary`. The command should summarize what the last 5 commits did. Which definition is BEST?

A) "/git-summary: Summarize git history"  
B) "/git-summary: Run `git log --oneline -5` and for each commit, provide: (1) commit hash (short), (2) one-sentence description of what changed, (3) which files were primarily affected. Output as a Markdown table with columns: Hash | Summary | Primary Files."  
C) "/git-summary: Use git to summarize changes"  
D) "/git-summary: Show recent commits"  

**Answer: B**  
**Explanation:** Complete slash command definition: specifies the exact command to run, the exact information to extract per commit, and the exact output format (Markdown table with specific columns). This produces consistent, useful output every time. The other options leave critical decisions to Claude's interpretation.

---

**Q17.** An MCP server for a team's internal wiki has been running for a year. The wiki has grown from 200 to 2,000 pages. Claude Code now sometimes retrieves outdated or incorrect information from it. What CLAUDE.md addition improves quality?

A) "The internal wiki MCP is available. Use it for reference."  
B) "When using the `wiki-mcp` tool, always note the page's last-updated date in your response. If the page is older than 6 months and concerns infrastructure or APIs, flag it as 'may be outdated' and recommend verifying against current code."  
C) Increase the MCP server's timeout  
D) Add more pages to the wiki  

**Answer: B**  
**Explanation:** CLAUDE.md can instruct Claude Code to critically evaluate tool output quality. For a growing, potentially stale wiki, instructing Claude Code to surface the freshness of information helps developers make informed decisions. Tool outputs should be treated as potentially outdated, especially for fast-moving technical content.

---

**Q18.** A security engineer reviews CLAUDE.md files across the organization. They find one that says: "You can execute any shell command the developer asks for." What is the risk?

A) No risk — developers can already run shell commands themselves  
B) Overly broad execution permission means Claude Code can run ANY command including destructive ones (`rm -rf`), without the developer consciously thinking through the implications; permission should specify allowed commands explicitly  
C) Shell commands require user-level configuration  
D) This is standard practice  

**Answer: B**  
**Explanation:** "Execute any shell command" is dangerously broad. Claude Code interprets "any" literally. A developer asking to "clean up temp files" might trigger a broader deletion than intended if Claude Code runs a generic cleanup command. Allowed operations should be explicit: specific commands are listed, or commands are scoped to safe operations (no rm, no chmod, no network commands without approval).

---

**Q19.** A team wants Claude Code to automatically format code after every change. Which CLAUDE.md instruction is correct?

A) "Format code when it looks messy."  
B) "After writing or modifying any file, run the appropriate formatter: `prettier --write` for JS/TS, `black` for Python, `gofmt` for Go. Treat formatting as part of completing any code change."  
C) "Code should be well-formatted."  
D) "Use the IDE's built-in formatter."  

**Answer: B**  
**Explanation:** Effective CLAUDE.md instructions specify: (1) the trigger (after writing/modifying), (2) the exact command per language/type, (3) the framing (formatting is part of completing a change). This makes code quality automatic. "Format when it looks messy" is subjective. "Well-formatted code" is unverifiable.

---

**Q20.** Two developers on the same team have user-level CLAUDE.md files with conflicting indentation preferences (tabs vs. spaces). The project CLAUDE.md says "2 spaces." When Claude Code writes code in the shared project:

A) It uses the first developer's preference  
B) It uses 2 spaces — the project-level CLAUDE.md overrides both user-level preferences for this project  
C) It alternates between tabs and spaces  
D) It asks the developer  

**Answer: B**  
**Explanation:** Project-level overrides user-level. This is the correct behavior — team standards defined in project CLAUDE.md win over personal preferences for that project. This is exactly why project-level configuration exists: to ensure consistency across all team members working in that codebase.

---

**Q21.** A CLAUDE.md for a microservices project says: "When adding a new service, follow the template in `services/template/`." Claude Code adds a service but skips the template. What likely caused this?

A) The template directory doesn't exist  
B) The CLAUDE.md instruction doesn't specify WHAT the template contains, WHICH files must be copied, or WHERE to put them; "follow the template" is too vague for autonomous execution  
C) Claude Code doesn't read template files  
D) The service was added in a subdirectory that has its own CLAUDE.md  

**Answer: B**  
**Explanation:** "Follow the template" requires Claude Code to understand what "following" means. Better: "When creating a new service: (1) Copy all files from `services/template/` to `services/{service-name}/`. (2) Replace all `TEMPLATE_NAME` strings with your service name. (3) Add the service to `docker-compose.yml` and `k8s/services/`. (4) Run `npm install` in the new directory."

---

**Q22.** An organization wants to prevent Claude Code from ever pushing to git remote branches. How should this be configured?

A) Remove git from Claude Code  
B) In the organization-wide project CLAUDE.md template: "Forbidden: git push, git push --force, git push origin, or any variation that uploads to a remote repository. All remote operations must be initiated by the developer manually."  
C) Configure git server permissions  
D) Use a pre-push hook  

**Answer: B**  
**Explanation:** CLAUDE.md Forbidden section is the correct control. The instruction must be explicit about variations (git push, git push --force, git push origin) because Claude may use different forms. This aligns with the operational safety principle: pushes affect shared systems and are hard to reverse (especially force pushes), warranting an explicit CLAUDE.md prohibition.

---

**Q23.** A team's MCP server connects to their Kubernetes cluster. The CLAUDE.md says: "Use `k8s-mcp` to check pod health when debugging." What additional constraint should be in CLAUDE.md given the production risk?

A) No additional constraint needed  
B) "The `k8s-mcp` tool connects to the production Kubernetes cluster. ONLY use it for READ operations: kubectl get, kubectl describe, kubectl logs. NEVER use it for: kubectl delete, kubectl apply, kubectl exec, or kubectl scale. These commands must be executed manually."  
C) "Use k8s-mcp carefully"  
D) "k8s-mcp is for advanced users only"  

**Answer: B**  
**Explanation:** Any tool with production access needs explicit read-only constraints in CLAUDE.md. The instruction must list the ALLOWED commands (get, describe, logs) AND the FORBIDDEN commands (delete, apply, exec, scale) — both lists reduce ambiguity. This protects against Claude Code accidentally running `kubectl delete pod` when debugging.

---

**Q24.** A CLAUDE.md uses the phrase "when appropriate." Example: "Run tests when appropriate." A developer notices that Claude Code rarely runs tests. Why and how should this be fixed?

A) Claude Code runs tests too infrequently by design  
B) "When appropriate" leaves the decision entirely to Claude's judgment; Claude defaults to "not appropriate" to avoid delays; specify explicit triggers: "Run the full test suite after modifying any .ts file, after adding a new function, and before marking any task as complete."  
C) Add more emphasis: "Run tests when VERY appropriate"  
D) "When appropriate" is clear enough  

**Answer: B**  
**Explanation:** Conditional instructions with undefined trigger conditions are effectively optional to Claude Code. It will interpret "when appropriate" conservatively (skipping tests to be faster). Replace conditions with explicit, enumerable triggers. If you want tests to always run, say "always" and specify the command.

---

**Q25.** A developer has been using Claude Code for 3 months and notices it's learned their preferences through conversation context. They're worried this "learning" won't persist after their session ends. What is the correct understanding and fix?

A) Claude Code permanently remembers all preferences from all sessions  
B) Claude Code has no cross-session memory by default; preferences from conversations don't persist; fix: explicitly add learned preferences to user-level CLAUDE.md after each session where you discover a preference that works well  
C) Claude Code stores preferences in a local SQLite database  
D) Session memory persists for 7 days  

**Answer: B**  
**Explanation:** Claude Code (and Claude generally) has no cross-session memory. Every new session starts fresh. The correct workflow: when you discover through a session that Claude Code behaves better with a specific instruction, add that instruction to CLAUDE.md so it's always present. CLAUDE.md is the persistence mechanism for learned preferences.

---

## Score: /25 | Pass: 19/25 (75%)
