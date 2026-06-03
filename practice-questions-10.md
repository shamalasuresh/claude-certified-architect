# Practice Questions 10 — MCP Transport, Authentication & Security

> Domain 4 deep-dive: stdio vs SSE transport decisions, OAuth, API key security, multi-tenant patterns, RBAC.  
> 25 questions | Recommended time: 45 minutes

---

**Q1.** A developer is choosing between stdio and SSE transport for an MCP server. The server will be used by a single developer on their local machine, integrating with local CLI tools. Which transport is correct?

A) SSE — it's more modern  
B) stdio — for single-user, local integration; no network overhead; simpler; no authentication required; appropriate for personal productivity tools  
C) Both are equivalent  
D) Neither — use HTTP REST instead  

**Answer: B**  
**Explanation:** stdio transport is designed for local, single-user, single-process scenarios: Claude Code on a developer machine integrating with local tools. No network binding, no authentication infrastructure needed, no port conflicts. SSE is for networked deployments serving multiple users. Match transport to deployment topology.

---

**Q2.** A company wants to deploy a shared MCP server accessible to all developers on their team (20 people). Which transport is required and why?

A) stdio — it's simpler  
B) SSE (Server-Sent Events) over HTTP/HTTPS — it enables a single server instance to serve multiple clients over the network; stdio only works for a single process; shared team servers require network transport  
C) WebSockets instead of SSE  
D) gRPC  

**Answer: B**  
**Explanation:** SSE transport enables: (1) Multiple concurrent clients. (2) Network access (multiple machines). (3) Authentication and authorization per client. (4) Centralized deployment and management. For a team of 20 developers, SSE is the only viable option. stdio is a one-to-one local connection, not a shared server.

---

**Q3.** A team deploys an SSE MCP server over plain HTTP on port 3000 in production. What is the critical security issue?

A) Port 3000 is not secure  
B) Plain HTTP transmits all data including API keys, tool inputs/outputs, and authentication tokens in plaintext; production SSE servers MUST use HTTPS/TLS to encrypt transport  
C) SSE should not be used in production  
D) The port should be 443  

**Answer: B**  
**Explanation:** Plain HTTP in production is a critical OWASP violation. Tool calls may include sensitive data (customer records, internal system credentials). Authentication tokens in plain HTTP are trivially interceptable. Production MCP servers require HTTPS with valid TLS certificates. This is non-negotiable for any production deployment.

---

**Q4.** An MCP server uses OAuth 2.0 for authentication. A client sends an access token as a URL query parameter: `?access_token=abc123`. What is the security problem?

A) OAuth should not be used with MCP  
B) Tokens in URL query parameters are logged in: server access logs, browser history, proxy logs, referrer headers — exposing the token to many systems; tokens must be sent in the Authorization header: `Authorization: Bearer abc123`  
C) The token format is wrong  
D) Query parameters are encrypted by HTTPS  

**Answer: B**  
**Explanation:** Even with HTTPS, URL parameters are logged and accessible in many places: web server logs, CDN logs, browser history, Referer headers in subsequent requests. Authorization tokens belong in the Authorization header where they are never logged by default. This is a classic OWASP API security mistake.

---

**Q5.** A developer stores MCP API keys as plain text in a configuration file committed to GitHub. Why is this a critical mistake?

A) Git doesn't support binary files  
B) Secrets in source code are a critical security vulnerability: once committed, they exist in git history even after removal; any person with repository access (or public GitHub access) can extract the key and use it without authorization  
C) Configuration files should be in a different format  
D) Only if the repository is public  

**Answer: B**  
**Explanation:** Secrets in source code is OWASP A02 (Cryptographic Failures) / A07 (Identification and Authentication Failures). Even in private repos: (1) git history preserves them forever. (2) Repo access leaks = secret leak. (3) Can be extracted from .git directory. Correct approach: environment variables, secret management systems (AWS Secrets Manager, HashiCorp Vault), .gitignore for .env files.

---

**Q6.** A multi-tenant MCP server serves customers from 3 different organizations. Each organization's data must be completely isolated. Which design correctly enforces tenant isolation?

A) Trust the `tenant_id` parameter passed by each client  
B) Extract tenant identity from the authenticated token (not from client-provided parameters); server-side validation of tenant identity prevents a client claiming another tenant's identity  
C) Use separate databases per tenant  
D) Both B and C  

**Answer: D**  
**Explanation:** Both are correct. B is the authorization principle: never trust client-provided tenant identity — extract it from server-validated tokens (JWT claims, session data). C is the data isolation principle: separate databases or schemas per tenant prevents data leaks via SQL bugs. Together they provide defense-in-depth: authentication layer + data layer isolation.

---

**Q7.** An MCP server needs to enforce that certain tools are only accessible to admin users. Where should this enforcement happen?

