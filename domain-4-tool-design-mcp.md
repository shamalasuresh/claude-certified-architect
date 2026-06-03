# Domain 4: Tool Design & MCP — 18%

> How to design tools that Claude can use effectively, and how MCP (Model Context Protocol) works as the standard for connecting models to the world.

---

## What This Domain Covers

- Model Context Protocol (MCP) fundamentals
- MCP server architecture
- Tool schema design
- Resource and prompt management
- Transport layers (stdio, SSE)
- Security considerations
- Error handling patterns for tools

---

## 1. What is MCP?

**Model Context Protocol (MCP)** is an open standard that defines how AI models like Claude communicate with external systems (databases, APIs, file systems, tools). Think of it as USB-C for AI — a universal connector between models and the outside world.

### Why MCP Exists

Before MCP, every AI integration was custom-built. MCP standardizes:
- How tools are discovered and described
- How tools are called and how results are returned
- How resources (files, data) are exposed
- How reusable prompts are shared

### MCP Ecosystem

```
Claude (client/host)
      │
      │ MCP Protocol
      │
      ▼
MCP Server
├── Tools (actions Claude can take)
├── Resources (data Claude can read)
└── Prompts (reusable templates)
```

---

## 2. MCP Architecture

### Components

| Component | What It Is | Example |
|-----------|-----------|---------|
| **Host** | The application using Claude | Claude Desktop, Claude Code |
| **Client** | MCP client built into the host | Handles protocol communication |
| **Server** | Your MCP server providing capabilities | GitHub MCP, Postgres MCP |
| **Transport** | How client and server communicate | stdio, SSE |

### MCP Server Capabilities

An MCP server can expose three types of capabilities:

#### 1. Tools
Actions that Claude can invoke. Tools have:
- A name
- A description
- An input schema (JSON Schema)
- A return type

#### 2. Resources  
Data that Claude can read. Resources have:
- A URI (identifier)
- A MIME type
- Contents (text or binary)

#### 3. Prompts
Reusable prompt templates stored on the server. They allow:
- Standardized instructions for common tasks
- Dynamic prompt generation with parameters
- Sharing prompts across teams/applications

---

## 3. Transport Layers

MCP supports two transport mechanisms. The exam tests when to use each.

### stdio (Standard Input/Output)

```
Claude Code / Host
      │
      │ stdin/stdout pipes
      │
      ▼
MCP Server Process (local)
```

**When to use stdio:**
- Local tools running on the same machine
- Command-line tools
- Development environments
- When security requires no network exposure

**Characteristics:**
- Process lifecycle tied to the host
- No network overhead
- Single client per server instance
- Simple setup for local tooling

**Config example:**
```json
{
  "command": "node",
  "args": ["./my-mcp-server.js"]
}
```

### SSE (Server-Sent Events)

```
Claude Code / Host
      │
      │ HTTP + SSE
      │
      ▼
MCP Server (remote/networked)
```

**When to use SSE:**
- Remote servers (API-backed tools)
- Shared MCP servers across multiple users/clients
- Web-based deployments
- When the server needs to push updates

**Characteristics:**
- Server can be remote
- Multiple clients can connect
- Requires network access
- More complex security considerations

**Config example:**
```json
{
  "url": "https://mcp.example.com/sse"
}
```

### Transport Decision Guide

| Scenario | Transport |
|----------|-----------|
| Local file system access | stdio |
| Developer's personal tools | stdio |
| Company-wide shared tool | SSE |
| Cloud-hosted MCP server | SSE |
| CI/CD pipeline tool | stdio |
| Multi-tenant SaaS tool | SSE |

---

## 4. Tool Schema Design

Tool schemas are the most important design element — they determine whether Claude can use the tool correctly.

### JSON Schema for Tools

```json
{
  "name": "search_codebase",
  "description": "Search for files or code patterns in the codebase. Use this to find where specific functions, classes, or patterns are defined.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "The search query — can be a function name, class name, or regex pattern"
      },
      "file_pattern": {
        "type": "string",
        "description": "Optional glob pattern to limit search scope (e.g., '**/*.ts' for TypeScript files)",
        "default": "**/*"
      },
      "case_sensitive": {
        "type": "boolean",
        "description": "Whether the search should be case-sensitive. Default false.",
        "default": false
      }
    },
    "required": ["query"]
  }
}
```

