# Practice Questions 4 — Claude Code Configuration: CLAUDE.md Design

> Domain 2 deep-dive: What goes in CLAUDE.md, scope hierarchy, structure, content quality, common mistakes.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A team of 8 developers all have different user-level CLAUDE.md files. The team lead wants to ensure all developers get the same code review behavior when working on the shared project. What is the correct approach?

A) Ask all developers to manually update their user-level CLAUDE.md  
B) Create a project-level CLAUDE.md with the code review behavior and commit it to the repository  
C) Distribute a standardized user-level CLAUDE.md template  
D) Use an environment variable to set CLAUDE.md path  

**Answer: B**  
**Explanation:** Team conventions belong in project-level CLAUDE.md committed to version control. This ensures every developer working on the project gets the same behavior automatically, without manual configuration. User-level configurations are personal and can't be enforced across a team.

---

**Q2.** A CLAUDE.md file contains: "Always be helpful and productive. Try to add value in every interaction." What is wrong with this instruction?

A) It is too short  
B) It is not actionable — "be helpful" provides no concrete behavioral guidance; Claude will interpret this vaguely; replace with specific, observable behaviors  
C) It is too positive  
D) Instructions should not start with "Always"  

**Answer: B**  
**Explanation:** Vague instructions in CLAUDE.md produce inconsistent behavior. "Be helpful" doesn't tell Claude what to do when "helpful" is ambiguous. Replace with specific, testable instructions: "When the user asks to fix a bug, also add a regression test. When making a PR, include a summary of changes."

---

**Q3.** A CLAUDE.md has the Forbidden section: "Do not access production systems." During a debugging session, a developer asks Claude Code to "check the production logs to see what's happening." What should Claude Code do?

A) Access production logs since the developer explicitly requested it  
B) Refuse the request, citing the forbidden rule, and suggest alternative debugging approaches  
C) Access the logs but warn the developer  
D) Ask for a password before accessing production  

**Answer: B**  
**Explanation:** Forbidden rules in CLAUDE.md are hard constraints. An explicit developer request in the session does not override CLAUDE.md rules — that's the entire point of having them. Claude Code should explain the restriction and offer alternatives (e.g., "I can't access production. Can you export the relevant logs locally?").

---

**Q4.** Which CLAUDE.md structure section should contain: "Run `./scripts/setup-dev.sh` before first run. Start dev server with `npm run dev`. Run tests with `npm test`. Build with `npm run build`."?

A) Overview  
B) Architecture  
C) Common Tasks  
D) Allowed Operations  

**Answer: C**  
**Explanation:** Common Tasks section is for operational commands — how to run, test, build, deploy. This is the section Claude Code references when it needs to execute project operations. Architecture is for design patterns. Overview is project description. Allowed Operations is for permission grants.

---

**Q5.** A project CLAUDE.md correctly lists the tech stack, conventions, and operations. However, Claude Code never uses the available MCP Jira tool even when working on bugs. What is the likely cause?

A) The MCP server is not properly installed  
B) CLAUDE.md doesn't mention the Jira MCP tool and when to use it  
C) Jira MCP requires user-level configuration  
D) MCP tools cannot be used during code editing  

**Answer: B**  
**Explanation:** MCP tools must be documented in CLAUDE.md to be reliably used. Even if the MCP server is properly configured in mcp_config.json, Claude Code won't know to use it for relevant tasks unless CLAUDE.md explicitly says: "Use the `jira` MCP tool to check for related issues before fixing bugs."

---

**Q6.** A security-conscious team wants to prevent Claude Code from ever modifying files in the `infrastructure/terraform/` directory without explicit confirmation. The project CLAUDE.md currently says nothing about this directory. What is the result?

A) Claude Code will never touch that directory by default  
B) Claude Code may modify files in that directory if instructed or if it determines changes are needed; without an explicit forbidden rule, the directory is implicitly accessible  
C) Claude Code always asks for confirmation before modifying any file  
D) Terraform files are automatically protected  

**Answer: B**  
**Explanation:** In CLAUDE.md, "not mentioned" means "not restricted." If you want Claude Code to treat a directory as off-limits or require confirmation, you must explicitly state this in the Forbidden section. Default behavior allows Claude Code to access any file it has the technical ability to access.

---

**Q7.** A CLAUDE.md file for a Node.js project says: "Use async/await, not callbacks. Use TypeScript strict mode. Follow the existing error handling pattern in `src/utils/errors.ts`." This content best exemplifies which PRECISE element?

A) P — Persona  
B) E — Explicit instructions  
C) C — Context  
D) I — Input format  

**Answer: B**  
**Explanation:** PRECISE-E: Explicit instructions. This content provides specific, actionable, verifiable behavioral rules. Claude Code can check whether it's following these rules. "Use async/await" is explicit. "Follow the pattern in this file" is explicit (points to a concrete example). This is high-quality CLAUDE.md content.