A) In the tool description: "Admin only"  
B) Server-side, in the tool handler: verify the authenticated user's role before executing the tool; refuse with a 403 Forbidden if the role check fails  
C) In the system prompt  
D) In Claude's instructions  

**Answer: B**  
**Explanation:** Authorization must be enforced server-side. Client-side or prompt-side controls are advisory, not enforced. A malicious client can ignore tool descriptions. The server must: (1) Authenticate the request. (2) Extract the user's role. (3) Check role against required permissions for the tool. (4) Return 403 if unauthorized. Server-side enforcement is the only real authorization.

---

**Q8.** A developer uses a single long-lived API key shared by all developers on a team for MCP authentication. What are the problems?

A) Long-lived keys are fine  
B) Shared keys cannot be individually revoked (removing one removes all), audit logs cannot attribute actions to specific individuals, and if one developer's machine is compromised, all access is compromised  
C) API key sharing is standard practice  
D) The key should be rotated quarterly  

**Answer: B**  
**Explanation:** Shared credentials violate the principle of individual accountability. Fix: each developer gets their own API key or OAuth token. Benefits: (1) Individual key revocation without disrupting others. (2) Audit logs show WHO performed each action. (3) Compromise of one account is contained. This is the non-repudiation principle — every action must be attributable to a specific identity.

---

**Q9.** An MCP server receives a tool call but the authentication token has expired. What should the server return?

A) A generic 500 error  
B) A 401 Unauthorized response with a clear error message that Claude can interpret and return to the user requesting re-authentication  
C) An empty response  
D) A 403 Forbidden  

**Answer: B**  
**Explanation:** 401 vs 403: 401 = "You are not authenticated (or your auth is invalid/expired)" → please authenticate. 403 = "You are authenticated, but not authorized for this resource." Token expiry is a 401 scenario. The error message should be Claude-readable: "Authentication token expired. Please re-authenticate to continue." Claude can then inform the user and guide re-authentication.

---

**Q10.** A team wants to allow Claude to call MCP tools in a CI/CD pipeline (automated, no human user). How should authentication be handled?

A) Use a developer's personal API key  
B) Use a service account with a dedicated API key scoped to only the permissions needed for the CI/CD workflow; store the key in the CI/CD system's secret management, not in code  
C) Disable authentication for CI/CD  
D) Use OAuth with user credentials  

**Answer: B**  
**Explanation:** Service accounts for automated systems: (1) Principle of least privilege — only permissions needed for the specific workflow. (2) No human user association — rotating the key doesn't affect human users. (3) CI/CD secret management (GitHub Secrets, Jenkins credentials) — not in code. (4) Audit logs clearly identify CI/CD vs human actions. Never use personal credentials for automated systems.

---

**Q11.** An MCP tool call includes user-provided data in the parameters. Before passing this data to a database query, what validation is mandatory?

A) Claude validates parameters  
B) All user-provided data must be treated as untrusted input and validated/sanitized server-side before use: type checking, length limits, allowed character sets, and parameterized queries for database operations (not string concatenation)  
C) JSON schema validation is sufficient  
D) Input validation is only needed for web applications  

**Answer: B**  
**Explanation:** Server-side input validation is mandatory regardless of client-side (or Claude-side) validation. SQL injection, command injection, path traversal — all exploit unvalidated inputs. Parameterized queries prevent SQL injection regardless of input content. JSON schema validation is a useful first check but doesn't replace the security validation that prevents injection attacks.

---

**Q12.** An SSE MCP server is deployed on a public internet URL without any authentication. It provides read access to internal company documentation. Why is this a critical problem?

A) SSE is not suitable for documentation  
B) Without authentication, anyone on the internet can read internal company documentation; no authentication = no access control; all internal data exposed publicly  
C) Only write access requires authentication  
D) The server should use REST instead  

**Answer: B**  
**Explanation:** "Read-only" without authentication is not safe for internal data. Authentication is required for all access to private data, not just write operations. An unauthenticated MCP server is functionally a public API — any data it can return is effectively public. Every production MCP server handling non-public data must require authentication.

---

**Q13.** A developer builds an MCP server that makes requests to third-party APIs using API keys stored in server-side environment variables. The tool passes these API keys back to Claude in the response for logging. What is the security issue?

A) Environment variables are not secure  
B) API keys returned to Claude may appear in: Claude's context window (visible in API responses), logs, conversation transcripts — unnecessarily expanding their exposure surface; third-party credentials should stay on the server and never appear in Claude's context  
C) Third-party API keys should be in the tool parameters  
D) This is the correct pattern  

**Answer: B**  
**Explanation:** Secrets have an exposure surface: the fewer places a secret appears, the lower the risk. Server-side API keys should exist only in the server's environment. They should never be passed to Claude (client-side). The tool performs the API call server-side and returns only the result (not the credential) to Claude. Minimize secret exposure surface.

