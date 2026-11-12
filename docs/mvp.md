# Weaver MVP: Private Agent Capability Layer

## Purpose

Weaver is the public, reusable capability and integration layer used by a separately operated Private Agent. Its first milestone makes GitHub issues, pull requests, and repositories available through GitHub's official MCP server in read-only mode.

Weaver does not host the Private Agent, store user credentials, enforce the private runtime's authorization policy, or publish private activity data.

## MVP boundary

```text
GitHub ── official GitHub MCP (read-only) ── Private Agent
                                               │
                                  optional MCP clients/adapters
                                               │
                                  private runtime and data stores

Weaver: public setup, reusable capability contracts, adapter guidance,
        and non-sensitive conformance checks
```

### Included

- A workspace MCP configuration for the official GitHub MCP server with `repos`, `issues`, and `pull_requests` toolsets in read-only mode.
- A beginner setup guide and troubleshooting notes.
- A provider catalog that distinguishes official services from community adapters and records unresolved access requirements.
- Reusable guidance for future retrieval (L3), tools and MCP adapters (L4), and non-sensitive conformance checks (L8).

### Not included

- A Weaver-hosted MCP proxy, agent orchestrator, credential broker, or user identity service.
- A public RAG index containing private repository or activity data.
- Write access to GitHub in the default configuration.
- X or Meta credentials, private account data, or third-party adapter code before a specific use case and its permissions have been reviewed.
- Production claims for the Zero Trust lab; it remains a prototype.

## GitHub setup

The workspace configuration is in [`.vscode/mcp.json`](../.vscode/mcp.json). It connects directly to GitHub's hosted MCP server and requests only repository, issue, and pull request toolsets with read-only mode enabled. It contains no token.

In VS Code:

1. Open the Weaver workspace with a current VS Code version that supports remote MCP.
2. Open the MCP server controls and start `github`.
3. Complete the GitHub authorization flow when prompted. Organization policy may require administrator approval.
4. Confirm the GitHub tools are visible in Agent mode.
5. Test a read-only request against a repository you are authorized to access, such as listing its open issues or pull requests.

The official service enforces the read-only tool surface, while GitHub authorization determines which repositories and records the account can access. Do not copy credentials into `.vscode/mcp.json`.

For other MCP hosts and PAT-based setup, see [GitHub MCP Quickstart](./github-mcp-quickstart.md).

## L3: retrieval and knowledge

Keep the source of truth in the source system. For the MVP, use GitHub's own search and read tools rather than copying all GitHub content into a Weaver-managed vector database.

If a later use case needs RAG:

- Keep private documents, embeddings, indexes, and access metadata in the private runtime.
- Filter retrieval by the caller's authorization before returning chunks to the model.
- Return source identifiers, URLs, and version or update timestamps with retrieved content.
- Treat retrieved issue, PR, and document text as untrusted input, not as instructions.
- Do not use vector similarity as an authorization decision.

## L4: tools and MCP adapters

Prefer a platform's official MCP server when it covers the requested capability. Add a Weaver adapter only when it provides a stable, reusable interface that is not already adequately exposed by an official server.

Future tools should be narrow and explicit: define inputs, outputs, required permissions, read/write behavior, failure cases, and whether human confirmation is required. Enforce authorization in the server or private runtime; descriptions and schema annotations alone are not security controls.

For write operations, require a separate opt-in configuration and server-side authorization. Require human confirmation for consequential or externally visible actions.

## L5-L6: private runtime controls

The private runtime, not this public repository, owns identity checks, authorization, secrets, rate limits, and data handling. Use the smallest provider permissions and account/repository scope available. Keep provider credentials in a runtime secret store, rotate and revoke them, and never expose them to browser code, model prompts, logs, or repository files.

Prompt-injection filtering and PII detection are defense-in-depth measures. They do not replace least-privilege tools, server-side authorization, source-data minimization, or human approval.

## L8: lightweight conformance and evidence

Weaver may hold public, non-sensitive evidence that its documented integration matches the intended interface:

- Configuration/schema validation.
- Contract and adapter tests using fixtures or mocked provider responses.
- Versioned capability descriptions and documented permission requirements.
- Change history for tool names, inputs, outputs, and read/write behavior.

Runtime audit records belong in the private runtime. Record only the metadata needed to investigate operations (for example, correlation ID, provider, tool, decision, result, and contract version); avoid tokens and unnecessary raw prompts, repository content, or personal data. Set access and retention rules for those records.

## MVP acceptance criteria

- An authorized VS Code Agent can discover GitHub repository, issue, and pull request tools using the checked-in configuration.
- The default configuration exposes only the selected GitHub toolsets in read-only mode.
- No PAT, OAuth token, private activity report, or private index is committed to Weaver.
- Documentation clearly distinguishes official GitHub MCP from Weaver and from the experimental Zero Trust lab.
- X and Meta support is described only to the level verified in the provider catalog; no unsupported endpoint or permission is presented as ready to use.

## Evolution rule

Add a provider or adapter only after identifying the specific user task, provider product/API, supported authentication flow, minimum permissions, data classification, and whether the operation is read-only or writes externally. Start read-only; add write capabilities separately after tests and human-approval behavior are defined.