---

**Q8.** A developer adds this to CLAUDE.md: "Database password: postgres_password_123. API key: sk-abc123xyz." 30 minutes later, the company's security scanner triggers an alert. Why?

A) The scanner is overly sensitive  
B) CLAUDE.md is committed to version control; credentials are now exposed in the git history, visible to all team members and potentially public  
C) Claude Code stored these credentials in a log file  
D) The credentials are weak  

**Answer: B**  
**Explanation:** CLAUDE.md is version-controlled documentation. Credentials placed there are: (1) Visible to all repository access. (2) Permanently in git history even after deletion. (3) Potentially exposed if the repo is ever made public or breached. Always use environment variables for credentials, referenced as `${VAR_NAME}` in configurations.

---

**Q9.** A slash command is defined as `/pr-review`. Two months later, a developer finds that the command sometimes checks security, sometimes checks tests, sometimes checks style — depending on context. What CLAUDE.md improvement is needed?

A) Rename the command to be more specific  
B) Add explicit scope to the command definition: exactly what it checks, in what order, and what format the output should follow  
C) Delete the command and rely on natural language  
D) Move the command to user-level CLAUDE.md  

**Answer: B**  
**Explanation:** Slash commands must have explicit, bounded definitions. Without specifying the exact checks and output format, Claude interprets "review" differently each time based on conversational context. A well-defined slash command is deterministic: "Check: (1) security vulnerabilities, (2) test coverage for modified code, (3) style against ESLint. Output: Markdown with a section per check."

---

**Q10.** A monorepo has root CLAUDE.md, and subdirectory CLAUDE.md files in `packages/api/`, `packages/frontend/`, and `packages/auth-service/`. The auth-service has: "Never modify authentication logic without running the full test suite." The root says: "Quick edits to fix typos can skip testing." Which rule applies in `packages/auth-service/`?

A) Root rule always wins  
B) Auth-service rule wins — more specific scope overrides more general scope  
C) Both rules apply simultaneously  
D) The developer's user-level CLAUDE.md decides  

**Answer: B**  
**Explanation:** Subdirectory scope overrides project scope for files within that directory. `packages/auth-service/CLAUDE.md` is more specific than the root — its rules win for files in that subdirectory. This is the correct and intended behavior: critical subdirectories can enforce stricter standards than the general project.

---

**Q11.** A developer wants to add a personal style preference to their CLAUDE.md: "When writing Python, use single quotes for strings." They also want this to apply in ALL their projects, not just one. Where does this belong?

A) In every project's CLAUDE.md they work on  
B) In their user-level `~/.claude/CLAUDE.md`  
C) In a global environment variable  
D) In their shell profile  

**Answer: B**  
**Explanation:** Personal preferences that should apply across all projects belong in user-level `~/.claude/CLAUDE.md`. This is exactly its purpose: developer-specific, cross-project preferences. Adding this to every project CLAUDE.md would require maintenance across all repos and would impose the preference on all team members.

---

**Q12.** A CLAUDE.md contains contradictory instructions: "Allowed: Modify any file in src/" AND "Forbidden: Modify configuration files in src/config/". Claude Code is working in `src/config/`. What happens?

A) Claude Code ignores all instructions due to contradiction  
B) Claude Code follows the more specific rule (Forbidden for src/config/) over the more general one  
C) Claude Code follows whichever rule appears last  
D) Claude Code asks for clarification  

**Answer: B**  
**Explanation:** When general and specific rules conflict, the more specific rule wins. "Forbidden: src/config/" is more specific than "Allowed: src/". This mirrors how most permission systems work — specific exceptions override general grants. This is why CLAUDE.md design requires careful review: contradictions with wrong specificity can override intended rules.

---

**Q13.** A project CLAUDE.md has 47 rules. A developer notices Claude Code is ignoring some of them. What is the likely cause and fix?

A) Claude Code only reads the first 10 rules  
B) Too many rules create noise; Claude may miss or deprioritize rules buried in a long list; organize rules by priority, use clear section headers, and trim redundant or obvious rules  
C) CLAUDE.md has a character limit  
D) Some rules require a special keyword to be activated  

**Answer: B**  
**Explanation:** Quality over quantity in CLAUDE.md. A bloated CLAUDE.md with 47 rules makes it harder to find and apply the important ones. Best practice: group rules by category, put the most critical ones first, eliminate redundancy, and keep the file focused. A concise, well-organized CLAUDE.md is more effective than an exhaustive one.

---

**Q14.** A team wants to add a `/security-audit` slash command that checks code for OWASP Top 10 vulnerabilities. The command should ONLY analyze staged git changes, not the entire codebase. How should this constraint be expressed in the command definition?