### Tool Description Best Practices

The description field is what Claude reads to decide **whether and how** to use the tool. Poor descriptions lead to incorrect tool selection.

| Bad Description | Good Description |
|-----------------|-----------------|
| "Gets data" | "Retrieves customer order history from the orders database. Use when you need historical purchase data for a specific customer." |
| "Sends message" | "Sends an email notification to the customer. Use ONLY after confirming an order change. This action cannot be undone." |
| "Search tool" | "Searches the internal knowledge base for product documentation. Does NOT search the web — only internal docs." |

### Schema Design Principles

**1. Be specific about types**
```json
// Bad
{"query": {"type": "string"}}

// Good
{"query": {"type": "string", "minLength": 1, "maxLength": 500}}
```

**2. Use enums for constrained choices**
```json
{"status": {
  "type": "string",
  "enum": ["open", "in_progress", "resolved", "closed"],
  "description": "The new status. Use 'resolved' when the issue is fixed but not yet verified."
}}
```

**3. Describe units and formats**
```json
{"timeout": {
  "type": "integer",
  "description": "Timeout in milliseconds. Minimum 100, maximum 30000.",
  "minimum": 100,
  "maximum": 30000
}}
```

**4. Make optionality explicit**
Required fields go in the `required` array. Optional fields should have defaults and descriptions of when to use them.

---

## 5. Tool Design Anti-Patterns

### Anti-pattern 1: Too Many Tools
Having 50+ tools overwhelms Claude's tool selection. Group related operations.

```
✗ Bad: create_file, read_file, update_file, delete_file, list_files, 
        rename_file, copy_file, move_file, get_file_info, ...

✓ Good: file_system_tool with an "operation" parameter:
        {"operation": "read|write|delete|list"}
```

### Anti-pattern 2: Ambiguous Names
```
✗ Bad: "process_data", "handle_request", "do_thing"
✓ Good: "calculate_invoice_total", "submit_support_ticket", "fetch_user_profile"
```

### Anti-pattern 3: Side Effects Hidden in Read Tools
Tools named like reads (get_, fetch_, list_) should not have side effects.
```
✗ Bad: get_user_data() that also increments access count and logs IP
✓ Good: fetch_user_data() reads only; log_access() is separate
```

### Anti-pattern 4: No Error Context in Returns
```json
// Bad return on error
{"success": false}

// Good return on error
{
  "success": false,
  "error_code": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Retry after 60 seconds.",
  "retry_after_seconds": 60
}
```

---

## 6. Resource Design

Resources expose data to Claude without it being an "action."

### When to Use Resources vs Tools

| Use Resources for | Use Tools for |
|-------------------|--------------|
| Documentation / reference material | Querying live data |
| Configuration files | Taking actions |
| Static or semi-static data | Creating/updating/deleting |
| Content Claude should read | Things with side effects |

### Resource URI Patterns

```
# File-based resources
file:///path/to/document.md

# Database-based resources  
db://table/row_id

# API-backed resources
https://api.example.com/resources/item-123

# Custom scheme
myapp://config/production
```

### Resource Contents
Resources return either:
- **Text content** (Markdown, JSON, code)
- **Blob content** (binary files, images — base64 encoded)

---

## 7. Error Handling in Tools

Claude needs actionable error information to decide what to do next. Cryptic errors cause incorrect retry behavior.

### Error Response Pattern

```json
{
  "isError": true,
  "content": [
    {
      "type": "text",
      "text": "Error: Database connection failed.\nError code: DB_CONN_TIMEOUT\nRetriable: true\nSuggested action: Wait 5 seconds and retry. If error persists after 3 attempts, escalate to human."
    }
  ]
}
```

### Error Categories and Claude's Expected Behavior

| Error Type | Should Claude Retry? | What Claude Should Do |
|------------|---------------------|----------------------|
| Network timeout | Yes (with backoff) | Retry up to 3 times |
| Rate limit | Yes (after delay) | Wait retry_after, then retry |
| Invalid input | No | Fix input and try again |
| Auth failure | No | Escalate to human |
| Not found | No (for read) | Report resource doesn't exist |
| Server error (500) | Maybe (once) | Retry once, then escalate |

