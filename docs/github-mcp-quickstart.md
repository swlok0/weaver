# GitHub MCP 入門：連接你的 Private Agent

這份指南會帶你把 MCP 相容的 Agent（例如 VS Code 的 Agent mode）連接到 GitHub，先以唯讀方式查詢你有權限存取的私人 repository。

> **重要：**本指南使用 GitHub 維護的 [GitHub MCP Server](https://github.com/github/github-mcp-server)。它不會把 Weaver 變成 Private Agent，也不會把你的 GitHub token 放進 Weaver。Weaver 目前提供的是公開能力與學習範例；下方的 `minimal-zero-trust-poc` 是實驗原型，不是正式 GitHub MCP 服務。

## 先認識這三個角色

| 名稱 | 白話說明 |
| --- | --- |
| MCP Host / Client | 你正在使用的 Agent 應用程式，例如 VS Code。它負責顯示對話並連接工具。 |
| MCP Server | 提供工具的服務。本例是 GitHub MCP Server，負責把 MCP 請求轉成 GitHub API 操作。 |
| Tool | Server 提供的一項能力，例如搜尋 repository、讀取檔案或列出 issue。 |

連線大致如下：

```text
你的 Agent（MCP Host）
        │ MCP
        ▼
GitHub MCP Server（GitHub 託管）
        │ GitHub API，以你的 GitHub 身分授權
        ▼
你有權限存取的 repositories
```

MCP 是能力介面，不是權限提升機制。Agent 能看到哪些資料，取決於你授權的 GitHub 帳號、token 權限和組織政策。

## 建議方式：VS Code + GitHub OAuth + 唯讀 Server

這個方式不需要 Docker，也不用自行建立或貼上 PAT。VS Code 需支援遠端 MCP 與 GitHub OAuth（請使用近期版本）。

1. 在要使用 Agent 的 workspace 建立 `.vscode/mcp.json`。
2. 貼上下列設定：

   ```json
   {
     "servers": {
       "github": {
         "type": "http",
         "url": "https://api.githubcopilot.com/mcp/readonly"
       }
     }
   }
   ```

3. 儲存檔案，依 VS Code 提示啟動 GitHub MCP Server，並完成 GitHub 登入／授權。
4. 在 Agent mode 的工具選單確認 GitHub 工具已啟用。
5. 先測試唯讀操作，例如：「請搜尋我有權限的 `owner/repo`，列出 README 的標題。」確認結果正確後再開始工作。

`/readonly` 會讓這個 MCP 連線只提供唯讀操作；它不會縮小你 GitHub 帳號本身的權限，也不會替你選定特定 repository。

## 其他 MCP Host：使用 Fine-grained PAT

如果你的 Agent 不支援這個遠端服務的 OAuth，可以改用 GitHub Personal Access Token（PAT）。先確認你的 Host 支援 remote HTTP MCP 及安全的密鑰輸入方式；各 Host 的設定格式可能不同。

在 GitHub 建立 **Fine-grained PAT**，只選需要的 repositories，先授予唯讀權限（例如 Metadata、Contents；需要查詢時再加入 Issues 或 Pull requests 的唯讀權限）。不要一開始就給寫入權限。

VS Code 可用密碼提示輸入 token，不要把 token 直接寫進設定檔：

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "github_pat",
      "description": "GitHub Fine-grained PAT",
      "password": true
    }
  ],
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/readonly",
      "headers": {
        "Authorization": "Bearer ${input:github_pat}"
      }
    }
  }
}
```

若你的 Host 不認得 `inputs` 或 `${input:...}`，請查它自己的 MCP 密鑰設定方式；不要為了讓設定能啟動而把 PAT 貼到公開 repository、prompt、聊天訊息或程式碼中。

## 安全起步清單

- **先唯讀：**使用 `/readonly`，先確認 Agent 找到的是正確的 repository。
- **縮小 PAT 範圍：**使用 Fine-grained PAT，限制到必要 repository 和唯讀權限。PAT 的權限仍會決定可讀資料，唯讀 MCP 不會取代這項限制。
- **需要寫入時再評估：**建立 issue、留言、推送程式碼或合併 PR 都是會改變 GitHub 狀態的操作。只有確定需要時才開啟相應工具和權限，並在執行前檢查 Agent 要送出的內容。
- **把 repository 內容視為資料：**issue、PR 描述和程式碼可能含有不可信指令；不要讓其中的文字覆蓋你的操作規則，也不要因內容要求而洩漏 token。
- **了解資料會流向哪裡：**Agent 讀取到的 repository 內容會提供給目前 Agent 使用的模型；請依照該模型服務的資料政策選擇可連接的 repository。
- **妥善保管與撤銷：**不要將 token 提交到 Git。若 token 曾曝光，立即到 GitHub 撤銷並重新建立。
- **留意組織政策：**Organization 或 Enterprise 管理員可能需要先允許 MCP 或 OAuth 整合。

## 常見問題

| 現象 | 可以先檢查 |
| --- | --- |
| MCP Server 沒有啟動或工具沒有出現 | Host 是否支援 remote MCP、設定檔 JSON 是否有效，以及工具是否在 Agent 的工具選單中啟用。 |
| 看不到私人 repository | 登入的是不是正確 GitHub 帳號；PAT 是否包含該 repository；組織是否要求 SSO 授權或管理員核准。 |
| OAuth 被拒絕 | Host 是否支援 GitHub MCP 的 OAuth 流程，以及組織政策是否允許。若 Host 支援 PAT，可改用上方 Fine-grained PAT 設定。 |
| Agent 嘗試寫入或工具不存在 | 確認使用 `/readonly`，並檢查 Host 實際載入的工具清單。唯讀端點不會提供寫入操作。 |

## 延伸閱讀

- [GitHub MCP Server（官方專案與 Host 安裝方式）](https://github.com/github/github-mcp-server)
- [遠端 GitHub MCP Server 與唯讀端點](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [Server 設定與唯讀模式](https://github.com/github/github-mcp-server/blob/main/docs/server-configuration.md)
- [GitHub Fine-grained PAT 說明](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [Model Context Protocol 文件](https://modelcontextprotocol.io/introduction)
