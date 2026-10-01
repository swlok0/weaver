# 🦾🏗️ Weaver

這是一個自用的 Agent 功能模組管理 Monorepo，採用 pnpm workspace 架構。核心目的在於**將 LLM 接入、MCP 協議、原子工具（Tools）與複合工作流（Skills）進行扁平化、集中化管理**，作為個人其他終端應用的底層依賴庫。

## Architecture and Release Scope

`weaver` is the public MCP server repository. Its sole responsibility is to expose a stable, versioned MCP capability surface.

### Current scope
This repository is responsible only for:

- MCP tools
- MCP resources
- MCP prompts
- protocol implementation
- public documentation
- usage examples

### Out of scope
This repository does not own:

- runtime orchestration
- private host policy
- client application logic
- experimental prototypes
- private routing or secrets management

Those responsibilities belong to separate repositories or private runtime layers.

---

## Repository Boundary

### `weaver-server`
Public capability repository.

Responsibilities:
- define and expose MCP capabilities
- keep the public interface stable
- version the server surface
- document how consumers should connect and use it

This repository should remain focused on the public MCP server surface only.

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

The public release boundary for `weaver` is the MCP server surface only.

That means:
- public changes must be capability-oriented
- internal orchestration must stay out of this repo
- experiments must not be mixed into the release surface
- reusable contracts may be extracted only if they directly support the server interface

---

## Experimental Work

Experimental work belongs in separate prototype or lab areas, not in the public server repo.

If an experiment proves useful, it may later be promoted into:
- `weaver-server`
- `weaver-contracts`
- or a dedicated reusable package

---

## Summary

`weaver` is intentionally narrow in scope.

It is not a full agent platform, not a host runtime, and not a client harness.
It is the public MCP server repository and should remain focused on that single responsibility.

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