A) "Run a security audit."  
B) "Run a security audit of staged git changes only (`git diff --staged`). Do not scan files not included in the current staged changes. Report: vulnerability name, location, severity (Critical/High/Medium/Low), and suggested fix."  
C) "Security audit — see security checklist."  
D) This cannot be expressed in a slash command  

**Answer: B**  
**Explanation:** Good slash command definitions are complete and bounded: they define (1) the scope (staged changes only), (2) what command to use to get the input (`git diff --staged`), (3) what to exclude (unchanged files), and (4) the exact output format. This produces consistent, reliable behavior every time the command is invoked.

---

**Q15.** A CLAUDE.md instruction says: "When you encounter a failing test, fix the test." A developer argues this is ambiguous — "fix" could mean fix the code under test OR fix the test assertions. How should this be rewritten?

A) "When you encounter a failing test, investigate why it's failing. Fix the production code if the test expectation is correct. Only modify the test if it contains a factual error (wrong expected value). Never delete failing tests."  
B) "Fix failing tests quickly."  
C) "Ask the developer before fixing any test."  
D) The instruction is fine as written  

**Answer: A**  
**Explanation:** CLAUDE.md instructions must be unambiguous. The rewrite specifies: (1) what to investigate first, (2) when to fix production code vs. test, (3) the specific exception (factual error in test assertion), (4) what's explicitly forbidden (deleting tests). Each decision point has a clear rule.

---

**Q16.** Which of the following demonstrates the BEST use of CLAUDE.md for a React project?

A) "This is a React project. Build good components."  
B) "Frontend stack: React 18 + TypeScript. State: Zustand. Styling: Tailwind CSS. Components go in `src/components/`. Follow compound component pattern (see `src/components/Button/` as example). New components need a Storybook story and a unit test in `__tests__/`."  
C) "Make the code work. Follow best practices."  
D) "Use React."  

**Answer: B**  
**Explanation:** Best CLAUDE.md content is: specific (exact versions), actionable (exactly where things go), pointed at examples (Button as reference), and includes completion criteria (Storybook + unit test). It eliminates ambiguity about stack, structure, and definition of done. Options A, C, D are too vague to produce consistent behavior.

---

**Q17.** A CLAUDE.md forbidden rule says: "Do not run destructive database operations." Claude Code is asked to "clean up duplicate records." It runs `DELETE FROM users WHERE id NOT IN (SELECT MIN(id) FROM users GROUP BY email)`. Did it violate the rule?

A) No — this is a legitimate maintenance operation  
B) Yes — DELETE is a destructive database operation; the rule is absolute and DELETE is explicitly destructive; Claude Code should propose the SQL, show a preview of affected rows, and wait for explicit confirmation  
C) No — the forbidden rule only covers DROP TABLE statements  
D) Ambiguous — depends on row count  

**Answer: B**  
**Explanation:** DELETE is a destructive operation. "Clean up duplicates" sounds benign but executes a potentially large DELETE. The forbidden rule should have been respected. Best practice: Claude Code proposes the SQL and shows the expected affected row count (`SELECT COUNT(...)`) before executing DELETE. The instruction should have been "do not execute destructive operations without showing a preview first."

---

**Q18.** A company has 3 environments: dev, staging, production. They want Claude Code available for dev and staging work but forbidden from touching production. How should this be configured?

A) One CLAUDE.md with a note saying "avoid production"  
B) Separate project-level CLAUDE.md files for dev/staging (where Claude Code is allowed) vs. production (which has strict forbidden rules and ideally Claude Code is not configured at all)  
C) User-level CLAUDE.md with environment detection  
D) A single CLAUDE.md covering all environments  

**Answer: B**  
**Explanation:** Environment isolation is critical. Production should have the most restrictive CLAUDE.md (or ideally Claude Code should not be enabled in production at all). Dev/staging can have appropriate permissions for development work. Environment-specific configs prevent "oops, I ran that in production" scenarios.

---

**Q19.** A developer reads the CLAUDE.md for a project they've just joined. They notice it's 800 lines long, last updated 2 years ago, and contains references to deprecated libraries. What should they do with it?

A) Trust it completely — it was written by the original team  
B) Treat stale CLAUDE.md content as potentially wrong; audit it against the current codebase; remove outdated sections; update to reflect current stack  
C) Delete it and start fresh  
D) Leave it — it's not their responsibility  

**Answer: B**  
**Explanation:** CLAUDE.md should be maintained like code. Stale instructions pointing to deprecated libraries or outdated patterns can cause Claude Code to make incorrect decisions. New team members are actually well-positioned to audit CLAUDE.md for accuracy since they bring fresh eyes. Treating it as authoritative when it's outdated is more dangerous than having no CLAUDE.md.