---

**Q14.** An MCP server needs to support both individual developer use (stdio) and team sharing (SSE). What architecture supports both?

A) Deploy two separate servers  
B) A single MCP server implementation can support both transports: use stdio transport for local developer use (spawned as a subprocess), and SSE transport when deployed as a network service; same codebase, different transport configuration  
C) Use a proxy server  
D) Different tools must be implemented for each transport  

**Answer: B**  
**Explanation:** MCP protocol is transport-agnostic. A well-designed MCP server implements the protocol layer independently from the transport layer. Configuration determines which transport to use. Same tools, same business logic — different deployment modes. This is the correct separation of concerns: protocol implementation separate from transport.

---

**Q15.** An MCP server exposes a `run_sql_query` tool. The `query` parameter accepts freeform SQL. What security controls are essential?

A) Log all queries  
B) Multiple controls required: (1) Allow-list of permitted SQL operations (SELECT only, no INSERT/UPDATE/DELETE/DROP). (2) Parameterized query execution. (3) Query timeout limits. (4) Access to read-only database replica, not primary. (5) Row limit enforcement. "Run arbitrary SQL" is extremely high-risk without these controls.  
C) Require admin authentication  
D) Both A and C  

**Answer: B**  
**Explanation:** Arbitrary SQL execution is a critical attack surface: SQL injection, data exfiltration, data modification, schema changes. Multiple layers: (1) Operation type allow-listing (SELECT only). (2) Read-only replica access (write protection at database level). (3) Row limits (prevent data exfiltration via large dumps). (4) Timeouts (prevent DoS via expensive queries). Defense-in-depth is mandatory for any query execution tool.

---

**Q16.** A developer uses JWT tokens for MCP authentication. They validate only the token signature, not the expiration time (`exp` claim) or issuer (`iss` claim). What attacks does this allow?

A) No attacks — signature validation is sufficient  
B) (1) Expired tokens can be reused indefinitely after account deletion or revocation. (2) Tokens from other applications/environments can be used if they share the same signing key (wrong issuer accepted). Both allow unauthorized access with technically "valid" tokens.  
C) Only replay attacks  
D) JWT signature validation is always sufficient  

**Answer: B**  
**Explanation:** JWT validation must check: (1) Signature — token integrity. (2) `exp` — token hasn't expired. (3) `iss` — token was issued by the expected authority. (4) `aud` — token is intended for this service. Skipping any of these creates vulnerabilities. Expired token reuse allows ex-employees or revoked clients continued access. Wrong issuer allows cross-application token reuse.

---

**Q17.** A team's MCP server logs all tool inputs and outputs for debugging. A `get_customer_data` tool logs the full customer record including PII. What is the problem?

A) Logging makes systems slow  
B) Logging PII creates data retention, access control, and compliance problems; logs become a secondary store of sensitive data requiring the same protection as the primary database; log structured IDs (customer_id) not full records  
C) Tool outputs should not be logged  
D) Debug logging is fine in production  

**Answer: B**  
**Explanation:** Sensitive data in logs violates GDPR/CCPA data minimization principles. Logs often have: weaker access controls, longer retention periods, wider audience (all ops staff). Correct pattern: log only what's needed for debugging. Log `customer_id: 12345, fields_returned: ["name", "balance"]` not the actual customer data. Structured logs with IDs over full records.

---

**Q18.** An MCP server does not implement rate limiting. A misbehaving Claude agent (or prompt injection attack) loops indefinitely, calling tools repeatedly. What is the consequence?

A) Claude stops after 10 tool calls automatically  
B) Without rate limiting: API costs increase uncontrollably, third-party API quotas are exhausted, DoS on dependent services; rate limiting per-client protects the server and downstream services  
C) The server will queue requests automatically  
D) Only external APIs need rate limiting  

**Answer: B**  
**Explanation:** Rate limiting protects against: (1) Agentic loops (Claude calling a tool in an infinite loop). (2) Prompt injection attacks that trigger tool spam. (3) Runaway automation. (4) Cost explosions. Implement: per-client rate limits, per-tool rate limits for expensive operations, circuit breakers for sustained high volume. These are essential operational controls for any production MCP server.

---

**Q19.** An MCP server needs to call another internal microservice. The developer passes the user's session token to the microservice. What is the security risk?

A) No risk — the user's token should be reused  
B) Passing user tokens to backend services is the "confused deputy" pattern — the microservice acts with the user's permissions for operations the user didn't explicitly authorize; use service-to-service authentication with scoped service credentials instead  
C) Session tokens are always reusable  
D) The microservice should generate its own tokens  