### Idempotent Tool Design

Design mutating tools to be safely re-runnable:

```json
// Include idempotency key in schema
{
  "name": "create_order",
  "inputSchema": {
    "properties": {
      "idempotency_key": {
        "type": "string",
        "description": "Unique key to prevent duplicate order creation. Generate from order contents + timestamp."
      }
    },
    "required": ["idempotency_key"]
  }
}
```

---

## 8. MCP Security

### Authentication in MCP

**stdio transport:** Authentication is implicit — only processes with file system access can connect. Apply OS-level permissions.

**SSE transport:** Requires explicit authentication:
- OAuth 2.0 for production systems
- API key in Authorization header for simpler cases
- JWT tokens for user-specific access

```json
// SSE with auth header
{
  "url": "https://mcp.example.com/sse",
  "headers": {
    "Authorization": "Bearer ${MCP_API_TOKEN}"
  }
}
```

### Tool Permission Scoping

Principle of least privilege applies to tools:
```
✓ Correct: Support agent's MCP server has tools: 
           read_ticket, update_ticket_status, add_comment

✗ Wrong: Support agent's MCP server has tools:
         read_all_tables, execute_sql, delete_record, modify_permissions
```

### Input Validation

MCP servers must validate ALL inputs before execution. Claude's schema describes expected input, but the server must enforce it:

```javascript
// Server-side validation (don't trust schema alone)
function handleCreateUser(input) {
  if (!input.email || !isValidEmail(input.email)) {
    throw new Error("Invalid email format");
  }
  if (input.email.length > 254) {  // RFC 5321
    throw new Error("Email too long");
  }
  // Proceed only after validation
}
```

### Preventing Sensitive Data Leakage

MCP server responses must not include:
- Full credential values (mask after first 4 chars)
- Other users' data (enforce row-level security)
- Internal system details (stack traces, paths)

---

## 9. MCP Server Implementation Patterns

### Minimal MCP Server Structure (Node.js)

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-mcp-server", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {} } }
);

// Register tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "search_docs",
      description: "Search internal documentation",
      inputSchema: { /* ... */ }
    }
  ]
}));

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "search_docs") {
    const results = await searchDocs(request.params.arguments.query);
    return { content: [{ type: "text", text: JSON.stringify(results) }] };
  }
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Tool Naming Conventions

```
{verb}_{noun}        → preferred
get_customer         → read, no side effect
create_ticket        → creates new resource
update_ticket_status → partial update
delete_attachment    → destructive, use sparingly
search_knowledge_base → search/filter
```

---

## 10. Exam Scenarios & Right Answers

### Scenario: "Claude is calling the wrong tool for a task"
**Right answer:** Improve tool descriptions — be specific about when to use the tool and when NOT to. Poor descriptions cause incorrect tool selection.

### Scenario: "Need to share an MCP server across multiple developers"
**Right answer:** SSE transport with authentication headers. stdio is per-process and can't be shared.

### Scenario: "MCP server has access to production database"
**Right answer:** Apply least privilege — create a separate read-only connection with access to only the tables needed. Use SSE with OAuth for multi-user access.

### Scenario: "Tool call fails and Claude retries in a loop"
**Right answer:** Error responses must include `retriable: false` for non-transient errors, or provide `retry_after` for rate limits. Claude can't make good retry decisions without this.

### Scenario: "Need to expose documentation files to Claude without making them 'actions'"
**Right answer:** Use MCP Resources, not tools. Resources are for readable data; tools are for actions.

### Scenario: "MCP server returns a 500 error with stack trace"
**Right answer:** Strip internal details from error responses. Return user-friendly error messages with error codes, not stack traces.

---

## 11. Quick Reference Card

### Transport Selection
```
Local / same machine  → stdio
Remote / shared       → SSE with auth
```

### Tool Description Template
```
"[Action verb] [what it does]. Use when [specific trigger]. 
[Important constraints or warnings]. [NOT to be confused with X]."
```

### MCP Capability Types
```
Tools     → Actions (have side effects)
Resources → Data (read-only)
Prompts   → Reusable templates
```

### Error Response Must Include
```
- isError: true
- Error code (machine-readable)
- Human-readable message
- Is it retriable? (true/false)
- Suggested action for Claude
```
