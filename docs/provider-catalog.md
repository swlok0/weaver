# Provider Catalog: Private Agent L3/L4

This catalog separates official provider capabilities from third-party adapters. Provider availability, terms, API access plans, and permissions can change; verify them against the provider's current documentation before deployment.

| Provider | Verified starting point | L3/L4 use | MVP status |
| --- | --- | --- | --- |
| GitHub | [GitHub's official MCP server](https://github.com/github/github-mcp-server), including the hosted remote server | Repository, issue, and pull request tools; GitHub remains the source of truth | **Included:** hosted MCP, selected toolsets, read-only |
| Meta developer tools | [Meta MCP documentation](https://developers.facebook.com/documentation/mcp/) and [Social Technologies MCP](https://developers.facebook.com/documentation/mcp/devtools-mcp/) | Developer workflows such as Meta app and platform documentation/tools; not a general adapter for private Facebook or Instagram content | **Document only:** evaluate the exact product and scopes |
| Meta Ads | [Meta Ads MCP overview](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-mcp-server/ads-mcp-server-overview) | Ads-oriented capabilities, subject to Meta account access, product availability, and authorization | **Document only:** no advertising actions enabled by Weaver |
| Facebook Pages / Instagram / Threads data | Specific Meta Graph APIs and eligible account types; an MCP adapter may be official or community-maintained | Read or publish only the supported objects and permissions for the selected API product | **Not included:** select product, account type, scopes, and review requirements first |
| X | [X Developer Platform](https://developer.x.com/) | Use the official X API as the authority for available data, authentication, and access plans; an MCP adapter may be community-maintained | **Not included:** no official X MCP endpoint has been verified for this MVP |

## Provider onboarding checklist

Before adding an adapter, record:

1. The specific provider product and user task (for example, search public posts, read a managed Page, or summarize GitHub PRs).
2. Whether the adapter/server is provider-official, host-provided, or community-maintained; include its source and maintenance status.
3. Authentication type, account ownership, scopes/permissions, API plan, review/approval requirements, and token rotation/revocation process.
4. Read-only versus write capabilities, with read-only as the initial setting.
5. Data returned, data sent to an LLM, retention, visibility, and deletion expectations.
6. Server-side authorization, confirmation requirements, error behavior, and tests for denied access.

Do not treat “supports MCP” as evidence that an adapter is official, secure, complete, or permitted for every account. Do not put credentials or private content in this public catalog.
