# 🦾🏗️ Weaver

這是一個自用的 Agent 功能模組管理 Monorepo，採用 pnpm workspace 架構。核心目的在於**將 LLM 接入、MCP 協議、原子工具（Tools）與複合工作流（Skills）進行扁平化、集中化管理**，作為個人其他終端應用的底層依賴庫。

## Architecture and Release Scope

`weaver` is the public capability and integration layer for a separately operated Private Agent. Its MVP connects to provider-hosted L3 retrieval and L4 tool/MCP capabilities; the private runtime remains responsible for identity, authorization, secrets, and private data. Weaver does not currently host a general-purpose MCP endpoint.

### Current scope
This repository is responsible for:

- MCP host configuration and provider integration examples
- reusable capability contracts and adapters when implemented
- public documentation and non-sensitive conformance tests
- experiments clearly labeled as prototypes

### Out of scope
This repository does not own:

- runtime orchestration
- private host policy
- client application logic
- private routing or secrets management

Those responsibilities belong to separate repositories or private runtime layers.

---

## Repository Boundary

### Weaver (this repository)
Public capability and integration layer.

Responsibilities:
- define reusable capability and adapter contracts
- connect MCP-compatible hosts to provider capabilities
- document permissions, behavior, and supported interfaces

This repository may include client configuration for official provider servers; that does not make Weaver a hosted MCP service. Prototype code under `labs/` is not part of the production capability surface.

### `weaver-contracts`
Optional shared contract repository.

Responsibilities:
- schema definitions
- manifests
- capability contracts
- naming and versioning rules

This repository should contain interface definitions, not runtime orchestration.

### `weaver-host`
Private runtime repository.

Responsibilities:
- orchestration
- deployment policy
- routing
- secrets
- session management
- LLM client integration

This repository is not part of the public capability surface.

### `weaver-client`
Optional client repository.

Responsibilities:
- consume MCP capabilities
- provide demo or harness applications
- connect to public server endpoints

This repository must not depend on private server internals.

---

## Release Principle

The public release boundary for `weaver` is reusable capability contracts, adapters, and their supporting documentation.

That means:
- public changes must be capability-oriented
- internal orchestration must stay out of this repo
- experiments must not be mixed into the release surface
- reusable contracts may be extracted only if they directly support a published capability

---

## Experimental Work

Experimental work belongs in separate prototype or lab areas, not in the stable capability surface.

If an experiment proves useful, it may later be promoted into:
- a reusable capability package
- `weaver-contracts`

---

## Summary

`weaver` is intentionally narrow in scope.

It is a public capability and integration layer, not a full agent platform, a private host runtime, a credential broker, or a store for private user data.

For a beginner-friendly guide to connecting a private MCP-compatible Agent to GitHub, see [GitHub MCP Quickstart](docs/github-mcp-quickstart.md). The guide uses GitHub's official MCP Server; it does not require sharing credentials with Weaver.

The MVP boundary, acceptance criteria, and provider evaluation are documented in [Weaver MVP](docs/mvp.md) and the [Provider Catalog](docs/provider-catalog.md). The workspace configuration connects VS Code to GitHub's official MCP server in read-only mode.

---

## 📂 扁平化目錄與職責邊界

```text
weaver/
...

```

---

## 📐 依賴防護原則 (Dependency Boundary)

為了確保各個本地模組能被乾淨地解耦複用，本倉庫嚴格遵守以下**單向依賴拓撲**：

```text
[skills]  ─── 組合/呼叫 ───>  [tools]
   │                             │
   └──────── 共同底層 ──────────> [llm & mcp]

```

* **tools（原子工具）**：保持絕對純粹。只處理基礎輸入與輸出，**內部不允許引入 llm**。工具本身不需要、也不應該知道是哪個模型在調用它。
* **skills（複合技能）**：業務邏輯與流程的聚合點。所有的 Prompt 拼接、Few-shot 範例引導、以及多步驟 Tool 的條件分支串聯（Workflow）皆收攏於此。
* **mcp（標準接口）**：將內部的 tools 或 skills 包裝為符合 Model Context Protocol 規範的標準功能，供本地其他桌面客戶端或網關直接調用。
