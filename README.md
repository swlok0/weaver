# 🦾🏗️ Weaver

這是一個自用的 Agent 功能模組管理 Monorepo，採用 pnpm workspace 架構。核心目的在於**將 LLM 接入、MCP 協議、原子工具（Tools）與複合工作流（Skills）進行扁平化、集中化管理**，作為個人其他終端應用的底層依賴庫。

## Release Structure & Repo Split

`weaver` is organized as a split architecture to keep public capability surfaces small, stable, and reusable, while isolating private runtime and experimental work.

### Public repos

#### `weaver-server`
The public MCP capability repo.

Responsibilities:
- expose MCP tools / resources / prompts
- implement the protocol surface
- keep the capability API stable and versioned
- provide examples and documentation for consumers

This repo should represent the public, repeatable surface of Weaver.

#### `weaver-contracts`
The public contract repo.

Responsibilities:
- define schemas, manifests, and capability contracts
- document versioning and naming rules
- provide shared interface definitions for clients and hosts

This repo should contain interface descriptions, not private runtime logic.

---

### Private repos

#### `weaver-host`
The private runtime / orchestration repo.

Responsibilities:
- manage runtime execution
- coordinate multiple MCP servers
- handle secrets, routing, and deployment policy
- integrate LLM clients and session management

This repo should not expose internal routing or secrets publicly.

#### `weaver-client`
The private or optional demo client repo.

Responsibilities:
- consume MCP capabilities
- provide internal harnesses, agent clients, or demo apps
- remain decoupled from private server internals

This repo may be public only if it is a simple demo client with no sensitive runtime logic.

---

### Experimental space

#### `labs/`
The experimental and prototype area.

Responsibilities:
- early prototypes
- failed experiments
- architecture exploration
- research branches and temporary demos

Anything proven useful in `labs/` should be distilled into:
- `weaver-server`
- `weaver-contracts`
- or a dedicated reusable tool/package

---

### Recommended release rule

- Public capability belongs in `weaver-server`
- Shared interface definitions belong in `weaver-contracts`
- Runtime/orchestration belongs in `weaver-host`
- Experimental work belongs in `labs/`

Do not mix public contract surface, private runtime behavior, and prototype code in the same release boundary.


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