**Answer: B**  
**Explanation:** Service-to-service calls should use service credentials, not user tokens. The user authorized the action via the MCP server — not directly with the microservice. Passing user tokens downstream: (1) Expands the token's exposure surface. (2) Can grant unintended permissions if the microservice uses the token for other operations. Use service accounts with specific service-to-service permissions.

---

**Q20.** A developer checks if an MCP server connection is using stdio by examining the connection object in code. Why might this be important?

A) stdio connections should be rejected  
B) Some security controls should only apply in networked deployments: authentication is required for SSE but not stdio; rate limiting is network-relevant; knowing the transport allows the server to apply appropriate security controls for each deployment mode  
C) stdio connections are always local and therefore safe  
D) Transport type doesn't matter for security  

**Answer: B**  
**Explanation:** Transport-aware security: stdio connections are local by definition (the MCP host and server are on the same machine, started by the same user). Authentication in stdio would be redundant. SSE connections are networked and require full authentication, authorization, and rate limiting. Transport-aware security applies the right controls for the right context.

---

**Q21.** A security audit finds that an MCP server's admin tools are accessible to all authenticated users, not just admins. No admin role enforcement exists. This is discovered when a regular user accidentally deletes production data. What control was missing?

A) Better user training  
B) Role-Based Access Control (RBAC) enforced server-side: the tool handler checks the authenticated user's role before execution; without RBAC, authentication (you are who you say you are) doesn't prevent unauthorized actions (you can do things you shouldn't)  
C) The admin tools should be in a separate server  
D) Both B and C  

**Answer: D**  
**Explanation:** Both fixes are valid. B (RBAC) is the primary fix: authentication ≠ authorization. Authenticated users need authorization checks for each privileged operation. C (separate server) adds physical separation — admin tools on a different endpoint with stronger access controls. For high-risk operations, both layers of protection are appropriate.

---

**Q22.** An organization wants to audit all Claude tool calls for compliance purposes. What must the MCP server log at minimum?

A) Only failed tool calls  
B) At minimum: WHO (authenticated user/service), WHAT (tool name + parameters, excluding PII), WHEN (timestamp with timezone), OUTCOME (success/failure + error type), and WHERE (source IP for network connections)  
C) Full request and response payloads  
D) Tool call count per user  

**Answer: B**  
**Explanation:** Compliance audit logging requires attribution (who), action (what, without PII), time, and outcome. Full payloads (C) over-collect sensitive data that creates its own compliance problems. Minimal structured logs with correlation IDs allow reconstructing what happened without retaining excessive PII. The 5 W's of audit logging: Who, What, When, Outcome, Where.

---

**Q23.** An MCP server's SSE endpoint doesn't validate the `Origin` header. What attack does this enable?

A) No attack — Origin headers are informational  
B) Cross-Site Request Forgery (CSRF) or Cross-Origin attacks: a malicious website can trigger SSE connections from a victim's browser, potentially authenticating with the victim's cookies or credentials and calling tools on their behalf  
C) Only relevant for browser-based clients  
D) SSE is not vulnerable to CSRF  

**Answer: B**  
**Explanation:** Even though Claude Code is not a browser, MCP servers must consider their full attack surface. If the SSE endpoint can be reached from a browser (for debugging or web-based clients), CSRF attacks are possible. Validate `Origin` header, implement CSRF tokens for browser-accessible endpoints, and use SameSite cookie attributes. Defense applies even if current clients don't use browsers.

---

**Q24.** A developer is implementing connection pooling for an MCP server that handles multiple concurrent Claude instances. What isolation must be maintained?

A) Connections can share state freely  
B) Each connection must have isolated authentication context and session state; pooled connections must not leak one client's session/credentials/data to another client; connection reuse must only happen for the same authenticated identity  
C) Connection pooling is not possible with MCP  
D) Isolation only matters for database connections  

**Answer: B**  
**Explanation:** Connection pool security: when connections are reused, the security context (user identity, permissions, session state) must not carry over between different clients. A connection used by Client A must be completely cleaned before being given to Client B. Authentication context leakage is a critical security bug in pooling implementations.

---

**Q25.** An MCP server is being promoted from development to production. What authentication change is mandatory?

A) No change needed if development tested authentication  
B) Development may use simplified auth (no-auth, self-signed certs, shared test tokens) — production requires: valid TLS certificates from a trusted CA, real authentication (OAuth or API keys with proper rotation), production secrets management, and revocation capabilities  
C) Just rotate the test API keys  
D) Authentication is the same in all environments  

**Answer: B**  
**Explanation:** Dev-to-prod promotion requires authentication hardening: (1) Self-signed certs → CA-issued TLS. (2) Test credentials → production credentials via secrets management. (3) Simplified auth → full auth with token rotation. (4) Shared dev credentials → individual production credentials. What's acceptable in development for convenience is not acceptable in production for security. These are distinct environments with distinct requirements.

---

## Score: /25 | Pass: 19/25 (75%)