---

**Q20.** A CLAUDE.md has a slash command `/deploy-staging`. The command definition doesn't specify what "staging" means — the URL, the deployment command, or which services to deploy. A developer runs it and Claude Code deploys to the wrong service. What was wrong?

A) The deployment should not be in CLAUDE.md at all  
B) The slash command lacks concrete specifics: which deployment command (`./scripts/deploy.sh staging`), which services (all, or specific list), what success looks like, and confirmation step before execution  
C) Staging deployments should require MCP authentication  
D) The command name was too short  

**Answer: B**  
**Explanation:** Slash commands for deployment MUST be fully specified: exact command to run, which environment (specific URL/target), which services are included, what the expected output looks like on success, and a confirmation step before execution. Vague deployment commands are dangerous — a wrong deployment can disrupt services.

---

**Q21.** A CLAUDE.md says "Use the company linting rules." But doesn't specify where the linting config is or how to run the linter. Claude Code guesses and uses a generic config. What fix makes this actionable?

A) "Use good code style"  
B) "Run `npm run lint` (uses `.eslintrc.json` in project root). Fix all errors. Warnings are acceptable but try to reduce them. Do not disable lint rules with eslint-disable comments without team approval."  
C) "Follow linting rules — see the team wiki"  
D) This is fine as written  

**Answer: B**  
**Explanation:** CLAUDE.md content must be self-contained and actionable. "Company linting rules" requires knowing where they are and how to run them. The improved version: specifies the exact command, the config file location, the acceptance criteria (fix errors, reduce warnings), and one explicit constraint (no disabling rules). Claude Code can now act on this without guessing.

---

**Q22.** Which CLAUDE.md instruction is MOST effective at preventing Claude Code from accidentally breaking the build?

A) "Be careful not to break the build."  
B) "After every code change, run `npm run build` and `npm test`. Only consider a task complete if both pass. If they fail, fix the failure before stopping."  
C) "Make sure the code works."  
D) "Run tests before submitting changes."  

**Answer: B**  
**Explanation:** The most effective instruction specifies the exact commands to run, when to run them (after every change), what constitutes "done" (both passing), and what to do if they fail (fix before stopping). This creates a verifiable completion criterion. "Be careful" and "make sure it works" are unverifiable platitudes.

---

**Q23.** A project uses GitHub Actions for CI. The team wants Claude Code to run the same checks locally that CI runs. Where should the CI check commands be documented?

A) In the GitHub Actions YAML files (already there)  
B) In CLAUDE.md Common Tasks section: the specific commands that replicate CI checks locally, so Claude Code uses them as the standard quality gate  
C) In the README only  
D) CI checks cannot be replicated locally  

**Answer: B**  
**Explanation:** CLAUDE.md should document the local equivalents of CI checks. When Claude Code finishes a task, it should run the same checks CI will run. Without this in CLAUDE.md, Claude Code may run different/fewer checks than CI, leading to PRs that pass local checks but fail CI. CLAUDE.md is the authoritative source of "definition of done" for Claude Code.

---

**Q24.** A team debates: should CLAUDE.md be reviewed in code review like regular code? Why or why not?

A) No — CLAUDE.md is documentation, not code  
B) Yes — CLAUDE.md directly shapes AI behavior; poor instructions cause bad outputs; it should be reviewed with the same rigor as any code that affects system behavior  
C) No — only the tech lead should modify CLAUDE.md  
D) Yes, but only for security-related sections  

**Answer: B**  
**Explanation:** CLAUDE.md is effectively a program that controls Claude Code's behavior. Poorly written CLAUDE.md = poor Claude Code behavior. Changes should be reviewed for: accuracy, consistency, security implications, and alignment with team standards. The bar for review should be higher for the Forbidden section (security implications) but the entire file deserves thoughtful review.

---

**Q25.** A CLAUDE.md instruction says: "If you're not sure about something, ask." In practice, Claude Code interrupts constantly with questions, slowing down the developer. What is the better instruction?

A) Remove the instruction entirely  
B) "Proceed with your best judgment for low-stakes decisions (formatting, variable names, test structure). Ask for clarification only for: (a) architectural decisions, (b) changes that affect more than 3 files, (c) actions in the Forbidden list that a user has explicitly requested."  
C) "Never ask questions."  
D) "Ask only once per session."  

**Answer: B**  
**Explanation:** "Ask if unsure" is too broad — Claude Code second-guesses everything. The improved version sets specific triggers for when to ask vs. when to proceed autonomously. This preserves the intent (don't make big decisions alone) while enabling the productive autonomous behavior that makes Claude Code valuable. Scope the interruptions to high-impact decisions.

---

## Score: /25 | Pass: 19/25 (75%)
